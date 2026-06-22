---
tags: [CSP, 动态规划, 状压DP]
difficulty: S
created: 2026-06-06
---

# 状压DP（状态压缩DP）

## 核心思想

**状压 DP** 用二进制整数表示集合状态，适用于元素数 `n ≤ 20` 的场景。将"选/不选"映射为 `1/0`，用 int 存储整个状态，通过位运算高效操作。

## 常用位运算速查

```cpp
int mask = 0;

// 基础操作
mask | (1 << k);           // 将第 k 位设为 1
mask & ~(1 << k);          // 将第 k 位设为 0
mask ^ (1 << k);           // 翻转第 k 位
(mask >> k) & 1;           // 判断第 k 位是否为 1

// 全集
(1 << n) - 1;              // n 位全 1 的 mask

// 统计 mask 中 1 的个数
__builtin_popcount(mask);  // GCC/Clang 内置函数

// 去掉最低位的 1
mask & (mask - 1);

// 取最低位的 1
mask & (-mask);            // lowbit

// 枚举 mask 的所有子集（包括空集和 mask 自身）
for (int sub = mask; ; sub = (sub - 1) & mask) {
    // 处理 sub
    if (sub == 0) break;
}
```

**枚举子集的子集**的时间复杂度：O(3^n)，因为每个元素在子集中有 3 种状态：属于 mask 且属于 sub、属于 mask 但不属于 sub、不属于 mask。

## TSP（旅行商问题）

n 个城市，访问每个城市恰好一次后回到起点，求最短路径。

状态定义：`dp[mask][i]` = 已访问城市集合为 `mask`，当前在城市 `i` 的最短路径长度。

```cpp
int tsp(vector<vector<int>>& dist) {
    int n = dist.size();
    int FULL = (1 << n) - 1;
    vector<vector<int>> dp(1 << n, vector<int>(n, INF));

    dp[1][0] = 0;  // 从城市0出发，只访问了0

    for (int mask = 1; mask <= FULL; mask++) {
        for (int i = 0; i < n; i++) {
            if (!(mask & (1 << i))) continue;     // i 不在 mask 中
            if (dp[mask][i] == INF) continue;
            for (int j = 0; j < n; j++) {
                if (mask & (1 << j)) continue;    // j 已访问，跳过
                int nxt = mask | (1 << j);
                dp[nxt][j] = min(dp[nxt][j], dp[mask][i] + dist[i][j]);
            }
        }
    }

    // 回到起点（城市0）
    int ans = INF;
    for (int i = 1; i < n; i++) {
        ans = min(ans, dp[FULL][i] + dist[i][0]);
    }
    return ans;
}
```

时间复杂度 O(n²·2^n)，空间 O(n·2^n)。若不需要回到起点，直接 `min(dp[FULL][i])`。

## 哈密顿路径计数

判断是否存在哈密顿路径（访问每个点恰好一次，不要求回路）。

状态定义类似 TSP，但最后答案取 `dp[FULL][i]` 的最小/最大/方案数。

## 互不侵犯 / 炮兵阵地（网格状压）

典型题：在 n×m 网格放置棋子，满足相邻限制。常一行一行 DP：`dp[row][state]`。

```cpp
// 炮兵阵地：每行用一个二进制数表示放置状态
// dp[i][mask] = 前 i 行，第 i 行状态为 mask 的最大炮兵数
// 预处理：valid[mask] 表示同行内不冲突且符合地形
// dp[i][mask] = max(dp[i-1][pre] + cnt[mask])
//      条件：mask 与 pre 不冲突
// 使用滚动数组优化：dp[2][mask]

vector<int> validStates;  // 预处理的合法状态列表
int dp[2][1 << m];        // 滚动数组
int cur = 0;

for (int mask : validStates) {
    dp[cur][mask] = cnt[mask];  // 第一行
}
for (int i = 1; i < n; i++) {
    cur ^= 1;
    memset(dp[cur], 0, sizeof(dp[cur]));
    for (int curMask : validStates) {
        for (int preMask : validStates) {
            if (curMask & preMask) continue;  // 同列冲突
            dp[cur][curMask] = max(dp[cur][curMask],
                dp[cur^1][preMask] + cnt[curMask]);
        }
    }
}
```

## 集合划分问题

将 n 个元素分成若干组，满足每组内元素约束，求最小分组数 / 最大收益。

```cpp
// cost[mask] = 子集 mask 能否独立成组（或组内代价）
int dp[1 << n];
dp[0] = 0;
for (int mask = 1; mask < (1 << n); mask++) {
    dp[mask] = INF;
    for (int sub = mask; sub; sub = (sub - 1) & mask) {
        if (valid[sub]) {  // sub 可以独立成一组
            dp[mask] = min(dp[mask], dp[mask ^ sub] + cost[sub]);
        }
    }
}
```

## 空间优化技巧

1. **滚动数组**：行数维度只保留 2 维
2. **用 map/unordered_map 存稀疏状态**：如果很多状态不可达，避免开满 2^n 空间
3. **bitset 辅助判定**：快速判断状态合法性

## ⚠️ 易错点

1. **n 太大不能用状压**：n > 24 时 2^n 爆空间/时间
2. **位运算优先级**：`mask & (1 << j) == 0` 会先算 `==`，应写成 `(mask & (1 << j)) == 0`
3. **TSP 初始化**：必须初始化某起点，只初始化 `dp[1 << start][start] = 0`
4. **INF 取值**：别取 `INT_MAX`，加法会溢出；用 `0x3f3f3f3f` 或 `1e9`
5. **子集枚举的 `--` 顺序**：`sub = (sub - 1) & mask`，不是 `sub-- & mask`
6. **滚动数组忘记清空**：切换行时 `memset` 或重置

## 相关笔记

- [[树形DP]]
- [[数位DP]]
- [[DP优化技巧]]
- [[08-MOC]]
- [[CSP-备考-MOC]]
