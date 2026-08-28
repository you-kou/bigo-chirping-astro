---
title: 子图最短路
description: '给定带权无向图，对每个点编号区间\([\ell,r]\)，取仅包含区间内点的导出子图；子图中点对不连通距离视为 0，求所有区间内全部\(u\le v\)点对的子图最短路总和，对\(10^9\)取模，\(n\le100\)。'
pubDate: 2026-03-15
tags: [动态规划, 增量式 Floyd 算法]
categories: [GESP]
---

> 给定包含 $n$ 个结点 $m$ 条边的**带权无向图** $G$，结点依次以 $1, 2, \dots, n$ 编号。第 $i$ ($1 \le i \le m$) 条边连接编号为 $u_i$ 与 $v_i$ 的两个结点，权值为 $w_i$。
>
> 对于指定的 $1 \le \ell \le r \le n$，按以下方式构造图 $G$ 的子图 $G(\ell, r)$：
>
> - 保留 $G$ 中编号在区间 $[\ell, r]$ 中的结点。删去其它编号不在 $[\ell, r]$ 中的结点以及与之相连的边。剩余的结点和边构成子图 $G(\ell, r)$。
>
> 对于 $G(\ell, r)$ 中的任意结点 $u, v$ 应有 $\ell \le u, v \le r$。记 $u, v$ 在子图 $G(\ell, r)$ 上的最短距离为 $d(\ell, r, u, v)$。特殊地，若 $u, v$ 在子图 $G(\ell, r)$ 上不连通，则认为 $d(\ell, r, u, v) = 0$。
>
> 你需要求出 $\sum_{\ell=1}^{n} \sum_{r=\ell}^{n} \sum_{u=\ell}^{r} \sum_{v=u}^{r} d(\ell, r, u, v)$ 对 $10^9$ 取模的结果。
>
> - 题目中的英文字母 $l$ 使用了特殊写法 $\ell$，以避免英文字母 $l$ 与数字 $1$ 混淆。
>
> **输入格式**
>
> 第一行，两个正整数 $n, m$，表示结点数与边数。
>
> 接下来 $m$ 行，第 $i$ ($1 \le i \le m$) 行包含三个正整数 $u_i, v_i, w_i$，表示一条连接结点 $u_i, v_i$ 的权值为 $w_i$ 的边。
>
> **输出格式**
>
> 输出一行，一个整数，表示 $\sum_{\ell=1}^{n} \sum_{r=\ell}^{n} \sum_{u=\ell}^{r} \sum_{v=u}^{r} d(\ell, r, u, v)$ 对 $10^9$ 取模的结果。
>
> **样例**
>
> **输入样例 1**
>
> ```
> 3 2
> 1 2 1
> 2 3 2
> ```
>
> **输出样例 1**
>
> ```
> 9
> ```
>
> **输入样例 2**
>
> ```
> 4 6
> 1 2 100
> 2 3 100
> 3 4 100
> 1 3 10
> 2 4 10
> 1 4 1
> ```
>
> **输出样例 2**
>
> ```
> 784
> ```
>
> **数据范围**
>
> 对于 $40\%$ 的测试点，保证 $2 \le n \le 20$。
>
> 对于所有测试点，保证 $2 \le n \le 100$，$2 \le m \le \frac{n(n-1)}{2}$，$1 \le u_i, v_i \le n$，$1 \le w_i \le 10^6$。图中可能存在重边。

题目要求对所有可能的区间 $[\ell, r]$，求出只保留编号在 $[\ell, r]$ 内的结点时，该子图中所有点对 $(u, v)$ 的最短路之和。

如果对每个区间 $[\ell, r]$ 都重新跑一遍 Floyd 算法：

- 区间个数有 $O(n^2)$ 个。
- 每个区间跑 Floyd 的复杂度是 $O((r-\ell+1)^3) = O(n^3)$。
- 总时间复杂度为 $O(n^5)$，对于 $n = 100$ 来说大约需要 $10^{10}$ 次运算，显然会超时。

## 核心优化思路：增量式 Floyd

我们可以**固定左端点 $\ell$**，然后让右端点 $r$ 从 $\ell$ 开始一步步向右扩展到 $n$。

当右端点从 $r-1$ 变成 $r$ 时，相当于在原有的子图 $G(\ell, r-1)$ 中**新加入了一个结点 $r$ 以及与 $r$ 相连的边**。此时，我们可以用 $O((r-\ell)^2)$ 的时间来更新所有的最短路，而不是从头计算：

1. **计算新节点 $r$ 到已有节点 $u \in [\ell, r-1]$ 的最短距离：**

   因为 $r$ 是新加入的端点，任何从 $u$ 到 $r$ 的最短路径，其最后一个中间节点必然是 $[\ell, r-1]$ 中的某个节点 $v$。因此：

   $$dist[u][r] = \min_{v \in [\ell, r-1]} \{ dist[u][v] + adj[v][r] \}$$

   *(其中 $adj[v][r]$ 为 $v$ 和 $r$ 之间的直接边权)*

2. **用新节点 $r$ 作为“中间节点”更新已有节点对之间的最短距离：**

   现在 $r$ 也可以作为中间跳板了，传统的 Floyd 状态转移方程适用：

   $$dist[u][v] = \min(dist[u][v], dist[u][r] + dist[r][v])$$

3. **统计答案：**

   更新完当前的 $dist$ 数组后，将当前区间 $[\ell, r]$ 内所有点对的最短路累加到总答案中即可。

## 复杂度分析

对于每个固定的 $\ell$，让 $r$ 从 $\ell$ 循环到 $n$。每次更新的复杂度与当前区间长度 $k = r-\ell+1$ 成平方关系，即 $O(k^2)$。

总时间复杂度为：

$$\sum_{\ell=1}^{n} \sum_{r=\ell}^{n} O((r-\ell)^2) = O(n^4)$$

当 $n = 100$ 时，$n^4 = 10^8$，实际运行中循环内部常数极小，可以通过。

## 参考代码

```c++
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

const long long INF = 1e18; // 用足够大的数表示正无穷
const int MOD = 1e9;

// 邻接矩阵，用来存原图的直接边
long long adj[105][105];
// dist[u][v] 表示当前子图中 u 到 v 的最短距离
long long dist[105][105];

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n, m;
    if (!(cin >> n >> m)) return 0;

    // 初始化邻接矩阵
    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= n; ++j) {
            if (i == j) adj[i][j] = 0;
            else adj[i][j] = INF;
        }
    }

    // 读入边，注意可能有重边，取最小的权值
    for (int i = 0; i < m; ++i) {
        int u, v;
        long long w;
        cin >> u >> v >> w;
        adj[u][v] = min(adj[u][v], w);
        adj[v][u] = min(adj[v][u], w);
    }

    long long total_ans = 0;

    // 枚举左端点 l
    for (int l = 1; l <= n; ++l) {
        // 初始化当前 l 对应的距离矩阵
        // 因为后面要动态往里面加点，这里先设为 INF
        for (int i = 1; i <= n; ++i) {
            for (int j = 1; j <= n; ++j) {
                dist[i][j] = (i == j) ? 0 : INF;
            }
        }

        // 枚举右端点 r
        for (int r = l; r <= n; ++r) {
            if (r > l) {
                // 步骤 1: 计算已有点 u 到新点 r 的最短距离
                for (int u = l; u < r; ++u) {
                    long long min_d = adj[u][r]; // 或者是直接连边
                    for (int v = l; v < r; ++v) {
                        if (dist[u][v] != INF && adj[v][r] != INF) {
                            min_d = min(min_d, dist[u][v] + adj[v][r]);
                        }
                    }
                    dist[u][r] = dist[r][u] = min_d;
                }

                // 步骤 2: 利用新点 r 作为中间点，更新已有的点对 (u, v)
                for (int u = l; u < r; ++u) {
                    for (int v = l; v < r; ++v) {
                        if (dist[u][r] != INF && dist[r][v] != INF) {
                            dist[u][v] = min(dist[u][v], dist[u][r] + dist[r][v]);
                        }
                    }
                }
            }

            // 步骤 3: 累加当前区间 [l, r] 内所有点对的最短路
            // 题目要求 u <= v，且 u == v 时距离为 0，所以从 v = u + 1 开始循环即可
            for (int u = l; u <= r; ++u) {
                for (int v = u + 1; v <= r; ++v) {
                    if (dist[u][v] != INF) {
                        total_ans = (total_ans + dist[u][v]) % MOD;
                    }
                }
            }
        }
    }

    cout << total_ans << "\n";

    return 0;
}
```

## 避坑提示

1. **重边问题**：题目说明中提到“图中可能存在重边”，因此在读入边权建立 `adj` 矩阵时，必须使用 `min(adj[u][v], w)` 取最小值。
2. **不连通处理**：题目规定如果不连通则距离记为 $0$。在代码中，若 `dist[u][v] == INF` 则代表不连通，此时直接跳过不累加到答案中即可。
3. **数据类型**：虽然答案最后对 $10^9$ 取模，但在求最短路累加的过程中，路径长度可能会超过 `int` 的范围，因此图的边权、`dist` 数组以及计算过程建议全部使用 `long long`。
