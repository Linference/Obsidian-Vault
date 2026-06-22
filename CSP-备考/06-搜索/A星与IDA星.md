---
tags: [CSP, 搜索]
difficulty: S
created: 2026-06-06
---

# A* 与 IDA*

## A* 算法

### 核心思想

**A* = BFS + 估价函数**。用优先队列替代普通队列，每次扩展**最有希望**的节点。

$$f(n) = g(n) + h(n)$$

- **g(n)**：起点到节点 n 的**实际代价**（已走步数）
- **h(n)**：节点 n 到终点的**估计代价**（启发函数）
- **f(n)**：经过 n 到终点的**估计总代价**

优先队列每次取 **f(n) 最小** 的节点扩展。

### h(n) 的要求

**h(n) 必须 ≤ 实际代价**（乐观估计 / 可纳性）。只有这样 A* 才能保证找到最优解。

- h(n) = 0 时，退化为普通的 Dijkstra/BFS
- h(n) 越接近实际值，搜索越快
- h(n) 过大（超过实际代价）→ 可能找不到最优解

---

## 八数码问题（A* 经典）

**状态**：3×3 棋盘，数字 1-8 + 空格，每次空格和相邻数字交换，求到目标状态的最少步数。

### 估价函数：曼哈顿距离

每个数字当前位置与目标位置的 `|dx| + |dy|` 之和。曼哈顿距离 ≤ 实际需要的步数（每步最多消除 1 点曼哈顿距离）。

### C++ 模板

```cpp
#include <bits/stdc++.h>
using namespace std;
using State = string;

// 目标状态
const State GOAL = "12345678x";

// 曼哈顿距离估价函数
int h(const State &s) {
    int dist = 0;
    for (int i = 0; i < 9; i++) {
        if (s[i] == 'x') continue;
        int num = s[i] - '1';
        dist += abs(i / 3 - num / 3) + abs(i % 3 - num % 3);
    }
    return dist;
}

// 方向数组：上下左右
const int dx[] = {-1, 1, 0, 0};
const int dy[] = {0, 0, -1, 1};
const char dir[] = "udlr";

int astar(const State &start) {
    // 逆序对判断是否可解（略）

    // 优先队列：{f, g, state}
    using Node = tuple<int, int, State>;
    priority_queue<Node, vector<Node>, greater<Node>> pq;
    unordered_map<State, int> dist;      // 记录到各状态的最小 g

    pq.push({h(start), 0, start});
    dist[start] = 0;

    while (!pq.empty()) {
        auto [f, g, cur] = pq.top(); pq.pop();

        if (cur == GOAL) return g;       // 找到最优解

        // ⚠️ 如果这个 g 大于已知最小 g，跳过（懒删除）
        if (g > dist[cur]) continue;

        int pos = cur.find('x');
        int x = pos / 3, y = pos % 3;

        for (int d = 0; d < 4; d++) {
            int nx = x + dx[d], ny = y + dy[d];
            if (nx < 0 || nx >= 3 || ny < 0 || ny >= 3) continue;

            State nxt = cur;
            swap(nxt[pos], nxt[nx * 3 + ny]);

            int ng = g + 1;
            if (!dist.count(nxt) || ng < dist[nxt]) {
                dist[nxt] = ng;
                pq.push({ng + h(nxt), ng, nxt});
            }
        }
    }
    return -1;   // 无解
}
```

### 逆序对判定可解性

```cpp
// 八数码中：空格上下移动不改变逆序对奇偶
// 空格左右移动改变奇偶 → 两个状态逆序对奇偶相同才可互相转化
bool solvable(const State &s) {
    int inv = 0;
    for (int i = 0; i < 9; i++) {
        if (s[i] == 'x') continue;
        for (int j = i + 1; j < 9; j++) {
            if (s[j] != 'x' && s[i] > s[j]) inv++;
        }
    }
    return inv % 2 == 0;   // 目标 GOAL 的逆序对为偶数
}
```

---

## IDA*（迭代加深 A*）

### 核心思想

**IDA* = DFS + 迭代加深 + A* 剪枝**。

A* 的问题是优先队列内存爆炸（八数码都数百万状态）。IDA* 用 DFS 的栈空间替代优先队列，用估价函数限制搜索深度：

- 设定阈值 `limit = f(初始状态)`
- DFS 搜索，**只扩展 `g + h ≤ limit` 的节点**
- 如果当前 limit 下找不到解，提高 limit，重新搜索
- limit 每次提高到本轮**超过 limit 的最小 f 值**

### 优势

- 空间 O(深度)，只需递归栈
- 适合状态空间大但 DFS 深度可控的问题
- 自带迭代加深：逐步放宽搜索深度

### 为什么 IDA* 效率高

虽然每轮都要重新搜索，但搜索树中**绝大多数节点在最后一层**，前面层的重复开销相对较小。

---

### 八数码 IDA* 实现

```cpp
const State GOAL = "12345678x";
const int dx[] = {-1, 1, 0, 0};
const int dy[] = {0, 0, -1, 1};
const char revDir[] = {1, 0, 3, 2};  // 反向移动索引（避免来回走）

int h(const State &s) {
    int dist = 0;
    for (int i = 0; i < 9; i++) {
        if (s[i] == 'x') continue;
        int num = s[i] - '1';
        dist += abs(i / 3 - num / 3) + abs(i % 3 - num % 3);
    }
    return dist;
}

// 返回 -1 表示找到解（在 path 中记录），否则返回下一个 limit
int dfs(State &cur, int g, int limit, int lastDir, vector<char> &path) {
    int f = g + h(cur);
    if (f > limit) return f;            // 估价超限，返回作为新的 limit 候选
    if (cur == GOAL) return -1;         // 找到解！

    int pos = cur.find('x');
    int x = pos / 3, y = pos % 3;
    int nextLimit = INT_MAX;

    for (int d = 0; d < 4; d++) {
        if (d == revDir[lastDir]) continue;   // 避免来回走
        int nx = x + dx[d], ny = y + dy[d];
        if (nx < 0 || nx >= 3 || ny < 0 || ny >= 3) continue;

        swap(cur[pos], cur[nx * 3 + ny]);
        path.push_back(dir[d]);

        int t = dfs(cur, g + 1, limit, d, path);
        if (t == -1) return -1;               // 找到解，立即返回
        nextLimit = min(nextLimit, t);        // 记录最小的超限值

        path.pop_back();
        swap(cur[pos], cur[nx * 3 + ny]);
    }
    return nextLimit;
}

vector<char> idaStar(const State &start) {
    vector<char> path;
    int limit = h(start);
    State cur = start;

    while (true) {
        int t = dfs(cur, 0, limit, -1, path);
        if (t == -1) return path;             // 找到
        if (t == INT_MAX) return {};          // 无解
        limit = t;                            // 提高阈值
    }
}
```

---

## A* vs IDA* 对比

| 维度 | A* | IDA* |
|------|-----|------|
| 数据结构 | 优先队列 (堆) | 递归栈 |
| 空间 | O(访问节点数) | O(递归深度) |
| 适用 | 状态数 ≤ 10⁶ | 状态数巨大但深度适中 |
| 实现 | 需优先队列 + 去重 | 只需递归 |
| 八数码 | 勉强能过 | 推荐方案 |
| 十五数码 | 不能过（状态太多） | 能过（已知深度短） |

---

## 实际竞赛选择建议

| 题目特征 | 推荐算法 |
|----------|----------|
| 状态数 < 10⁵，求最短路径 | BFS |
| 状态数大但 ≤ 5×10⁵，有启发函数 | A* |
| 状态数巨大，已知解在较浅层 | IDA* |
| 没有好的启发函数 | 双向 BFS |
| 无权重 / 全等权边 | BFS（A* 退化同 BFS） |

---

## CSP 真题场景

- **八数码 / 十五数码**：IDA* 标准模板题
- **骑士精神 (Knight Moves)**：IDA* + 估价剪枝
- **S2023 T3** 结构体 — 搜索型题目可考虑 A* 优化
- 各省省选中启发式搜索偶尔出现（A* 在 CSP-S 范围属进阶内容）

## ⚠️ 易错点

1. **h(n) 必须 ≤ 实际代价**：用曼哈顿距离时确保每步最多减少 1
2. **A* 优先队列的"懒删除"**：同一个状态可能被多次入队，`g > dist[cur]` 跳过旧记录
3. **IDA* 的 limit 更新**：不是 `limit++`，而是取 `min(nextLimit, t)`，否则迭代次数过多
4. **逆序对判无解**：八数码中，目标逆序对为偶数，起始为奇数 → 无解，直接返回避免无限搜索
5. **IDA* 每轮都要从头搜**：这是设计使然，空间换时间的典型取舍
6. **估价函数的设计**：并非越精确越好——太复杂的估价函数本身耗时不划算

## 相关笔记

- [[BFS与DFS]] — 普通搜索的基础
- [[剪枝与记忆化搜索]] — IDA* 本质是估价剪枝
- [[04-图论]] — Dijkstra 可视作 A* 的特例 (h=0)
