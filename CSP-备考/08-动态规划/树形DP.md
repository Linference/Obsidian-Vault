---
tags: [CSP, 动态规划, 树形DP]
difficulty: S
created: 2026-06-06
---

# 树形DP

## 核心思想

**树形 DP** 在树上做动态规划，利用树的递归结构自底向上（后序遍历）计算。当前节点的状态由子节点状态推导。

关键：DFS 作为 DP 的计算顺序，天然保证拓扑序。

## 树的最大独立集

在树上选出尽可能多的节点，使得任意两个相邻节点不同时被选。

状态定义：
- `dp[u][0]`：不选节点 u 时，子树 u 的最大独立集大小
- `dp[u][1]`：选节点 u 时，子树 u 的最大独立集大小

```cpp
vector<int> tree[MAXN];
int dp[MAXN][2];

void dfs(int u, int parent) {
    dp[u][0] = 0;          // 不选 u
    dp[u][1] = 1;          // 选 u（贡献1个节点）
    for (int v : tree[u]) {
        if (v == parent) continue;
        dfs(v, u);
        dp[u][0] += max(dp[v][0], dp[v][1]);   // u不选，子节点可任选
        dp[u][1] += dp[v][0];                  // u选了，子节点不能选
    }
}
// 答案: max(dp[root][0], dp[root][1])
```

### 变种：带权独立集

将 `dp[u][1] = 1` 改为 `dp[u][1] = weight[u]`。典型题：**没有上司的舞会**。

## 树的最小点覆盖

选出最少的点，使得每条边至少有一个端点被选中。

```cpp
// dp[u][0] = 不选 u，则所有子节点必须被选 → sum(dp[v][1])
// dp[u][1] = 选 u → sum(min(dp[v][0], dp[v][1]))

void dfs(int u, int parent) {
    dp[u][0] = 0;
    dp[u][1] = 1;
    for (int v : tree[u]) {
        if (v == parent) continue;
        dfs(v, u);
        dp[u][0] += dp[v][1];
        dp[u][1] += min(dp[v][0], dp[v][1]);
    }
}
```

## 树的直径（DP 做法）

直径 = 树上最远两点间的路径长度。DP 做法：对每个节点，计算经过它的最长路径。

```cpp
int diameter = 0;

int dfs_diameter(int u, int parent) {
    int max1 = 0, max2 = 0;  // 子树中最长的两条路径
    for (int v : tree[u]) {
        if (v == parent) continue;
        int len = dfs_diameter(v, u) + 1;  // 子链长度 + 边(u,v)
        if (len > max1) {
            max2 = max1; max1 = len;
        } else if (len > max2) {
            max2 = len;
        }
    }
    diameter = max(diameter, max1 + max2);  // 经过u的最长路径
    return max1;  // 返回以u为起点的最长链
}
```

> 另一种方法：两次 BFS/DFS，从任意点出发找最远点 a，再从 a 出发找最远点 b。

## 树的重心

删除重心后，剩余最大连通块最小。可以 DP 做：`sz[u]` 子树大小，`max_part = max(max_part, n - sz[u])`。

```cpp
int n, centroid, minBalance = n;

int dfs_centroid(int u, int parent) {
    int sz = 1, maxPart = 0;
    for (int v : tree[u]) {
        if (v == parent) continue;
        int sub = dfs_centroid(v, u);
        sz += sub;
        maxPart = max(maxPart, sub);
    }
    maxPart = max(maxPart, n - sz);  // 父节点方向的连通块
    if (maxPart < minBalance) {
        minBalance = maxPart;
        centroid = u;
    }
    return sz;
}
```

## 树上背包

在树上有依赖的背包问题：每个节点选或不选，选了子节点才能选父节点（或反之）。

状态定义：`dp[u][j]` 表示以 u 为根的子树中选择 j 个节点（且必须选 u）的最大价值。

```cpp
void dfs_knap(int u, int parent) {
    dp[u][1] = value[u];  // 至少选自己
    sz[u] = 1;
    for (int v : tree[u]) {
        if (v == parent) continue;
        dfs_knap(v, u);
        // 合并背包（01背包风格，逆序）
        for (int j = sz[u]; j >= 1; j--) {
            for (int k = 1; k <= sz[v]; k++) {
                dp[u][j + k] = max(dp[u][j + k], dp[u][j] + dp[v][k]);
            }
        }
        sz[u] += sz[v];
    }
}
```

时间复杂度看似 O(n³)，实为 O(n²)（每对节点在 LCA 处合并一次）。

## 换根DP (Re-rooting)

当需要以**每个节点为根**分别计算答案时，通过两次 DFS 避免 O(n²)。核心：第一次按固定根计算，第二次利用父节点信息推子节点。

```cpp
int dp1[N];  // 以固定根（如0）计算的结果
int ans[N];  // 每个节点为根的最终答案

void dfs1(int u, int p) { /* 正常树形DP，计算dp1 */ }
void dfs2(int u, int p) {  // 换根
    ans[u] = /* 计算以u为根的答案 */;
    for (int v : tree[u]) if (v != p) {
        // 修改 dp1，把 v 当成根
        dfs2(v, u);
        // 恢复 dp1
    }
}
```

## 基环树DP

基环树 = 树 + 一条额外边（n 个点 n 条边）。处理方式：
1. 找到环
2. 对环上每条边，考虑断不断开（两种可能状态）
3. 对环上每个节点，其子树做普通树形 DP

## ⚠️ 易错点

1. **忘记传 parent 参数**：树是双向边，不传 parent 会死循环
2. **领接表 vs 邻接矩阵**：树的边数 = n-1，邻接矩阵浪费空间
3. **树上背包内层循环顺序**：容量 j 逆序，避免重复算同一子节点
4. **直径 DP 的返回值**：返回从 u 出发的最长链，而不是直径本身
5. **换根 DP 的回退操作**：修改全局状态后必须恢复

## 相关笔记

- [[线性DP]]
- [[区间DP]]
- [[状压DP]]
- [[08-MOC]]
- [[CSP-备考-MOC]]
