---
tags: [CSP, 动态规划, 区间DP]
difficulty: S
created: 2026-06-06
---

# 区间DP

## 核心思想

区间 DP 将状态定义为**区间 `[l, r]`**，通过枚举分割点 `k`，将大区间拆分为两个子区间合并求解。本质是"分治 + 重叠子问题"。

## 通用模板

```cpp
// dp[l][r] 表示区间 [l, r] 的最优解
// 初始化：dp[i][i] = 基础值（长度为1的区间）; 其余为 INF 或 0

for (int len = 2; len <= n; len++) {           // 枚举区间长度
    for (int l = 0; l + len - 1 < n; l++) {    // 枚举左端点
        int r = l + len - 1;                    // 右端点
        for (int k = l; k < r; k++) {           // 枚举分割点
            dp[l][r] = min(dp[l][r], dp[l][k] + dp[k+1][r] + cost(l, r));
        }
    }
}
```

**循环顺序很关键**：必须按 `len` 从小到大计算，保证子区间先被算出。

## 石子合并（经典入门题）

N 堆石子排成一排，每次合并相邻两堆，代价为两堆石子数之和。求最小总代价。

```cpp
#include <bits/stdc++.h>
using namespace std;

int stoneMerge(vector<int>& a) {
    int n = a.size();
    vector<int> prefix(n + 1, 0);
    for (int i = 1; i <= n; i++) prefix[i] = prefix[i-1] + a[i-1];
    auto sum = [&](int l, int r) { return prefix[r+1] - prefix[l]; };

    vector<vector<int>> dp(n, vector<int>(n, INT_MAX));
    for (int i = 0; i < n; i++) dp[i][i] = 0;  // 单堆代价为0

    for (int len = 2; len <= n; len++) {
        for (int l = 0; l + len - 1 < n; l++) {
            int r = l + len - 1;
            for (int k = l; k < r; k++) {
                dp[l][r] = min(dp[l][r],
                    dp[l][k] + dp[k+1][r] + sum(l, r));
            }
        }
    }
    return dp[0][n-1];
}
```

时间复杂度 O(n³)，空间 O(n²)。

## 环形石子合并（破环成链）

将环形数组复制一份接在后面，长度变为 `2n`，DP 后在 `dp[i][i+n-1]`（i=0..n-1）中取最值。

```cpp
// 环形：a[0..n-1]，复制为 b[0..2n-1]
vector<int> b(2 * n);
for (int i = 0; i < n; i++) b[i] = b[i + n] = a[i];

int ans = INT_MAX;
for (int i = 0; i < n; i++) {
    ans = min(ans, dp[i][i + n - 1]);  // dp 按 b 的 [0, 2n) 计算
}
```

## 矩阵链乘

给定 n 个矩阵的维度 `(p[0]×p[1]), (p[1]×p[2]), ..., (p[n-1]×p[n])`，求最少标量乘法次数。

```cpp
int matrixChain(vector<int>& p) {
    int n = p.size() - 1;  // 矩阵个数
    vector<vector<int>> dp(n, vector<int>(n, 0));
    for (int len = 2; len <= n; len++) {
        for (int i = 0; i + len - 1 < n; i++) {
            int j = i + len - 1;
            dp[i][j] = INT_MAX;
            for (int k = i; k < j; k++) {
                dp[i][j] = min(dp[i][j],
                    dp[i][k] + dp[k+1][j] + p[i] * p[k+1] * p[j+1]);
            }
        }
    }
    return dp[0][n-1];
}
```

## 最长回文子序列

```cpp
int longestPalindromeSubseq(string s) {
    int n = s.size();
    vector<vector<int>> dp(n, vector<int>(n, 0));
    for (int i = 0; i < n; i++) dp[i][i] = 1;

    for (int len = 2; len <= n; len++) {
        for (int i = 0; i + len - 1 < n; i++) {
            int j = i + len - 1;
            if (s[i] == s[j])
                dp[i][j] = dp[i+1][j-1] + 2;
            else
                dp[i][j] = max(dp[i+1][j], dp[i][j-1]);
        }
    }
    return dp[0][n-1];
}
```

## 括号序列

给定包含 `()` `[]` 的字符串，求最长合法括号子序列长度。合法对 `()` 或 `[]`，或由合法对拼接/嵌套。

```cpp
// dp[l][r] = 区间 [l,r] 的最长长度
// 若 s[l] 与 s[r] 匹配: dp[l][r] = dp[l+1][r-1] + 2
// 枚举分割 k: dp[l][r] = max(dp[l][r], dp[l][k] + dp[k+1][r])
```

## 四边形不等式优化（简介）

当 `cost(l, r)` 满足 **四边形不等式** 时（即 `cost(a,c) + cost(b,d) <= cost(a,d) + cost(b,c)` for `a<=b<=c<=d`），最优决策点 `k` 满足 `opt[l][r-1] <= opt[l][r] <= opt[l+1][r]`，可将 O(n³) 优化为 O(n²)。

```cpp
int opt[n][n];  // 记录最优分割点
for (int len = 2; len <= n; len++) {
    for (int l = 0; l + len - 1 < n; l++) {
        int r = l + len - 1;
        int kl = opt[l][r-1], kr = opt[l+1][r];
        for (int k = kl; k <= kr; k++) {  // k 的范围缩到 O(1)
            // 更新 dp[l][r]
        }
    }
}
```

## ⚠️ 易错点

1. **循环顺序**：必须按 `len` 从小到大，不能按 `l` 从 0 到 n
2. **初始化遗漏**：`dp[i][i]` 必须初始化为正确基础值
3. **合并代价遗漏**：区间 DP 往往需要在合并时加上该区间的总代价（如 `sum(l,r)`）
4. **环形问题忘记 2n 长度**：复制数组后 DP 范围是 `[0, 2n)`
5. **INT_MAX 溢出**：累加时可能溢出，改用 `long long` 或初始化时加判断

## 相关笔记

- [[线性DP]]
- [[DP优化技巧]]
- [[08-MOC]]
- [[CSP-备考-MOC]]
