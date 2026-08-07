---
title: "噪声多项式插值与noiseCRT"
description: ""
slug: noise-crt
date: 2025-04-07 00:00:00+0000
categories:
    - 密码学
tags:
    - crypto
    - note
math: true
---

论文复现[Noisy Polynomial Interpolation  and Noisy Chinese Remaindering](https://www.iacr.org/archive/eurocrypt2000/1807/18070053-new.pdf)
## 1.
## Problem 1 (Noisy polynomial interpolation)
![](https://github.com/komiko0831/picx-images-hosting/raw/master/QQ20250407-001351.54xzbrh31y.png)
![](https://github.com/komiko0831/picx-images-hosting/raw/master/QQ20250407-001340.2dox3ov4nj.png)
你有一个未知的多项式 \(P(x)\)，它的次数最多是 \(k\)，
你已知的：
- 一堆不同的点 \(x_1,x_2,\ldots,x_n\in\mathbb{F}\)，这些是你希望插值的横坐标；
- 对应的集合 \(S_1,S_2,\cdots.S_n\)，每个集合 \(S_i\)里装了 m个元素，其中：
  - 只有一个是真正的 \(y_i = P(x_i)\)  - 剩下 m - 1是noise
目标是：
从这些模糊的集合中找回原始的多项式 \(P(x)\)
问题代码化也就是：
```Python
# Noisy Polynomial Interpolation Instance
from sage.all import ZZ,PolynomialRing,GF,Matrix
from random import shuffle
from Crypto.Util.number import getPrime
DEBUG = True

def noisy_polynomial_interpolation_instance(n,m,F=None,k = None):
    # Generate n distinct random points
    if F == None: F = ZZ
    
    while True:
        Xs = [F.random_element() for i in range(n)]
        if len(set(Xs)) == n:break

    if k == None:
        k = ZZ.random_element(0,n-1)
        print(f"k = {k}")

    assert n > k + 1 , "Err:Not Enough Points, n must be greater than k + 1"
    P = PolynomialRing(F,'x')
    x = P.gen()
    f = x^k + 1
    # Construct Quotient Ring Q by mod X^n + 1
    Q = P.quotient(f)
    p = Q.random_element()
    p = p.lift()

    if DEBUG:print(f"Test polynomial: {p}\nWith parent: {p.parent()}")
    
    Ys = [p(x) for x in Xs]
    # Add noise
    S = []
    for i in range(n):
        while True:
            Y = [Ys[i]] + [F.random_element() for j in range(m-1)]
            if len(set(Y)) == m:break
        shuffle(Y)
        S.append(Y)
    if DEBUG:
        print(f"Ys: {Ys}")
        print(f"Xs: {Xs}")
        print(f"S: {S}")
        points = [(Xs[i],Ys[i]) for i in range(n)]
        pp =  LagrangeInterpolation(points)
        print(f"Lagrange Interpolation Polynomial: {pp}")
        print(f"Test polynomial: {p}")
        print(f"Interpolation Polynomial Correctness: {check_poly_Lagrange(points,pp)}")
        return Ys,Xs,S
    return S

def ConstructMatrixFromSn(S):
    """
    Construct a matrix from the noisy polynomial interpolation data.
    
    Args:
        S (list): List of lists containing noisy y-values.
    
    Returns:
        Matrix: Constructed matrix.
    """

    M = Matrix(ZZ, S)
    if DEBUG:
        print(f"Constructed Matrix:\n{M}")
        print(f"Matrix Size: {M.nrows()} x {M.ncols()}")
        print(f"Matrix Rank: {M.rank()}")
    return M




```
#### 1.0.1 拉格朗日插值法
拉格朗日插值法是一种用来通过已知数据点构造多项式的方法。它的主要思想是，通过一组已知的点，找到一个多项式，使这个多项式在这些点上和已知的函数值相等。
假设我们有 n+1 个点 \((x_0,y_0),(x_1,y_1),\ldots,(x_n,y_n)\)，其中 \(y_i = f(x_i)\)。
拉格朗日多项式：
对于每一个数据点 \((x_i,y_i)\)，我们构造一个基多项式 \(L_i(x)\):
$$
L_i(x)=\prod_{0\leq j\leq n}\frac{x-x_j}{x_i-x_j}
$$
我们可以发现这个多项式在 \(x_i\)处取 1，在其他所有 \(x_j\)处取 0.
最终的插值多项式 \(P(x)\)是所有基多项式的加权和:
$$
P(x)=\sum_{i=0}^ny_iL_i(x)
$$
以下是按自己的理解/收集到的拉格朗日插值法恢复多项式的板子
- 按定义来
```Python
def poly_add(a,b,p):
    res = []
    for i in range(max(len(a),len(b))):
        coef_a = a[i] if i < len(a) else 0
        coef_b = b[i] if i < len(b) else 0
        res.append((coef_a + coef_b) % p)
    return res

def poly_mul(a,b,p):
    res = [0]*(len(a)+len(b)-1)
    for i in range(len(a)):
        for j in range(len(b)):
            res[i+j] = (res[i + j] + a[i]*b[j])%p
    return res

def lagrange_interpolation(x_s,y_s,p):
    k = len(x_s)
    assert k == len(set(x_s))
    P = [0]
    for i in range(k):
        numerator = [1]
        denominator = 1
        for j in range(k):
            if i != j:
                numerator = poly_mul(numerator,[ -x_s[j]%p,1],p)
                denominator = (denominator*(x_s[i]-x_s[j]))%p
        inv_denominator = pow(denominator,-1,p)
        L_i = [(coef*inv_denominator)%p for coef in numerator]
        Li_yi = [(coef * y_s[i]) % p for coef in L_i]
        P = poly_add(P,Li_yi,p)
    return P

p = 17
shares = [(1,8), (2,11)]

x_s = [x for x, _ in shares]
y_s = [y for _, y in shares]

P = lagrange_interpolation(x_s, y_s, p)
print(P)
```
- 普通矩阵乘法
```Python
def solve(enc):
    e, p, PT = enc
    t = len(PT)
    A = [[pow(PT[i][0], j, p) for j in range(t)] for i in range(t)]
    b = [PT[i][1] for i in range(t)]
    A = matrix(Zmod(p), A)
    b = vector(Zmod(p), b)
    x = A.solve_right(b)
    cc = x[0]
    print(f'{cc = }')
    print(f'{e = }')
    print(f'{p = }')

solve(enc)
```
- 范德蒙德矩阵
```Python
# Lagrange Interpolation Polynomial with Sage
from sage.all import *
DEBUG = True

def InterpolationBasePoly_Li(F,points:list,i:int):
    R.<x> = PolynomialRing(F)
    try:
        xi,_ = points[i]
    except IndexError:
        raise ValueError("Index out of range")
    # Compute the Lagrange basis polynomial
    Li = prod([(x - xj)/(xi - xj) for j, (xj,_) in enumerate(points) if j != i])
    
    return Li 


def InterpolationPoly(F,points:list):
    """
    This function computes the Lagrange Interpolation Polynomial
    given a set of points.
    """
    # Initialize the polynomial ring
    R.<x> = PolynomialRing(F)
    # Compute the Lagrange Interpolation Polynomial with InterpolationBasePoly_Li
    P = sum([yi * InterpolationBasePoly_Li(F, points, i) for i, (_, yi) in enumerate(points)])

    
    return P

def LagrangeInterpolation(points:list):
    """
    This function computes the Lagrange Interpolation Polynomial
    given a set of points.
    Cao
    """
    # Get the field of the points
    F = points[0][0].parent()
    print(F)
    # Compute the Lagrange Interpolation Polynomial
    P = InterpolationPoly(F,points)
    
    return P

def check_poly_Lagrange(points:list,p):

    P = LagrangeInterpolation(points)
    Result = False
    for i, (x,y) in enumerate(points):
        Result &=  (P(x) == p(x))
        print(f"P(x) =? p(x) ,{P(x) == p(x)}")
    return True

'''
R = PolynomialRing(GF(7^3),'x')
print(R.lagrange_polynomial(points))
print(LagrangeInterpolation(points))
'''


points = [(0,1),(1,2),(2,3)]

def Vandermonde_matri_from_Points(Points:list,k):
    """
    Construct a Vandermonde matrix from a given list of values.
    
    Args:
        v (list): A list of values [v1, v2, ..., vn].
    
    Returns:
        Matrix: The Vandermonde matrix.
    """
    Xs, Ys = zip(*Points)
    Ys = Ys + [0]*(k-len(Ys))

    n = len(Points)
    Vander = matrix([[Xs[i]**j for j in range(k)] for i in range(n)])
    Values = matrix(vector(Ys)).transpose()

    if DEBUG:
        print(f"Vandermonde Matrix:\n{Vm}")
        print(f"Vandermonde Values:\n{Vv}")


    return Vander, Values


# Matrix Version of Lagrange Interpolation
def MarixLagrangeInterpolation(points:list,k):
    Vm,Vv = Vandermonde_matri_from_Points(P,k)
    coef = Vm.solve_right(Vv)
    F = points[0][0].parent()
    print(f"Vandermonde Coefficients:\n{coef}")
    R.<x> = PolynomialRing(F)
    f = R(coef.list())
    if DEBUG:
        print(f"Vandermonde Reconstruct Polynomial:\n{f}")
    return f




```
- sage上有函数
```Python
F = GF(p)
R.<x> = PolynomialRing(F)
points = [(F(a), F(b)) for (a, b) in points]
P = R.lagrange_polynomial(points)
```
### 1.1 A Gr ̈obner Basis Method
关于Goebner基的用法可以看我的[这篇文章](https://www.komiko.top/article/Groebner-basis)
思想是将其转化为**求解一个多变量多项式方程组**的问题。
多项式的系数表示法：
未知的多项式 \(P(x)\)可以表示为：
$$
P(X)=\sum_{i=0}^ka_iX^i
$$
我们需要求的只是未知系数 \(a_0,a_1,\cdots,a_k\)。
对于每个 i，我们知道在集合 \(S_i=\{y_{i,1},\ldots,y_{i,m}\}\)中总有一个值是真的，也就是说：
$$
P(x_i)=y_{i,j}
$$
也就意味着能构造方程：
$$
\prod_{j=1}^m(P(x_i)-y_{i,j})=0=(P(x_i)-y_{i,1})(P(x_i)-y_{i,2})\cdots(P(x_i)-y_{i,m})=0
$$
展开之后是一个关于 \(a_0,\cdots,a_k\)的多项式
我们一共收集了 n个这样的等式后，就可以变成一个用Groebner基求解多变量方程组的问题。
但！这个方法在变量数或次数稍大时会爆炸，比如：k 大于 10~20，Gröbner 基就已经慢到不行了；
### 1.2 Lattice-Based Methods for Noisy Polynomial Interpolation
我们使用的拉格朗日插值的形式：
$$
P(x)=\sum_{i=0}^ny_iL_i(x)=\sum_{i=0}^nP(x_i)L_i(x)
$$
其中：
$$
L_i(x)=\prod_{0\leq j\leq n}\frac{x-x_j}{x_i-x_j}
$$
新定义一个选择变量 \(\delta_{i,j}\):
如果 \(P(x_i) = y_{i,j}\)，那么 \(\delta_{i,j}=1\)，否则为0.
那么多项式还可以表示为：
$$
P(X)=\sum_{i=1}^n\sum_{j=1}^m\delta_{i,j}y_{i,j}L_i(X).
$$
我们注意到 \(\delta \in \{1,0\}\)，求出 \((\delta_{1,1},\delta_{1,2},\ldots,\delta_{n,m})\)就相当于恢复了选出来正确的插值，可以恢复多项式。
1. 构造格基矩阵
$$
\mathbf{L}=\begin{bmatrix}\mathrm{coeff}_0(y_{1,1}L_1)&\mathrm{coeff}_0(y_{1,2}L_1)&\ldots&\mathrm{coeff}_0(y_{n,m}L_n)\\\mathrm{coeff}_1(y_{1,1}L_1)&\mathrm{coeff}_1(y_{1,2}L_1)&\ldots&\mathrm{coeff}_1(y_{n,m}L_n)\\\varvdots&\varvdots&\ddots&\varvdots\\\mathrm{coeff}_d(y_{1,1}L_1)&\mathrm{coeff}_d(y_{1,2}L_1)&\ldots&\mathrm{coeff}_d(y_{n,m}L_n)\end{bmatrix}\quad\in\mathbb{Q}^{(d+1)\times(n\cdot m)}
$$
每一列是一个多项式的系数向量（从常数项到 \(X^d\))
（可以类比于我们的子集和问题）
令：
$$
\sum_{i=1}^n\sum_{j=1}^m\delta_{i,j}\cdot y_{i,j}\cdot L_i(X)=f(X)
$$
展开后：
$$
\mathbf{L}\cdot\vec{\delta}=\vec{f}
$$
\(\vec{\delta}\in\{0,1\}^{n\cdot m}\)是要找的向量， \(\vec{f}\in\mathbb{Q}^{d+1}\)是多项式的系数向量, 规约出它就代表求出了未知的多项式。
## 2.
### Problem 2 (Noisy Chinese remaindering)
![](https://github.com/komiko0831/picx-images-hosting/raw/master/QQ20250407-011247.8z6qupzh77.png)
给出一组互素的整数 \(p_1,p_1,\cdots,p_n\)。给出n个集合 \(S_1,S_2,\cdots,S_n\),其中每个 \(S_i=\{r_{i,j}\}_{1\leq j\leq m}\)包含 \(m-1\)个在 \(\mathbb{Z}_{p_i}\)上的随机元素，只有一个真正的 \(N \mod p_i\)。在满足解唯一的情况下，恢复整数 N。
#### 2.1 Lattice-Based Methods
直接看题
#### 2.1.1 noisy-CRT 
- **题一([NeepuCtf 2023]loud)**
题目：
```Python
from Crypto.Util.number import *
import random
flag = randint(1, 2^4096)
print(flag)
p = [getPrime(256) for _ in range(20)]
c = [[randint(0, p[i] - 1) for __ in range(3)] + [int(flag % p[i])] for i in range(20)]
for c0 in c:
    random.shuffle(c0)
print(p)
print(c)
```
我们已知：
$$
p = (p_1,p_2,\cdots,p_{20})
$$
给出每个 \(c_i\)，是一个长度为4的列表，包含前三个为干扰的随机值（noise），以及一个真实模值 \(flag \mod p_i\)
shuffle函数对 \(c\)进行随机打乱
分析：
如果没有被打乱，这就是一个简单的CRT，也就是对c的最后一列进行CRT处理就能得到flag的值，这道题就在于c被打乱，无法确定 \(c_i\)到底是真实模值还是随机数。是一个典型的noisyCRT问题。
复习一下CRT吧：
![](https://github.com/komiko0831/picx-images-hosting/raw/master/s.102dznkgbd.png)
令： \(P = prod(p)\)， \(P_i = P//p_i\)
$$
c=\begin{pmatrix}a_{1,1}&a_{1,2}&a_{1,3}&a_{1,4}\\a_{2,1}&a_{2,2}&a_{2,3}&a_{2,4}\\&\cdots&\ddots&\cdots\\a_{20,1}&a_{20,2}&a_{20,3}&a_{20,4}\end{pmatrix}
$$
$$
K_i = P_i*inverse(P_i,p_i)
$$
$$
C=\begin{pmatrix}K_1*a_{1,1}&K_1*a_{1,2}&K_1*a_{1,3}&K_1*a_{1,4}\\K_2*a_{2,1}&K_2*a_{2,2}&K_2*a_{2,3}&K_2*a_{2,4}\\&\cdots&\ddots&\cdots\\K_{20}*a_{20,1}&K_{20}*a_{20,2}&K_{20}*a_{20,3}&K_{20}*a_{20,4}\end{pmatrix}
$$
看作近似子集和问题会更清晰：
- 我有80个候选值，选中为1，未选中为0；
- 80选20，使它们加起来等于 flag。
构造格M：
$$
M = \left(\begin{array}{cccc|c}
1 &        &        &        & K_1 \cdot a_{1,1} \\
  & 1      &        &        & K_1 \cdot a_{1,2} \\
  &        & \ddots &        & \vdots        \\
  &        &        & 1      & K_i \cdot a_{i,1} \\
  &        &        &        & K_i \cdot a_{i,2} \\
  &        &        &        & \vdots        \\
  &        &        &        & K_{20} \cdot a_{20,4} \\\hline
0 & 0      & \cdots & 0      & P
\end{array}\right)
$$
$$
(l_{1,1},l_{1,2},...,l_{20,20},l)*M=(l_{1,1}*,l_{1,2},...,flag)
$$
其中 \(l_i \in \{0,1\}\)，因为flag是 \(4096bits\)，为了规约成功，对角线还需要乘上 \(2^{4096}\)
```Python
from Crypto.Util.number import *
from tqdm import tqdm
P = prod(p)
pc = []
for i,cc in zip(p,c):
    pp = P // i
    temp = pp * ZZ(inverse(pp,i))
    for j in cc:
        pc.append(j * temp % P)
pc.append(P)
M = matrix.identity(80)    
v = [0] * 80
M = M.stack(vector(v))
M = M.augment(matrix(ZZ,pc).T)
M[:,:-1] *= 2**4096
print('begin LLL')
M = M.LLL()
print('end LLL')
M[:,:-1] /= 2**4096

for i in tqdm(range(81)):
    L = M.row(i).list()[:-1]
    flag = True
    for m in L:
        if m != 0 and m != 1 and m != -1: # 这里是and,想一下它的否命题就明白了
            flag = False
            break
    if flag:
        print(M[i][-1])
        print(i)

```
- **Noise CRT**
```Python
N = [getRandomNBitInteger(3211) for _ in range(15)]

def leak(N):
    p,S = [],[]
    for i in range(15):
        p.append(getPrime(321))
        r = [N[_]%p[i] for _ in range(15)]
        shuffle(r)
        S.append(r)
    return p, S

p, S = leak(N)
```
正常来说：
$$
A=\begin{bmatrix}N_1\pmod{p_0}&N_2\pmod{p_0}&\cdots&N_n\pmod{p_0}\\N_1\pmod{p_1}&N_2\pmod{p_1}&\cdots&N_n\pmod{p_1}\\\vdots&\vdots&\vdots&\vdots\\N_1\pmod{p_m}&N_2\pmod{p_m}\end{bmatrix}
$$
对每一列进行一个CRT就能求出 \(N_i\)，但是shuffle对每一行都进行了置换，我们得到的每个 \(S_i\)是打乱的15个数，我们的目标是需要从乱序的A矩阵中还原出 \((N_1,N_2,\cdots,N_n)\)。
