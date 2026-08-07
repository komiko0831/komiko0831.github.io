---
title: "Extending Wiener‘Attack学习"
description: ""
slug: extending-wiener-attack
date: 2024-10-13 00:00:00+0000
categories:
    - 密码学
tags:
    - crypto
    - note
math: true
---

## 一．介绍该论文想要解决的问题
基于维纳攻击，解决对于给定N，有多个加密指数e，并且都带有小解密指数d的RSA问题。
## 二．一些需要铺垫的知识
### 连分数
连分数就是对一个数进行连续分式展开，形式为：
$$
a_0+\cfrac{1}{a_1+\cfrac{1}{a_2+\cfrac{1}{\ddots+\cfrac{1}{a_n}}}},
$$
$$
一般记为[a_0 ;a_1 ,a_2 ,...,a_n ]
$$
### 连分数的收敛
连分数的收敛：连分式的每一次分式展开，对于每一个收敛都是从两边不断逼近这个数
$$
\forall i\in[0,n],\quad c_i=[a_0;a_1,...,a_i]
$$
### 勒让德定理
$$
当满足|a-\frac cd|<\frac1{2d^2}时，\frac cd是a的一个连分数收敛。
$$
### 维纳攻击
若e与N大小大致相同，且有较小的私钥d < N^1/4，可根据勒让德定理计算e/N的连分数来找到与其近似的数k/dg，从而分解出N。
原理与流程如下：
设：
$$
\begin{aligned}&\lambda(N)= \frac{(p-1)(q-1)}g\\&g=gcd(p-1,q-1)\\&s=1-p-q\\&phi = (p-1)(q-1)\\&ed=1+k\lambda(N) \\&ed = 1+k\frac{phi}g\\&edg=g+k*phi\\
\end{aligned}
$$
因为d < N^1/4,满足勒让德定理，所以对e/N进行展开连分数展开得到k/dg。
$$
\left|\frac{e}{N}-\frac{k}{dg}\right|=\left|\frac{1}{dN}+\frac{ks}{dgN}\right|<\frac{ks}{dgN}<\frac{1}{2(dg)^2}
$$
不等式的证明如下
$$
\begin{aligned}证明如下&edg=k(N+s)+g\\&edg=kN+ks+g\\同除dgN,&\frac{e}{N}-\frac{k}{dg}=\frac{g+ks}{dgN}=\frac{1}{dN}+\frac{ks}{dgN}\\&\frac{1}{dN}\text{忽略不计}\\&\frac{kg}{dgN}\approx\frac{1}{N^{1.25}}\\&\frac{1}{2(gd)^{2}}\approx\frac{1}{N^{0.5}}\\&\frac{ks}{dgN}<\frac{1}{2(gd)^{2}}\end{aligned}
$$
我们已经知道k/dg
$$
\begin{aligned}\frac{edg}{k}=\varphi(N)+\frac{g}{k}(g不大，故可忽略不计) \\取整[\frac{edg}{k}]=(p-1)(q-1) \\\because\frac{pq-(p-1)(q-1)+1}{2}=\frac{P+q}{2} \\\Rightarrow\frac{N-Phi+1}{2}=\frac{P+q}{2}\end{aligned}
$$
所以通过检验（（p-q）/2）^2是否为平方数来确定我们找到的连分数是否正确，然后已知p+q，p*q，可以通过维达求解出p，q实现大数N的分解。
sagemath实现：
```Python
#wiener_attack
def possible(e,alist,N): 
    for x in alist:
        if x==0:
            continue
        phi = floor(e*(1/x))
        if (N-phi+1)%2==0 and sqrt(pow((N-phi+1)//2,2)-N).is_integer():
                (p,q)=var('p,q')
                x=solve([(p-1)*(q-1)==phi, p*q==N],p,q)
                return int(str(x[0][0]).split('==')[1])
        else:
            continue

def wiener_attack(e,N):
    c=continued_fraction(e/N)
    alist=c.convergents()
    return possible(e,alist,N)

n=
e=
t=wiener_attack(e,n)
print t

```
##   三.针对这个研究问题，前人怎么解决该问题，并且有哪些不足？
### 3.1 Guo's Approach（e_1,e_2,e_3)
$$
\begin{aligned}&e_{1}d_{2}g-g=k_{2}(p-1)(q-1) \\&e_2d_2g-g=k_2(p-1)(q-2) \\&\text{两式相除}\Rightarrow k_2d_1e_1-k_1d_2e_2=k_2-k_1 \\&\text{同除}k_2d_1e_2\Rightarrow\frac{e_1}{e_2}-\frac{k_1d_2}{k_2d_1}=\frac{k_2-k_1}{k_2d_1e_2}\end{aligned}
$$
![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fe549a579-2c2f-4a01-ab69-720e615313ef%2Fc2289465-0a75-4b83-887c-d5bc53913416%2FQQ20241011-213818.png?table=block&id=120f19d3-b212-80d9-8666-c2d736e82251&spaceId=e549a579-2c2f-4a01-ab69-720e615313ef&width=2000&cache=v2)
因此可以通过e1/e2的连分数展开得到（k1d2）/（k2d1）
#### 存在的不足
1.得到（k1d2）/（k2d1）
之后还是不能对N进行分解，也无法找到d2或者k1
2.k1d2和d2k1很有可能不互素，也就是得到的连分时展开是化简后的分式。
（后续Guo有提议尝试对k1d2进行分解，但是这里不做详细阐述哩）
## 四. Extending Wiener’s Attack
### 4.1 Overview
作者为了将分析扩展到n个加密指数e，解密指数d很小*（首先假设d<N^alpha)，**同时结合了Wiener和Guo的方法**，构造出多项式组，展开矩阵方程，通过放缩，使得满足LLL算法条件，求解多项式，解出私钥d。
![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fe549a579-2c2f-4a01-ab69-720e615313ef%2F0407b543-d269-42d3-b2c7-11699e9c8d86%2Fimage.png?table=block&id=120f19d3-b212-80fa-b553-c235ead90a87&spaceId=e549a579-2c2f-4a01-ab69-720e615313ef&width=2000&cache=v2)
### 4.2  Let’s start Extending Wiener’s Attack with 2 small decrypto exponents.
#### 1>选取多项式
1,W1 , G(1,2) , W1*W2
$$
\begin{aligned}&&& k_1k_2 = k_1k_2\\&&W1:d_1ge_1-k_1N& =g+k_1s, \\&&G_{(1,2)}:k_1d_2e_2-k_2d_1e_1& =k_1-k_2, \\&&W_1W_2:d_{1}d_{2}g^{2}e_{1}e_{2}-d_{1}gk_{2}e_{1}N-d_{2}gk_{1}e_{2}N+k_{1}k_{2}N^{2}& =(g+k_1s)(g+k_2s). \end{aligned}
$$
#### 2>构造矩阵
对W1乘以k2，G(1,2)乘以g这样四个等式左边都含有：
![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fe549a579-2c2f-4a01-ab69-720e615313ef%2F857047c2-9891-4708-9ae9-515721c47c0f%2F%25E5%259B%25BE%25E7%2589%25871.png?table=block&id=120f19d3-b212-8037-b3ad-d9e2293b3e13&spaceId=e549a579-2c2f-4a01-ab69-720e615313ef&width=2000&cache=v2)
$$
b = (k_1k_2,d_1gk_2,d_2gk_1,d_1d_2g^2)\\L=\begin{pmatrix}1&-N&0&N^2\\0&e_1&-e_1&-e_1N\\0&0&e_2&-e_2N\\0&0&0&e_1e_2\end{pmatrix}\\a=\begin{pmatrix}k_1k_2,k_2(g+k_1s),g(k_1-k_2),(g+k_1s)(g+k_2s)\end{pmatrix}
$$
所以v*L = w
$$
\begin{aligned}\text{也就是}\\(k_{1}k_{2},d_{1}gk_{2},d_{2}gk_{1},d_{1}d_{2}g^{2})&\left(\begin{array}{ccc}1&-N&0&N^{2}\\&e_{1}&-e_{1}&-e_{1}N\\&&e_{2}&-e_{2}N\\&&&e_{1}e_{2}\end{array}\right)=\\&(k_{1}k_{2},k_{2}(g+k_{1}s),g(k_{1}-k_{2}),(g+k_{1}s)(g+k_{2}s).\end{aligned}
$$
#### 3>使用LLL求解最短向量w（近似）
简单介绍LLL格基规约算法吧
---
Lenstra-Lenstra-Lovasz对LLL算法的初步理解
1.通过近似求解最短向量问题的算法（并不是直接找最短，而是在一定范围内近似找最短）
2.在一定限制下*解出vL = w中的w（不一定需要n个变量n个方程）
*：最短向量的上界([https://en.wikipedia.org/wiki/Minkowski's_theorem](https://en.wikipedia.org/wiki/Minkowski%27s_theorem))
![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fe549a579-2c2f-4a01-ab69-720e615313ef%2F59970343-1d93-4e57-bc9d-55a16235088e%2FQQ20241009-234211.png?table=block&id=120f19d3-b212-8088-bf71-fb03df328888&spaceId=e549a579-2c2f-4a01-ab69-720e615313ef&width=2000&cache=v2)
对于n维的格L，有非零向量v属于L，满足
||v||表示格基的数量积，**n为维度**，det(L)为格L(矩阵)的行列式
---
在扩展维纳这里理解为在满足一个不等式的前提下,可在未知v的情况下求解出w。
我们这里的||a||显然不满足，故进行放缩
#### 4>构造对角线矩阵。
![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fe549a579-2c2f-4a01-ab69-720e615313ef%2Fb1ebde44-13cf-491d-8236-a6891667bc87%2FQQ20241012-232700.png?table=block&id=120f19d3-b212-8079-8f2a-dc422a6cd95d&spaceId=e549a579-2c2f-4a01-ab69-720e615313ef&width=2000&cache=v2)
![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fe549a579-2c2f-4a01-ab69-720e615313ef%2Fdcd70cb2-3930-42d1-bce4-0f6359d68564%2F%25E5%259B%25BE%25E7%2589%25871.png?table=block&id=120f19d3-b212-808b-96f5-d8bcf1e4db91&spaceId=e549a579-2c2f-4a01-ab69-720e615313ef&width=2000&cache=v2)
$$
\text{最终我们构造的矩阵为}\\\\L_2=\begin{pmatrix}1&-N&0&N^2\\&e_1&-e_1&-e_1N\\&&e_2&-e_2N\\&&&e_1e_2\end{pmatrix}*D
$$
![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fe549a579-2c2f-4a01-ab69-720e615313ef%2F6dd34be9-9a1a-4c87-b629-b76fe8539f64%2F%25E5%259B%25BE%25E7%2589%25873.png?table=block&id=120f19d3-b212-8008-b9ec-c44362df7895&spaceId=e549a579-2c2f-4a01-ab69-720e615313ef&width=2000&cache=v2)
b*L2 = a2
![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fe549a579-2c2f-4a01-ab69-720e615313ef%2F4c620c4c-8599-421c-8b66-328113cf4478%2FQQ20241013-134712.png?table=block&id=120f19d3-b212-809b-8d89-d54a6b68187b&spaceId=e549a579-2c2f-4a01-ab69-720e615313ef&width=2000&cache=v2)
提一下关于alpha的大小，原论文给出了当n = 2，alpha是等于0.36。这里不做详细证明，后面给出了推广到n个密钥对的alpha通式。
![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fe549a579-2c2f-4a01-ab69-720e615313ef%2F74070984-8753-4898-a449-99e8640ba8b0%2F%25E5%259B%25BE%25E7%2589%25874.png?table=block&id=120f19d3-b212-8033-b8a2-f1058581665b&spaceId=e549a579-2c2f-4a01-ab69-720e615313ef&width=2000&cache=v2)
把alpha带入M2，这样就可以使用LLL解出最短向量a2，乘上L2的逆可以求出b
b（b1，b2，b3，b4）
由b的前两项
b2/b1 = d1g/k1
K *Phi = edg - g
Phi = edg/k （- g/k）
那么到这里就可以解出d了
### 4.3  3 small decrypto exponents.
对于三个加密指数e，我们选取的多项式有：
W1 , G(1,2) , W1*W2，G(1,3), W1*G(2,3) , W2*G(1,3）
$$
\begin{aligned}&\text{} \\&B=( k_1k_2k_3\quad d_1gk_2k_3\quad k_1d_2gk_3\quad d_1d_2g^2k_3\quad k_1k_2d_3g\quad k_1d_3g\quad k_1d_3g\quad d_1d_2d_3g^3)\\&然后我们便可以构造格 \\&\text{L} =\begin{pmatrix}1&-N&0&N^2&0&0&0&-N^3\\0&e_1&-e_1&-Ne_1&-e_1&0&Ne_1&N^2e_1\\0&0&e_2&-Ne_2&0&Ne_2&0&N^2e_2\\0&0&0&e_1e_2&0&-e_1e_2&-e_1e_2&-Ne_1e_2\\0&0&0&0&e_3&-Ne_3&-Ne_3&N^2e_3\\0&0&0&0&0&e_1e_3&0&-Ne_1e_3\\0&0&0&0&0&0&e_2e_3&-Ne_2e_3\\0&0&0&0&0&0&0&e_1e_2e_3\end{pmatrix} \end{aligned}
$$
$$
D=diag(\begin{matrix}N^{\frac{3}{2}}&N&N^{a+\frac{3}{2}}&\sqrt{N}&N^{a+\frac{3}{2}}&N^{a+1}&N^{a+1}&1\end{matrix}）
$$
同样我们可以得到
$$
\|bL_2\|<\sqrt{8}N^{3/2+2\alpha_3}
$$
则当
$$
\alpha_3<2/5-\epsilon^{\prime}
$$
时可以通过格基规约求出向量b
### 4.4   4 small decrypto exponents.
对于四个加密指数选取的W1 , G(1,2) , W1*W2，G(1,3), W1*G(2,3) , W2*G(1,3)，G(1,4),W1   *G(2,4),G(1,2)*G(3,4),
G(1,3)*G(2,4),W1*W2*G(3,4),W1*W3*G(2,4),W2*W3*G(1,4),W1W2W3W4
![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fe549a579-2c2f-4a01-ab69-720e615313ef%2F2522c7ee-17c6-4ae7-a4f1-6d9d654fbd28%2FQQ20241012-235804.png?table=block&id=120f19d3-b212-80b0-b5fe-e2114766504a&spaceId=e549a579-2c2f-4a01-ab69-720e615313ef&width=2000&cache=v2)
### 4.5  extend to n.
扩展维纳攻击结合上述三个例子已经详细的阐明了方法细节，但是其中没有讲解如何选取复合关系。其实在原文的附录中给出了复合关系的选取，以及给出了alpha_n的表达式。
![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fe549a579-2c2f-4a01-ab69-720e615313ef%2Fbb009edf-7d28-4287-84e4-f9426f8858b1%2FQQ20241012-150921.png?table=block&id=120f19d3-b212-807c-ae1b-dfa604e39c3e&spaceId=e549a579-2c2f-4a01-ab69-720e615313ef&width=2000&cache=v2)
![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fe549a579-2c2f-4a01-ab69-720e615313ef%2F3dc718b1-747d-49c6-a2bd-c934170b04d3%2FQQ20241012-185318.png?table=block&id=120f19d3-b212-80cd-b5d2-cfb2abff8ebf&spaceId=e549a579-2c2f-4a01-ab69-720e615313ef&width=2000&cache=v2)
