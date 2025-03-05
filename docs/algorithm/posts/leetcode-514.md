---
draft: false
date:
  created: 2025-03-01
  updated: 2025-03-02
categories:
  - Algorithm
  - Note
authors:
  - BaCl2
  - zhazi
---

# Leetcode 514. 自由之路

???+ info "自由之路"

    电子游戏“辐射4”中，任务 “**通向自由**” 要求玩家到达名为 “**Freedom Trail Ring**” 的金属表盘，并使用表盘拼写特定关键词才能开门。

    给定一个字符串 `ring`，表示刻在外环上的编码；给定另一个字符串 `key`，表示需要拼写的关键词。您需要算出能够拼写关键词中所有字符的**最少**步数。

    最初，`ring` 的第一个字符与 `12:00` 方向对齐。您需要顺时针或逆时针旋转 `ring` 以使 **key** 的一个字符在 `12:00` 方向对齐，然后按下中心按钮，以此逐个拼写完 **`key`** 中的所有字符。

    旋转 `ring` 拼出 `key` 字符 `key[i]` 的阶段中：

    1. 您可以将 **ring** 顺时针或逆时针旋转一个位置 ，计为1步。旋转的最终目的是将字符串 **`key`** 的一个字符与 `12:00` 方向对齐，并且这个字符必须等于字符 **`key`**。
    2. 如果字符 **`key[i]`** 已经对齐到 `12:00` 方向，您需要按下中心按钮进行拼写，这也将算作 **1** 步。按完之后，您可以开始拼写 **key** 的下一个字符（下一阶段）, 直至完成所有拼写。

??? tips "提示"

    - `1 <= ring.length, key.length <= 100`
    - `ring` 和 `key` 只包含小写英文字母
    - **保证**字符串 `key` 一定可以由字符串 `ring` 旋转拼出

<!-- more -->

??? example "示例1"

    <img src="https://assets.leetcode.com/uploads/2018/10/22/ring.jpg" style="width:25%;" />

    **输入**: ring = "godding", key = "gd"  
    **输出**: 4  
    **解释**:  
    > 对于 key 的第一个字符 'g'，已经在正确的位置, 我们只需要1步来拼写这个字符。 
    > 对于 key 的第二个字符 'd'，我们需要逆时针旋转 ring "godding" 2步使它变成 "ddinggo"。  
    > 当然, 我们还需要1步进行拼写。  
    > 因此最终的输出是 4。  


??? example "示例2"

    **输入**: ring = "godding", key = "godding"  
    **输出**: 13

## 思考
整个拼写过程就是不断的找字母，然后按下中心按钮。找字母的过程可能要遍历整个字符串，如果字母在 `ring` 中有多个对应，那么还会出现分支情况。

可以看作是只有两个方向的寻路问题考虑递归搜索。当前所处的状态可以表示为 `(key[i], ring[j])`，即：当前已经输入 `key` 的前 `i-1` 个字母，正在寻找 `key[i]`，此时 `12:00` 方向字母为 `ring[j]`。那么定义一个递归函数 `f`，接受两部分参数：1. 当前的状态，2. 下一步的方向，返回选择这个方向后还需要的最少步数就可以开始递归了。

### 定义递归函数
具体来说，定义递归函数 `f(ring, key, n, m, i, j, d)`，对问题进行递归求解。其中，`n` 是 `ring` 的长度，`m` 是 `key` 的长度，前四个参数都是不变的参数，`i` 表示当前考虑 `key[i]`，`j` 表示当前考虑 `s[j]`，这两个参数表示了当前的状态，`d` 在 `{0,1}` 中取值，表示下一步方向是顺时针或者逆时针。最后，再加上一点点优化——记忆化搜索，加一个参数 `dp` 表来记录已经解决的状态。因此，最终定义的 `f` 为 **`f(ring, key, n, m, i, j, d, dp)`**，`dp` 表格定义为 **`dp[i][j][d]`**。

### 递归过程
看 `s[j]` 是否可以填入 `key[i]`，如果可以，我们递归处理 `key[i+1]`；否则，我们按照当前选择的方向移动 `j`，递归:

```c++
int ans = 0;
if (key[i] == ring[j]) {
    ans = std::min(f(ring, key, n, m, i + 1, j, 0, dp), f(ring, key, n, m, i + 1, j, 1, dp)) + 1;
} else {
    if (d == 0) {
        ans = f(ring, key, n, m, i, j - 1, 0, dp) + 1;
    } else {
        ans = f(ring, key, n, m, i, j + 1, 1, dp) + 1;
    }
}
```

### 边界情况
当 `i` 来到 `m` 时，`key[i]` 越界，说明 `key` 已经填好，返回剩余操作数是 0:

```c++
if (i == m) {
    return 0;
}
```

## 代码

```cpp linenums="1"
class Solution {
public:
    int f(std::string &ring, std::string &key, int n, int m, int i, int j, int d,std::vector<std::vector<std::vector<int>>> &dp) {
        if (i == m) {
            return 0;
        }

        // NOTE: ring 是环形数组
        if (j == n) {
            j = 0;
        }
        if (j == -1) {
            j = n - 1;
        }

        if (dp[i][j][d] != -1) {
            return dp[i][j][d];
        }

        int ans = 0;
        if (key[i] == ring[j]) {
            ans = std::min(f(ring, key, n, m, i + 1, j, 0, dp), f(ring, key, n, m, i + 1, j, 1, dp)) + 1;
        } else {
            if (d == 0) {
                ans = f(ring, key, n, m, i, j - 1, 0, dp) + 1;
            } else {
                ans = f(ring, key, n, m, i, j + 1, 1, dp) + 1;
            }
        }

        dp[i][j][d] = ans;
        return ans;
    }

    int findRotateSteps(string ring, string key) {
        int n = ring.size(), m = key.size();
        std::vector<std::vector<std::vector<int>>> dp(m, std::vector<std::vector<int>> (n, std::vector<int> (2, -1)));

        return std::min(f(ring, key, n, m, 0, 0, 0, dp), f(ring, key, n,m, 0, 0, 1, dp));
    }
};
```
