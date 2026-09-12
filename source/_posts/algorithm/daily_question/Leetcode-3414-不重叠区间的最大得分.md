---
title: Leetcode-3414-不重叠区间的最大得分
date: 2026-09-12 10:07:59
tags: [LeetCode, 动态规划, 数组, 二分查找, 排序, 每日一题]
categories: [算法题解]
permalink: maximum-score-of-non-overlapping-intervals/
---

## 题目

[3414. 不重叠区间的最大得分](https://leetcode.cn/problems/maximum-score-of-non-overlapping-intervals/description/)

给你一个二维整数数组 `intervals`，其中 `intervals[i] = [l_i, r_i, weight_i]`。区间 `i` 的起点为 `l_i`，终点为 `r_i`，权重为 `weight_i`。你**最多可以选择 4 个互不重叠**的区间。所选择区间的**得分**定义为这些区间权重的总和。

返回一个数组，包含从 `intervals` 中选出的至多 4 个区间的**下标**（原始下标），使得得分最大。如果有多个方案得分相同，返回**字典序最小**的那个。

注意：区间 `[a, b]` 包含端点，即两个区间共享端点也算重叠。

**示例 1：**

```text
输入：intervals = [[1,3,2],[4,5,2],[1,5,5],[6,9,3],[6,7,1],[8,9,1]]
输出：[2,3]
解释：可以选择下标为 2 和 3 的区间，其权重分别为 5 和 3。
```

**示例 2：**

```text
输入：intervals = [[5,8,1],[6,7,7],[4,7,3],[9,10,6],[7,8,2],[11,14,3],[3,5,5]]
输出：[1,3,5,6]
解释：可以选择下标为 1、3、5 和 6 的区间，其权重分别为 7、6、3 和 5。
```

**提示：**

- `1 <= intervals.length <= 5 * 10^4`
- `intervals[i].length == 3`
- `1 <= l_i <= r_i <= 10^9`
- `1 <= weight_i <= 10^9`

## 思路

这道题是经典的**区间调度问题（Interval Scheduling）**的变种，核心矛盾在于：**选互不重叠的区间让总权重最大，但最多只能选 4 个**，且还要返回**字典序最小**的下标集合。

### 四个标签如何协同解题

```
┌──────────────────────────────────────────────────────┐
│                   完整解题流程                         │
│                                                      │
│  ① 数组 ──→ 存储原始区间数据 [l, r, weight, origIdx] │
│                       │                              │
│                       ▼                              │
│  ② 排序 ──→ 按右端点升序排列，方便冲突判断和二分       │
│                       │                              │
│                       ▼                              │
│  ③ 动态规划 ──→ 状态转移中需要查找前驱不重叠区间       │
│                       │                              │
│                       ▼                              │
│  ④ 二分查找 ──→ 在有序的右端点数组中 O(log n) 定位    │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## 标签详解

### 一、排序：为什么必须按右端点排？

区间问题的经典套路——**按右端点排序**，原因有三：

1. **简化冲突判断**：排序后，对于区间 `i`，所有 `j < i` 的区间右端点都 ≤ `arr[i].r`。因此区间 `j` 和 `i` **不重叠** 等价于 `arr[j].r < arr[i].l`（因为 `j` 在 `i` 左边）。
2. **右端点单调 → 可二分**：排序后右端点数组递增，才能用二分快速定位前驱。
3. **贪心直觉**：右端点越小越靠前，越早结束，给后面留更多空间。

#### 关键陷阱：原始下标绑定

排序会打乱原始下标，但题目要求返回**原始位置**的字典序最小集合。所以排序时必须把原始下标带着一起排：

```java
int n = intervals.size();
int[][] arr = new int[n][4];
for (int i = 0; i < n; i++) {
    arr[i][0] = intervals.get(i).get(0); // l
    arr[i][1] = intervals.get(i).get(1); // r
    arr[i][2] = intervals.get(i).get(2); // weight
    arr[i][3] = i;                        // originalIdx
}
// 按右端点升序排序
Arrays.sort(arr, (a, b) -> Integer.compare(a[1], b[1]));
```

以示例 1 为例，排序前后对照：

```
原始:  [[1,3,2], [4,5,2], [1,5,5], [6,9,3], [6,7,1], [8,9,1]]
       idx=0     idx=1     idx=2     idx=3     idx=4     idx=5

排序后 (按 r):
       [1,3,2,0] [4,5,2,1] [1,5,5,2] [6,7,1,4] [6,9,3,3] [8,9,1,5]
       r=3       r=5       r=5       r=7       r=9       r=9
```

排序后右端点单调递增 `[3, 5, 5, 7, 9, 9]`，这为后续二分查找打下了基础。

---

### 二、数组：数据的载体

数组在这道题中扮演多重角色：

| 数组 | 作用 |
|------|------|
| `intervals` | 输入数据，存储原始区间 `[l, r, weight]` |
| `arr` | 绑定了原始下标 `[l, r, weight, originalIdx]` 的排序后数组 |
| `rightEnds` | 从 `arr` 提取的纯右端点数组，用于二分查找 |
| `State[][] dp` | 动态规划状态表，存储每个状态的权重和索引集合 |

#### 二维数组 vs 滚动数组

观察状态转移：`dp[i][k]` 依赖 `dp[i-1][k]`（不选）和 `dp[j+1][k-1]`（选，其中 `j+1` 可能远小于 `i`）。因为 `j+1` 可以是 0，**不能直接用滚动数组覆盖前面的状态**，需要完整的二维数组。但 `k` 维度只有 5（0~4），实际空间 `O(n × 5) = O(n)`。

---

### 三、动态规划：核心状态与转移

#### 为什么是动态规划？

这是经典的**区间调度问题（Interval Scheduling）**的变种：选互不重叠的区间让总权重最大。与经典问题的区别在于：
- **最多选 4 个**（k ≤ 4）
- 返回**字典序最小**的方案

#### 状态定义

```
dp[i][k] = 考虑排序后的前 i 个区间（arr[0] ~ arr[i-1]），
           从中恰好选 k 个互不重叠的区间时，
           能获得的最大权重值 + 对应的原始下标集合
```

其中：
- `i ∈ [0, n]`：处理到第 i 个区间
- `k ∈ [0, 4]`：选了多少个区间（最多 4 个）

#### 状态转移

对于排序后的第 `i` 个区间（即 `arr[i-1]`），有两种选择：

**方案 1：不选第 i 个区间**
```
dp[i][k] = dp[i-1][k]
```

**方案 2：选第 i 个区间（记为 A = arr[i-1]）**

先在 A 左边找到**最后一个与 A 不重叠**的区间位置 `j`（用二分查找），然后从 `dp[j+1][k-1]` 转移过来：

```
dp[i][k] = dp[j+1][k-1].weight + A.weight     （前提是 dp[j+1][k-1] 有效）
dp[i][k].indices = dp[j+1][k-1].indices + [A.originalIdx]
```

#### 转移决策图

```
考虑第 i 个区间 A = arr[i-1]:
┌──────────────────────────────────────────────────┐
│                                                  │
│  方案1: 不选 A                                    │
│    dp[i][k] = dp[i-1][k]                         │
│                                                  │
│  方案2: 选 A (需要 k >= 1 且存在有效前驱)          │
│    pos = upperBound(rightEnds, A.left)            │
│    j = pos - 1  ← 最后一个右端点 < A.left 的位置  │
│    dp[i][k] = dp[j+1][k-1] + {A.weight, A.idx}   │
│                                                  │
│  取 maxWeight 更大的方案                           │
│  若 maxWeight 相同 → 选 indices 字典序更小的方案    │
│                                                  │
└──────────────────────────────────────────────────┘
```

#### 初始化

```
dp[0][0] = {weight=0, indices=[]}     ← 选 0 个区间，有效
dp[0][k] = {weight=-1, indices=[]}    ← k > 0 时前 0 个区间选不出来，无效

dp[i][0] = {weight=0, indices=[]}     ← 对所有 i，选 0 个都是有效的
```

---

### 四、二分查找：快速定位前驱

#### 解决什么问题？

在状态转移的「选 A」分支中，需要找到：

> 所有在 A 左边、且与 A 不重叠的区间中，**最靠右**的那个的位置

如果用线性查找，每次 O(n)，总复杂度 O(n²)。但因为 `rightEnds[]` 已经排序，可以用二分。

#### 核心逻辑

```java
int[] rightEnds = {3, 5, 5, 7, 9, 9};  // 排序后的右端点数组
int L = 6;                              // 当前区间的左端点

// upperBound: 在 rightEnds 中找第一个 >= L 的位置
// rightEnds 中值为 7 的那个位置（下标 3）是第一个 >= 6 的
int pos = upperBound(rightEnds, L);  // 返回 3

// pos - 1 就是最后一个 < L 的位置
// rightEnds[2] = 5 < 6 ✓，rightEnds[3] = 7 >= 6
int j = pos - 1;  // j = 2
```

#### 完整流程

```java
// upperBound: 在有序数组中找第一个 >= target 的下标
private int upperBound(int[] sorted, int target) {
    int left = 0, right = sorted.length;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (sorted[mid] >= target) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }
    return left;
}

// 对于当前区间 A = arr[i-1]，左端点为 l = A[0]
int pos = upperBound(rightEnds, l);
int prevIdx = pos - 1;  // 最后一个右端点 < l 的区间位置

// prevIdx + 1 表示前 prevIdx+1 个区间（arr[0] ~ arr[prevIdx]）
// 从 dp[prevIdx + 1][k - 1] 转移过来
```

#### 为什么用 `< l` 而不是 `<= l`？

题目明确定义：**区间 `[a, b]` 包含端点，两个区间共享端点也算重叠**。所以：

```
[l1, r1] 和 [l2, r2] 不重叠 ⟺ r1 < l2（严格小于）
```

不能写成 `r1 <= l2`，否则共享端点的两个区间（如 `[1, 3]` 和 `[3, 5]`）会被误判为不重叠。

#### 复杂度影响

| 方式 | 每次查询 | 总复杂度 |
|------|---------|---------|
| 线性扫描 | O(n) | O(n²) |
| 二分查找 | O(log n) | O(n log n) |

---

### 五、字典序维护：最容易被忽略的难点

题目不仅要求**权重最大**，还要求**原始位置字典序最小**。这是这道题区别于经典区间调度 DP 的关键。

#### 为什么必须贯穿 DP 过程？

如果只在最后比较，中间状态可能已经丢失了字典序更优但权重暂时相同的方案。必须在**每次状态转移时**就比较字典序。

#### State 类设计

把权重和索引集合打包在一起：

```java
class State {
    long weight;           // 最大权重（用 long 防溢出）
    List<Integer> indices; // 原始下标语

    static State better(State a, State b) {
        // 先比权重
        if (a.weight != b.weight) {
            return a.weight > b.weight ? a : b;
        }
        // 权重相同，比原始下标字典序
        return isLexSmaller(a.indices, b.indices) ? a : b;
    }
}
```

#### 字典序比较规则

和字符串类似：逐位比较，第一个不同的位置决定大小；短的数组字典序更小。

```java
private boolean isLexSmaller(List<Integer> a, List<Integer> b) {
    int minLen = Math.min(a.size(), b.size());
    for (int i = 0; i < minLen; i++) {
        if (!a.get(i).equals(b.get(i))) {
            return a.get(i) < b.get(i);
        }
    }
    return a.size() < b.size();
}
```

举例：`[1, 5]` 比 `[2, 3]` 字典序小（第一位 1 < 2）。

#### 为什么权重用 long？

`weight_i` 最大 10^9，最多选 4 个，总和可达 4 × 10^9 ≈ 40 亿，超过 `int` 范围（约 ±21 亿），必须用 `long`。

---

## 代码

```java
import java.util.*;

class Solution {
    public int[] maximumWeight(List<List<Integer>> intervals) {
        int n = intervals.size();

        // ① 数组：绑定原始下标
        int[][] arr = new int[n][4];
        for (int i = 0; i < n; i++) {
            arr[i][0] = intervals.get(i).get(0); // l
            arr[i][1] = intervals.get(i).get(1); // r
            arr[i][2] = intervals.get(i).get(2); // weight
            arr[i][3] = i;                        // originalIdx
        }

        // ② 排序：按右端点升序
        Arrays.sort(arr, (a, b) -> Integer.compare(a[1], b[1]));

        // 提取右端点数组，用于二分查找
        int[] rightEnds = new int[n];
        for (int i = 0; i < n; i++) {
            rightEnds[i] = arr[i][1];
        }

        // ③ 动态规划
        // dp[i][k] = 前 i 个区间中选 k 个的最优状态
        State[][] dp = new State[n + 1][5];

        // 初始化：选 0 个区间，权重 0，索引空
        for (int i = 0; i <= n; i++) {
            dp[i][0] = new State(0, new ArrayList<>());
        }
        // k > 0 时，初始为无效状态（权重 -1，因为 weight >= 1）
        for (int k = 1; k <= 4; k++) {
            dp[0][k] = new State(-1, new ArrayList<>());
        }

        for (int i = 1; i <= n; i++) {
            for (int k = 1; k <= 4; k++) {
                // 方案1：不选第 i 个区间
                State skip = dp[i - 1][k];

                // 方案2：选第 i 个区间（arr[i-1]）
                State take = new State(-1, new ArrayList<>());
                int l = arr[i - 1][0];
                int weight = arr[i - 1][2];
                int origIdx = arr[i - 1][3];

                // ④ 二分查找：找第一个右端点 >= l 的位置
                int pos = upperBound(rightEnds, l);
                // 最后一个右端点 < l 的位置
                int prevIdx = pos - 1;
                // 前 prevIdx+1 个区间，选 k-1 个
                State prevState = dp[prevIdx + 1][k - 1];
                if (prevState.weight >= 0) {
                    take.weight = prevState.weight + weight;
                    take.indices = new ArrayList<>(prevState.indices);
                    take.indices.add(origIdx);
                }

                dp[i][k] = State.better(skip, take);
            }
        }

        // 遍历 dp[n][1] ~ dp[n][4]，找权重最大、字典序最小的
        State ans = dp[n][1];
        for (int k = 2; k <= 4; k++) {
            ans = State.better(ans, dp[n][k]);
        }

        // 按原始位置升序排序后返回
        Collections.sort(ans.indices);
        int[] result = new int[ans.indices.size()];
        for (int i = 0; i < result.length; i++) {
            result[i] = ans.indices.get(i);
        }
        return result;
    }

    private int upperBound(int[] sorted, int target) {
        int left = 0, right = sorted.length;
        while (left < right) {
            int mid = left + (right - left) / 2;
            if (sorted[mid] >= target) {
                right = mid;
            } else {
                left = mid + 1;
            }
        }
        return left;
    }

    static class State {
        long weight;
        List<Integer> indices;

        State(long weight, List<Integer> indices) {
            this.weight = weight;
            this.indices = indices;
        }

        static State better(State a, State b) {
            if (a.weight != b.weight) {
                return a.weight > b.weight ? a : b;
            }
            return isLexSmaller(a.indices, b.indices) ? a : b;
        }

        static boolean isLexSmaller(List<Integer> a, List<Integer> b) {
            int minLen = Math.min(a.size(), b.size());
            for (int i = 0; i < minLen; i++) {
                if (!a.get(i).equals(b.get(i))) {
                    return a.get(i) < b.get(i);
                }
            }
            return a.size() < b.size();
        }
    }
}
```

## 复杂度分析

- **时间复杂度**：`O(n log n)` — 排序 O(n log n)，动态规划 O(n × 5) 次状态转移，每次二分 O(log n)。
- **空间复杂度**：`O(n)` — dp 数组 O(n × 5)，右端点数组 O(n)。

## 关键点

1. **按右端点排序**：区间问题的经典套路，简化冲突判断并为二分铺路。
2. **原始下标绑定**：排序时必须保留原始位置，因为最终答案需要返回原始下标，且字典序比较基于原始下标。
3. **k ≤ 4 的常数优化**：因为最多选 4 个，dp 第二维只有 5 个状态（0~4），空间复杂度 O(n)。
4. **共享端点算重叠**：判断不重叠用 `r1 < l2`（严格小于），不能用 `r1 <= l2`。
5. **字典序维护要贯穿始终**：不仅最后比，每次状态转移时就比较，否则中间过程会丢失更优方案。
6. **权重用 long**：`weight_i` 最大 10^9，4 个总和达 40 亿，超过 int 范围（约 21 亿）。
7. **prevIdx = -1 边界处理**：当所有区间右端点都 >= 当前区间左端点时，`pos = 0`，`prevIdx = -1`，`prevIdx + 1 = 0`，`dp[0][0]` 的 weight=0 可以直接加上当前 weight，选 1 个区间自动成立。

## 相关题目

- **区间调度类**：
  - [435. 无重叠区间](https://leetcode.cn/problems/non-overlapping-intervals/) — 经典贪心，按右端点排序后选最多不重叠区间。
  - [452. 用最少数量的箭引爆气球](https://leetcode.cn/problems/minimum-number-of-arrows-to-burst-balloons/) — 区间覆盖，等价于最多选多少不重叠区间。
  - [646. 最长数对链](https://leetcode.cn/problems/maximum-length-of-pair-chain/) — 按右端点排序 + DP。
  - [1235. 规划兼职工作](https://leetcode.cn/problems/maximum-profit-in-job-scheduling/) — **和本题核心算法完全相同**（排序 + DP + 二分），但没有最多选 4 个的限制，也不需要字典序。