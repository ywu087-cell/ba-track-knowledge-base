# 第 04 课 - 哈希表、集合与 K-Sum 问题

日期：2026-07-12
来源：BA track 课程 PDF，主题是 Hash Table、Set、2 Sum、3 Sum 和 4 Sum
整理方式：中文解释为主，保留关键英文术语；不是逐页照搬 PDF

## 一句话总结

哈希表 (Hash Table) 把“按值查找”从逐个扫描的 O(n) 问题，转为通常可在 O(1) 完成的定位问题；2 Sum 到 4 Sum 的关键，是根据数组是否有序、是否允许排序和输出要求，选择双指针 (Two Pointers)、集合 (Set) 或字典 (Dictionary) 来降低复杂度。

## 快速目录

1. 为什么需要哈希表
2. 哈希函数、桶和冲突
3. Hash Table、Dictionary 与 Set 的关系
4. 2 Sum：有序数组与无序数组
5. 重复值与索引输出
6. 3 Sum：固定一个数再做 2 Sum
7. 4 Sum：降维与 Pair Sum
8. 复杂度选择框架
9. BA 视角总结
10. 复习问题与术语表

## 核心概念索引

- 哈希表 (Hash Table)：通过哈希函数把 key 映射到存储位置，以便快速插入和查询的数据结构。
- 哈希函数 (Hash Function)：把 key 转成整数哈希值 (hash number) 的函数。
- 桶 (Bucket)：哈希表中一个索引位置对应的存储单元。
- 索引 (Index)：通常由 `hash(key) % table_size` 得到的位置编号。
- 冲突 (Collision)：两个不同的 key 被映射到同一个 index。
- 链地址法 (Separate Chaining)：每个 bucket 存一个列表，冲突元素放在同一个 bucket 的列表中。
- 开放寻址法 (Open Addressing)：冲突后继续寻找其他空位置存放元素。
- 字典 (Dictionary / `dict`)：Python 中常见的 key-value 哈希表。
- 集合 (Set / `set`)：只存 key、不关心 value 的哈希结构，常用于去重和成员判断。
- 补数 (Complement)：为达到 target 所缺少的值，例如 target=11、当前值=10，则 complement=1。
- 双指针 (Two Pointers)：用左右两个索引向中间移动，常用于有序数组。
- K-Sum：找出 k 个数之和等于 target 的一类问题。
- 时间复杂度 (Time Complexity)：输入规模变大时运行时间的增长趋势。
- 空间复杂度 (Space Complexity)：算法额外占用内存的增长趋势。

## 1. 为什么需要哈希表

假设要保存很多 `key -> value` 对，例如：

```text
video_id -> title
customer_id -> customer profile
campaign_id -> campaign metrics
```

如果数据只放在普通列表中，为了找某个 key，通常要从头扫到尾，最坏需要 O(n)。哈希表的目标是：先根据 key 直接算出大致位置，再只检查该位置，因此平均查询、插入和删除都可以接近 O(1)。

核心流程：

```text
key -> hash(key) -> hash number -> index -> bucket -> value
```

课程示例中，若哈希值为 121、表长为 10：

```python
index = 121 % 10   # 1
```

所以数据会被放到第 1 个 bucket。取模 (modulo) 的目的，是把可能很大的 hash number 压缩到合法的数组下标 `0` 到 `N - 1`。

## 2. 哈希函数、桶和冲突

### 2.1 哈希函数不是“保证唯一”

不同 key 可能算出不同的 hash number，但经过 `% N` 后仍会落到同一个 index：

```text
121 % 10 = 1
131 % 10 = 1
```

这就是冲突 (Collision)。冲突是正常现象，不表示哈希表失效；真正的问题是如何保存和查找这些落在同一 bucket 的元素。

### 2.2 链地址法 (Separate Chaining)

课程重点介绍的是链地址法：每个 bucket 存一个小列表。

```text
bucket[1] -> [(121, value_a), (131, value_b), (141, value_c)]
```

插入时：

1. 计算 `hash(key)`。
2. 计算 `index = hash(key) % N`。
3. 到 `bucket[index]` 检查是否已有相同 key。
4. 没有则加入；有则更新其 value。

查询时：

1. 用同一 key 计算 index。
2. 只检查对应 bucket 内的少量元素。
3. 找到相同 key 后返回 value。

理想情况下，key 会比较均匀地分散在各 bucket，单个 bucket 很短，因此平均查找为 O(1)。极端情况下所有 key 都撞在一起，会退化为 O(n)。

### 2.3 开放寻址法 (Open Addressing)

开放寻址法不把多个元素挂在同一个列表里，而是在冲突后按照规则继续探测其他位置，例如线性探测 (Linear Probing)。

本课的重点不是手写完整哈希表，而是理解：**哈希表的高效来自“直接定位”，但前提是要有合理的哈希和冲突处理策略。** 实务中直接使用 Python 的 `dict` 和 `set` 即可。

### 2.4 字符串哈希：理解即可

课程用 Rabin-Karp 的思路说明：字符串可以被看成某个进制的数，例如将 `a` 到 `z` 映射为 0 到 25，再计算多项式哈希。

```text
hash("abc") = 0 * 26^2 + 1 * 26^1 + 2 * 26^0
```

这部分用于理解“字符串为何能被映射为数字”。实际 Python 项目中，通常使用内置的 `hash()`、`dict` 或 `set`，不需要自己实现这种哈希函数。

## 3. Hash Table、Dictionary 与 Set 的关系

| 结构 | 存什么 | 最常见用途 | Python |
| --- | --- | --- | --- |
| Hash Table | `key -> value` | 按 key 快速找 value | `dict` |
| Dictionary | `key -> value` | 映射、频次统计、索引保存 | `dict` |
| Set | 只存唯一的 key | 去重、判断某值是否出现过 | `set` |

可以把 Set 理解为简化版 Hash Table：只关心 key 是否存在，value 不重要。

```python
seen_customer_ids = {101, 102, 103}
101 in seen_customer_ids  # True，平均 O(1)
```

## 4. 2 Sum：有序数组与无序数组

2 Sum 的基本问题：给定整数数组和 `target`，判断是否存在两个不同位置的数相加等于 target。

### 4.1 有序数组：双指针 (Two Pointers)

如果数组已排序，例如 `[1, 2, 3, 4]`，target=6，可以从最小和最大值开始：

```text
left=1, right=4 -> 1+4=5，太小，left 右移
left=2, right=4 -> 2+4=6，找到
```

为什么可以这样移动？因为数组有序：

- 当前和太小，右指针左移只会让和更小，所以应让左指针右移。
- 当前和太大，左指针右移只会让和更大，所以应让右指针左移。

```python
def two_sum_sorted(nums, target):
    left, right = 0, len(nums) - 1

    while left < right:
        current_sum = nums[left] + nums[right]

        if current_sum == target:
            return True
        if current_sum < target:
            left += 1
        else:
            right -= 1

    return False
```

复杂度：时间 O(n)，额外空间 O(1)。

### 4.2 无序数组：Set 保存已见过的数

无序数组不能安全地移动双指针。此时遍历每个数 `x`，并检查补数 `target - x` 是否已经出现过。

```python
def two_sum_unsorted(nums, target):
    seen = set()

    for value in nums:
        complement = target - value
        if complement in seen:
            return True
        seen.add(value)

    return False
```

例如 `[10, 13, 1]`、target=11：

```text
读到 10：需要 1，尚未出现 -> 记录 10
读到 13：需要 -2，尚未出现 -> 记录 13
读到 1：需要 10，已经出现 -> True
```

复杂度：时间 O(n)，空间 O(n)。这是一种典型的 **用空间换时间 (space-time trade-off)**。

## 5. 重复值与索引输出

### 5.1 如果只需要“有无”：Set 足够

`two_sum_unsorted` 只回答 True/False。若数组为 `[2, 2]`、target=4，第二个 `2` 检查时能看到第一个 `2`，因此不会误把同一个元素使用两次。

### 5.2 如果要统计有多少对：用频次字典

对于 `[10, 2, 2, 13, 2]`、target=4，三个 `2` 在不同 index 上可组成 `C(3, 2)=3` 对。

```python
def count_two_sum_pairs(nums, target):
    frequency = {}
    count = 0

    for value in nums:
        complement = target - value
        count += frequency.get(complement, 0)
        frequency[value] = frequency.get(value, 0) + 1

    return count
```

这里先加 `frequency[complement]`，再记录当前 value，确保只将“之前出现过的元素”与当前元素配对，避免重复计数。

### 5.3 如果要输出具体 index：字典保存 index

```python
def two_sum_indices(nums, target):
    index_by_value = {}

    for index, value in enumerate(nums):
        complement = target - value
        if complement in index_by_value:
            return [index_by_value[complement], index]
        index_by_value[value] = index

    return None
```

注意：如果题目要求返回所有重复组合，单个 `value -> index` 不够，需要保存 `value -> [index1, index2, ...]`。

## 6. 3 Sum：固定一个数再做 2 Sum

3 Sum 要判断是否存在三个数之和等于 target。

### 6.1 思路：降成 2 Sum

固定第一个数 `nums[i]` 后，剩余问题变成：在后面的数里找两个数，和等于 `target - nums[i]`。

```text
3 Sum = 固定一个数 + 2 Sum
```

如果每次使用无序 2 Sum，整体为 O(n^2) 时间、O(n) 额外空间。

### 6.2 排序后用双指针

课程的主解法是先排序，再对每个 `i` 使用两个指针：

```python
def three_sum_exists(nums, target):
    nums = sorted(nums)

    for i in range(len(nums) - 2):
        left, right = i + 1, len(nums) - 1

        while left < right:
            current_sum = nums[i] + nums[left] + nums[right]

            if current_sum == target:
                return True
            if current_sum < target:
                left += 1
            else:
                right -= 1

    return False
```

复杂度：排序 O(n log n)，外层循环 O(n)，每轮双指针最多 O(n)，合计 O(n^2)。指针本身只占 O(1) 空间；实际排序函数可能额外使用内部内存。

若题目要求返回所有不重复三元组，还必须跳过重复值；否则相同的值组合会被返回多次。

## 7. 4 Sum：降维与 Pair Sum

4 Sum 需要找四个数之和等于 target。最朴素的四层循环是 O(n^4)，通常太慢。

### 7.1 方案一：固定一个数，转成 3 Sum

```text
4 Sum = 固定一个数 + 3 Sum
```

在排序数组中，这通常可做到 O(n^3) 时间。它延续了 K-Sum 的基本递归思路：每增加一个数，就多固定一层。

### 7.2 方案二：Pair Sum Hash Table

把所有两数之和预先保存：

```text
a + b + c + d = target
(a + b) + (c + d) = target
```

遍历时寻找 `target - pair_sum` 是否已出现过；必须检查四个 index 不重复。

```python
def four_sum_exists(nums, target):
    pairs_by_sum = {}

    for right in range(len(nums)):
        for left in range(right):
            pair_sum = nums[left] + nums[right]
            complement = target - pair_sum

            for i, j in pairs_by_sum.get(complement, []):
                if len({i, j, left, right}) == 4:
                    return True

        for left in range(right):
            pair_sum = nums[left] + nums[right]
            pairs_by_sum.setdefault(pair_sum, []).append((left, right))

    return False
```

这类 Pair Sum 方法的典型复杂度是 O(n^2) 时间和 O(n^2) 空间，但实现时必须清楚输出要求、重复值规则和 index 是否可以复用。对于很大的数据，O(n^2) 内存本身可能无法接受。

## 8. 复杂度选择框架

遇到“找若干数的和”问题时，先问这五件事：

| 问题 | 对算法选择的影响 |
| --- | --- |
| 数组是否已排序？ | 已排序优先考虑双指针。 |
| 能否修改原数组？ | 可以排序时，K-Sum 通常更容易处理。 |
| 只要 True/False、计数，还是具体 index/组合？ | 决定用 Set、频次 `dict`，还是索引列表。 |
| 是否有重复值？ | 需要定义“不同值”还是“不同 index”，并防止重复计数。 |
| 数据规模多大？ | O(n^2) 或 O(n^3) 在大数据上可能不可用。 |

简化决策：

```text
2 Sum + sorted     -> Two Pointers
2 Sum + unsorted   -> Set / dict + complement
需要频次或 index    -> dict 保存频次或 index
3 Sum              -> sort + fix one + Two Pointers
4 Sum              -> fix one + 3 Sum，或 Pair Sum（用更多内存换时间）
```

## 9. BA 视角总结

这节课不是直接讲营销指标，但它训练的结构选择会频繁出现在 BA 数据处理中：

| 业务任务 | 可借鉴的结构/思路 |
| --- | --- |
| 按 `customer_id` 合并订单与客户表 | `dict` / hash join 的快速 key 查找 |
| 检查重复 user_id、coupon code 或 transaction_id | `set` 成员判断与去重 |
| 统计某个值或某种组合出现次数 | `dict` 频次统计 |
| 检查两笔金额能否匹配某个差额或预算 | complement 思路 |
| 大数据分析 | 先估算 O(n)、O(n^2) 的成本，避免把可线性完成的问题写成嵌套循环 |

最重要的 BA 习惯是：不要因为代码能运行，就忽略算法规模。对 4,000 行数据，O(n^2) 可能仍可接受；对 4,000,000 行数据，同一做法可能完全不可用。

## 10. 复习问题

1. `hash(key) % N` 的目的是什么？为什么会产生 collision？
2. Separate Chaining 如何解决 collision？它在什么情况下会退化为 O(n)？
3. 为什么已排序 2 Sum 可以使用双指针，而无序数组不可以直接这样做？
4. 无序 2 Sum 中，为什么要先检查 complement、再把当前值加入 `seen`？
5. 如果要统计所有 pair 数量，为什么 Set 不够，而需要 frequency dictionary？
6. 3 Sum 为什么通常先排序？它如何转化为多个 2 Sum？
7. Pair Sum 能把 4 Sum 的时间复杂度降到什么量级？代价是什么？
8. 在真实业务数据中，什么情况下你会优先用 `set`，什么情况下用 `dict`？

## 11. 术语表

| 中文 | English | 说明 |
| --- | --- | --- |
| 哈希表 | Hash Table | 按 key 快速定位 value 的数据结构。 |
| 哈希函数 | Hash Function | 把 key 映射成整数的函数。 |
| 哈希值 | Hash Value / Hash Number | 哈希函数的输出。 |
| 桶 | Bucket | 哈希表的一个存储位置。 |
| 冲突 | Collision | 多个 key 映射到同一 index。 |
| 链地址法 | Separate Chaining | 将冲突元素放进同一个 bucket 的列表。 |
| 开放寻址法 | Open Addressing | 冲突后继续寻找其他位置。 |
| 集合 | Set | 只记录元素是否存在的唯一值集合。 |
| 字典 | Dictionary | 保存 key-value 对的 Python `dict`。 |
| 补数 | Complement | 与当前值相加能得到 target 的值。 |
| 双指针 | Two Pointers | 在有序数组中用两个方向相反的指针搜索。 |
| K-Sum | K-Sum | 查找 k 个数之和等于 target 的问题族。 |
| 时间复杂度 | Time Complexity | 运行时间随输入规模增长的趋势。 |
| 空间复杂度 | Space Complexity | 额外内存随输入规模增长的趋势。 |
