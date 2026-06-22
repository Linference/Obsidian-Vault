---
tags: [CSP, 字符串]
difficulty: S
created: 2026-06-06
---

# Manacher 与 Z 函数

## Manacher 算法

### 核心思想

求字符串的**最长回文子串**，暴力 O(n²)，Manacher 做到 **O(n)**。

关键技巧：
1. **预处理填充**：在每两个字符间插入 `#`，将奇偶回文统一为奇数长度处理。`"abba"` → `"#a#b#b#a#"`
2. **利用对称性**：维护已找到的最右回文右边界 R 及其中心 C，当计算位置 i 的回文半径时，利用关于 C 的对称点 `i' = 2C - i` 的信息加速。

### 回文半径数组

`d1[i]`：以 i 为中心、奇数长度的回文半径（含中心，即半径字符数）。
在填充后的字符串中，`d[i]` 表示以 i 为中心的最长回文半径 — 实际回文长度为 `d[i] - 1`。

### C++ 模板

```cpp
#include <bits/stdc++.h>
using namespace std;

// 返回最长回文子串的长度
int manacher(const string &s) {
    // 预处理：插入 #
    string t = "#";
    for (char ch : s) {
        t += ch;
        t += '#';
    }

    int n = t.size();
    vector<int> d(n, 1);    // d[i]：以 i 为中心的回文半径
    int C = 0, R = 0;       // 最右回文的中心和右边界
    int maxLen = 1;

    for (int i = 0; i < n; i++) {
        // 充分利用对称性
        if (i < R) {
            int mirror = 2 * C - i;
            d[i] = min(d[mirror], R - i);
        }

        // 暴力扩展
        while (i - d[i] >= 0 && i + d[i] < n
               && t[i - d[i]] == t[i + d[i]]) {
            d[i]++;
        }

        // 更新最右回文
        if (i + d[i] - 1 > R) {
            C = i;
            R = i + d[i] - 1;
        }

        maxLen = max(maxLen, d[i] - 1);
    }
    return maxLen;
}

// 返回所有回文半径（可进一步定位每个回文子串）
vector<int> manacherFull(const string &t) {
    int n = t.size();
    vector<int> d(n, 1);
    int C = 0, R = 0;
    for (int i = 0; i < n; i++) {
        if (i < R)
            d[i] = min(d[2 * C - i], R - i);
        while (i - d[i] >= 0 && i + d[i] < n
               && t[i - d[i]] == t[i + d[i]])
            d[i]++;
        if (i + d[i] - 1 > R) {
            C = i;
            R = i + d[i] - 1;
        }
    }
    return d;
}

// 判断子串 s[l..r] 是否回文 (0-indexed)
bool isPalindrome(int l, int r, const vector<int> &d) {
    int center = (l + r + 2);  // 映射到 t 中的中心位置
    return d[center] - 1 >= r - l + 1;
}
```

### 复杂度分析

虽然是两层循环，但内层 while 每次成功扩展都会推动 R 向右移动，R 最多移动 n 次，总复杂度 **O(n)**。

---

## Z 函数 (Z Algorithm)

### 核心思想

**Z[i]**：字符串 s 和 s[i..] 的**最长公共前缀 (LCP)** 的长度。

$$Z[i] = \text{LCP}(s[0..], s[i..])$$

特殊定义：Z[0] = 0（或 n，随实现而定）。

### C++ 模板

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> zFunction(const string &s) {
    int n = s.size();
    vector<int> z(n, 0);
    int L = 0, R = 0;         // 最右匹配段 [L, R)
    for (int i = 1; i < n; i++) {
        if (i < R)
            z[i] = min(R - i, z[i - L]);
        while (i + z[i] < n && s[z[i]] == s[i + z[i]])
            z[i]++;
        if (i + z[i] > R) {
            L = i;
            R = i + z[i];
        }
    }
    return z;
}
```

### 应用场景

| 场景 | 用法 |
|------|------|
| 字符串匹配 | 构造 `t = P + '#' + T`，Z 值 = \|P\| 的位置即匹配 |
| 求 border | Z 函数与 next 数组可以互相转化 |
| LCP 查询 | Z 函数天然支持 |
| 字符串周期 | 结合 Z 函数判断最小循环节 |

### Z 函数 vs Manacher

| 维度 | Manacher | Z 函数 |
|------|----------|--------|
| 目标 | 回文子串 | LCP（最长公共前缀） |
| 对称性 | 利用回文的对称 | 利用已匹配区间的对称 |
| 核心变量 | 中心 C + 右边界 R | 左边界 L + 右边界 R |

两者思想一脉相承：**维护一个"已知的最右区间"，新位置尽可能复用已有信息，不能复用再暴力扩展。**

## CSP 真题场景

- **S2022 T2** 策略游戏 — 可能需要回文判断
- **J2019 T3** 纪念品 — 可用回文做辅助判断
- 字符串中回文相关的题目几乎都能用 Manacher 做到最优

## ⚠️ 易错点

1. **Manacher 填充字符**：必须用原字符串中不出现的字符（`#` 最常用）
2. **半径取 min**：`d[i] = min(d[mirror], R - i)` 中的 `R - i` 不能超过右边界
3. **回文长度换算**：填充后半径 d[i]，实际回文长度 = `d[i] - 1`
4. **Z[0] 的定义**：通常设为 0；设为 n 也可以但要统一
5. **Z 函数 while 条件**：`s[z[i]] == s[i + z[i]]` 不是 `s[i]`
6. **边界条件**：扩展时注意数组下标不要越界

## 相关笔记

- [[字符串哈希]] — 哈希 + 二分也能 O(n log n) 求最长回文，实现更简单
- [[KMP]] — KMP 的 next 数组与 Z 函数互有联系
- [[Trie与AC自动机]] — 回文相关的多模式匹配可用 AC 自动机
