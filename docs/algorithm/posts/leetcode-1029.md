---
draft: false
date:
  created: 2025-03-03
  updated: 2025-03-06
categories:
  - Algorithm
  - Note
authors:
  - BaCl2
  - zhazi
---

# Leetcode 1029. 两地调度

???+ info "两地调度"

    公司计划面试 `2n` 人。给你一个数组 `costs `，其中 **costs[i] = [aCost<sub>i</sub>, bCost<sub>i</sub>]** 。第 `i` 人飞往 a 市的费用为 aCost<sub>i</sub> ，飞往 b 市的费用为 **bCost<sub>i</sub>** 。

    返回将每个人都飞到 a 、b 中某座城市的最低费用，要求每个城市都有 `n` 人抵达。

??? tips "提示"

    - `2 * n == costs.length`
    - `2 <= costs.length <= 100`
    - `costs.length` 为偶数
    - 1 <= aCost<sub>i</sub>, bCost<sub>i</sub> <= 1000

<!-- more -->

??? example "示例1"

    **输入**：costs = [[10,20],[30,200],[400,50],[30,20]]  
    **输出**：110  
    **解释**：  
    > 第一个人去 a 市，费用为 10。
    > 第二个人去 a 市，费用为 30。
    > 第三个人去 b 市，费用为 50。
    > 第四个人去 b 市，费用为 20。
    > 
    > 最低总费用为 10 + 30 + 50 + 20 = 110，每个城市都有一半的人在面试。

??? example "示例2"

    **输入**：costs = [[259,770],[448,54],[926,667],[184,139],[840,118],[577,469]]  
    **输出**：1859  

??? example "示例3"

    **输入**：costs = [[515,563],[451,713],[537,709],[343,819],[855,779],[457,60],[650,359],[631,42]]  
    **输出**：3086  

## 思考

### 暴力解
一个简单的想法是，对每个人讨论去 a 地和去 b 地两种情况。同 [Leetcode 514.](./leetcode-514.md) 一样，很容易写出一个递归函数 `f(consts, i, a, b)`，其中 `consts` 是不变的参数，`i, a, b` 用于表示当前的状态，即：前 `i-1` 个人已经分配完毕，开始分配第 `i` 个人，此时，a 地已经分配 `a` 人，b 地已经分配 `b` 人。`f` 返回这一状态下的最小代价。当然，数据量不允许这样暴力求解。

```c++ title="递归函数"
int f(std::vector<std::array<int, 2>> &costs, int i, int a, int b) {
    if (i == costs.size()) {
        return 0;
    }
    
    int ans1 = INF, ans2 = INF;

    if (a > 0) {
        ans1 = f(costs, i + 1, a - 1, b) + costs[i][0];
    }

    if (b > 0) {
        ans2 = f(costs, i + 1, a, b - 1) + costs[i][1];
    }

    return std::min(ans1, ans2);
}
```

### 贪心
不难注意到：记 `dif` 为某个人去 a 地的花费和去 b 地的花费的差（不取绝对值)，如果这个差很小，说明他去 a 地更好，反之去 b 地更好。因此我们考虑一个贪心策略，对 `costs` 排序（按 `dif` 升序)，然后让前一半去 a 地，后一半去 b 地。

```c++ title="贪一下"
struct Cmp {
    bool operator()(const std::array<int, 2>& a, const std::array<int, 2>& b) const {
        int dif1 = a[0] - a[1], dif2 = b[0] - b[1];
        return dif1 < dif2;
    }
};

int solve2(std::vector<std::array<int, 2>>& costs) {
    int n = costs.size() >> 1;
    
    std::sort(costs.begin(), costs.end(), Cmp());

    int ans = 0;
    for (int i = 0; i < n; i++) {
        ans += costs[i][0];
    }

    for (int i = n; i < n << 1; i++) {
        ans += costs[i][1];
    }

    return ans;
}
```

### 对数器
证明贪心是困难的，因此在手上有一个暴力解的时候，写一个对数器做实验证明是(~~对菜鸡而言~~)更合理的选择。

```c++ title="对数器"
int n = 14;
vector<array<int, 2>> costs(n);

for (int i = 0; i < n; i++) {
    int a = rand() % 1000 + 1;
    int b = rand() % 1000 + 1;
    costs[i] = {a, b};
}

int solvel(vector<array<int, 2>>& costs);
int solve2(vector<array<int, 2>>& costs);

int ans1 = solvel(costs), ans2 = solve2(costs);
if (ans1 != ans2) {
    cout << "[";
    for (int i = 0; i < n; i++) {
        cout << "[" << costs[i][0] << ", " << costs[i][1] << "]";
        if (i < n - 1) cout << ", ";
    }
    cout << "]" << endl;
    cout << ans1 << " " << ans2 << endl;
}
```

实验证明上述贪心策略在经验上是正确的。那么，提交。
 
## 代码

```c++ linenums="1"
#include <bits/stdc++.h>

using namespace std;

struct Cmp {
	bool operator()(const std::vector<int> &a, const std::vector<int> &b) const {
		int dif1 = a[0] - a[1], dif2 = b[0] - b[1];
		return dif1 < dif2;
	}
};

class Solution {
public:
    int twoCitySchedCost(vector<vector<int>>& costs) {
        int n = costs.size() >> 1;

        std::sort(costs.begin(), costs.end(), Cmp());

        int ans = 0;
        for (int i = 0; i < n; i++) {
            ans += costs[i][0];
        }
        for (int i = n; i < n << 1; i++) {
            ans += costs[i][1];
        }

        return ans;
    }
};

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(nullptr);

    Solution problem;

    auto startT = std::chrono::high_resolution_clock::now();

    std::vector<std::vector<int>> cost = {{10,20},{30,200},{400,50},{30,20}};
    problem.twoCitySchedCost(cost);

    auto endT = std::chrono::high_resolution_clock::now();
	std::chrono::duration<double> duration = endT - startT;
	std::cout << "Time taken: " << duration.count() << " seconds\n\n";

    return 0;
}
```

## 结果

![](./images/leetcode-1029-提交结果.png)

结果的确如此。
{ .caption }
