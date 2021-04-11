---
title: 动态规划
author: lijiabao
date: 2021-04-11 20:30
categories: ["动态规划", "DataStruture and Alogrithm"]
tags: ["动态规划", "DataStruture and Alogrithm"]
---

动态规划，和分治方法类似，都是通过子问题的组合求解。通常动态规划用于求解最优化问题，这类问题可以有很多可行解，每个解都有一个，希望寻找具有最优值（最大或最小）的解

## 动态规划

动态规划最基础的解决就是穷举法，然后利用穷举法的实现再去找寻优化的穷举方法。找出穷举

### 动态规划解题步骤和实现方法

动态规划算法涉及内容：

1. 刻画一个最优解的结构特征
2. 递归定义最优解的值
3. 计算最优解的值，通常采用自底向上
4. 利用计算出的解信息构建一个最优解

#### 动态规划解题步骤

1. 确定子问题和状态：也就是确定原问题和子问题中变化的变量（找到问题答案的最后一步并找到子问题）
2. 确定dp（状态转移方程）函数的定义
3. 确定边界条件和初始条件：也就是递归终止条件或者最开始的最小子问题
4. 确定计算顺序

#### 两种等价实现方法

1. 第一种是带备忘的自顶向下的计算方法：由于存在多个子问题重复求解的问题，因此为了减少子问题的重复求解，可以添加一个备忘，类似缓存的机制，将第一次计算的结果保存，之后遇见这个问题直接把值拿过来使用就好了，这种解法是一种典型的空间换时间的做法。
2. 第二种方法是自底向上的解法（使用DP-table）：每次的问题计算都依赖于更小的子问题的解，因此求解问题时先将问题的规模从小到大排列起来，从小到大求解。当需要求解某个子问题时，所依赖的更小子问题已经求解完毕，结果保存可以直接使用，每个子问题只需要求解一次。但是求解该子问题的前提是他依赖的子问题已经求解完毕。

上面两种方法的效率分析：由于自底向上的求解方法少了反复调用递归函数的开销，因此运行时间更少，运行更快。

### 动态规划三要素

- 重叠子问题：就是在求解暴力求解的过程种会碰到有些子问题被重复计算或者重复出现，这类子问题称为重叠子问题
- 最优子结构：原最优问题分解的子问题也需要是最优的才能保证原问题是最优的。
- 状态转移方程：动态规划问题的数学描述，DP-table的使用和这个状态转移方程相关，是解决问题的核心，而往往状态转移方程代表着暴力解的实现。<font color='red'>动态规划问题最难最重要的一步就是列出状态转移方程，即写出暴力解</font>

### 动态规划问题实例

#### 最长回文字符串

对于最长回文字符串的解法简单的有两种：

1. 动态规划解
2. 中心扩散解

##### 动态规划解

1. 首先找到最后一步和子问题：最后一步就是最长的回文字符串，那么这时候的子问题就通过最大回文字符串左右都减少一个字符得到，这时候只要子问题满足回文字符串且左右减少的字符相等，最后一步也就是满足条件的回文字符串
2. 确定转移方程：`f(i,j) = f(i+1, j-1) && (s[i] == s[j])`
3. 确定边界条件和初始条件：当字符串长度为1，都满足回文字符串，字符串长度为2，需要两个字符串相等时才是回文字符串
4. 计算顺序：从最小的字符串开始往后不断查找满足条件的回文字符串，直到最后找到最大的字符串

```java
class Solution {
    public String longestPalindrome(String s) {
        int n = s.length();
        boolean[][] dp = new boolean[n][n];
        String ans = "";
        for (int l = 0; l < n; ++l) {
            for (int i = 0; i + l < n; ++i) {
                int j = i + l;
                if (l == 0) {
                    dp[i][j] = true;
                } else if (l == 1) {
                    dp[i][j] = (s.charAt(i) == s.charAt(j));
                } else {
                    dp[i][j] = (s.charAt(i) == s.charAt(j) && dp[i + 1][j - 1]);
                }
                if (dp[i][j] && l + 1 > ans.length()) {
                    ans = s.substring(i, i + l + 1);
                }
            }
        }
        return ans;
    }
}
```

时间：O(n\*n)，空间O(n\*n)

##### 中心扩散法

1. 首先找到所有满足条件的回文字符串（长度为一或者为二的回文字符串）
2. 然后对这些所有满足条件的字符串向左右两边扩散
3. 然后输出最大的回文字符串

```java
class Solution {
    public String longestPalindrome(String s) {
        if (s == null || s.length() < 1) {
            return "";
        }
        int start = 0, end = 0;
        for (int i = 0; i < s.length(); i++) {
            int len1 = expandAroundCenter(s, i, i);
            int len2 = expandAroundCenter(s, i, i + 1);
            int len = Math.max(len1, len2);
            if (len > end - start) {
                start = i - (len - 1) / 2;
                end = i + len / 2;
            }
        }
        return s.substring(start, end + 1);
    }

    public int expandAroundCenter(String s, int left, int right) {
        while (left >= 0 && right < s.length() && s.charAt(left) == s.charAt(right)) {
            --left;
            ++right;
        }
        return right - left - 1;
    }
}
```

时间O(n\*n)，空间O(1)

#### 最长有效匹配括号

##### 动态规划解

1. 确定最后一步和子问题：最后一步就是最长的有效匹配括号，子问题有两种情况，如果这个字符串末尾和倒数第二个元素都为右括号，或者这两个括号匹配。将字符串索引为i处最大括号匹配为`dp[i]`

2. 动态转移方程：
   这个问题的转移方程分析需要较为细致一些，首先分析需要从最长匹配的末尾元素开始分析，如果最后元素不是右括号，显然不可能满足有效括号匹配
   如果是右括号，需要向前查看一位

   1. 如果前面一位是左括号，则直接有一个匹配括号，因此变成一个子问题`dp[i] = dp[i-2] + 2`，且此处的i需要满足大于等于2
   2. 如果还是右括号，则需要进行更细致一点的分析，和i处所在的右括号匹配位置是`i-dp[i-1]-1`，如果此处为左括号，则匹配，长度需要添加2，即`dp[i] = dp[i-1] + 2`，类似这种情况下，还有一种特殊的情况，就是之前的子序列如果是一种特殊序列`(...)`，还需要再加上一种评判，`i-dp[i-1]-2`子序列的匹配长度
   
   因此子问题就变成：
   
   1. `dp[i] = dp[i-2] + 2`
   2. `dp[i] = dp[i-1] + dp[i-dp[i-1]-2] + 2`
   
3. 初始条件：`dp[0] = 0`.
   边界条件：`i-2 > 0`和`i-dp[i-1]-2 >= 0`

4. 计算顺序：从0开始计算1，2，3...，n，依次计算完毕后，获取最大匹配数量

```java
public class Solution {
    public int longestValidParentheses(String s) {
        int maxans = 0;
        int[] dp = new int[s.length()];
        for (int i = 1; i < s.length(); i++) {
            if (s.charAt(i) == ')') {
                if (s.charAt(i - 1) == '(') {
                    dp[i] = (i >= 2 ? dp[i - 2] : 0) + 2;
                } else if (i - dp[i - 1] > 0 && s.charAt(i - dp[i - 1] - 1) == '(') {
                    dp[i] = dp[i - 1] + ((i - dp[i - 1]) >= 2 ? dp[i - dp[i - 1] - 2] : 0) + 2;
                }
                maxans = Math.max(maxans, dp[i]);
            }
        }
        return maxans;
    }
}
```

时空复杂度都是O(n)

##### 利用数据结构栈来解决

使用优秀的数据结构改善算法时空复杂度较好的例子

具体做法是我们始终保持栈底元素为当前已经遍历过的元素中「最后一个没有被匹配的右括号的下标」，这样的做法主要是考虑了边界条件的处理，栈里其他元素维护左括号的下标：

对于遇到的每个 ‘(’，我们将它的下标放入栈中
对于遇到的每个 ‘)’，我们先弹出栈顶元素表示匹配了当前右括号：
如果栈为空，说明当前的右括号为没有被匹配的右括号，我们将其下标放入栈中来更新我们之前提到的「最后一个没有被匹配的右括号的下标」
如果栈不为空，当前右括号的下标减去栈顶元素即为「以该右括号为结尾的最长有效括号的长度」
我们从前往后遍历字符串并更新答案即可。

需要注意的是，如果一开始栈为空，第一个字符为左括号的时候我们会将其放入栈中，这样就不满足提及的「最后一个没有被匹配的右括号的下标」，为了保持统一，我们在一开始的时候往栈中放入一个值为 −1 的元素。

```java
public class Solution {
    public int longestValidParentheses(String s) {
        int maxans = 0;
        Deque<Integer> stack = new LinkedList<Integer>();
        stack.push(-1);
        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) == '(') {
                stack.push(i);
            } else {
                stack.pop();
                if (stack.empty()) {
                    stack.push(i);
                } else {
                    maxans = Math.max(maxans, i - stack.peek());
                }
            }
        }
        return maxans;
    }
}
```

时空复杂度都是O(n)

##### 不适用额外空间的方法

在此方法中，我们利用两个计数器 left 和 right 。首先，我们从左到右遍历字符串，对于遇到的每个 ‘(’，我们增加 left 计数器，对于遇到的每个 ‘)’ ，我们增加 right 计数器。每当 left 计数器与 right 计数器相等时，我们计算当前有效字符串的长度，并且记录目前为止找到的最长子字符串。当 right 计数器比 left 计数器大时，我们将 left 和 right 计数器同时变回 0。

这样的做法贪心地考虑了以当前字符下标结尾的有效括号长度，每次当右括号数量多于左括号数量的时候之前的字符我们都扔掉不再考虑，重新从下一个字符开始计算，但这样会漏掉一种情况，就是遍历的时候左括号的数量始终大于右括号的数量，即 (() ，这种时候最长有效括号是求不出来的。

解决的方法也很简单，我们只需要从右往左遍历用类似的方法计算即可，只是这个时候判断条件反了过来：

当 left 计数器比 right 计数器大时，我们将 left 和 right 计数器同时变回 0
当 left 计数器与 right 计数器相等时，我们计算当前有效字符串的长度，并且记录目前为止找到的最长子字符串
这样我们就能涵盖所有情况从而求解出答案。

```java
public class Solution {
    public int longestValidParentheses(String s) {
        int left = 0, right = 0, maxlength = 0;
        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) == '(') {
                left++;
            } else {
                right++;
            }
            if (left == right) {
                maxlength = Math.max(maxlength, 2 * right);
            } else if (right > left) {
                left = right = 0;
            }
        }
        left = right = 0;
        for (int i = s.length() - 1; i >= 0; i--) {
            if (s.charAt(i) == '(') {
                left++;
            } else {
                right++;
            }
            if (left == right) {
                maxlength = Math.max(maxlength, 2 * left);
            } else if (left > right) {
                left = right = 0;
            }
        }
        return maxlength;
    }
}
```

时间是O(n)，空间O(1)