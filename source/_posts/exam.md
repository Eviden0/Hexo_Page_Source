---
title: 小垃圾
date: '2024-5-17 17:35'
categories: 新手村的操作
description: GO!
abbrlink: b757dff5
---

1:

```cpp
#include <bits/stdc++.h>
using namespace std;
int n, k;
long long totalSum, huaNum, cnt, deleteNum;
bool cmp(int x, int y)
{
    return x > y;
}
int solve(vector<int> &a)
{
    sort(a.begin(), a.end(), cmp);
    // cout << a.capacity() << endl;
    // 取k个数
    for (int i = 0; i < k; i++)
    {
        huaNum += a[i];
    }
    deleteNum = huaNum;
    // 已经直接大于小华的分数
    if (totalSum > 2 * huaNum)
    {
        return 0;
    }
    // 尝试删除大数
    // 必胜的话没有平局,也就是一定要大于
    int j = 0;
    for (int i = k; i < n; ++i)
    {
        // 3 2 1
        huaNum += a[i];
        huaNum -= a[j++];
        // cout << huaNum << ' ';

        deleteNum += a[i];
        // if(totalSum - deleteNum)
        if (totalSum - deleteNum > huaNum)
        {
            // return i + 1 - k;
            return j;
        }
    }

    return -1;
}
int main()
{
    scanf("%d %d", &n, &k);
    vector<int> a(n);
    for (int i = 0; i < n; i++)
    {
        scanf("%d", &a[i]);
        totalSum += a[i];
    }
    cout << solve(a);
    // for (int i : a)
    //     printf("%d ", i);
}
```

----

**2:**

为了求解这个问题，我们可以通过遍历所有可能的值来找到满足条件的 `(a, p, k)` 组合。这个过程需要考虑到等比数列的和的公式，并逐步检查每种可能的情况。

等比数列和的公式： 𝑆=𝑎𝑝𝑘−1𝑝−1*S*=*a**p*−1*p**k*−1

其中：

- 𝑆*S* 是等比数列的和。
- 𝑎*a* 是首项。
- 𝑝*p* 是公比。
- 𝑘*k* 是项数。

我们需要确保公式中的所有变量都满足整数条件，并且计算出的和等于给定的 𝑆*S*。

下面是一个 C++ 程序来实现这个要求：

```cpp
cpp复制代码#include <iostream>
#include <vector>
#include <cmath>
#include <algorithm>

using namespace std;

vector<tuple<int, int, int>> findGeometricSeries(int S) {
    vector<tuple<int, int, int>> solutions;

    for (int k = 2; ; ++k) {
        bool found = false;
        for (int p = 2; ; ++p) {
            long long pk_minus_1 = pow(p, k) - 1;
            if (pk_minus_1 > S * (p - 1)) break;
            if (S * (p - 1) % pk_minus_1 == 0) {
                int a = S * (p - 1) / pk_minus_1;
                solutions.emplace_back(a, p, k);
                found = true;
            }
        }
        if (!found) break;
    }

    sort(solutions.begin(), solutions.end());
    return solutions;
}

int main() {
    int S;
    cin >> S;

    vector<tuple<int, int, int>> solutions = findGeometricSeries(S);
    cout << "Case " << solutions.size() << ":" << endl;
    for (const auto& [a, p, k] : solutions) {
        cout << a << " " << p << " " << k << endl;
    }

    return 0;
}
```

### 程序解释

1. **输入**：程序从标准输入读取一个整数 𝑆*S*，即等比数列的和。

2. 查找解

   ：

   - 使用嵌套循环遍历所有可能的 𝑘*k* 和 𝑝*p*。
   - 对于每个 𝑘*k*，从 𝑝=2*p*=2 开始，计算 𝑝𝑘−1*p**k*−1。
   - 如果 𝑝𝑘−1*p**k*−1 超过了 𝑆×(𝑝−1)*S*×(*p*−1)，则停止增加 𝑝*p*。
   - 检查 𝑆×(𝑝−1)*S*×(*p*−1) 是否能被 𝑝𝑘−1*p**k*−1 整除，如果可以，则 𝑎=𝑆×(𝑝−1)𝑝𝑘−1*a*=*p**k*−1*S*×(*p*−1)，并且 𝑎,𝑝,𝑘*a*,*p*,*k* 组成一个有效解。
   - 将找到的解存入 `solutions` 容器中。

3. 输出解

   ：

   - 对找到的解按照 𝑎,𝑝,𝑘*a*,*p*,*k* 进行排序。
   - 打印解的数量和每个解。

这个程序首先找到所有可能的解，然后按要求排序并输出结果。这样可以确保所有满足条件的解都被找出，并且按照字典序输出。

----

4.

```cpp
#include <iostream>
#include <vector>
#include <cmath>

struct TreeNode
{
    int value;
    int color; // 0 for black, 1 for white
    TreeNode *left;
    TreeNode *right;
    TreeNode(int val, int col) : value(val), color(col), left(nullptr), right(nullptr) {}
};

TreeNode *insert(TreeNode *root, int value, int color)
{
    if (root == nullptr)
    {
        return new TreeNode(value, color);
    }
    if (value < root->value)
    {
        root->left = insert(root->left, value, color);
    }
    else
    {
        root->right = insert(root->right, value, color);
    }
    return root;
}

std::pair<int, int> countColors(TreeNode *root)
{
    if (root == nullptr)
    {
        return {0, 0};
    }
    auto leftColors = countColors(root->left);
    auto rightColors = countColors(root->right);
    int black = leftColors.first + rightColors.first + (root->color == 0 ? 1 : 0);
    int white = leftColors.second + rightColors.second + (root->color == 1 ? 1 : 0);
    return {black, white};
}

std::pair<bool, int> isBlackWhiteTree(TreeNode *root)
{
    if (root == nullptr)
    {
        return {true, 0};
    }
    auto colors = countColors(root);
    if (std::abs(colors.first - colors.second) > 1)
    {
        return {false, 0};
    }
    auto leftCheck = isBlackWhiteTree(root->left);
    auto rightCheck = isBlackWhiteTree(root->right);
    if (leftCheck.first && rightCheck.first)
    {
        return {true, leftCheck.second + rightCheck.second + 1};
    }
    return {false, 0};
}

int largestBlackWhiteTree(TreeNode *root)
{
    if (root == nullptr)
    {
        return 0;
    }
    auto check = isBlackWhiteTree(root);
    if (check.first)
    {
        return check.second;
    }
    return std::max(largestBlackWhiteTree(root->left), largestBlackWhiteTree(root->right));
}

void processCase(int n, const std::vector<std::pair<int, int>> &nodes)
{
    TreeNode *root = nullptr;
    for (const auto &node : nodes)
    {
        root = insert(root, node.first, node.second);
        int maxSize = largestBlackWhiteTree(root);
        std::cout << maxSize << std::endl;
    }
}

int main()
{
    int T;
    std::cin >> T;
    for (int t = 1; t <= T; ++t)
    {
        int n;
        std::cin >> n;
        std::vector<std::pair<int, int>> nodes(n);
        for (int i = 0; i < n; ++i)
        {
            std::cin >> nodes[i].first >> nodes[i].second;
        }
        std::cout << "case#" << t << ":" << std::endl;
        processCase(n, nodes);
    }
    return 0;
}

```

---

5.

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <algorithm>

using namespace std;

const int INF = 1e9;

struct Edge
{
    int to, weight;
};

vector<int> dijkstra(int n, vector<vector<Edge>> &graph, int start)
{
    vector<int> distances(n + 1, INF);
    distances[start] = 0;
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
    pq.push({0, start});

    while (!pq.empty())
    {
        int current_distance = pq.top().first;
        int current_node = pq.top().second;
        pq.pop();

        if (current_distance > distances[current_node])
        {
            continue;
        }

        for (const auto &edge : graph[current_node])
        {
            int neighbor = edge.to;
            int weight = edge.weight;
            int distance = current_distance + weight;

            if (distance < distances[neighbor])
            {
                distances[neighbor] = distance;
                pq.push({distance, neighbor});
            }
        }
    }

    return distances;
}

void find_best_drop_off(int n, int m, int a, int b, vector<vector<Edge>> &graph)
{
    vector<int> distances_from_1 = dijkstra(n, graph, 1);
    vector<int> distances_from_a = dijkstra(n, graph, a);
    vector<int> distances_from_b = dijkstra(n, graph, b);

    int min_distance = INF;
    vector<pair<int, int>> results;

    for (int i = 1; i <= n; ++i)
    {
        if (distances_from_1[i] + distances_from_a[i] == distances_from_1[a])
        { // Valid drop-off point
            if (distances_from_b[i] < min_distance)
            {
                min_distance = distances_from_b[i];
                results = {{i, distances_from_1[i]}};
            }
            else if (distances_from_b[i] == min_distance)
            {
                results.push_back({i, distances_from_1[i]});
            }
        }
    }

    sort(results.begin(), results.end(), [](const pair<int, int> &a, const pair<int, int> &b)
         { return a.second != b.second ? a.second < b.second : a.first < b.first; });

    cout << min_distance << endl;
    for (const auto &result : results)
    {
        cout << result.first << " " << result.second << endl;
    }
}

int main()
{
    int n, m, a, b;
    cin >> n >> m >> a >> b;

    vector<vector<Edge>> graph(n + 1);
    for (int i = 0; i < m; ++i)
    {
        int u, v, w;
        cin >> u >> v >> w;
        graph[u].push_back({v, w});
        graph[v].push_back({u, w});
    }

    find_best_drop_off(n, m, a, b, graph);

    return 0;
}

```

