---
title: 斐波那契数列
description: '斐波那契数列指从 0、1 开始，后续每一项都等于前两项之和的数列，即 0，1，1，2，3，5，8……，它频繁出现在植物生长、自然形态中，同时在算法、递归、黄金比例相关问题里有着广泛应用。'
pubDate: '1202-01-01'
tags: [递归, 动态规划]
categories: [基础算法]
heroImage: './斐波那契递归调用树.png'
heroImageAlt: '斐波那契数列递归调用树，计算 fib (6) 时，大量相同子问题被反复重复计算。'
---

**斐波那契数列（Fibonacci sequence）** 是数学和计算机科学中最著名的数列之一。它的定义非常简单，但蕴含着丰富的数学规律，并且在算法设计中是演示递归、动态规划与数学优化的绝佳案例。

## 什么是斐波那契数列？

斐波那契数列由 0 和 1 开始，后面的每一项都是前两项之和。

- **前几项**：$0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, \dots$

- **数学递推公式**：

  $$F(0) = 0$$

  $$F(1) = 1$$

  $$F(n) = F(n-1) + F(n-2) \quad (n \ge 2)$$

> **黄金分割点**：当 $n$ 趋近于无穷大时，相邻两项的比值 $\frac{F(n)}{F(n-1)}$ 会无限趋近于黄金分割率 $\phi \approx 1.6180339887...$。

## 算法思维的演化：从直觉到动态规划

求解斐波那契数列的过程，完美体现了计算机科学中“如何将一个指数级开销的问题优化到线性乃至常数级”的思考脉络。

### 朴素递归：自顶向下的“直觉解法”

朴素递归直接遵照数学定义。它假设子问题已经被解决，通过将大问题 $F(n)$ 拆解为两个更小的子问题 $F(n-1)$ 和 $F(n-2)$，不断向边界条件（$F(0)=0, F(1)=1$）靠拢。

```c++
int fib(int n) {
    if (n <= 0) return 0;
    if (n == 1) return 1;
    return fib(n - 1) + fib(n - 2); // 递归拆分
}
```

虽然代码极简，但其背后的计算过程会展开为一棵巨大的**递归树**：

```
                 fib(5)
               /        \
          fib(4)        fib(3)
          /    \        /    \
      fib(3)  fib(2)  fib(2) fib(1)
      /   \
  fib(2) fib(1)
```

可以看到，`fib(3)` 被计算了 2 次，`fib(2)` 被计算了 3 次。随着 $n$ 的增长，子问题的重复计算呈现**指数级暴涨**（节点总数达 $O(2^n)$）。这意味计算第 50 项就需要几十亿次计算，在工程上完全不可接受。

### 记忆化搜索：带备忘录的剪枝优化

既然重复计算导致了性能崩塌，一个极其自然的优化直觉随之诞生：**将已经计算过的子问题答案存起来**。

记忆化搜索（Memoization）保留了“自顶向下”拆解问题的结构，但在递归前增加了一道“缓存检查”：

1. 每次求解子问题前，先查询“备忘录”（数组或哈希表）；
2. 若已命中，**直接返回**，避免向下递归；
3. 若未命中，计算后**先存入备忘录**再返回。

```c++
#include <vector>

int fibMemo(int n, std::vector<int>& memo) {
    if (n <= 0) return 0;
    if (n == 1) return 1;
    
    // 1. 检查备忘录（已计算则直接返回）
    if (memo[n] != -1) return memo[n];
    
    // 2. 求解并将结果写入备忘录
    memo[n] = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
    return memo[n];
}

int fib(int n) {
    std::vector<int> memo(n + 1, -1); // 初始化备忘录为 -1
    return fibMemo(n, memo);
}
```

引入备忘录后，原本庞大的**递归树**被剪枝压缩为一条单向**递推链**。时间复杂度直接从指数级的 $O(2^n)$ 暴降至线性级的 **$O(n)$**。

### 动态规划：自底向上的状态转移

记忆化搜索虽然解决了时间效率问题，但仍依赖**函数递归**，存在隐性的**系统调用栈开销**（$n$ 极大时可能引发栈溢出）。

既然我们知道求解 $F(n)$ 必须依赖 $F(0)$ 到 $F(n-1)$ 的全部已知结果，**为何不直接从最基础的已知条件出发，自底向上递推？** 这正是动态规划（Dynamic Programming, DP）的核心范式：

1. **定义状态**：$dp[i]$ 表示第 $i$ 个斐波那契数；
2. **确定边界**：$dp[0] = 0, dp[1] = 1$；
3. **状态转移方程**：$dp[i] = dp[i-1] + dp[i-2]$。

```c++
#include <vector>

int fibDP(int n) {
    if (n <= 0) return 0;
    if (n == 1) return 1;
    
    std::vector<int> dp(n + 1);
    dp[0] = 0;
    dp[1] = 1;
    
    // 自底向上递推
    for (int i = 2; i <= n; ++i) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }
    return dp[n];
}
```

观察转移方程 $dp[i] = dp[i-1] + dp[i-2]$ 可以发现：计算第 $i$ 项时，**仅需要紧挨着的两项**，更早的历史数据完全可以丢弃。

我们无需维护整个 DP 数组，仅需两个临时变量交替更新即可，这就是**滚动变量优化**：

```c++
int fibOptimized(int n) {
    if (n <= 0) return 0;
    if (n == 1) return 1;
    
    int prev2 = 0; // dp[i-2]
    int prev1 = 1; // dp[i-1]
    int curr = 0;  // dp[i]
    
    for (int i = 2; i <= n; ++i) {
        curr = prev1 + prev2;
        prev2 = prev1; // 滚动向前更新
        prev1 = curr;
    }
    return curr;
}
```

此时，空间复杂度被进一步压缩至常数级 **$O(1)$**，达到了时间和空间的极佳平衡。

## 跨越线性瓶颈：高阶数学优化

当 $n$ 达到上亿甚至更大（如 $n = 10^{18}$）时，线性的 $O(n)$ 循环依然太慢。此时需要借助数学工具实现跨越。

### 矩阵快速幂法

借助线性代数，斐波那契递推式可以改写为矩阵乘法形式：

$$\begin{pmatrix} F(n) \\ F(n-1) \end{pmatrix} = \begin{pmatrix} 1 & 1 \\ 1 & 0 \end{pmatrix}^{n-1} \begin{pmatrix} 1 \\ 0 \end{pmatrix}$$

将求解第 $n$ 项的问题转化为求矩阵的 $n-1$ 次幂。结合**快速幂算法**（二分思想），可以在对数时间内得出结果：

```c++
#include <vector>

using Matrix = std::vector<std::vector<long long>>;

Matrix multiply(const Matrix& A, const Matrix& B) {
    Matrix C = {{0, 0}, {0, 0}};
    for (int i = 0; i < 2; ++i) {
        for (int j = 0; j < 2; ++j) {
            for (int k = 0; k < 2; ++k) {
                C[i][j] += A[i][k] * B[k][j];
            }
        }
    }
    return C;
}

long long fibonacci_matrix(int n) {
    if (n <= 0) return 0;
    if (n == 1) return 1;

    Matrix result = {{1, 0}, {0, 1}}; // 单位矩阵
    Matrix base = {{1, 1}, {1, 0}};

    int p = n - 1;
    while (p > 0) {
        if (p & 1) result = multiply(result, base);
        base = multiply(base, base);
        p >>= 1;
    }
    return result[0][0];
}
```

- **时间复杂度**：$O(\log n)$ —— 对数级耗时，面对超大数值时性能极强。
- **空间复杂度**：$O(1)$。

### 通项公式法

使用闭式解通项公式直接求值：

$$F(n) = \frac{1}{\sqrt{5}} \left[ \left(\frac{1+\sqrt{5}}{2}\right)^n - \left(\frac{1-\sqrt{5}}{2}\right)^n \right]$$

```c++
#include <cmath>

long long fibonacci_formula(int n) {
    double sqrt5 = std::sqrt(5.0);
    double phi = (1.0 + sqrt5) / 2.0;
    return std::round(std::pow(phi, n) / sqrt5);
}
```

- **时间复杂度**：$O(1)$ （取决于 `pow` 的底层实现）。
- **致命局限**：受限于计算机浮点数精度（`double` 溢出与有效位数缺失），当 $n$ 稍大时结果就会失真，通常仅用于数学理论推导或小范围近似计算。