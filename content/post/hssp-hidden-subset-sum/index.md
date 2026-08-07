---
title: "HSSP隐藏子集和问题"
description: ""
slug: hssp-hidden-subset-sum
date: 2025-02-16 00:00:00+0000
categories:
    - 密码学
tags:
    - crypto
    - note
math: true
---

学习SSP进阶之HSSP问题（Hidden Subset Sum  Problem），参考学习并复现【[A Polynomial-Time Algorithm for Solving the Hidden Subset Sum  Problem](https://eprint.iacr.org/2020/461.pdf)】和【[Provably Solving the Hidden Subset Sum Problem via Statistical  Learning](https://eprint.iacr.org/2021/1007.pdf)】这两篇论文。
## The hidden subset-sum problem
令 \(q\)是一个正整数，随机整数 \(\alpha_1,\cdots,\alpha_n\in\mathbb{Z}_q\)，设 \(\mathrm{x}_1,\ldots,\mathrm{x}_n\in\mathbb{Z}^m\)是分布在 \(\{0,1\}\)中的随机向量，设 \(h=(h_1,\cdots,h_m)\in\mathbb{Z}_m\)满足：
$$
\mathbf{h}=\alpha_1\mathbf{x}_1+\alpha_2\mathbf{x}_2+\cdots+\alpha_n\mathbf{x}_n\quad(\mathrm{mod}\:q)
$$
只给出 \(q\)和 \(h\)，我们需要恢复出 \(\alpha=(\alpha_{1},\ldots,\alpha_{n})\)和向量 \(x_i\)。
回想一下已知权重 \(\alpha_i\)的经典子集和问题，我们可以基于格基约简的算法在多项式时间内求解，但是对于隐藏权重 \(\alpha_i\)的情况下，显然原来的格攻击不再适用。 \(Nguyen 和Stern\)，提出了基于正交格的攻击算法：Nguyen-Stern 算法来有效解决新的隐藏子集和问题。
```Python
with open("flag.txt", "rb") as f:
    flag = int.from_bytes(f.read(), "big")

n = 70
m = flag.bit_length()
assert n < m
p = random_prime(2**512)


F = GF(p)
x = random_matrix(F, 1, n)
A = random_matrix(ZZ, n, m, x=0, y=2)
A[randint(0, n-1)] = vector(ZZ, Integer(flag).bits())
h = x*A

print(p)
print(list(h[0]))
```
## Background on lattices
### 格与向量空间
线性代数中的向量空间大家都不陌生，我们可以给出格的定义：一个**n 维格（Lattice）**是由**n**个线性无关的基向量的**整数线性组合**构成的集合：
$$
L=\left\{\sum_{i=1}^na_i\mathbf{v}_i\mid a_i\in\mathbb{Z}\right\}
$$
其中, \(\mathbf{v}_1,\mathbf{v}_2,...,\mathbf{v}_n\)是n维空间中的一组基向量
### 正交格
正交格是指**格**的一种特殊形式，其中基向量彼此正交。
#### 正交格的定义
正交格 \(L\)是由一组两辆正交的基向量 \(\mathbf{v}{1},\mathbf{v}{2},...,\mathbf{v}_{n}\)通过整数线性组合构成的集合。
$$
L=\left\{\sum_{i=1}^na_i\mathbf{v}_i\mid a_i\in\mathbb{Z}\right\}
$$
其中基向量满足正交条件: \(\mathbf{v}_i\cdot\mathbf{v}_j=0\quad(i\neq j)\).
#### 零空间（又称核(Kernel)）
给定一个 \(m\times n\)矩阵 \(A\),其零空间定义为：
$$
N(A)=\{\mathbf{x}\in\mathbb{R}^n\mid A\mathbf{x}=0\}
$$
零空间也就是指包含所有满足 \(A\mathbf{x}=0\)的向量 \(x\)。
直观来看， \(N(A)\) 是在列空间的“剩余”维度上存在的自由解空间。
因为行空间与零空间相互垂直，行空间与零空间可以张成整个 \(\mathbb{R}^n\)空间，所以**求一个格的正交格，就是求他的零空间（离散的）**。
#### 正交补关系
前面我们描述行空间与零空间相互垂直，在向量空间中，零空间 \(N(A)\)和行空间 \(R(A^T)\)互为正交补。
$$
N(A)=R(A^T)^\perp 
$$
任何属于零空间的向量都**垂直于**矩阵的行向量
#### 格基正交化的方法
我们学过的最常见的正交化方法是 **Gram-Schmidt 正交化**
1. 对于给定的格基 \(b1,b2,...,bn\),我们可以通过如下公式构造一个新的正交基 \(\mathbf{b}_1^*,\mathbf{b}_2^*,...,\mathbf{b}_n^*\)
$$
\mathbf{b}_{1}^{*}=\mathbf{b}_{1}\\\mathbf{b}_{2}^{*}=\mathbf{b}_{2}-\frac{\langle\mathbf{b}_{2},\mathbf{b}_{1}^{*}\rangle}{\|\mathbf{b}_{1}^{*}\|^{2}}\mathbf{b}_{1}^{*}\\\mathbf{b}_{3}^{*}=\mathbf{b}_{3}-\sum_{i=1}^{2}\frac{\langle\mathbf{b}_{3},\mathbf{b}_{i}^{*}\rangle}{\|\mathbf{b}_{i}^{*}\|^{2}}\mathbf{b}_{i}^{*}
$$
- 这个新基 \(b_i^*\)** **互相正交，但它们不一定仍然是整数向量因此这个新基通常不再是原格的基（因为格需要整数线性组合）
1. 而各类格基规约算法（如LLL算法）本质上就是通过一系列的数值调整使得基向量尽可能短，在整数范围内尽量接近正交。
## The Nguyen-Stern Algorithm
The Nguyen-Stern Algorithm分两个步骤
1. 从样本 \(h\)中，确定格 \(\bar{\mathcal{L}}_{\mathrm{x}}\)，其中 \(\mathcal{L}_{\mathrm{x}}\)是由 \(x_i\)生成的格。
1. 从 \(\bar{\mathcal{L}}_{\mathrm{x}}\)中恢复出来隐藏向量 \(x_i\)，然后由线性关系恢复出 \(a_i\)。
### First step
第一步我们的目标是将已知的 \(h\)和 \(q\)构造格基，设 \(L_0\)是与  \(h\)模 \(q\)正交（垂直）的向量的格
$$
\mathcal{L}_0:=\Lambda_q^\perp(\mathbf{h})=\{\mathbf{u}\in\mathbb{Z}^n\mid\langle\mathbf{u},\mathbf{h}\rangle\equiv0\quad(\mathrm{mod}\:q)\:\}
$$
对于任意的  \(\mathbf{u}\in\mathcal{L}_{0}\)，有 \(\boldsymbol{u}\cdot\boldsymbol{h}\equiv\sum_{i=1}^n\alpha_i(\boldsymbol{u}\cdot\boldsymbol{x}_i)\equiv0\quad(\mathrm{mod}\:q)\)
令向量 
$$
\mathbf{p_u}=(\langle\mathbf{u},\mathbf{x_1}\rangle,\ldots,\langle\mathbf{u},\mathbf{x_n}\rangle)
$$
我们能获得
$$
\langle\mathbf{u},\mathbf{h}\rangle\equiv\alpha_1\langle\mathbf{u},\mathbf{x}_1\rangle+\cdots+\alpha_n\langle\mathbf{u},\mathbf{x}_n\rangle\equiv0\quad(\mathrm{mod}\:q)
$$
然后，如果 \(u\)是短向量，\(\mathbf{p_u}\)也是短向量，如果 \(\mathbf{p_u}\)是一个短于与 \(\alpha\: mod\:q\)正交的最短非零向量，那么就只能是 \(\mathbf{p_u}=0\)
即 \(u\cdot x_i=0\)，令 \(L_x\)是以 \(\boldsymbol{x}_1,\cdots,\boldsymbol{x}_n\)为基的格，可以得到 \(\mathbf{u}\in\mathcal{L}_{\mathbf{x}}^{\perp}\)，也就是说 \(u\)是 \(L_x\)的正交格 \(\mathcal{L}_{\mathbf{x}}^{\perp}\)中的向量。
问题就可以转化为我们要先求出 \(u_i\)，这相当于找到了 \(\mathcal{L}_{\mathbf{x}}^{\perp}\)
首先造格我们需要找到多项式
拆开
$$
\boldsymbol{u}\cdot\boldsymbol{h}\equiv0\quad(\mathrm{mod}q)
$$
得到
$$
\sum_{i=1}^mu_ih_i\equiv0\quad(\mathrm{mod}q)
$$
也就是
$$
\sum_{i=1}^mu_ih_i -kq=0
$$
令
$$
\boldsymbol{v}=(-k,u_1,u_2,u_3,\cdots,u_m)
$$
由 \(h\)和 \(q\)构造格
$$
L=\begin{bmatrix}M&&&&&\\h_1&1&&&&\\h_2&&1\\\vdots&&&\ddots&\\h_m&&&&1\end{bmatrix}_{(m+1)\times(m+1)}
$$
也就是
$$
w =vL = \boldsymbol{w}_2=(0,u_1,u_2,u_3,\cdots,u_m)
$$
```Python
L = block_matrix(ZZ,[
    [matrix(ZZ,m*[0])],
    [identity_matrix(m)]
])
R = matrix(ZZ,[M]+h).transpose()
M = R.augment(L)
#print(M)
L = M.LLL()
```
得到 L的前 \(m-n\)来构造 \(L_{x}^{\perp}\)，对 \(L_{x}^{\perp}\)求零空间得到 \(L_x\)的正交补 \(\bar{L}_{x}\)
```Python
Lxo = matrix(ZZ, L[:m-n])
Lxc = Lxo.right_kernel(algorithm='pari').matrix() # faster
print('right_kernel done.')

Lx_real = matrix(ZZ, [xi + [0] * (m - len(xi)) for xi in X])
rsc = Lxc.row_space()
print([xi in rsc for xi in Lx_real])
```
### Second step
Nguyen-Stern 算法的第二步涉及使用**BKZ**规约来找到格 \(L_x\)的最短向量。
由于**LLL**规约的近似因子较大，得到的基向量可能远大于原始的二进制向量 \(x_i\)，因此，需要 BKZ 规约
来获得更紧的逼近，从而恢复原始的二进制向量。
分析：
直接贴下原文
恢复 \(x_i\):
由于短向量可能是 \(x_i-x_j\)形式，直接从 BKZ 规约得到的基可能不是 \(x_i\)本身。
需要构造新格
$$
L_x^{\prime}=2L_x+e\mathbb{Z}，e=(1,1,\cdots,1)
$$
因为
\(2x_i\)变为 \(\{-2,0,2\}\), \(2x_i-e\)变成 \(\{-1,-1\}\)（更短）
这里类似于背包密码里对 \(x_1,\cdots,x_n\)处理为
$$
2\boldsymbol{x}_{1}-1,2\boldsymbol{x}_{2}-1,\cdots,2\boldsymbol{x}_{n}-1
$$
```Python

 def checkMatrix(M, wl=[-1, 1]):
  M = [list(_) for _ in list(M)]
  ml = list(set(flatten(M)))
  logging.debug(ml)
  return sorted(ml) == sorted(wl)
    
e = matrix(ZZ, [1] * m)
B = block_matrix([[-e], [2*Lxc]])
Lx = B.BKZ()
assert checkMatrix(Lx)
assert len(set(Lx[0])) == 1
```
恢复权重 \(\alpha_i\)：
最后解一个线性方程
$$
h=\alpha_1x_1+\alpha_2x_2+\cdots+\alpha_nx_n\quad\mathrm{mod}M
$$
令
$$
X^{\prime}=(x_1,x_2,\ldots,x_n)
$$
$$
h^{\prime}=(h_1,h_2,\ldots,h_n)
$$
求解
$$
\alpha=X^{\prime{-1}}h^{\prime}\mod M
$$
```Python
Lx = Lx[1:]
E = matrix(ZZ, [[1 for c in range(Lxc.ncols())] for r in range(Lxc.nrows())])
Lx = (Lx + E) / 2

Lx2 = []
e = vector(ZZ, [1] * m)
rsc = Lxc.row_space()
for lx in Lx:
  if lx in rsc:
    Lx2 += [lx]
    continue
  lx = e - lx
  if lx in rsc:
    Lx2 += [lx]
    continue
  print('Something wrong?')
Lx = matrix(Zmod(M), Lx2)
vh = vector(Zmod(M), h)
va = Lx.solve_left(vh)
```
## 整点题题
### Hgame2025 SPiCa
```Python
from Crypto.Util.number import *
from secrets import flag
from sage.all import *
 
def derive_M(n):
    iota=0.035
    Mbits=int(2 * iota * n^2 + n * log(n,2))
    M = random_prime(2^Mbits, proof = False, lbound = 2^(Mbits - 1))
    return Integer(M)
 
m = bytes_to_long(flag).bit_length()
n = 70
p = derive_M(n)
 
 
F = GF(p)
x = random_matrix(F, 1, n)
A = random_matrix(ZZ, n, m, x=0, y=2)
A[randint(0, n-1)] = vector(ZZ, list(bin(bytes_to_long(flag))[2:]))
h = x*A
 
with open("data.txt", "w") as file:
    file.write(str(m) + "\n")
    file.write(str(p) + "\n")
    for item in h:
        file.write(str(item) + "\n")
```
HSSP
```Python
def checkMatrix(M, wl=[-1, 1]):
    M = [list(i) for i in list(M)]
    ml = list(set(flatten(M)))
    return sorted(ml) == sorted(wl)
 
def hssp_solve(n,m,M,h):
    ge1 = [[0]*m for _ in range(m)]
    tmp = pow(h[0], -1, M)
    for i in range(1,m):
        ge1[i][0] = -h[i]*tmp
        ge1[i][i] = 1
    ge1[0][0] = M
 
    Ge1 = Matrix(ZZ,ge1)
 
    L1 = Ge1.BKZ()
 
    Lx_orthogonal = Matrix(ZZ, L1[:m-n])
    Lx = Lx_orthogonal.right_kernel(algorithm='pari').matrix()
 
    e = Matrix(ZZ, [1] * m)
    B = block_matrix([[-e], [2*Lx]])
 
    L2 = B.BKZ()
    assert checkMatrix(L2)
 
    E = matrix(ZZ, [[1]*L2.ncols() for _ in range(L2.nrows())])
    L2 = (L2 + E) / 2
 
    assert set(L2[0]) == {0}
 
    L2 = L2[1:]
 
    space = Lx.row_space()
 
    Lx2 = []
    e = vector(ZZ, [1] * m)
    for lx in L2:
        if lx in space:
            Lx2 += [lx]
            continue
        lx = e - lx
        if lx in space:
            Lx2 += [lx]
            continue
        return None
 
    Lx = matrix(Zmod(M), Lx2)
 
    vh = vector(Zmod(M), h)
    va = Lx.solve_left(vh)
    return Lx, va
 
with open("data.txt", "r") as file:
    n = 70
    m = int(file.readline())
    M = int(file.readline())
    h = list(map(int, file.readline()[1:-2].split(", ")))
 
A,a = hssp_solve(n,m,M,h)
 
for row in A:
    ans = "".join(str(i) for i in row)
    try:
        print(long_to_bytes(int(ans,2)).decode())
    except:
        None
 
# hgame{U_f0und_3he_5pec14l_0n3!}
```
