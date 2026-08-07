---
title: "One vector to rule them all: Key recovery from one vector in UOV schemes"
description: ""
slug: uov-key-recovery-one-vector
date: 2025-07-27 00:00:00+0000
categories:
    - 密码学
    - 论文阅读
tags:
    - crypto
    - paper
math: true
---

## 概述：
给定的有 \(G = [G₁, ..., Gₘ]\)：一组 **m 个二次型**，构成 UOV 公钥。 \(v ∈ O\)，一个属于秘密子空间 O 的非零向量。
我们根据 \(v\)找出秘密子空间O的完整基，即恢复了整个等效私钥。

接下来我会一边解释论文中提到的步骤，一边逐行讲解对应代码实现。
## 核心攻击
论文的核心攻击基于以下引理：
![](https://github.com/komiko0831/picx-images-hosting/raw/master/image-(2).1hsindxry9.png)
它表明，给定油子空间 O 中的一个非零向量 \(\mathbf{x}\)，我们可以构造一个线性系统 \(J(\mathbf{x})\)，其核 \(K(\mathbf{x}) = \text{ker}(J(\mathbf{x}))\) 包含 O。
###  \(J(\mathbf{x})\)
雅可比矩阵，学过高数的大家应该都对它比较熟悉，
> 对多个多项式 \(f_i(x)\)，它们在某点 \(v\) 的偏导数组成了雅可比矩阵：
$$
J(v)=\left[\frac{\partial f_i}{\partial x_j}(v)\right]_{i,j}
$$
对于一个二次型 \(f_i(x) = x^T G_i x\)，我们有：
$$
\frac{\partial f_i}{\partial x_j}=2(G_ix)_j\quad\Rightarrow\quad\nabla f_i(x)=2G_ix
$$
所以：
$$
J(v)=\begin{bmatrix}v^TG_1\\\vdots\\v^TG_m\end{bmatrix}\in\mathbb{F}_q^{m\times n}
$$
### 全迷向子空间
设 \(f\) 是一个二次型，即 \(f(x) = x^T G x\)。所有向量 x 满足 f(x)=0，即满足：子空间中每个向量都在该二次型下为 0，且向量两两之间也“互相正交”
也就是UOV中的油变量子空间 \(O\).
### 证明：
![](https://github.com/komiko0831/picx-images-hosting/raw/master/image-(3).175ou8ivml.webp)
\(x\in\mathcal{O},\:z\in\mathcal{O}\)
\(\mathcal{O}\)是全迷向的：
$$
\boldsymbol{x}^TG_i\boldsymbol{z}=0\quad\forall i
$$
所以
$$
J(\boldsymbol{x})\cdot\boldsymbol{z}=\begin{bmatrix}\boldsymbol{x}^TG_1\boldsymbol{z}\\\varvdots\\\boldsymbol{x}^TG_m\boldsymbol{z}\end{bmatrix}=\boldsymbol{0}\Rightarrow\boldsymbol{z}\in\ker(J(\boldsymbol{x}))
$$
得证
$$
\mathcal{O}\subset\ker(J(\boldsymbol{x}))
$$
## Step1:构造 J(x) 矩阵
```Python
J = matrix([v*g for g in G])
```

对每个二次型 \(g\in G\)，我们左乘向量 \(v\)（即 \(v^T * G_i\)）
## Step2:求解 J 的核 K(v)
```Python
B = matrix(J.right_kernel().basis())
```

若 \(K(v)\) 的维度是 \(r = n-m\)，说明我们已经将搜索空间从 \(F_q^n\) 缩小到了一个仅维度为 \(n-m\)的子空间。
## Step3:将 G 限制到 K(v) 中
```Python
for g in G :
    ghat = B*g*B.transpose()
```
原本函数是作用在 n 维变量上的，现在变成作用在 n-m 维的变量上
## Step 4：处理特征域为 2 
```Python
if charac == 2 :
    ghat = ghat + ghat.transpose()
```
在特征为 2 的有限域中，二次型矩阵不是自动对称的，需要手动对称化。
 \(x^T A x = x^T (A + A^T) x\)（当 \(A\) 不是对称矩阵时）
## Step 5：寻找公共核空间中的向量
```Python
for b in ghat.kernel().basis() :
    if len(B2) == 0 or b not in span(B2) :
        B2.append(b)
    if len(B2) == m :
        break
```
遍历每个 `ghat` 的核中的向量 `b`
每个 `ker(ghat)` 是一个子空间，理想情况下它们的交集就是秘密子空间 O 在 \(K(v)\) 中的表示。由于 \(G_1,G_2,\cdots,G_m\) 都在 O 上全各向同性，因此这些核交集会包含 O。
$$
\hat{O}=\bigcap_{i=1}^m\ker(\hat{G}_i)
$$
表现为：
如果 `b` 没有包含在已有集合 `B2` 的线性张成空间中，就添加进去。直到找到m哥线性无关的向量为止
## Step 6：从子空间升回原空间
```Python
B3 = matrix(B2)
C = B3*B
```
完整代码如下：
```Python
def attack(G,v) :
    """ 
    This function is the one that completes the attack.

    Given G a set of m quadratic forms admitting a common totally isotropic subspace O
    of dimension at least (n-m)/2, and v in O, find a basis of O as a whole.
    """
    m = len(G) 
    J = matrix([v*g for g in G]) 
    B = matrix(J.right_kernel().basis())
    B2 = []
    charac = G[0].base_ring().characteristic()
    for g in G :
        ghat = B*g*B.transpose()  #restriction of G to the kernel of J
       	if charac == 2 :
            ghat = ghat + ghat.transpose()
        for b in ghat.kernel().basis() :
            if len(B2) == 0 or b not in span(B2) :
                B2.append(b)
        if len(B2) == m :
            break
    B3 = matrix(B2)
        
    C = B3*B
    return C
```
