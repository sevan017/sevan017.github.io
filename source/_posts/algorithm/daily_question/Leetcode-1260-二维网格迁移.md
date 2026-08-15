---
title: Leetcode-1260-二维网格迁移
date: 2026-07-20 09:20:02
tags: [LeetCode, 数组, 矩阵, 模拟, 每日一题]
categories: [算法题解]
permalink: shift-2d-grid/
---

## 题目

[1260. 二维网格迁移](https://leetcode.cn/problems/shift-2d-grid/description/)

给你一个 `m` 行 `n` 列的二维网格 `grid` 和一个整数 `k`。你需要将 `grid` 迁移 `k` 次。

每次「迁移」操作会引发下述活动：

- 位于 `grid[i][j]`（`j < n - 1`）的元素会移动到 `grid[i][j + 1]`。
- 位于 `grid[i][n - 1]` 的元素会移动到 `grid[i + 1][0]`。
- 位于 `grid[m - 1][n - 1]` 的元素会移动到 `grid[0][0]`。

请返回 `k` 次迁移操作后最终得到的 **二维网格**。

**示例 1：**

![示例1](https://assets.leetcode.com/uploads/2019/11/05/e1.png)

```text
输入：grid = [[1,2,3],[4,5,6],[7,8,9]], k = 1
输出：[[9,1,2],[3,4,5],[6,7,8]]
```

**示例 2：**

```text
输入：grid = [[3,8,1,9],[19,7,2,5],[4,6,11,10],[12,0,21,13]], k = 4
输出：[[12,0,21,13],[3,8,1,9],[19,7,2,5],[4,6,11,10]]
```

**示例 3：**

```text
输入：grid = [[1,2,3],[4,5,6],[7,8,9]], k = 9
输出：[[1,2,3],[4,5,6],[7,8,9]]
```

**提示：**

- `m == grid.length`
- `n == grid[i].length`
- `1 <= m <= 50`
- `1 <= n <= 50`
- `-1000 <= grid[i][j] <= 1000`
- `0 <= k <= 100`

## 思路

### 一维展开

核心思路是将二维网格**展开为一维数组**来思考。

对于一个 `m × n` 的二维网格，元素 `grid[i][j]` 在一维中的位置（下标）为：

```text
index = i × n + j
```

每次迁移相当于所有元素在一维数组中**向右移动一位**，最后一个元素移动到第一个位置。迁移 `k` 次后，原来位于一维下标 `index` 的元素，新位置为：

```text
newIndex = (index + k) % (m × n)
```

最后再将一维下标 `newIndex` 映射回二维坐标：

```text
row = newIndex / n   // 行：商
col = newIndex % n   // 列：余数
```

### 举例

以 `grid = [[1,2,3],[4,5,6],[7,8,9]]`，`k = 1` 为例：

一维展开：`[1, 2, 3, 4, 5, 6, 7, 8, 9]`

迁移 1 次后：`[9, 1, 2, 3, 4, 5, 6, 7, 8]`

还原为二维：

```text
[[9, 1, 2],
 [3, 4, 5],
 [6, 7, 8]]
```

### 复杂度分析

- **时间复杂度**：`O(m × n)`，遍历每个元素一次。
- **空间复杂度**：`O(1)`（不计返回结果所需的空间）。

## 代码

```java
class Solution {
    public List<List<Integer>> shiftGrid(int[][] grid, int k) {
        int m = grid.length, n = grid[0].length;
        int total = m * n;

        // 初始化结果二维列表
        List<List<Integer>> ans = new ArrayList<>();
        for (int i = 0; i < m; i++) {
            List<Integer> row = new ArrayList<>();
            for (int j = 0; j < n; j++) {
                row.add(0);
            }
            ans.add(row);
        }

        // 遍历原网格每个元素，计算迁移后的位置并填入
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                // 一维展开后迁移 k 步的新下标
                int index = (i * n + j + k) % total;
                // 还原为二维坐标
                int row = index / n;
                int col = index % n;
                ans.get(row).set(col, grid[i][j]);
            }
        }

        return ans;
    }
}
```

## 关键点

1. **下标映射**：`(i, j) → i × n + j`，这是二维数组一维化的核心公式。
2. **取模运算**：`% total` 处理了循环移动，当 `k` 大于总元素数时自动取余。
3. **取余与取模的区别**：Java 中 `%` 对正数运算即为取余，由于 `index` 和 `k` 均为非负数，直接使用 `%` 即可。

## 相关题目

- [189. 轮转数组](https://leetcode.cn/problems/rotate-array/) — 一维数组的轮转，本题是二维版本。
- [48. 旋转图像](https://leetcode.cn/problems/rotate-image/) — 二维矩阵的原地旋转。
