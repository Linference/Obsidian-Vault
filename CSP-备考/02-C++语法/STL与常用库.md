---
tags: [CSP, C++, STL, 标准库]
difficulty: J+S
created: 2026-06-06
---

# STL与常用库

## 一、STL 是什么

**STL（Standard Template Library）** 是 C++ 标准模板库，提供了一套高效的**容器、算法和迭代器**。CSP 中 STL 用得越熟练，编码速度越快——大多数数据结构和算法不用从零手写。

三大组件：
- **容器（Containers）**：vector, stack, queue, set, map ...
- **算法（Algorithms）**：sort, binary_search, next_permutation ...
- **迭代器（Iterators）**：begin(), end()，连接容器和算法

> CSP-J 至少掌握 vector/sort/stack/queue。CSP-S 要掌握 set/map/priority_queue/next_permutation。

## 二、vector —— 动态数组

```cpp
#include <vector>

vector<int> v;                     // 空 vector
vector<int> v(10);                 // 10 个元素，默认初始化为 0
vector<int> v(10, 5);              // 10 个元素，每个值为 5
vector<int> v = {1, 2, 3};        // 初始化列表

// 常用操作
v.push_back(4);                    // 末尾添加元素
v.pop_back();                      // 删除末尾元素
v.size();                          // 元素个数
v.empty();                         // 是否为空
v.clear();                         // 清空
v.resize(n);                       // 调整大小为 n（新元素默认 0）
v.reserve(n);                      // 预留空间（避免反复扩容）

// 访问
v[2] = 10;                         // 下标访问（不检查越界）
v.at(2) = 10;                      // 下标访问（检查越界，越界抛异常）
v.front();                         // 第一个元素
v.back();                          // 最后一个元素

// 遍历
for (int i = 0; i < v.size(); i++) cout << v[i] << " ";
for (int x : v) cout << x << " ";  // C++11 range-for（推荐）
for (auto it = v.begin(); it != v.end(); ++it) cout << *it << " ";

// 二维 vector
vector<vector<int>> mat(3, vector<int>(4, 0));  // 3 行 4 列，初始化为 0
mat[0].push_back(5);                            // 第 0 行添加元素
```

> **注意**：`v.size()` 返回 `size_t`（无符号类型）。`v.size() - 1` 当 v 为空时溢出为极大值，循环条件 `i < v.size() - 1` 可能出错。

## 三、stack —— 栈

```cpp
#include <stack>

stack<int> s;
s.push(1);          // 入栈
s.push(2);
s.pop();            // 出栈（无返回值！）
int top = s.top();  // 访问栈顶（不出栈）
s.empty();          // 是否为空
s.size();           // 元素个数
```

> **CSP 陷阱**：`pop()` 没有返回值！必须先 `top()` 取值，再 `pop()`。

## 四、queue —— 队列

```cpp
#include <queue>

queue<int> q;
q.push(1);          // 入队（到队尾）
q.push(2);
q.pop();            // 出队（从队首，无返回值）
int front = q.front(); // 队首元素
int back  = q.back();  // 队尾元素
q.empty();
q.size();
```

## 五、deque —— 双端队列

```cpp
#include <deque>

deque<int> dq;
dq.push_back(1);    // 队尾添加
dq.push_front(2);   // 队首添加
dq.pop_back();      // 队尾移除
dq.pop_front();     // 队首移除
dq.front();
dq.back();
// 支持随机访问：dq[0], dq[1] ...
```

## 六、priority_queue —— 优先队列（堆）

```cpp
#include <queue>

// 默认：大根堆（最大的在队首）
priority_queue<int> pq;
pq.push(3);
pq.push(1);
pq.push(5);
cout << pq.top();  // 5（最大元素）
pq.pop();          // 移除 5

// 小根堆（最小的在队首）
priority_queue<int, vector<int>, greater<int>> pq_min;
pq_min.push(3);
pq_min.push(1);
cout << pq_min.top();  // 1

// 自定义比较（结构体）
struct Node {
    int val, idx;
};
struct cmp {
    bool operator()(Node a, Node b) {
        return a.val > b.val;  // 小根堆：val 小的优先
    }
};
priority_queue<Node, vector<Node>, cmp> pq_custom;
```

> **记忆**：`greater<int>` = 小根堆，`less<int>`（默认）= 大根堆。和 `sort` 的方向相反！

## 七、set / multiset —— 有序集合

```cpp
#include <set>

set<int> s;              // 自动升序，元素不重复
s.insert(5);
s.insert(3);
s.insert(5);            // 重复插入无效！
s.erase(3);             // 删除值为 3 的元素
s.erase(s.begin());     // 删除迭代器指向的元素
if (s.find(5) != s.end()) { ... }  // 查找，找不到返回 s.end()
s.count(5);             // 是否存在（对 set，返回 0 或 1）
s.lower_bound(5);       // 第一个 >= 5 的迭代器
s.upper_bound(5);       // 第一个 > 5 的迭代器
s.size();
s.empty();
s.clear();

// multiset：允许重复元素
multiset<int> ms;
ms.insert(5);
ms.insert(5);           // ✅ 允许重复
ms.erase(5);            // ⚠️ 删除所有值为 5 的元素！
ms.erase(ms.find(5));   // ✅ 只删除一个值为 5 的元素
```

> **复杂度**：插入、删除、查找均为 **O(log n)**。底层是红黑树。

## 八、map —— 键值对

```cpp
#include <map>

map<string, int> mp;     // 按 key（string）自动升序
mp["apple"] = 5;         // 插入或修改
mp["banana"] = 3;
mp["apple"] = 10;        // 修改已有键

cout << mp["apple"];     // 10
cout << mp["orange"];    // ⚠️ 如果 key 不存在，会自动插入并初始化为 0！

if (mp.count("apple")) { ... }  // 判断 key 是否存在
mp.erase("apple");               // 删除键

// 遍历
for (auto &p : mp) {
    cout << p.first << " " << p.second << endl;  // p.first=key, p.second=value
}
```

> **关键陷阱**：`mp[key]` 如果 key 不存在，会自动插入！想只查询不插入，用 `mp.find(key)` 或 `mp.count(key)`。

## 九、unordered_map / unordered_set

哈希实现，**O(1) 平均复杂度**，但无序。

```cpp
#include <unordered_map>
#include <unordered_set>

unordered_map<string, int> ump;
unordered_set<int> us;

// 用法与 map/set 相同
// 区别：不保证顺序，不支持 lower_bound/upper_bound
// 自定义类型作为 key 需要提供 hash 函数
```

> **什么时候用 map 什么时候用 unordered_map**：需要有序遍历 → map；只需要快速查找 → unordered_map。CSP 中如果数据大且只查不遍历，unordered_map 能省一个 log。

## 十、algorithm 常用算法

```cpp
#include <algorithm>

int a = 10, b = 20;
max(a, b);            // 最大值
min(a, b);            // 最小值
swap(a, b);           // 交换

int arr[5] = {3, 1, 4, 1, 5};
sort(arr, arr + 5);   // 升序排序 [1, 1, 3, 4, 5]

vector<int> v = {3, 1, 4, 1, 5};
sort(v.begin(), v.end());  // 升序
sort(v.begin(), v.end(), greater<int>());  // 降序

// 自定义排序
struct cmp {
    bool operator()(int a, int b) {
        return a > b;  // 降序
    }
};
sort(v.begin(), v.end(), cmp());

// 用 lambda（C++11）
sort(v.begin(), v.end(), [](int a, int b) { return a > b; });
```

### 其他常用算法

```cpp
// 反转
reverse(v.begin(), v.end());

// 去重（需要先排序！）
sort(v.begin(), v.end());
auto it = unique(v.begin(), v.end());
v.erase(it, v.end());  // 真正删除重复元素

// 查找
auto pos = find(v.begin(), v.end(), 5);  // 线性查找，返回迭代器
bool found = binary_search(v.begin(), v.end(), 5);  // 二分查找，O(log n)
auto lb = lower_bound(v.begin(), v.end(), 5);   // 第一个 >= 5 的位置
auto ub = upper_bound(v.begin(), v.end(), 5);   // 第一个 > 5 的位置

// 计数
int cnt = count(v.begin(), v.end(), 5);

// 最大值/最小值元素
auto max_it = max_element(v.begin(), v.end());
auto min_it = min_element(v.begin(), v.end());
```

### next_permutation —— 全排列

```cpp
int arr[] = {1, 2, 3};
sort(arr, arr + 3);  // ⚠️ 必须先排序才能生成所有排列！
do {
    cout << arr[0] << arr[1] << arr[2] << endl;
} while (next_permutation(arr, arr + 3));
// 输出：
// 1 2 3
// 1 3 2
// 2 1 3
// 2 3 1
// 3 1 2
// 3 2 1
```

> 返回值：生成下一个更大的排列则返回 true，已是最大排列则返回 false。

## 十一、pair

```cpp
#include <utility>

pair<int, string> p = {1, "hello"};
pair<int, string> p2 = make_pair(2, "world");
cout << p.first << " " << p.second;  // 1 hello

// pair 可以比较（先比较 first，再比较 second）
// 所以可以直接放进 vector 然后 sort
```

## 十二、C++11/14/17 新特性

### auto —— 自动类型推导

```cpp
auto x = 42;                          // int
auto y = 3.14;                        // double
auto it = v.begin();                  // vector<int>::iterator
map<string, int> mp;
for (auto &p : mp) { ... }           // 遍历 map 超方便
```

### range-for（基于范围的 for 循环）

```cpp
vector<int> v = {1, 2, 3};
for (int x : v) cout << x;           // 拷贝
for (int &x : v) x *= 2;             // 引用，可修改
for (const int &x : v) cout << x;    // const 引用，避免拷贝且不修改
```

### lambda 表达式

```cpp
// 基本语法：[捕获](参数) -> 返回类型 { 函数体 }
auto add = [](int a, int b) -> int { return a + b; };
cout << add(3, 5);  // 8

// 捕获外部变量
int base = 10;
auto mul = [base](int x) { return base * x; };  // 值捕获 base

// 最常用：sort 的自定义比较
sort(v.begin(), v.end(), [](int a, int b) { return a > b; });
```

## ⚠️ 易错点

1. **stack/pop 无返回值**——先 `top()`/`front()` 取值再 `pop()`。
2. **priority_queue 的比较方向与 sort 相反**——`greater<int>` 在小根堆中是升序 top 最小，在 sort 中却是降序。
3. **map 的 `[]` 运算符**——访问不存在的 key 会自动插入。用 `find()` 或 `count()` 进行只读查询。
4. **multiset 的 `erase(x)` 删除所有值为 x 的元素**——只删一个用 `erase(find(x))`。
5. **`v.size() - 1` 为无符号数**——当 v 为空时变成极大值，循环条件异常。
6. **`lower_bound` / `upper_bound` 需要有序**——无序序列结果未定义。
7. **`unique` 只是把重复元素移到末尾**——实际删除需要配合 `erase`。
8. **set / map 的迭代器不能做 `it + k` 操作**——只能用 `++it`（双向迭代器）。但 vector 和 deque 的迭代器可以（随机访问迭代器）。
9. **for-each 循环中修改容器结构**——在 range-for 中插入/删除元素是未定义行为。
10. **lambda 中值捕获 vs 引用捕获**——`[=]` 值捕获所有，`[&]` 引用捕获所有，`[this]` 捕获 this 指针。

## 相关笔记

- [[数组与字符串]] — vector 替代 C 数组，string 替代 char[]
- [[控制结构]] — range-for 是更安全的遍历方式
- [[函数与递归]] — lambda 作为匿名函数参数
- [[数据结构-栈]]、[[数据结构-队列]]、[[数据结构-堆]] — 对应 STL stack/queue/priority_queue
