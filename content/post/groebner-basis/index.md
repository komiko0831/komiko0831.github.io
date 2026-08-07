---
title: "Gröbner 基"
description: ""
slug: groebner-basis
date: 2025-04-01 00:00:00+0000
categories:
    - 密码学
tags:
    - crypto
    - note
math: true
---

在 crypto CTF中 用`**Ideal.groebner_basis()**` 来解方程组并不少见，今天学习了一下，希望能通过这篇文章来理解一下什么是Ideal什么是groebner basis。
## 前置：
### 单项式序
令 \(K\) 为域,考虑 \(K\) 上的 \(n\) 元多项式环 \(K[x1, ..., xn]\)。
> 💡 
#### 定义(单项式序)
\(K[x1, ..., xn]\) 上的单项式序是其上所有(首一)单项式 \(x^a=x_1^{a_1}\ldots x_n^{a_n}\) 之间满足如下两条性质的全序 :
1. 常数多项式为最小元。也即对任意的 \(a\in\mathbb{N}^n\setminus\{0\},1<x^a\)。
1. 对任意的 \(a, b, c ∈ \mathbb N^n\), 若 \(x^a < x^b\) 则有 \(x^{a+c} < x^{b+c}\)。
对于一般的单项式来说,差一个非零常数因子意义下相同的单项式在序关系 < 下相等, 这也就是同类项在多项式的典范表示中合并的道理。
> 💡 
#### 注:
由定义,一元多项式环 \(K[x]\) 上只有唯一的单项式序,也即单项式的次数作为自然数诱导的序:
\(1<x<x^2<\cdots<x^m<x^{m+1}<\cdots \)
给定多项式环 \(K[x1, ..., xn]\) 上的单项式序 \(<\) 后,每个多项式 f 都有唯一的首项,也就是在 f 中出现的非零系数的 \(<\) 最大的单项式 \(\lambda_ax^a\), 记为 \(LT_<(f)\)。首项系数 \(λ_a\) 记为 \(LC_<(f)\),对应的首一单项式 \(x^a\) 记为 \(LM_<(f)\)。在不引起混淆时也可以省略 < 记号。
> 💡 
#### 定义1.3(首项理想)
给定多项式环 \(K[x_1,\cdots,x_m]\)上的单项式序 < 以及理想 I, I的首项理想是由 I中所有多项式的首项生成的理想, 记为 \(LT(I)\).
#### 例 1.4
\(K[x_1,\cdots,x_m]\)上的理想 \(I = <x_1+1>\)对应的首项理想是 \((x_1)\). 这说明理想与其对应的首项理想之间一般不存在相互包含关系.
## Gröbner Basis 的定义
### 我们关注它能做什么：
1. **求解多项式方程组**
  - Gröbner 基可以用于确定多项式方程组是否有解，并找到所有可能的解。  - 其作用可以类似于线性代数中的 **高斯消元法**，用于非线性多项式系统的求解。
1. **Gröbner 基在公钥密码学中的应用**
  - 基于 **多变量多项式** 的密码系统，**Gröbner 基**可以用来求解这些方程组
这样听起来还是十分难以理解，下面给出比较直观的理解：
### **Gröbner Basis 的直观理解**
在一元线性方程中，我们可以通过**高斯消元法**来解方程组。例如：
$$
\begin{cases}3x+2y=5\\6x+4y=10&\end{cases}
$$
我们发现第二个方程是第一个方程的两倍，所以这个方程组本质上只有一个独立的方程。
在**一元多项式**的情况下，我们使用 **最大公因式（GCD）** 来找到一个公共约束：
$$
f_1(x)=x^3-x\\f_2(x)=x^2-1
$$
它们的 **GCD** 是 \(x-1\)，所以它们的解就是 \(x= 1\).
那么，在 **多元多项式**（如 \(x,y,z\)）的情况下，我们需要一种类似的工具来消去变量，并得到一个更简单的等价表示，这就是 **Gröbner Basis** 的作用。
#### 定义：
设 \(k[x_1,x_2,\cdots,x_3]\)是一个多项式环，给定一个多项式的集合
$$
F=\{f_1,f_2,...,f_m\}\subset k[x_1,x_2,...,x_n]
$$
记作以 \(x\)为变量的多项式环 \(R[x]\)
> 💡 
**Ideal：**
一个 理想 \(I\)是 \(R[x]\)的一个子集，
$$
I=\langle f_1,f_2,...,f_m\rangle 
$$
满足以下两个条件：
1. 若 \(f(x)\in I\)，且 \(g(x)\in I\)，则 \(f(x)+g(x)\in I\)（加法封闭性）
1. 若 \(f(x)\in I\)，且 \(g(x)\in R[x]\)，则 \(f(x)*g(x) \in I\)（乘法封闭性）

Gröbner 基的最重要性质是：**它允许我们用 "除法" 操作来测试一个多项式是否属于该理想**，就像一元多项式可以用**辗转相除法**来判断是否能整除一样。

## **Ideal.groebner_basis()用法**
sage的`**Ideal.groebner_basis()**` 函数用来求groebner基非常的方便
以一个简单的RSA问题作为例子;
```Python
from Crypto.Util.number import *

p, q = getPrime(256), getPrime(256)
N = p * q
m1 = bytes_to_long(b"flag{12345678901234567890")
m2 = bytes_to_long(b"1234567890123456789012345")
m3 = bytes_to_long(b"6789012345678901234567890}")
e = 17
c1 = pow(m1, e, N)
c2 = pow(m2, e, N)
c3 = pow(m3, e, N)
s = m1 + m2 + m3
print(c1, c2, c3, s)

```
题目给了我们在 \(Zmod(N)\)下有四个多项式
$$
x ^{17}-c _1,\\y ^{17}-c_2,\\z^{17}-c_3,\\x+y+z-s
$$
```Python
R = PolynomialRing(Zmod(N), 'x, y, z')
x, y, z = R.gens()
I = Ideal([x^e - c1, y^e - c2, z^e - c3, s - x - y - z])
for g in I.groebner_basis():
    print(g)
```
\(x,y,z\)作为参数构造模N下的环，先求四个多项式的理想，然后直接用`I.groebner_basis()` 解就好了。
会有以下的输出
```Python
x + 8200508147491951798120906107533560089138740799523516997878446708243664748597643056570935725894533617402947792194137765592419341413867744140575655826790573
y + 8200508147491951798120906107533560089138740799523516997878446708243664748597643056570935725894867730375893531643622682181338943540709032181545167583747496
z + 8200508147491951798120906107533560089138740799523516997878446708243664748597643056570935725808055265682740432338427024728626953197650464755288088055292256
```
我们得到了 Gröbner 约化基的形式如下：
\(x+a_1=0,\quad y+a_2=0,\quad z+a_3=0\)
通过取模 \(N\)取相反数，我们可以得到解：
```Python
x = (-a1) % N
y = (-a2) % N
z = (-a3) % N

m1 = long_to_bytes(x)
m2 = long_to_bytes(y)
m3 = long_to_bytes(z)

flag = m1 + m2 + m3
print(flag.decode())
#flag{1234567890123456789012345678901234567890123456789012345678901234567890}
```
## 再多看几个例题
### Lv 1 ：LCG+Gröbner basis
感觉groebner basis和lcg放在一起考察比较多
#### **题目：Dragon Knight CTF**
```Python
from Crypto.Util.number import *
 
m = b'flag{********}'
a =  getPrime(247)
b =  getPrime(247)
n =  getPrime(247)
 
seed = bytes_to_long(m)
 
class LCG:
    def __init__(self, seed, a, b, m):
        self.seed = seed  
        self.a = a  
        self.b = b  
        self.m = m  
 
    def generate(self):
        self.seed = (self.a * self.seed + self.b) % self.m
        self.seed = (self.a * self.seed + self.b) % self.m
        return self.seed
 
seed = bytes_to_long(m)
 
output = LCG(seed,a,b,n)
 
for i in range(getPrime(16)):
    output.generate()
 
print(output.generate())
print(output.generate())
print(output.generate())
print(output.generate())
print(output.generate())
```
#### 分析：
LCG生成种子： \(x_2 = (x_1 *a+b）mod\ n\)
\(x_3= (a *(x_1*a+b)+b )mod\ n\)
给了五组输出，有a,b,n三个未知数，考虑构造groebner basis解方程组
#### exp：
```JavaScript
v = [5944442525761903973219225838876172353829065175803203250803344015146870499,
141002272698398325287408425994092371191022957387708398440724215884974524650,
42216026849704835847606250691811468183437263898865832489347515649912153042,
67696624031762373831757634064133996220332196053248058707361437259689848885,
19724224939085795542564952999993739673429585489399516522926780014664745253]
 
P.<a,b,n>=PolynomialRing(ZZ)
#a = A^2 ,b = ab+b
F = [a*v[i-1]+b - v[i] for i in range(1,len(v))]
ideal = Ideal(F)
I = ideal.groebner_basis()
print(I)
# 求解参数a b n
res=[x.constant_coefficient() for x in I]
n = res[2]
a = -res[0]%n
b = -res[1]%n
 
n = 155908129777160236018105193822448288416284495517789603884888599242193844951
a = 64626559347206047394060455019203284495581399273613957197572789423633157340
b = 70007026214493722017071763643415747654706858246084218560508331533479420820
a_ = inverse_mod(a,n)
x = v[0]
for i in range(0x10000):
    x = (x-b)*a_%n 
    if i>0x8000 and b'flag' in long_to_bytes(int(x)):
        print(long_to_bytes(int(x)))
#flag{Hello_CTF}

```
#### 我的改进
考虑到这里直接print，显然降低了难度，我们改成交互的
把他作为chall1
增加LCG pro max 作为 chall 2
如果chall 2 通过
send flag
flag从环境变量中获取，做动态处理
动态生成flag’？
### Lv 2：和上一题几乎一致
#### 题目:**LinearEquations**
```Python
from Crypto.Util.number import*
from secret import flag
assert flag[:5] == b'cazy{'
assert flag[-1:] == b'}'
flag = flag[5:-1]
assert(len(flag) == 24)

class my_LCG:
    def __init__(self, seed1 , seed2):
        self.state = [seed1,seed2]
        self.n = getPrime(64)
        while 1:
            self.a = bytes_to_long(flag[:8])
            self.b = bytes_to_long(flag[8:16])
            self.c = bytes_to_long(flag[16:])
            if self.a < self.n and self.b < self.n and self.c < self.n:
                break
    
    def next(self):
        new = (self.a * self.state[-1] + self.b * self.state[-2] + self.c) % self.n
        self.state.append( new )
        return new

def main():
    lcg = my_LCG(getRandomInteger(64),getRandomInteger(64))
    print("data = " + str([lcg.next() for _ in range(5)]))
    print("n = " + str(lcg.n))

if __name__ == "__main__":
    main() 

# data = [2626199569775466793, 8922951687182166500, 454458498974504742, 7289424376539417914, 8673638837300855396]
# n = 10104483468358610819
```
#### 分析：
改了一些的LCG；由五组数据，我们有：
$$
\begin{aligned}&x_2=(a\cdot x_1+b\cdot x_0+c)\mod n\\&x_3=(a\cdot x_2+b\cdot x_1+c)\mod n\\&x_4=(a\cdot x_3+b\cdot x_2+c)\mod n\end{aligned}
$$
有三个未知参数 \(a,b,c\),同样建立方程用groebner basis求解。
#### exp：
```Python
d = [2626199569775466793, 8922951687182166500, 454458498974504742, 7289424376539417914, 8673638837300855396]
n = 10104483468358610819
PR.<a,b,c> = PolynomialRing(Zmod(n))
f1 = (a*d[1]+b*d[0]+c-d[2])
f2 = (a*d[2]+b*d[1]+c-d[3])
f3 = (a*d[3]+b*d[2]+c-d[4])
Fs = [f1, f2, f3]
I = Ideal(Fs)
B = I.groebner_basis()
m = b''
for b in B:
    assert b.degree() == 1
    mi = ZZ(-b(0, 0, 0))
    m += bytes.fromhex(hex(mi)[2:])
print(m)

# b'L1near_Equ4t1on6_1s_34sy'
```
