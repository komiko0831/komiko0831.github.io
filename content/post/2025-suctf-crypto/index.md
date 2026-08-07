---
title: "2025 SUCTF Crypto"
description: ""
slug: 2025-suctf-crypto
date: 2025-01-18 00:00:00+0000
categories:
    - CTF
tags:
    - ctf
    - note
math: true
---

题目复现
## SU-signin
### 题目：
```Python
from Crypto.Util.number import *
from secret import flag

bit_length = len(flag) * 8

p = 0x1a0111ea397fe69a4b1ba7b6434bacd764774b84f38512bf6730d2a0f6b0f6241eabfffeb153ffffb9feffffffffaaab
K = GF(p)
E = EllipticCurve(K, (0, 4))
o = 793479390729215512516507951283169066088130679960393952059283337873017453583023682367384822284289
n1, n2 = 859267, 52437899

while(1):
    G1, G2 = E.random_element(), E.random_element()
    if(G1.order() == o and G2.order() == o):
        P1, P2 = (o//n1)*G1, (o//n2)*G2
        break

cs = [(randrange(0, o) * P1 + randrange(0, o) * G2).xy() if i == "1" else (randrange(0, o) * G1 + randrange(0, o) * P2).xy() for i in bin(bytes_to_long(flag))[2:].zfill(bit_length)]
print(cs)
```
由题目 ：
G1，G2的阶同为o；
$$
\begin{aligned}P_1&=\frac o{n_1}\cdot G_1\\P_2&=\frac o{n_2}\cdot G_2\end{aligned}
$$
这里带入一组P1和P2去计算一下
```Python
P1.order()
#859267
P2.order()
#52437899
```
会发现P1,P2实际上分别是n1，n2阶点；
然后根据flag的每一位bits是0还是1分别计算密文C：
$$
\begin{cases}C_1=aP_1+bG_2=a(o//n_1)G_1+bG_2\\C_2=cG_1+dP_2=cG_1+d(o//n_2)G_2\end{cases}\\(a,b,c,d为随机数）
$$
我们需要从给出的一大坨C里恢复原始bits
因为这题用的是BLS12-381，一个pairing-friendly curve，所以可以很轻易的计算pairing。
又因为flag是填充过后的，所以第一个bit的点K一定是0，采用的是 \(aG_1+bP_2\)的方式加密，而由于 \(P_2\)是  \(n_2\)阶的点，有 \(P_2=\frac o{n_2}\cdot G_2\text{ ,}\)    
$$
n_2K=n_2(aG_1+bP_2)
$$
> 💡 
对于阶为 \(o\)的点P，Q，Weil pairing \(e(P,Q),满足：\)
                                        \(e(P,Q)^o=1\)
由题目 \(G_1\)的阶为 \(o\)。 \(n_2*K\)是 o-阶点的倍点，根据上面的性质：
$$
e(n2·K,G1)=1
$$
因此，上式化简为：
$$
e(n_2\cdot K,C)=1\cdot e(n_2\cdot K,P_2)^{rand_2}=e(n_2\cdot K,P_2)^{rand_2}
$$
因此如果weilpairing的结果是1，说明当前位是“0”。
如果密文点对应加密 "1"：
$$
e(n_2K,rand_1\cdot P_1+rand_2\cdot G_2)=e(n_2K,G_2)^{rand_1}
$$
这里\(e(n_2K,G_2)\neq1\)，可以区分
### exp
```Python
from Crypto.Util.number import *

p = 0x1a0111ea397fe69a4b1ba7b6434bacd764774b84f38512bf6730d2a0f6b0f6241eabfffeb153ffffb9feffffffffaaab
K = GF(p)
E = EllipticCurve(K, (0, 4))
o = 793479390729215512516507951283169066088130679960393952059283337873017453583023682367384822284289
n1, n2 = 859267, 52437899

points = []

flag = "0"
K = E(points[0])
for i in points[1:]:
    if((n2*K).weil_pairing(E(i), o) == 1):
        flag += "0"
    else:
        flag += "1"
print(long_to_bytes(int(flag, 2)))
```
### notes
#### 简略的抽象代数定义
> 
- 什么是群：
![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fe549a579-2c2f-4a01-ab69-720e615313ef%2Ffe345630-ff1f-4918-8586-ecd499c2c470%2Fimage.png?table=block&id=94107cdf-ee0a-47b8-8cbf-c48c8fd41eb4&spaceId=e549a579-2c2f-4a01-ab69-720e615313ef&width=2000&cache=v2)
> 💡 
在群中定义求幂运算为重复使用群中的运算，例对（G，+），\(a^4=a+a+a+a\)。规定 \(a^0=e\)为单位元。
> 💡 
如果一个群的所有元素都是a 的幂 \(a^k\),则称这个群是一个**循环群，**这里的k是整数。a也被称为这个群的生成元。
#### 双线性配对
> 双线性配对是一种数学运算，它在两个群（通常是椭圆曲线上的点构成的群）之间建立关系，并映射到一个第三个群（通常是乘法群）上。
这里我们主要关注一下椭圆曲线的双线性配对，椭圆曲线与双线性配对结合，可以构造出新的加密算法，比如基于身份的加密（IBE）、短签名和零知识证明等
> 为了简便，我们专注于对称的双线性配对，即 \(G_1=G_2\),设 \(G_1\)为椭圆曲线的点群, \(G_2\)是一个有限群（假设是加法群），我们可以构造双线性映射 \(e(P,G)\colon G_{1}\times G_{1}\rightarrow G_{T}\),满足：
> 💡 
1. 双线性：对于所有的 \(a,b\in Z\),以及 \(P,Q\in G_1\),有 \(\mathrm{e}(aP,bQ)=ab\mathrm{e}(P,Q)\)
> 💡 
1. 非退化性：如果P和Q均不是群G1的无穷远点O，则 \({e}(P,Q)\neq0\).
> 💡 
1. 可计算性：存在多项式时间的有效算法计算任意 \(P,Q\in G_1\)的配对 \(e(P,Q)\).
下面这张图，便是选取椭圆曲线 \(G_1\)的两个点P,Q并映射到另一个群 \(G_r\)中。
![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fe549a579-2c2f-4a01-ab69-720e615313ef%2Fa4459961-1def-4f0a-9809-e43bef178437%2F34-1.png?table=block&id=f80ed900-289c-44a5-9aa9-92bec8f8e5f2&spaceId=e549a579-2c2f-4a01-ab69-720e615313ef&width=2000&cache=v2)
#### 学习weil配对之前需要知道的数学工具
> **挠点（torsion points）**指在椭圆曲线上具有有限阶的点。即存在正整数 \(m \geq 1\)，使得对于椭圆曲线 \(E\) 上的点 \(P\)，有 \(mP = O\)，其中 \(O\) 是曲线的单位元（无穷远点）。点 \(P\) 的阶即为    m，此时 P 被称为挠点。
我们可以将所有阶为 m的倍数的点集合起来，就构成了 m-挠群（torsion group），定义为：
$$
E[m] = \set{P \in E: mP = O}
$$
挠群 \(E[m]\) 是椭圆曲线 E 的一个加法子群，其中单位元是无穷远点 O。如果 \(P, Q \in E[m]\)，则 \(P+Q\) 和 \(Q-P\) 也属于 \(E[m]\)，说明挠群具有封闭性和存在逆元，因此构成一个群结构.
当椭圆曲线定义在域 \(K\) 上时，m-挠群记为 \(E(K)[m]\)。例如，定义在 \(\mathbb{F}_p\) 上的 \(E\) 的 m-挠群写作 \(E(\mathbb{F}_p)[m]\).
---
**除子（Divisor）**是椭圆曲线上点的形式和，可以看作是一种“加权”点的概念，用于追踪有理函数上的零点和极点。如果在 \(\alpha_1\) 处有 \(e_1\) 个零点，就记为 \(e_1 [\alpha_1]\)；如果在 \(\beta_1\) 处有 \(d_1\) 个极点，就记为 \(- d_1 [\beta_1]\)，然后再把它们形式的加起来（不是普通的加法运算）：
$$
\text{div}(f) = e_1 [\alpha_1] + ... + e_r [\alpha_r] - d_1 [\beta_1] - ... - d_s [\beta_s] 
$$
举个例子，多项式 \(f(x) = \frac{x^3 - 2x}{2(x^2 - 5)}\)，它的零点为 \(\set{0, \sqrt{2}, - \sqrt{2}}\)，极点为 \(\set{\sqrt{5}, - \sqrt{5}}\)，这些零点和极点的重数都为 1。因此它的除子：
$$
\text{div}(f) = [0] + [\sqrt{2}] + [- \sqrt{2}] - [\sqrt{5}] - [- \sqrt{5}] 
$$
这些都很抽象啦，下面我们举个结合挠群和除子的例子
我们选一个定义在 \(F_5\) 上的椭圆曲线 \(E: y^2 = x^3 - x \pmod{5}\)。
![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fe549a579-2c2f-4a01-ab69-720e615313ef%2F9e9e4660-a1dc-462d-bfc4-332b214870aa%2F35-3.png?table=block&id=d27dba72-1a7f-420c-96b9-bb4be7d58e0f&spaceId=e549a579-2c2f-4a01-ab69-720e615313ef&width=2000&cache=v2)
它有3个根，分别为 \(0, -1, 1\)，对应椭圆曲线上的3个点 \(P_1(0,0), P_2(-1,0), P_3(1,0)\)。由于他们满足 \(2P_1 = 2P_2 = 2P_3 = O\)，因此它们加上无穷远点 \(O\) 构成了椭圆曲线的 \(2\)-挠群 \(E[2] = \set{P_1, P_2, P_3, O}\)。可以看到 \(E[2]\) 包含4个点，同构于 \(\mathbb{Z}/2\mathbb{Z} \times \mathbb{Z}/2\mathbb{Z}\)。
我们可以定义在 \(E[2]\) 上的有理函数 \(f(x, y) = \frac{x + 1}{x-1}\)。这个函数在 \(P_2\) 有一个零点，在 \(P_3\) 有一个极点。因此，它的除子可以写为：
$$
D = \text{div}(f) = [P_2] - [P_3]
$$
这时候就有同学会问了，主播主播，这两个数学工具和我们的weil配对有什么关系呢？
> 💡 
 \(m\)-挠群包含椭圆曲线上阶为 \(m\) 的点，为 Weil 配对提供了一个有限的、结构良好的点群；
> 💡 
除子基于有理函数，是椭圆曲线上点的形式和，支持 Weil 配对在数学上精确地描述点之间的配对关系。
我们接着来看weilpairing
#### weilpairing
Weil 配对是一种基于椭圆曲线的双线性配对。这里偷个懒省去了一系列的推导过程，我们直接来看它的性质和常用形式
> 性质
> 💡 
性质1. 双线性：对于 \(E[m]\) 上的挠点 \(P, Q, R\)，满足 \(e_m(P + R, Q) = e_m(P, Q) e_m(R, Q)\) 和 \(e_m(P, Q + R) = e_m(P, Q) e_m(P, Q)\).  【\(e([m]P,[n]Q)=e(P,Q)^{mn}\)】
> 💡 
性质2. 非退化性：对于所有的 \(P \in E[m]\)，有 \(e_m(P,Q) = 1\) 成立，那么 \(Q = O\)。
> 💡 
性质3.交错性： \(e_m(P, Q) = e_m(Q, P)^{-1}\)，且 \(e_m(P, P) = 1\)
> 💡 
性质4.阶的性质：如果P和Q的阶为 \(o\),则  \(e(P,Q)^o=1\)
> weil配对的常用形式
$$
e_m(P, Q) = \frac{f_P(Q+X)}{f_P(X)} / \frac{f_Q(P-X)}{f_Q(-X)}
$$
其中点 \(P, Q\) 属于椭圆曲线的 \(m\)-挠群 \(E[m]\)，点 \(X\) 为椭圆曲线上满足 \(X \notin \set{O, P, -Q, P-Q}\) 的任意一点，函数 \(f_P\) 和 \(f_Q\) 为定义在椭圆曲线上的有理函数，满足：
$$
\text{div}(f_P) = m[P] - m[O]
$$
$$
\text{div}(f_Q) = m[Q] - m[O]
$$
这个形式的 Weil 配对 \(e_m(P, Q) \in \mu_m\)，同样满足上一节的性质：双线性，非退化性，和交错性。
