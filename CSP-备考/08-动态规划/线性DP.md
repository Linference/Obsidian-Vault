---
tags: [CSP, 动态规划, 线性DP]
difficulty: S
created: 2026-06-06
---

# 线性DP

## 核心思想

**线性DP** 是最基础的 DP 类型，状态按线性顺序排列（一维数组或二维矩阵），转移方向通常是前向后。特征：状态维度少，转移关系简单，但需要深入理解"无后效性"。

## 最长上升子序列 (LIS)

### O(n²) 做法

状态定义：`dp[i]` = 以 `a[i]` 结尾的 LIS 长度。

```cpp
int LIS(vector<int>& a) {
    int n = a.size();
    vector<int> dp(n, 1);  // 每个元素本身长度为1
    int ans = 1;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (a[j] < a[i]) {
                dp[i] = max(dp[i], dp[j] + 1);
            }
        }
        ans = max(ans, dp[i]);
    }
    return ans;
}
```

### O(n log n) 做法（贪心 + 二分）

维护 `tails` 数组，`tails[k]` 表示长度为 `k+1` 的上升子序列的最小末尾值。

```cpp
int LIS_fast(vector<int>& a) {
    vector<int> tails;
    for (int x : a) {
        auto it = lower_bound(tails.begin(), tails.end(), x);
        if (it == tails.end()) {
            tails.push_back(x);  // 扩展长度
        } else {
            *it = x;             // 更新最小末尾
        }
    }
    return tails.size();
}
```

> 注意：`lower_bound` 求**严格上升**；若要求**不下降**，用 `upper_bound`。

## 最长公共子序列 (LCS)

状态定义：`dp[i][j]` = `s1` 前 `i` 个字符与 `s2` 前 `j` 个字符的 LCS 长度。

```cpp
int LCS(string& s1, string& s2) {
    int n = s1.size(), m = s2.size();
    vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            if (s1[i-1] == s2[j-1]) {
                dp[i][j] = dp[i-1][j-1] + 1;
            } else {
                dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
            }
        }
    }
    return dp[n][m];
}
```

时间复杂度 O(nm)，空间 O(nm) 可优化为 O(min(n,m))。

### LCS 输出具体序列

从 `dp[n][m]` 回溯：

```cpp
string getLCS(string& s1, string& s2) {
    // 已计算 dp 数组
    int i = s1.size(), j = s2.size();
    string res;
    while (i > 0 && j > 0) {
        if (s1[i-1] == s2[j-1]) {
            res += s1[i-1];
            i--; j--;
        } else if (dp[i-1][j] > dp[i][j-1]) {
            i--;
        } else {
            j--;
        }
    }
    reverse(res.begin(), res.end());
    return res;
}
```

### 特殊优化

若 s1 是排列（1~n 各出现一次），LCS 可转化为 LIS：
- 将 s1 中字符映射为位置，s2 按映射转换，求转换后序列的 LIS，O(n log n)。

## 最短编辑距离 (Edit Distance)

求将 s1 变为 s2 的最少操作次数（插入、删除、替换）。

```cpp
int editDistance(string& s1, string& s2) {
    int n = s1.size(), m = s2.size();
    vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));
    for (int i = 0; i <= n; i++) dp[i][0] = i;  // 全部删除
    for (int j = 0; j <= m; j++) dp[0][j] = j;  // 全部插入
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            if (s1[i-1] == s2[j-1]) {
                dp[i][j] = dp[i-1][j-1];
            } else {
                dp[i][j] = min({dp[i-1][j] + 1,     // 删除
                                dp[i][j-1] + 1,     // 插入
                                dp[i-1][j-1] + 1}); // 替换
            }
        }
    }
    return dp[n][m];
}
```

## 最大子段和

状态定义：`dp[i]` = 以 `a[i]` 结尾的最大子段和。

```cpp
int maxSubArray(vector<int>& a) {
    int cur = a[0], ans = a[0];
    for (int i = 1; i < a.size(); i++) {
        cur = max(a[i], cur + a[i]);
        ans = max(ans, cur);
    }
    return ans;
}
```

空间 O(1)，无需完整 dp 数组（滚动思想）。

### 变种：最大子矩阵和

枚举上下边界，对每列求和转化为一维最大子段和，O(n³)。

## 数字三角形

从顶部出发，每次向下或右下移动，求路径最大和。

```cpp
int triangle(vector<vector<int>>& a) {
    int n = a.size();
    vector<vector<int>> dp(n, vector<int>(n, 0));
    dp[0][0] = a[0][0];
    for (int i = 1; i < n; i++) {
        dp[i][0] = dp[i-1][0] + a[i][0];         // 左边界
        for (int j = 1; j < i; j++) {
            dp[i][j] = max(dp[i-1][j-1], dp[i-1][j]) + a[i][j];
        }
        dp[i][i] = dp[i-1][i-1] + a[i][i];       // 右边界
    }
    return *max_element(dp[n-1].begin(), dp[n-1].end());
}
```

## ⚠️ 易错点

1. **LIS 初始化**：`dp[i]` 至少为 1（元素自身），不要初始化为 0
2. **LCS 坐标**：`dp[i][j]` 对应 `s1[i-1]` 和 `s2[j-1]`，注意下标偏移
3. **编辑距离边界**：`dp[0][j] = j`（插入 j 个字符），`dp[i][0] = i`（删除 i 个字符）
4. **最大子段和**：全负数时 `max` 返回的是最大负数，不是 0
5. **降维陷阱**：LCS 降维时逆序枚举防止覆盖

## 相关笔记

- [[背包问题全集]]
- [[区间DP]]
- [[08-MOC]]
- [[CSP-备考-MOC]]
