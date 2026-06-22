---
tags: [CSP, 动态规划, 数位DP]
difficulty: S
created: 2026-06-06
---

# 数位DP

## 核心思想

**数位 DP** 统计区间 `[l, r]` 内满足特定条件的整数个数。利用差分：`solve(r) - solve(l-1)`。

核心技巧：**记忆化搜索**，按位 DFS，维护 `tight`（是否受限）、`lead`（是否有前导零）等状态。

## 通用记忆化搜索模板

```cpp
int dp[pos][other_states];
int digits[20];

int dfs(int pos, bool tight, bool lead, /* 其他状态 */) {
    if (pos == -1) return 1;
    if (!tight && !lead && dp[pos][/*状态*/] != -1)
        return dp[pos][/*状态*/];

    int up = tight ? digits[pos] : 9;
    int res = 0;
    for (int d = 0; d <= up; d++) {
        // 根据 d 更新状态，限制条件写这里
        res += dfs(pos - 1, tight && (d == up), lead && (d == 0), /* 新状态 */);
    }

    if (!tight && !lead)
        dp[pos][/*状态*/] = res;  // 缓存
    return res;
}

int solve(int x) {
    if (x < 0) return 0;
    int len = 0;
    while (x) { digits[len++] = x % 10; x /= 10; }
    memset(dp, -1, sizeof(dp));
    return dfs(len - 1, true, true, /* 初始状态 */);
}
// 答案: solve(r) - solve(l-1)
```

## 经典例题：不含62和4的数字

统计 `[l, r]` 中不含数字 `4` 且不含连续 `62` 的整数个数。

```cpp
int dp[10][2];  // dp[pos][preIs6]: 前一位是否为6
int digits[10];

int dfs(int pos, bool tight, int preIs6) {
    if (pos == -1) return 1;
    if (!tight && dp[pos][preIs6] != -1) return dp[pos][preIs6];

    int up = tight ? digits[pos] : 9;
    int res = 0;
    for (int d = 0; d <= up; d++) {
        if (d == 4) continue;
        if (preIs6 && d == 2) continue;
        res += dfs(pos - 1, tight && (d == up), d == 6);
    }

    if (!tight) dp[pos][preIs6] = res;
    return res;
}

int solve(int x) {
    if (x < 0) return 0;
    int len = 0;
    while (x) { digits[len++] = x % 10; x /= 10; }
    memset(dp, -1, sizeof(dp));
    return dfs(len - 1, true, 0);
}
```

## 各位数字之和

统计 `[l, r]` 中所有数字的各位数字之和。

```cpp
int dp[20][200];  // dp[pos][sum]

int dfs(int pos, bool tight, int sum) {
    if (pos == -1) return sum;  // 注意：返回sum而非1
    if (!tight && dp[pos][sum] != -1) return dp[pos][sum];

    int up = tight ? digits[pos] : 9;
    int res = 0;
    for (int d = 0; d <= up; d++) {
        res += dfs(pos - 1, tight && (d == up), sum + d);
    }
    if (!tight) dp[pos][sum] = res;
    return res;
}
```

## Windy数

相邻两个数字之差至少为2的正整数（不含前导零）。

```cpp
int dfs(int pos, bool tight, bool lead, int pre) {
    if (pos == -1) return 1;
    if (!tight && !lead && dp[pos][pre] != -1) return dp[pos][pre];

    int up = tight ? digits[pos] : 9;
    int res = 0;
    for (int d = 0; d <= up; d++) {
        if (!lead && abs(d - pre) < 2) continue;
        res += dfs(pos - 1, tight && (d == up), lead && (d == 0), d);
    }
    if (!tight && !lead) dp[pos][pre] = res;
    return res;
}
```

## 状态设计要点总结

| 参数 | 含义 | 何时需要 |
|------|------|---------|
| `pos` | 当前位下标 | 必须 |
| `tight` | 是否受限 | 必须 |
| `lead` | 前导零 | 当"前导零影响判定"时需要 |
| `pre` | 前一位数字 | 连续条件（如62、相邻差） |
| `sum` | 前缀和 | 统计数字之和 |
| `mod` | 取模余数 | 整除性判定 |

## 进制变种

数位DP不限于十进制。`up = tight ? digits[pos] : 9` 中的 `9` 改为 `进制-1`，按位拆分也改为目标进制即可。

## ⚠️ 易错点

1. **记忆化条件**：只有 `!tight && !lead` 时才能缓存（tight时状态唯一，不需要记忆化）
2. **前导零处理**：前导零可能导致错误判断（如Windy数：02不合法但应等处理到非零位再判断）
3. **solve(x)的边界**：x<0返回0，x==0要正确处理（len=0时）
4. **dp数组维度**：注意不同状态组合是否超过数组大小
5. **dp初始化为-1**：每次solve前需要memset
6. **递归深度**：最多20位，递归不会栈溢出

## 相关笔记

- [[状压DP]]
- [[DP优化技巧]]
- [[08-MOC]]
- [[CSP-备考-MOC]]
