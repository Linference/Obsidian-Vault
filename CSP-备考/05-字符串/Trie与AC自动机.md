---
tags: [CSP, 字符串]
difficulty: S
created: 2026-06-06
---

# Trie 与 AC 自动机

## Trie（字典树 / 前缀树）

### 核心思想

用树形结构高效存储和查询字符串集合。每个节点代表一个前缀，边代表字符。公共前缀共享路径，节省空间 + 加速查询。

### C++ 模板

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 100005;     // 总字符数
const int CHARSET = 26;      // 小写字母

struct Trie {
    int tr[MAXN][CHARSET];   // tr[u][c] = 节点 u 沿字符 c 走到的子节点
    int cnt[MAXN];           // 以该节点结尾的单词数
    int tot;                 // 节点总数（根为 0）

    Trie() : tot(0) {
        memset(tr, 0, sizeof(tr));
        memset(cnt, 0, sizeof(cnt));
    }

    void insert(const string &s) {
        int u = 0;
        for (char ch : s) {
            int c = ch - 'a';
            if (!tr[u][c]) tr[u][c] = ++tot;
            u = tr[u][c];
        }
        cnt[u]++;             // 结尾标记
    }

    int query(const string &s) {
        int u = 0;
        for (char ch : s) {
            int c = ch - 'a';
            if (!tr[u][c]) return 0;  // 不存在
            u = tr[u][c];
        }
        return cnt[u];         // 返回出现次数
    }

    // 检查是否存在以 s 为前缀的字符串
    bool startsWith(const string &s) {
        int u = 0;
        for (char ch : s) {
            int c = ch - 'a';
            if (!tr[u][c]) return false;
            u = tr[u][c];
        }
        return true;
    }
};
```

### 复杂度

| 操作 | 时间 |
|------|------|
| 插入 | O(|S|) |
| 查询 | O(|S|) |

### 应用扩展

**01-Trie（最大异或对）**：将整数二进制表示插入 Trie，每次贪心走相反位。

```cpp
// 查询与 x 异或最大值
int maxXor(int x) {
    int u = 0, ans = 0;
    for (int i = 30; i >= 0; i--) {
        int bit = (x >> i) & 1;
        if (tr[u][bit ^ 1]) {       // 尽量走相反位
            ans |= (1 << i);
            u = tr[u][bit ^ 1];
        } else {
            u = tr[u][bit];
        }
    }
    return ans;
}
```

## AC 自动机 (Aho-Corasick)

### 核心思想

**AC 自动机 = Trie + KMP**。解决**多模式匹配**：在一段文本中同时查找多个模式串。

在 Trie 上添加 **fail 指针**（类似 KMP 的 next 数组），fail[u] 指向 u 节点表示的前缀的**最长后缀**（也是一个模式串的前缀）。

### 构建 fail 指针（BFS）

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 500005;
const int CHARSET = 26;

struct ACautomaton {
    int tr[MAXN][CHARSET];
    int fail[MAXN];
    int cnt[MAXN];           // 该节点结尾的模式串数量
    int tot;

    ACautomaton() : tot(0) {
        memset(tr, 0, sizeof(tr));
        memset(fail, 0, sizeof(fail));
        memset(cnt, 0, sizeof(cnt));
    }

    void insert(const string &s) {
        int u = 0;
        for (char ch : s) {
            int c = ch - 'a';
            if (!tr[u][c]) tr[u][c] = ++tot;
            u = tr[u][c];
        }
        cnt[u]++;
    }

    void build() {
        queue<int> q;
        // 根节点的直接子节点的 fail 指向根
        for (int c = 0; c < CHARSET; c++) {
            if (tr[0][c]) {
                fail[tr[0][c]] = 0;
                q.push(tr[0][c]);
            }
        }
        while (!q.empty()) {
            int u = q.front(); q.pop();
            for (int c = 0; c < CHARSET; c++) {
                int v = tr[u][c];
                if (v) {
                    // BFS 时 fail 已经算好了，直接跳
                    fail[v] = tr[fail[u]][c];
                    // 累计该节点的模式串数（优化：直接继承 fail 的 cnt）
                    cnt[v] += cnt[fail[v]];
                    q.push(v);
                } else {
                    // 关键优化：路径压缩！不存在的边直接指向 fail 对应边
                    tr[u][c] = tr[fail[u]][c];
                }
            }
        }
    }

    // 查询文本串 t 中出现的所有模式串（总数）
    int query(const string &t) {
        int u = 0, ans = 0;
        for (char ch : t) {
            int c = ch - 'a';
            u = tr[u][c];           // 路径压缩后无需 while 跳 fail
            ans += cnt[u];
            cnt[u] = 0;             // 避免重复计数（看题意）
        }
        return ans;
    }
};
```

### 匹配过程

- 每读入一个字符，沿着 Trie 边走
- fail 指针已经通过 **`cnt[v] += cnt[fail[v]]`** 累加，所以直接 `ans += cnt[u]` 即可
- 经典写法需要用 while 沿 fail 链跳，但路径压缩后只需一步

### 复杂度

| 步骤 | 复杂度 |
|------|--------|
| 插入所有模式串 | O(总字符数) |
| 构建 fail | O(节点数 × 字符集) |
| 匹配 | O(文本串长度) |

## CSP 真题场景

- **J2021 T3** 网络连接 — 可用 Trie 做前缀匹配
- **S2020 T2** 动物园 — 01-Trie 相关
- **各种字符串统计、自动补全** 题型

## ⚠️ 易错点

1. **Trie 节点数开够**：总字符数可能很大，MAXN 要留有裕量
2. **AC 自动机 `tr[u][c]` 路径压缩**：不存在的边指向 `tr[fail[u]][c]`，这是正确性和效率的关键
3. **fail 指针的 BFS 顺序**：必须 BFS（而非 DFS），保证 fail 祖先已经被计算
4. **重复模式串**：多模式串相同时，cnt 累加；query 完后清零避免重复计数
5. **01-Trie 位数**：int 范围用 30 位（0..30），long long 用 62 位

## 相关笔记

- [[KMP]] — AC 自动机就是多模式的 KMP
- [[字符串哈希]] — 多模式匹配也可以用哈希（逐个比）
- [[BFS与DFS]] — BFS 在 AC 自动机构建中的应用
