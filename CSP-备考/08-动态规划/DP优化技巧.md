---
tags: [CSP, 动态规划, 优化]
difficulty: S
created: 2026-06-06
---

# DP优化技巧

## 核心思想

DP 的瓶颈往往在时间复杂度或空间复杂度上。掌握常见优化是 CSP-S 高分的关键，能把 O(n²) 降到 O(n log n) 甚至 O(n)。

## 滚动数组

空间优化：当 `dp[i]` 只依赖 `dp[i-1]` 时，用 2 行轮换。

```cpp
int dp[2][V + 1];  // 仅两行
int cur = 0;
for (int i = 0; i < n; i++) {
    cur ^= 1;
    for (int j = 0; j <= V; j++) {
        dp[cur][j] = dp[cur ^ 1][j];  // 先复制
        if (j >= w[i]) dp[cur][j] = max(dp[cur][j], dp[cur ^ 1][j - w[i]] + v[i]);
    }
}
```

更彻底：倒序压到一维（背包问题的标准做法）。

```cpp
// 当 dp[i] 只依赖 dp[i-1] 且枚举顺序满足时
int dp[V + 1];
for (int i = 0; i < n; i++)
    for (int j = V; j >= w[i]; j--)
        dp[j] = max(dp[j], dp[j - w[i]] + v[i]);
```

## 单调队列优化DP

**适用条件**：转移方程形如 `dp[i] = max/min(dp[k] + f(k))` 且 `k` 的范围是一个滑动窗口。

### 模板：滑动窗口最大值

```cpp
// 求 dp[i] = max(dp[k]) for k in [i-m, i-1]
deque<int> q;  // 存下标，保证 dp[q.front()] 是窗口内最大值（递减队列）

for (int i = 1; i <= n; i++) {
    // 1. 弹出队头过期的元素
    while (!q.empty() && q.front() < i - m) q.pop_front();
    // 2. dp[i] = dp[q.front()] + ...（当前窗口最大值）
    // 3. 将 i 加入队列，维护单调性
    while (!q.empty() && dp[q.back()] <= dp[i]) q.pop_back();
    q.push_back(i);
}
```

### 例题：多重背包的单调队列优化

```cpp
// dp[j] 表示容量 j 的最大价值
// 按余数分类，每组用单调队列求最大值
int dp[V + 1];
for (int i = 0; i < n; i++) {  // n 件物品
    for (int r = 0; r < w[i]; r++) {  // 按余数分类
        deque<int> q;
        for (int j = r; j <= V; j += w[i]) {
            // 维护滑动窗口 [j - c[i]*w[i], j]
            while (!q.empty() && q.front() < j - c[i] * w[i]) q.pop_front();
            while (!q.empty() && dp[q.back()] + (j - q.back()) / w[i] * v[i] <= dp[j])
                q.pop_back();
            q.push_back(j);
            // 计算dp[j]
        }
    }
}
```

## 斜率优化DP

**适用条件**：dp 转移形如 `dp[i] = min(dp[j] + a[i] * b[j] + c[j])`。

核心思想：将 dp[j] 视为自变量和因变量，a[i] 是"斜率"，维护下凸壳/上凸壳。

```cpp
// 经典形式：dp[i] = min(dp[j] + (sum[i] - sum[j])^2)  即 a[i] = -2*sum[i]
// 整理为：dp[j] + sum[j]^2 = 2*sum[i] * sum[j] + dp[i] - sum[i]^2
// 令 Y[j] = dp[j] + sum[j]^2, X[j] = sum[j]
// dp[i] = min(Y[j] - 2*sum[i] * X[j]) + sum[i]^2

// 询问斜率单调递增 → 单调队列维护凸壳
struct Point { long long x, y; };
// 判断 (a,b,c) 三点构成上凸：叉积 (c-a)×(b-a) <= 0 则删b

int q[N], head = 0, tail = 0;  // 队列存下标
q[tail++] = 0;  // 初始点

for (int i = 1; i <= n; i++) {
    // 弹出队头不优的
    while (head + 1 < tail && slope(q[head], q[head+1]) <= a[i]) head++;
    int j = q[head];
    dp[i] = dp[j] + ...;  // 转移
    // 将 i 加入凸壳
    while (head + 1 < tail && cross(q[tail-2], q[tail-1], i) >= 0) tail--;
    q[tail++] = i;
}
```

> 斜率优化需要对推导式非常熟悉，建议多写几个例题找感觉。

## 四边形不等式优化（决策单调性）

已在 [[区间DP]] 中介绍。当 `opt[l][r-1] <= opt[l][r] <= opt[l+1][r]` 时，可将一维枚举压到分摊 O(1)。

```cpp
int opt[N][N];  // 最优分割点
for (int len = 2; len <= n; len++)
    for (int l = 0; l + len - 1 < n; l++) {
        int r = l + len - 1;
        int kl = opt[l][r-1], kr = opt[l+1][r];
        for (int k = kl; k <= kr; k++)
            if (dp[l][r] > dp[l][k] + dp[k+1][r] + cost(l, r))
                dp[l][r] = dp[l][k] + dp[k+1][r] + cost(l, r), opt[l][r] = k;
    }
```

## Bitset 优化

用 `bitset` 做布尔 DP 或 01 背包判定问题，O(nV/64)。

```cpp
bitset<MAXV> dp;
dp[0] = 1;
for (int i = 0; i < n; i++) {
    dp |= dp << w[i];          // 01 背包（本质是 dp[j] |= dp[j-w[i]]）
    // dp = dp | (dp << w[i]); // 完全背包（重复选）
}
// dp[j] == 1 表示可以恰好凑出容量 j
```

## 其他优化技巧

| 技巧 | 适用场景 | 复杂度改善 |
|------|---------|-----------|
| 滚动数组 | 只依赖前一行 | 空间 O(n²)→O(n) |
| 单调队列 | 滑动窗口 max/min | 时间 O(n²)→O(n) |
| 斜率优化 | dp[j]+a[i]×b[j] 形式 | 时间 O(n²)→O(n) |
| 四边形不等式 | 区间DP决策单调 | 时间 O(n³)→O(n²) |
| bitset | 布尔DP/背包 | 时间/N → 时间/64 |
| wqs 二分 | 带限制的凸DP | 去掉一维状态 |
| CDQ 分治 | 二维DP | O(n²)→O(n log n) |
| 矩阵快速幂 | 线性递推 | O(n)→O(log n) |

## 例题

### 例题1：单调队列优化

> 给定数组 a，求长度为 m 的连续子段的最大和（允许不选）。

```cpp
// dp[i] = max(0, max(dp[j])) + a[i] for j in [i-m, i-1]
// 相当于求 a[i] 加一个滑动窗口的最大前缀
```

### 例题2：斜率优化

> 玩具装箱（HNOI2008）：dp[i] = min(dp[j] + (sum[i] - sum[j] + i - j - 1 - L)^2)

整理后使用斜率优化，经典斜率优化入门题。

## ⚠️ 易错点

1. **滚动数组不清空**：每一轮切换行时忘初始化
2. **单调队列的方向**：最大值用递减队列（队头最大），最小值用递增队列
3. **斜率优化整数溢出**：叉积计算用 `long long`
4. **斜率单调性判断**：必须先推导清楚斜率是否单调
5. **bitset 左移越界**：`dp << w[i]` 默认高位超出被丢弃，符预期
6. **决策单调性的前提**：cost 必须满足四边形不等式，不是所有区间 DP 都适用

## 相关笔记

- [[背包问题全集]]
- [[区间DP]]
- [[线性DP]]
- [[08-MOC]]
- [[CSP-备考-MOC]]
