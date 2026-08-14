---
title: Leetcode-3090-每个字符最多出现两次的最长子字符串
date: 2026-08-14 08:14:00
tags: [LeetCode, 滑动窗口, 字符串, 哈希表, 每日一题]
categories: [算法题解]
permalink: maximum-length-substring-with-two-occurrences/
---

## 题目

[3090. 每个字符最多出现两次的最长子字符串](https://leetcode.cn/problems/maximum-length-substring-with-two-occurrences/description/)

给你一个字符串 `s`，请返回满足以下条件的最长子字符串的长度：

- 每个字符最多出现两次。

**示例 1：**

```text
输入：s = "bcbbbcba"
输出：4
解释：以下子字符串长度为 4，并且每个字符最多出现两次："bcba"（b 出现 2 次，c 出现 1 次，a 出现 1 次）。
```

**示例 2：**

```text
输入：s = "aaaa"
输出：2
解释：以下子字符串长度为 2，并且每个字符最多出现两次："aa"。
```

**提示：**

- `2 <= s.length <= 100`
- `s` 仅包含小写英文字母。

## 思路

### 滑动窗口

本题是 [3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/) 的变体，只需把「每个字符最多出现 1 次」放宽为「最多出现 2 次」，套用同一个**不定长滑动窗口**模板：

1. `right` 不断右移，将新字符加入窗口，频次 +1；
2. 若该字符频次超过 2，说明窗口不合法，从 `left` 开始收缩：移除 `left` 处字符，频次 -1，`left++`，直到该字符频次 ≤ 2；
3. 此时窗口 `[left, right]` 一定合法，用 `right - left + 1` 更新答案。

因为 `left`、`right` 都只向右移动，每个字符至多被加入、移除各一次，时间复杂度 `O(n)`；频次表最多存 26 个小写字母，空间复杂度 `O(26)`。

### 示例推演

以 `s = "bdbbabccad"` 为例：

| right | 字符 | 收缩后窗口 | 窗口内频次 | ans |
| ----- | ---- | ---------- | ---------- | --- |
| 0 | b | `[0,0]`="b" | b:1 | 1 |
| 1 | d | `[0,1]`="bd" | b:1, d:1 | 2 |
| 2 | b | `[0,2]`="bdb" | b:2, d:1 | 3 |
| 3 | b | `[1,3]`="dbb"（b 超 2，移除 left=0 的 b） | b:2, d:1 | 3 |
| 4 | a | `[1,4]`="dbba" | b:2, d:1, a:1 | 4 |
| 5 | b | `[3,5]`="bab"（b 超 2，依次移除 d、b） | b:2, a:1 | 4 |
| 6 | c | `[3,6]`="babc" | b:2, a:1, c:1 | 4 |
| 7 | c | `[3,7]`="babcc" | b:2, a:1, c:2 | 5 |
| 8 | a | `[3,8]`="babcca" | b:2, a:2, c:2 | 6 |
| 9 | d | `[3,9]`="babccad" | b:2, a:2, c:2, d:1 | 7 |

返回 7。

## 出错信息（备注）

写代码时踩了一个坑，记录在此：收缩窗口的 `while` 循环里错误地写了

```java
ch = s.charAt(left);
```

复用了外层变量 `ch`。问题在于：`ch` 是 `right` 处字符——正是它触发频次超限，`while` 条件 `occ.get(ch) > 2` 要靠它判断是否继续收缩。在循环内把它改成 `left` 处字符后，条件判断的对象变成了**刚被移除的那个字符**，而它的频次刚刚减 1，大概率已经 ≤ 2，于是循环**提前退出**，超限字符的频次根本没降下来，窗口仍然不合法。

反例：`s = "abbbc"`，正确答案是 3（如 "abb"、"bbc"）。

- right=3 时窗口 "abbb"，b 出现 3 次，进入收缩循环；
- 错误写法把 `ch` 改为 `s.charAt(0) = 'a'`，移除 a 后 `occ['a'] = 0`，条件 `0 > 2` 不成立，循环立刻退出；
- 此时窗口 "bbb" 中 b 仍出现 3 次，却按合法窗口参与计算，最终返回 4。

正确做法：用一个**新变量**接收 `left` 处字符，循环条件里的 `ch` 保持不动：

```java
while (occ.get(ch) > 2) {
    char lch = s.charAt(left);
    occ.put(lch, occ.getOrDefault(lch, 0) - 1);
    left++;
}
```

本质：`while` 条件的判断对象（`right` 处超限字符）和循环体的操作对象（`left` 处被移除字符）**不是同一个字符**，必须用两个变量区分。

## 代码

```java
class Solution {
    public int maximumLengthSubstring(String s) {
        int ans = 0, left = 0;
        Map<Character, Integer> occ = new HashMap<>();
        for (int right = 0; right < s.length(); right++) {
            char ch = s.charAt(right);
            occ.put(ch, occ.getOrDefault(ch, 0) + 1); // 元素加入窗口，频次 + 1
            while (occ.get(ch) > 2) {                 // 窗口不合法则收缩左边界
                char lch = s.charAt(left);
                occ.put(lch, occ.getOrDefault(lch, 0) - 1);
                left++;
            }
            ans = Math.max(ans, right - left + 1);    // 窗口 [left, right] 长度
        }
        return ans;
    }
}
```

**复杂度**：时间 `O(n)`，空间 `O(26)`（`HashMap` 最多存 26 个键，也可用 `int[26]` 数组进一步精简）。

## 关键点

1. **模板识别**：「每个字符最多出现 k 次」是经典滑动窗口模型，本题 k=2。窗口内频次超过 k 就收缩左边界，收缩后的窗口必然合法，此时更新答案。
2. **收缩条件**：`while` 只需判断**当前加入的字符**（`right` 处）是否超限——收缩过程中其他字符只会变少，不会新产生超限，因此不必检查整个频次表。
3. **变量隔离**：循环条件的判断对象与循环体的操作对象是不同字符，不要复用同一个变量（见「出错信息」一节的反例）。
4. **复杂度直觉**：左右指针均单调右移，均摊 `O(n)`。

## 相关题目

- [3. 无重复字符的最长子字符串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/) — 同模板，k=1 的特例。
- [2958. 最多 K 个重复元素的最长子数组](https://leetcode.cn/problems/length-of-longest-subarray-with-at-most-k-frequency/) — 本题的泛化版本。
- [1695. 删除子数组的最大得分](https://leetcode.cn/problems/maximum-erasure-value/) — 滑动窗口 + 窗口内元素去重计数。