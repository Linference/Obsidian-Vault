---
tags: [CSP, 字符串]
difficulty: S
created: 2026-06-06
---

# KMP 算法

## 核心思想

KMP（Knuth-Morris-Pratt）解决**单模式匹配**：在文本串 T 中找模式串 P 的所有出现位置。

暴力做法 O(n·m)，KMP 做到 **O(n+m)**。核心是**失配时不回退 i（文本指针）**，只回退 j（模式指针），利用已经匹配过的信息跳过不必要的比较。

## next 数组（失配函数）

`next[i]`：P[0..i] 这个前缀中，**最长相等前后缀**的长度。

- 前缀：从开头到某个位置的子串（不含最后一个字符）
- 后缀：从某个位置到末尾的子串（不含第一个字符）
- 例如 P = "ababc"，next[4] = 2，因为 "ab" 既是前缀也是后缀

**直观理解**：当 P[j] 和 T[i] 失配时，已匹配的部分是 P[0..j-1]。既然 next[j-1] 是这个已匹配部分的最长公共前后缀，我们就直接把 P 向右滑动到那个位置继续匹配。

## C++ 模板

```cpp
#include <bits/stdc++.h>
using namespace std;

// 构造 next 数组
vector<int> buildNext(const string &p) {
    int m = p.size();
    vector<int> nxt(m, 0);
    // nxt[0] = 0，单个字符无真前后缀
    for (int i = 1, j = 0; i < m; i++) {
        while (j > 0 && p[i] != p[j])
            j = nxt[j - 1];       // 回退：尝试更短的前缀
        if (p[i] == p[j])
            j++;
        nxt[i] = j;
    }
    return nxt;
}

// KMP 匹配：返回所有匹配起始位置 (0-indexed)
vector<int> kmp(const string &t, const string &p) {
    vector<int> nxt = buildNext(p);
    vector<int> ans;
    int n = t.size(), m = p.size();
    for (int i = 0, j = 0; i < n; i++) {
        while (j > 0 && t[i] != p[j])
            j = nxt[j - 1];       // 失配：j 回退
        if (t[i] == p[j])
            j++;
        if (j == m) {             // 完全匹配
            ans.push_back(i - m + 1);
            j = nxt[j - 1];       // 继续找下一个匹配
        }
    }
    return ans;
}
```

### 关键细节解释

**构造 next 的回退 `j = nxt[j-1]`**：这其实是在"用自己匹配自己"。当 P[i] != P[j] 时，我们已知 P[0..j-1] 的 next 值，所以跳到更短的可能前缀继续尝试。

**匹配中的 `j = nxt[j-1]`**：找到一次完整匹配后，不重新开始，而是利用 next 跳过已知重叠。

## 最小循环节

**定理**：若 `m % (m - next[m-1]) == 0`，则字符串的最小循环节长度为 `m - next[m-1]`。

```cpp
int minCycle(const string &p) {
    auto nxt = buildNext(p);
    int m = p.size();
    int cycle = m - nxt[m - 1];
    return (m % cycle == 0) ? cycle : m;
}
```

- "abcabcabc" → next[8] = 6，周期 = 9 - 6 = 3 ✓
- "abcab" → next[4] = 2，5 % (5-2) ≠ 0，无完整循环节

## 复杂度分析

| 步骤 | 复杂度 | 说明 |
|------|--------|------|
| 构造 next | O(m) | j 最多增加 m 次，每次回退至少减少 1 |
| 匹配过程 | O(n) | 同理，j 的增加次数不超过 n |
| 总计 | O(n+m) | 线性 |

## KMP vs 字符串哈希

| 维度 | KMP | 字符串哈希 |
|------|-----|-----------|
| 单模式匹配 | O(n+m)，稳定 | O(n+m)，可能哈希冲突 |
| 实现难度 | 较高（next 数组） | 低 |
| 多模式匹配 | 需扩展为 AC 自动机 | 需要逐一计算 |
| 额外功能 | 求循环节 | 求 LCP、任意子串比较 |
| 适用 | 精确匹配，不信任随机 | 需要灵活子串操作 |

**CSP 建议**：两个都学。哈希实现快，KMP 思想更重要（next 数组和 AC 自动机的基础）。

## CSP 真题场景

- **J2020 T2** 字符串匹配 — 可直接套 KMP
- **S2019 T1** 格雷码 — 可以用字符串匹配思路理解
- **J2023 T3** 一元二次方程 — 可能需要字符串处理

## ⚠️ 易错点

1. **next 数组下标**：`next[i]` 是 P[0..i] 的最长公共前后缀**长度**，不是下标
2. **回退条件 `j > 0`**：忘记会导致死循环或越界
3. **匹配成功后的处理**：`j = nxt[j-1]` 不能写成 `j = 0`，否则 "aaa" 中找 "aa" 会漏掉重叠匹配
4. **循环节判断**：必须 `m % cycle == 0` 才成立，否则最小循环节就是 m 本身
5. **下标从 0 还是 1**：习惯 0-indexed，但有些教材用 1-indexed，注意转换

## 相关笔记

- [[字符串哈希]] — 替代方案，更灵活的子串操作
- [[Trie与AC自动机]] — KMP 扩展到多模式匹配
- [[Manacher与Z函数]] — Z 函数与 next 数组有类似思想
