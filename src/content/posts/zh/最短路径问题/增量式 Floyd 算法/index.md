---
title: '增量式 Floyd 算法'
description: '增量式 Floyd 在标准 Floyd 基础上，按顺序逐步加入中间节点 k，动态更新任意两点间最短路径，支持节点逐个新增的场景；它复用之前已算好的路径结果，不必每次从头完整重跑，适合图中节点不断增加的问题，时间复杂度仍为\(O(n^3)\)。'
pubDate: 1962-06
tags: [图, 动态规划]
categories: [基础算法, 最短路径问题]
heroImage: './Floyd-Warshall cover.webp'
heroImageAlt: 'Floyd‑Warshall 算法扁平化科普插画，对比原始网络与处理后的路径'
---

**增量式 Floyd**（Incremental Floyd）是一种在图结构发生动态变化（如**插入新节点**或**降低边权**）时更新全源最短路的算法。

标准 Floyd 算法的时间复杂度为 $O(N^3)$。当图中动态新增一个节点或一条边时，若重新运行 Floyd 算法开销极大；增量式 Floyd 利用原有的最短路状态，仅需 **$O(N^2)$** 的时间即可完成更新。

## 核心原理

标准 Floyd 的动态规划状态转移方程为：

$$d_{k}[i][j] = \min\Big(d_{k-1}[i][j],\, d_{k-1}[i][k] + d_{k-1}[k][j]\Big)$$

最外层循环 $k$ 的本质是**逐步将节点 $k$ 加入到“允许作为中间过渡节点的集合”中**。

增量式 Floyd 正是基于这一特性：如果现有的最短路矩阵 $d$ 已经包含了前 $N$ 个节点之间的最优路径，当加入第 $N+1$ 个节点 $u$ 时，旧节点对之间的最优路径要么保持不变，要么需要经过 $u$ 进行松弛。

## 更新步骤

### 动态新增节点 $u$

假设原图已有 $N$ 个节点（编号 $0 \dots N-1$），现新增节点 $u$（编号为 $N$）。更新过程必须严格按照以下 **“先四周，后中心”** 的顺序执行：

**1.边界初始化：** 建立直连关系。

设置新节点到自身的距离为 0，并根据新输入的边，初始化旧节点与新节点之间的直连边权（无直连则赋值为 $\infty$）：

$$d[u][u] = 0$$

$$d[i][u] = \text{weight}(i \to u), \quad d[u][i] = \text{weight}(u \to i) \quad (\text{其中 } 0 \le i < N)$$

**2.更新「旧节点 $\to$ 新节点」的最短路：** 利用旧图向内汇聚（更新第 u 列）。

遍历所有旧节点 $i$，利用另一个旧节点 $k$ 作为中间跳板，松弛到新节点 $u$ 的距离：

$$d[i][u] = \min_{0 \le k < N} \big(d[i][k] + d[k][u]\big) \quad (\text{其中 } 0 \le i < N)$$

> **原理**：从旧节点 $i$ 走到 $u$，必然是先在旧图内部走一段最优路径 $d[i][k]$，再通过直连边 $d[k][u]$ 跨越到 $u$。

**3.更新「新节点 $\to$ 旧节点」的最短路：** 利用旧图向外扩散（更新第 u 行）。

遍历所有旧节点 $j$，利用旧节点 $k$ 作为中间跳板，松弛从新节点 $u$ 出发的距离：

$$d[u][j] = \min_{0 \le k < N} \big(d[u][k] + d[k][j]\big) \quad (\text{其中 } 0 \le j < N)$$

> **注意**：Step 2 与 Step 3 互不依赖（分别更新行列），但都必须在 Step 4 之前完成。

**4.全图同步松弛：** 利用新节点 u 辐射全图。

当新节点 $u$ 自身的行列距离都已达到最优后，将其作为全图的“新跳板”，松弛所有已有的旧节点对 $(i, j)$：

$$d[i][j] = \min\big(d[i][j],\, d[i][u] + d[u][j]\big) \quad (\text{其中 } 0 \le i, j < N)$$

> **重要依赖**：此步必须用 Step 2 和 Step 3 算出的**最终最短路** $d[i][u]$ 和 $d[u][j]$ 进行计算，否则会漏掉复合路径。

> 💡 **时间复杂度**：Step 1 为 $O(N)$，Step 2、3、4 均为双重循环 $O(N^2)$。因此，单次插入新节点的整体更新复杂度为 **$O(N^2)$**。

### 动态新增/强化单条边（边权变更）

当在已有图上新增一条边，或降低某条已有边的权重 $(x \to y)$ 且新权值 $w < d[x][y]$ 时：

1. **更新直接距离**：

   $$d[x][y] = w$$

2. **全图同步松弛**：

   遍历全图所有的节点对 $(i, j)$，检查它们如果强行绕道这条新边 $(x \to y)$，路径是否会变短：

   $$d[i][j] = \min\big(d[i][j],\, d[i][x] + w + d[y][j]\big) \quad (\text{其中 } 0 \le i, j < N)$$

> 💡 **时间复杂度**：只需双重循环遍历所有 $(i, j)$ 节点对，更新复杂度同样为 **$O(N^2)$**。

## 参考代码

```c++
#include <vector>
#include <algorithm>

const int INF = 1e9;

class IncrementalFloyd {
private:
    int n; // 当前节点数
    std::vector<std::vector<int>> d; // 距离矩阵

public:
    IncrementalFloyd(int max_nodes) : n(0) {
        d.assign(max_nodes, std::vector<int>(max_nodes, INF));
    }

    // 新增节点 u，传入旧节点到 u 的边权 in_weight，以及 u 到旧节点的边权 out_weight
    void add_node(const std::vector<int>& in_weight, const std::vector<int>& out_weight) {
        int u = n;
        d[u][u] = 0;

        // Step 1: 初始化直接边
        for (int i = 0; i < n; ++i) {
            d[i][u] = in_weight[i];
            d[u][i] = out_weight[i];
        }

        // Step 2: 更新 旧节点 -> 新节点 的最短路
        for (int i = 0; i < n; ++i) {
            for (int k = 0; k < n; ++k) {
                if (d[i][k] != INF && d[k][u] != INF) {
                    d[i][u] = std::min(d[i][u], d[i][k] + d[k][u]);
                }
            }
        }

        // Step 3: 更新 新节点 -> 旧节点 的最短路
        for (int j = 0; j < n; ++j) {
            for (int k = 0; k < n; ++k) {
                if (d[u][k] != INF && d[k][j] != INF) {
                    d[u][j] = std::min(d[u][j], d[u][k] + d[k][j]);
                }
            }
        }

        // Step 4: 将新节点 u 作为中间过渡节点松弛所有已有点对
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < n; ++j) {
                if (d[i][u] != INF && d[u][j] != INF) {
                    d[i][j] = std::min(d[i][j], d[i][u] + d[u][j]);
                }
            }
        }

        n++; // 扩充节点集合
    }

    // 动态添加/更新单条边 (x -> y，权值为 w)
    void update_edge(int x, int y, int w) {
        if (w >= d[x][y]) return;
        d[x][y] = w;
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < n; ++j) {
                if (d[i][x] != INF && d[y][j] != INF) {
                    d[i][j] = std::min(d[i][j], d[i][x] + w + d[y][j]);
                }
            }
        }
    }

    int get_dist(int u, int v) const { return d[u][v]; }
};
```

## 经典应用场景

1. **动态图在线查询**：网络节点或链路逐步开启、扩展，需实时回答全源最短路径问题。
2. **离线倒序处理（删点/删边问题）**：
   - 当题目要求“动态删除节点”并查询全源最短路时，直接删点维护最短路极其困难。
   - **常用技巧**：离线记录所有操作，**倒序**将“删点”转化为“向图中逐步插入节点”。每次插入节点使用增量式 Floyd，将总体时间复杂度控制在 $O(N^3)$。
3. **传递闭包（Transitive Closure）维护**：结合 `std::bitset` 将 `min/+` 操作替换为位运算，能在 $O(N^2 / 64)$ 的时间内维护点之间的动态连通性。