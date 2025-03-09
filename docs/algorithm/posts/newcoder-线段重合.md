---
draft: false
date:
  created: 2025-03-06
  updated: 2025-03-08
  updated: 2025-03-09
categories:
  - Algorithm
  - Note
authors:
  - BaCl2
  - zhazi
---

# Newcoder 线段重合

???+ info "线段重合"

    每一个线段都有 `start` 和 `end` 两个数据项，表示这条线段在 X 轴上从 `start` 位置开始到 `end` 位置结束。
    给定一批线段，求所有重合区域中最多重合了几个线段，首尾相接的线段不算重合。
    例如：线段 `[1,2]` 和线段 `[2,3]` 不重合。线段 `[1,3]` 和线段 `[2,3]` 重合

    **输入描述：**

    > 第一行一个数N，表示有N条线段
    > 接下来N行每行2个数，表示线段起始和终止位置

    **输出描述：**

    > 输出一个数，表示同一个位置最多重合多少条线段

??? tips "备注"

    - $N \le 10^4$
    - $1 \le start, end \le 10^5$

<!-- more -->

??? example "示例1"

    ```console
    输入: 3
          1 2
          2 3
          1 3
    输出: 2
    ```

## 思考

有多个解法，都挺有意思，一并记录在这里。


### 模拟([扫描线](https://oi-wiki.org/geometry/scanning/))

把线段视为一个个事件，目标是确定最多有几个事件同时进行。考虑到这里的“时间”是有限的（整型，范围是 $[1,10^5]$），我们完全可以进行模拟，基本思想是：是**“模拟一条扫描线从一个方向扫过所有事件”**，在扫描过程中维护一个数据结构来追踪当前的状态（例如活动区间的数量、最小值、最大值等）。

对于本题，我们申请一个 $10^5+5$ 大小的数组 `times`，把事件的起点 `times[start]` 记为 1, 终点 `times[end]` 记为 -1，其他位置置为 0，然后模拟时间流动，维护一个 `cur` 记录当前时间点上同时进行的事件数，再维护一个 `ans` 记录答案。

```c++
// 记录
while (n--) {
    scanf("%d %d",&start,&end);
    times[start] += 1;
    times[end] -= 1;
}

// 模拟
for(int i = 1; i < 100001; i++) {
    cur += times[i];
    ans = max(ans, cur);
}
```

时间复杂度: $O(m)$，空间复杂度：$O(m)$，$m$ 与线段端点的取值范围有关，在本题中为 $10^5$。

### 差分

上面的记录方式实际上是一种差分的思想，我们记录下每个位置的**变化量**然后进行模拟，因此也可以考虑把 `times` 加工成一个前缀和数组从而省掉 `cur` 变量。

```c++
for(int i = 1; i < 100001; i++) {
    times[i] += times[i-1];
    ans = max(ans, times[i]);
}
```

时间复杂度和空间复杂度与上面一样。

### 离散化

模拟的做法简单直观，但是在数据范围比较大，甚至数据类型为浮点型的时候就不可行了。那么有没有一种办法可以解决这些问题呢？有的兄弟，有的，我们可以考虑**离散化**。把每个事件的端点按时间顺序塞到数组里面，然后在这个数组上进行模拟，就可以完美的解决这些问题。这里再注意一下起点和终点相同时起点在前，所以给 `start` 和 `end` 分配一个数字来区分，前面的 `+1` 和 `-1` 看起来就挺不错。

```c++
vector<pair<int, int>> events;

scanf("%d",&n);
while (n--) {
    scanf("%d %d",&start,&end);
    events.push_back({start, 1}); 
    events.push_back({end, -1}); 
}
```

之后的做法和前面一样，时间复杂度为: $O(Nlog(N))$ (多了一道排序)，空间复杂度：$O(N)$。


### 数据结构（堆）

还有一个比较 trick 的做法：使用堆结构来维护**“正在进行中的事件”**。前面我们一直以时间点为单位进行处理，现在我们转变思路，以事件为单位进行处理。

我们的基本思想是：**“按顺序记录事件，然后在新的事件开始时，检查哪些事件结束了，把这些事件移除。”**因此我们需要维护所有记录中的事件的终点 `record_ends`，以及新的事件的起点 `new_start`，然后把 `record_ends` 中小于 `new_start` 的事件移除。因此我们希望 `record_ends` 的记录是有序的，并且在移除元素后保持有序，最好还能高效的移除元素，那么使用堆结构维护 `record_ends` 的结论就呼之欲出了。

具体来说，我们按 `start` 升序对线段排序。之后遍历线段，每次把线段的 `end` 放入小根堆，若遍历到某条线段有 `new_start >= record_ends.top()`，说明这条线段与记录中的某些线段没有重叠，那么弹出堆顶元素，直到 `new_start < record_ends.top()`，这样就保证了**“堆中的线段都与新加入的线段有重叠。”**我们记录每次堆内线段的数量，维护一个最大值作为我们的答案即可。

```c++
std::priority_queue<int, std::vector<int>, std::greater<int>> record_ends;

int ans = 0;
for (int i = 0; i < n; i++) {
  while (!record_ends.empty() && lines[i][0] >= record_ends.top()) {
    record_ends.pop();
  }
  record_ends.push(lines[i][1]);
  ans = std::max(ans, (int)record_ends.size());
}
```

时间复杂度为: $O(Nlog(N))$，空间复杂度：$O(N)$。


### 线段树

主播主播，你的贪心确实很强，但还是太吃操作了，有没有更粗暴的方法推荐一下？有的兄弟，有的，像这样强势的方法还有很多，比如**线段树**。我们可以维护一棵线段树，每个叶子的值是经过该点的线段的数量，最终答案就是所有叶子结点中的的最大值。需要注意的是，向线段树添加数据时，应该是加入一个左闭右开的区间 `[start, end)`。 此外，如果数据量更大，可以进一步离散化 `start` 和 `end`，建立更小的线段树(未实现)。

时间复杂度为: $O(Nlog(N))$，空间复杂度：$O(N)$。

## 代码

### 模拟

```c++ linenums="1"
#include <bits/stdc++.h>
#include <cstdio>
#include <ctime>
using namespace std;

int times[10005];

int main() {
    int n, start, end;
    int cur = 0, ans = 0;
    scanf("%d",&n);

    while (n--) {
        scanf("%d %d",&start,&end);
        times[start] += 1;
        times[end] -= 1;
    }
    for(int i = 1; i < 100001; i++) {
        cur += times[i];
        ans = max(ans, cur);
    }
    printf("%d",ans);
}
```

### 差分

```c++ linenums="1"
#include <bits/stdc++.h>
#include <cstdio>
#include <ctime>
using namespace std;

int times[10005];

int main() {
    int n, start, end;
    int ans = 0;
    scanf("%d",&n);

    while (n--) {
        scanf("%d %d",&start,&end);
        times[start] += 1;
        times[end] -= 1;
    }
    for(int i = 1; i < 100001; i++) {
        times[i] += times[i-1];
        ans = max(ans, times[i]);
    }
    printf("%d",ans);
}
```

### 离散化

```c++
#include <bits/stdc++.h>
#include <cstdio>
#include <ctime>
using namespace std;

int main() {
    int n, start, end;
    int cur = 0, ans = 0;
    vector<pair<int, int>> events;

    scanf("%d",&n);
    while (n--) {
        scanf("%d %d",&start,&end);
        events.push_back({start, 1}); 
        events.push_back({end, -1}); 
    }

    sort(events.begin(), events.end(), [](const pair<int, int>& a, const pair<int, int>& b) {
        if (a.first == b.first) {
            return a.second < b.second;
        }
        return a.first < b.first;
    });

    for (const auto& event : events) {
        cur += event.second;
        ans = max(cur, ans);
    }
    printf("%d",ans);
}
```

### 数据结构（堆）

```c++
#include <bits/stdc++.h>

using i64 = long long;
using u64 = unsigned long long;
using u32 = unsigned;
using u128 = unsigned __int128;

struct Cmp {
	bool operator()(const std::array<int, 2> &a, const std::array<int, 2> &b) const {
		return a[0] < b[0];
	}
};

int main() {
	std::ios::sync_with_stdio(false);
	std::cin.tie(nullptr);

	int n;
	std::cin >> n;

	std::vector<std::array<int, 2>> lines(n);
	for (auto &line : lines) {
		std::cin >> line[0] >> line[1];
	}

	std::sort(lines.begin(), lines.end(), Cmp());

	std::priority_queue<int, std::vector<int>, std::greater<int>> record_ends;

	int ans = 0;
	for (int i = 0; i < n; i++) {
		while (!record_ends.empty() && lines[i][0] >= record_ends.top()) {
			record_ends.pop();
		}
		record_ends.push(lines[i][1]);
		ans = std::max(ans, (int)record_ends.size());
	}

	std::cout << ans << "\n";

	return 0;
}
```

### 线段树

```c++
#include <bits/stdc++.h>
 
using i64 = long long;
using u64 = unsigned long long;
using u32 = unsigned;
using u128 = unsigned __int128;
 
constexpr int N = 1e5 + 7;
 
struct Node {
    int max;
    int add;
} tree[N << 2];
int n;
 
void pushUp(int root) {
    tree[root].max = std::max(tree[root << 1].max, tree[root << 1 | 1].max);
}
 
void lazy(int root, int lazy) {
    tree[root].max += lazy;
    tree[root].add += lazy;
}
 
void pushDown(int root) {
    if (tree[root].add > 0) {
        lazy(root << 1, tree[root].add);
        lazy(root << 1 | 1, tree[root].add);
        tree[root].add = 0;
    }
}
 
void build(int l = 1, int r = N - 7, int root = 1) {
    if (l == r) {
        tree[root].max = 0;
        tree[root].add = 0;
        return ;
    }
 
    int mid = l + ((r - l) >> 1);
 
    build(l, mid, root << 1);
    build(mid + 1, r, root << 1 | 1);
 
    pushUp(root);
}
 
void add(int jobl, int jobr, int jobv, int l = 1, int r = N - 7, int root = 1) {
    if (l >= jobl && r <= jobr) {
        lazy(root, jobv);
        return ;
    }
    int mid = l + ((r - l) >> 1);
 
    pushDown(root);
 
    if (mid >= jobl) {
        add(jobl, jobr, jobv, l, mid, root << 1);
    }
    if (mid + 1 <= jobr) {
        add(jobl, jobr, jobv, mid + 1, r, root << 1 | 1);
    }
 
    pushUp(root);
}
 
int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(nullptr);
 
    std::cin >> n;
 
    for (int i = 0; i < n; i++) {
        int a, b;
        std::cin >> a >> b;
 
        add(a, b - 1, 1);
    }
 
    std::cout << tree[1].max << "\n";
 
    return 0;
}
```
