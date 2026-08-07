---
title: "Unbalanced Oil and Vinegar Signature Scheme"
description: ""
slug: uov-signature-scheme
date: 2025-05-29 00:00:00+0000
categories:
    - 密码学
    - 论文阅读
tags:
    - crypto
    - paper
math: true
---

Unbalanced Oil and Vinegar Signature Scheme
## UOV
### background
多变量多项式密码学（Multivariate Polynomial Cryptography, MPC）是一种基于**多变量多项式方程组求解难题**的密码学方法，是后量子密码的一个重要分支。其中**UOV签名**方案基于油和醋变量的不平衡设计，能够实现高效且安全的数字签名。
### **UOV（Unbalanced Oil and Vinegar）签名流程**
先从人话开始讲，也就是用最少的数学概念来通俗解释这个签名的流程。
UOV签名方案基于多变量二次多项式。该方案分为两个变量组：油变量和醋变量。
- 油变量（Oil Variables）：记作 \(x_1,x_2,\ldots,x_o\)
- 醋变量（Vinegar Variables）：记作 \(y_1,y_2,\cdots,y_v\)
其中，通常 \(o>v\)，也就是多放醋少放油会更安全。
我有 \(o\)份油醋配方作为公钥：
\(f_{i}(x_{1},x_{2},\ldots,x_{n})=\sum_{j=1}^{n}a_{ij}x_{j}^{2}+\sum_{j<k}b_{ijk}x_{j}x_{k}\)
其中，**没有 油，醋一次项，油二次项，以及常数项**。这是UOV的特点（齐次）
**流程：**
1. 选 vinegar 变量
1. 带入 \(y\)，构造线性方程求解 \(x\)
1. 给定消息 \(m\)，计算其哈希值 \(h = H(m)\)
1. 我们构造的签名就是所有的油和醋
   \(\sigma=(y_0,y_1,\cdots,y_v,x_0,x_1,\cdots,x_o)\)，使得 \(P(x,y) = h =H(m)\)，则验证签名成功
以下是一个 \(o = 2,v = 3\)的OUV签名实例
```Python
import numpy as np

def generate_key_pair(o, v):
    oil_coeffs = np.random.randint(1, 10, (o, o))
    vinegar_coeffs = np.random.randint(1, 10, (v, v))
    public_key = (oil_coeffs, vinegar_coeffs)
    private_key = (oil_coeffs, vinegar_coeffs)
    return public_key, private_key

def sign(message, private_key):
    oil_coeffs, vinegar_coeffs = private_key
    signature = np.dot(oil_coeffs, message) + np.dot(vinegar_coeffs, message)
    return signature

def verify(message, signature, public_key):
    oil_coeffs, vinegar_coeffs = public_key
    expected_signature = np.dot(oil_coeffs, message) + np.dot(vinegar_coeffs, message)
    return np.array_equal(signature, expected_signature

o = 2
v = 3
public_key, private_key = generate_key_pair(o, v)
message = np.random.randint(1, 10, (o, 1))
signature = sign(message, private_key)
is_valid = verify(message, signature, public_key)

print("Public Key:", public_key)
print("Private Key:", private_key)
print("Message:", message)
print("Signature:", signature)
print("Is valid:", is_valid)

```
### 二次型（**Quadratic form）**
所谓**二次型**，是关于一些变量的二次齐次多项式，例如 \(3x^2 + 4xy +2y^2\) 是关于变量 \(x\)和 \(y\)的二次型。
二次型一定是齐次的，不等同于二次函数/二次方程。
在向量空间中：任何非零的n维二次型在一个 (n-1) 维的投影空间中定义了一个 \((n-2)\)维的二次曲面，3维二次型可视化为圆锥曲线。
#### 另一种形式：
任何2维二次形式可以被写为：
$$
F(x,y)=ax^2+by^2+cxy
$$
这个向量空间的任何向量可以表示为 \(x = (x,y)\)，二次形式*F*可以表达为矩阵。
$$
M=\begin{bmatrix}a&\frac c2\\\frac c2&b\end{bmatrix}
$$
接着矩阵乘法给我们下列等式：
$$
F(x)=x^{T}M x
$$
那么对于n维的UOV公钥二次型：
设有一个变量向量： \(x = (x_1,x_2,\cdots,x_n)^T \in \mathbb{F}_{q}^n\)
对于二次型：
$$
f_{i}(x_{1},x_{2},\ldots,x_{n})=\sum_{j=1}^{n}a_{ij}x_{j}^{2}+\sum_{j<k}b_{ijk}x_{j}x_{k}
$$
也有更简洁的形式：
$$
f(x ) =x^TAx
$$
特别的： \(A\)是一个上三角矩阵。
$$
A=\begin{pmatrix}A_1&A_2\\0&A_3\end{pmatrix}
$$
![](attachment:0eba587b-e808-49c3-8681-38acae17315b:image.png)
#### 性质：
- Q服从平行四边形定律：
$$
Q(u+v)+Q(u-v)=2Q(u)+2Q(v)
$$
- **极化形式（polar form）(很重要！）：**
  \(p^{\prime}(\boldsymbol{x},\boldsymbol{y})=p(\boldsymbol{x}+\boldsymbol{y})-p(\boldsymbol{x})-p(\boldsymbol{y})\)  展开：  \(p(\boldsymbol{x}+\boldsymbol{y})=\boldsymbol{x}^T\boldsymbol{A}\boldsymbol{x}+\boldsymbol{x}^T(\boldsymbol{A}+\boldsymbol{A}^T)\boldsymbol{y}+\boldsymbol{y}^T\boldsymbol{A}\boldsymbol{y}\)  得到：  $$
  p^{\prime}(x,y)=x^{T}(A+A^{T})y
  $$
因此，极化形式对 \(x\)和 \(y\)都是对称和线性的。
### 多元二次映射P(x)
多元二次映射 \(P\)是由 \(m\)个多元二次函数组成的向量，对每个 \(p_i\)都进行P映射
怎么理解 \(\mathcal{P}\::\:\mathbb{F}_q^n\:\to\:\mathbb{F}_q^m\)
这是在有限域 \(\mathbb{F}_{q}\)上从 \(n\)维向量空间到 \(m\)维向量空间的线性映射，从UOV签名中更直观的来看：
输入： \(x \in \mathbb{F}_q^n\)
输出： \(\mathcal{P}(\mathbf{x})\in\mathbb{F}_q^m\)
$$
n>m
$$
### 签名流程:
要签名 \(m \in \mathbb{F}_q^n\)，我们需要找到 \(x \in \mathbb{F}_q^m\)使得 \(P(x) = m\);
这个过程是NP-hard
但如果我们知道一个子空间 \(O\subset\mathbb{F}_q^n\) ，维度为 \(m\)，对于所有 \(o\in O\)都有 \(P(o) = 0\)，我们就可以快速签名（ 子空间 \(O\)也就是私钥的一部分）
接下来：
我们可以随机选一个 \(v\)，然后找一个 \(o \in O\)，使得 \(P(v+o) = m\)
展开：
$$
P(v+o)=P(v)+P^{\prime}(v,o)+P(o)
$$
因为：
$$
P(o) = 0
$$
所以：
$$
P^{\prime}(v,o)=t-P(v)
$$
因为 \(P^{\prime}(v,o)\)对 \(o\)是线性的，而 \(O\)的维度正好是 \(m\)，所以我们可以用高斯消元法求解；
## Attack
### oil泄露
#### challenge
```Python
from sage.all import *
from sock import Sock

q, n, m = 251, 106, 44
F = GF(q)
f = Sock("be.ax 31105")
f.read_line()
data = bytes.fromhex(f.read_line().strip().decode())
data = [F(x) for x in data]
off = 0
def getdata(num):
    global off
    ret = data[off:off+num]
    off += num
    return ret

PM = []
z = zero_matrix(F, m, (n - m))
for i in range(m):    
    P1 = matrix(F, (n - m), (n - m), getdata((n-m)**2))
    P2 = matrix(F, (n - m), m, getdata((n-m)*m))
    P3 = matrix(F, m, m, getdata(m**2))
    P = block_matrix([ [P1, P2], [z, P3]])
    PM.append(P)
# Hints:

# P(g)
y = vector(F, bytes.fromhex(f.read_line().strip().decode()))
# ^o = o + g
og = vector(F, bytes.fromhex(f.read_line().strip().decode()))
# signature challenge
chall = vector(F, bytes.fromhex(f.read_line().strip().decode()))

```
#### Solve
在这个挑战中，它生成一个**多变量二次映射**：
 \(P: \mathbb{F}_q^{106} \to \mathbb{F}_q^{44}\)
这个参数规模略小于 NIST Level 1 的安全等级。同时，我们还获得了子空间 \(O\) 的一组**基向量**。
此外，还提供了两个提示（oil spills）：
- 一个向量 \(g + o\)，其中：
  -  \(g\) 是由 **0 和 1 组成的向量**  - \(o \in O\)
- 以及 \(P(g)\) 的值。
---
### 解题思路
我们希望从已知的提示中恢复出秘密向量 \(g\)。核心思想是造格，使得 \(g\) 是其中一个**很短的向量**，从而借助 BKZ 等格规约算法恢复出来。
---
#### 推导过程
1. 设我们猜测 \(o\) 的近似值为 \(\hat{o}\)，我们有如下公式：
1. 由于 \(\hat{o} - g = o\) 是 \(O\) 的一个元素，我们可以令：
1. 对于单个输出分量 \(p'_i(x, y)\)，可以写成：
---
#### 造格
考虑如下格子矩阵 \(B\)
$$
\begin{gathered}\boldsymbol{B}=\begin{bmatrix}1&0&0&\ldots&\hat{\boldsymbol{o}}(\boldsymbol{P}_0+\boldsymbol{P}_0^T)[0]&\ldots&\hat{\boldsymbol{o}}(\boldsymbol{P}_m+\boldsymbol{P}_m^T)[0]\\0&1&0&\ldots&\hat{\boldsymbol{o}}(\boldsymbol{P}_0+\boldsymbol{P}_0^T)[1]&\ldots&\hat{\boldsymbol{o}}(\boldsymbol{P}_m+\boldsymbol{P}_m^T)[1]\\\vdots&\vdots&\vdots&\ddots&\vdots&\vdots&\vdots\\0&0&0&\ldots&q&0&0\\0&0&0&\ldots&0&q&0\\0&0&0&\ldots&0&0&\ddots\\0&0&0&\ldots&k_0+\boldsymbol{g}^T\boldsymbol{P}_0\boldsymbol{g}&\ldots&k_m+\boldsymbol{g}^T\boldsymbol{P}_m\boldsymbol{g}\end{bmatrix}\end{gathered}
$$
构造出如下结构后，考虑线性组合向量：
\(t = (g \Vert w_0, w_1, w_2, \dots \Vert -1)\)
其中 \(w_i\) 是模 \(q\) 下的权值，乘以该基 \(B\) 有：
\(tB = (g \Vert 0)\)
这是一个非常短的向量，因此只需要通过 BKZ（块大小约为 15）规约即可恢复。
当然还可以降维和使用 1/2的技巧，用 LLL也是可行的
```Python
M = matrix(F, [og*(Pi+Pi.T) for Pi in PM])
v = vector(F, [og*Pi*og + yi for (Pi, yi) in zip(PM, y)])
g0 = M.solve_right(v)
R = M.right_kernel().matrix()
L = R.change_ring(ZZ).stack(251 * identity_matrix(106))
L = (L*2).stack(vector(ZZ, [int(a)*2-1 for a in g0])) # 1/2 trick
Lll = L.LLL(0.99999)    

for row in Lll:
    if row and -1 <= min(row) <= max(row) <= 1:
        sol = row
        print(row)

g = vector(F, [(a+1)//2 for a in sol])

o = og - g

for Pi in PM:
    assert o * Pi * o == 0
```
---
#### 恢复整个子空间
我们使用下面的攻击函数来恢复整个 \(\mathcal{O}\) 子空间的基：
```Python
def attack(G,v):
    m = len(G)
    J = matrix([v*g for g in G]) 
    B = matrix(J.right_kernel().basis())
    B2 = []
    for g in G :
        ghat = B*g*B.transpose()
        for b in ghat.kernel().basis() :
            if len(B2) == 0 or b not in span(B2) :
                B2.append(b)
        if len(B2) == m :
            break
    B3 = matrix(B2)
    C = B3*B
    return C
```
- 该函数对每个对称矩阵 \(G_i\)，将其限制在 \(v\) 的核空间 \(B\) 上，并进一步找到限制形式的核。
```JavaScript
from ast import literal_eval
from tqdm import tqdm
from pwn import process, remote
from hashlib import shake_128
from sage.all import *

m, n = 73, 180
q = 0x10001
F = GF(q)

def recv_pk():
    io.recvuntil("🌻 ")
    seed = int(io.recvline())
    set_random_seed(seed)
    pub = []
    for _ in range(m):
        P1 = random_matrix(F, n-m, n-m)
        P2 = random_matrix(F, n-m, m)
        P3 = matrix(F, m, m, literal_eval(io.recvline().decode()))
        pub.append(block_matrix(F, [[P1, P2], [zero_matrix(F, m, n-m), P3]]))
    return pub
        
# io = process(["sage", "task.py"])
io = remote("39.106.16.204", "17131")
pks = recv_pk()

for _ in tqdm(range(79)):
    io.sendlineafter("> ", "R")
    pks.extend(recv_pk())

import time
start_time = time.time()

A = []
u = []
for pk in tqdm(pks):
    a = []
    for i in range(n-m):
        for j in range(i, n-m):
            if i != j:
                a.append(pk[i,j]+pk[j,i])
            else:
                a.append(pk[i,j])
    for i in range(n-m):
        a.append(pk[n-m,i]+pk[i,n-m])
    u.append(pk[n-m,n-m])
    A.append(a)

print(len(A), len(A[0]))
A = matrix(F, A)
u = vector(F, u)

print("[+] Start right kernel...")
ker = A.right_kernel().matrix()[:,-(n-m):]
print("[+] End right kernel...")

print("[+] Start Eq solve...")
x = A.solve_right(-u)[-(n-m):]
print("[+] End Eq solve...")

dim = n-m
H = ker.echelon_form()
L = matrix(ZZ, dim+1, dim+1)
L[:H.nrows(), :dim] = H
L[H.nrows():dim, H.nrows():dim] = identity_matrix(dim-H.nrows())*q
L[dim, :] = matrix(1, dim+1, list(x)+[2**5])
print("[+] Start LLL...")
basis = L.LLL()
end_time = time.time()
print("[+] Time Cost", end_time-start_time)
for item in basis[:10]:
    if abs(item[-1]) == 32:
        print("[+] FIND!", item)
        break
else:
    print("[-] Fail :(")

o1 = vector(F, list(basis[0][:-1])+[1]+[0]*(m-1))
print(o1, o1*pks[0]*o1)

E = zero_matrix(F, len(pks), n)
for i in range(len(pks)):
    E[i,:] = o1*pks[i]
print("E Rank", E.rank())
O = E.right_kernel().matrix().T
print("O row col", O.nrows(), O.ncols())

pub = pks[-m:]

def Hash(msg):
    h = int(shake_128(msg).hexdigest(3*m),16)
    return vector(GF(q),[(h:=h//q)%q if i else h%q for i in range(m)])

def sign(msg, O):
    F = GF(q)
    v = random_vector(F, n, 1)
    M = matrix(F, [v*(pub[i]+pub[i].T)*O for i in range(m)])
    u = Hash(msg.encode())-vector([(v*pub[i]*v) for i in range(m)])
    return v+O*M.solve_right(u)

io.sendlineafter("> ", "V")
io.sendlineafter("📝 ", str(list(sign("Unwind On Vacation", O))))
io.interactive()
```
