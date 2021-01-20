---
layout: post
title: Disjoint LIS 杨表
date: 2021-01-21 01:00:32
categories: 算法
tags: 杨表 LIS
typora-root-url: ..
mathjax: true
---

* content
{:toc}
### 题意分析

计数这样的n的排列: 定义L为LIS长度, 此排列存在两个不相交的LIS(长度也为L)

由Diworth, 排列的特性转化为: 有这样一个子序列: 长度为2L, LDS=2.

我们定义k-LDS为LDS长度不超过k的最长子序列, 利用杨表解决其计数问题.

<!-- more -->

### 杨氏矩阵

#### 插入 S←x

1. 在当前行寻找最小的比x大的数y.
2. 成功: x替换y, 并让y在下一行重复(1).
3. 失败: x放在当前行的末尾.

- 记录插入的位置所对应的插入时间.

#### 删除 (i, j)

1. 在上一行寻找最大的比x小的数y.
2. 用x替换y, 并让y重复操作(1)直到第一行.

#### 重构

通过上述的插入算法, 我们构建了两个杨表.

$P_X$为插入得到的表, $Q_X$为记录表.

之后依据$Q_X$对$P_X$执行删除操作就可以重构回序列.

这也说明了两个杨表与一个排列是一一对应的关系.

#### 性质

1. 杨表第一行的长度为LIS的长度.
2. 将序列转置后$P_X$也转置.($Q_X$不一定)
3. 杨表第一列的长度为LDS的长度.
4. 杨表前k行的列数和为k-LDS的长度.
5. 杨表前k列的行数和为k-LIS的长度.

#### 计数

$$
\begin{aligned}
&(shape)f_{shape}={n! \over \prod H(i, j)}\\
&(sum)f_n=f_{n-1}+(n-1)f_{n-2}
\end{aligned}
$$

### 具体思路

使用Dfs构造出来前两行的长度相同的杨表.

预处理好阶乘和逆元, 都可以$O(n)$.

计算好行数和列数, 计算出Hook函数的值.

利用钩子计数公式累加答案.

### Code

```c++
#include<iostream>
#include<cstdio>
#include<algorithm>
typedef long long LL;

const int N=80, M=998244353;
int n, rows_by_col[N], cols_by_row[N];
LL inv[N], fac[N], ans;

void Pre()
{
    fac[1]=inv[1]=1;
    for(int i=2; i<=n; i++)
    {
        inv[i]=(M-M/i)*inv[M%i]%M;
        fac[i]=fac[i-1]*i%M;
    }
} //preprocess fac and inv

inline int hook(int row, int col)
{
    return rows_by_col[col]+cols_by_row[row]-row-col+1;
} //value of hook in young table

void Calc(int row)
{
    for(int i=1; i<=cols_by_row[1]; i++)
        rows_by_col[i]=0;
    for(int i=1; i<=row; i++)
        rows_by_col[cols_by_row[i]]++;
    for(int i=n-1; i>=1; i--)
        rows_by_col[i]+=rows_by_col[i+1];
  //use dif to calculate rows_by_cal
    LL sum=fac[n];
    for(int i=1; i<=row; i++)
        for(int j=1; j<=cols_by_row[i]; j++)
            sum=sum*inv[hook(i, j)]%M;
    ans=(ans+sum*sum)%M;
} //n!/prod(hook(i, j))

void Dfs(int row, int cols_pre, int left)
{
    if(left==0) Calc(row-1);
    int enum_lim=std::min(left, cols_pre);
    for(int i=1; i<=enum_lim; i++)
    {
        cols_by_row[row]=i;
        Dfs(row+1, i, left-i);
    }
} //enumerate legal form

void Test()
{
    freopen("temp\\in.txt", "r", stdin);
}
int main()
{
    // Test();
    scanf("%d", &n);
    Pre();
    for(int i=1; i+i<=n; i++)
    { 
        cols_by_row[1]=cols_by_row[2]=i;
        Dfs(3, i, n-i-i);
    } //i represent L
    std::cout<<ans;

    return 0;
}
```

