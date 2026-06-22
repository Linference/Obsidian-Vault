---
tags: [CSP, 搜索]
difficulty: J+S
created: 2026-06-06
---

# BFS 与 DFS

## DFS（深度优先搜索）

### 核心思想

一条路走到黑，走不通再回溯。用**递归**或**显式栈**实现。核心框架是**回溯**：

```
尝试 → 深入 → 撤销尝试 → 尝试下一个分支
```

### 回溯模板

```cpp
#include <bits/stdc++.h>
using namespace std;

void dfs(状态) {
    if (满足结束条件) {
        记录答案;
        return;
    }
    for (每个可选分支) {
        if (!合法) continue;
        做选择;               // 标记/修改状态
        dfs(新状态);
        撤销选择;             // 恢复状态（回溯的精髓）
    }
}
```

### 全排列（经典 DFS 回溯）

```cpp
int n;
vector<int> path;
bool used[20];

void dfs(int u) {
    if (u == n) {
        for (int x : path) cout << x << " ";
        cout << "\n";
        return;
    }
    for (int i = 1; i <= n; i++) {
        if (used[i]) continue;
        used[i] = true;
        path.push_back(i);
        dfs(u + 1);
        path.pop_back();
        used[i] = false;       // 撤销！
    }
}
```

### 八皇后问题

```cpp
int n, ans = 0;
int col[20], diag1[40], diag2[40];   // 列、主对角线、副对角线

void dfs(int row) {
    if (row == n) { ans++; return; }
    for (int c = 0; c < n; c++) {
        if (col[c] || diag1[row + c] || diag2[row - c + n])
            continue;
        col[c] = diag1[row + c] = diag2[row - c + n] = 1;
        dfs(row + 1);
        col[c] = diag1[row + c] = diag2[row - c + n] = 0;
    }
}
```

---

## BFS（广度优先搜索）

### 核心思想

一层一层地搜，用**队列**实现。核心属性：**BFS 搜到的第一条路径即是最短路径**（边权相等时）。

### BFS 模板

```cpp
#include <bits/stdc++.h>
using namespace std;

int bfs() {
    queue<状态> q;
    q.push(初始状态);
    vis[初始状态] = 1;

    int step = 0;
    while (!q.empty()) {
        int sz = q.size();       // 当前层的节点数
        for (int i = 0; i < sz; i++) {   // 逐层处理
            auto cur = q.front(); q.pop();
            if (cur == 目标) return step;

            for (每个扩展方向) {
                auto nxt = cur 的下一个状态;
                if (合法 && !vis[nxt]) {
                    vis[nxt] = 1;
                    q.push(nxt);
                }
            }
        }
        step++;
    }
    return -1;                   // 无法到达
}
```

### 迷宫最短路径

```cpp
const int dx[] = {-1, 1, 0, 0};
const int dy[] = {0, 0, -1, 1};

struct Node { int x, y; };

int mazeBFS(vector<vector<char>> &grid, int sx, int sy, int tx, int ty) {
    int n = grid.size(), m = grid[0].size();
    vector<vector<int>> dist(n, vector<int>(m, -1));
    queue<Node> q;

    q.push({sx, sy});
    dist[sx][sy] = 0;

    while (!q.empty()) {
        auto [x, y] = q.front(); q.pop();
        if (x == tx && y == ty) return dist[x][y];

        for (int d = 0; d < 4; d++) {
            int nx = x + dx[d], ny = y + dy[d];
            if (nx < 0 || nx >= n || ny < 0 || ny >= m) continue;
            if (grid[nx][ny] == '#' || dist[nx][ny] != -1) continue;
            dist[nx][ny] = dist[x][y] + 1;
            q.push({nx, ny});
        }
    }
    return -1;     // 不可达
}
```

---

## 洪水填充 (Flood Fill)

### 连通块计数

```cpp
void floodFill(vector<vector<int>> &grid, int x, int y, int color) {
    int n = grid.size(), m = grid[0].size();
    if (x < 0 || x >= n || y < 0 || y >= m) return;
    if (grid[x][y] != 1) return;    // 1 代表未填充的陆地

    grid[x][y] = color;             // 染色
    for (int d = 0; d < 4; d++)
        floodFill(grid, x + dx[d], y + dy[d], color);
}
```

也可以用 BFS 实现 flood fill，避免递归栈溢出。

---

## DFS vs BFS 选择

| 场景 | 选择 | 原因 |
|------|------|------|
| 求最短路径（等权边） | BFS | 层序 = 最短 |
| 求所有路径 | DFS | 回溯天然枚举所有可能 |
| 连通块 / 岛屿 | 两者都可以 | DFS 递归可能栈溢出用 BFS |
| 状态空间巨大，解在深处 | DFS (IDDFS) | BFS 内存爆炸 |
| 图很大但解在浅层 | BFS | DFS 可能走太深 |
| 拓扑排序 | DFS 后序 | 天然支持 |

## 状态去重

- **vis 数组**：适用于状态可编码为整数
- **`unordered_set<string/状态>`**：适用于状态映射
- **剪枝去重**：排序 / 规定搜索顺序避免等价状态重复

```cpp
struct State { int x, y, keys; };
// 去重：把 (x, y, keys) 编码为唯一标识
int encode(int x, int y, int keys) {
    return (x << 16) | (y << 8) | keys;
}
```

## CSP 真题场景

- **J2022 T4** 上升点列 — BFS/DFS 变体
- **J2021 T4** 小熊的果篮 — 搜索 + 链表
- **J2020 T4** 方格取数 — DFS 回溯 / DP
- **S2021 T1** 廊桥分配 — 可用 BFS 模拟

## ⚠️ 易错点

1. **DFS 忘记回溯**：修改状态后必须恢复，否则影响其他分支
2. **BFS 入队前标记 visited**：必须在 push 时标记，不能等到 pop 时（会导致重复入队）
3. **递归深度**：DFS 递归层数过大（> 10^5）会栈溢出，改用显式栈或 BFS
4. **方向数组**：dx/dy 的对应关系别搞混（四方向 vs 八方向）
5. **BFS 分层技巧**：`int sz = q.size()` 逐层处理可以获得当前步数

## 相关笔记

- [[剪枝与记忆化搜索]] — DFS 的进阶优化
- [[A星与IDA星]] — BFS 的启发式扩展
- [[03-动态规划]] — 记忆化搜索与 DP 等价
