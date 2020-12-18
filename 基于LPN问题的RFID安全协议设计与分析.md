# 基于LPN问题的RFID安全协议设计与分析

> 因为低成本的tag具有例如计算能力低、存储空间小等特点与人类类似，从而RFID协议设计者们考虑将“Human-computer"认证协议应用到FRID的安全认证之中。同时以此为基础，积极寻找合适的计算问题来构建安全的认证协议。



## LPN问题以及HB类协议

### LPN问题

#### 背景

假设User 与Computer之间共享$k$比特密钥$x$, U想向C证明自己的身份，认证过程如下：

1. C产生一个随机的$k$比特矢量$a$发送给U
2. U收到后计算$c=a\cdot x$响应给C（其中$\cdot$表示GF(2)上的点乘）
3. C收到响应后检查，如果$c = a\cdot x$则认证通过，反之认证不通过

说明：

在一轮验证过程中，C接受到一个假冒用户的概率为1/2，重复$r$轮之后，理论上接受到一个假冒用户的概率是$2^{-r}$ ，而被动攻击这只需要观察$O(k)$次”challenge-response pair“，就可以通过高斯消元法得到密钥$x$，进而伪装成U。为了防止这一点，我们引入一个噪声参数$\eta \in (0, \frac{1}{2})$ ，在U响应中参加一些错误的回答，这样被动攻击者就无法通过简单的高斯消元发而得到密钥$x$，这就是LPN(Learning parity with the presence of Noise)问题，

#### 定义

$假设D是一个随机的q\times k比特矩阵，x是一个随机的k比特矢量，噪声参数\eta\in(0,\frac{1}{2}),$ $v是一个随机的q比特矢量，其汉明距离|v| \leq \eta q。$ $已知D，\eta以及z = (D\cdot x)\oplus v，找到一个k比特矢量x^{'}满足|(D\cdot x^{'})\oplus v|\le \eta q$

---

### HB协议

[HB协议$^{[2]}$](##参考文献) 是Hopper和Blum基于LPN问题提出的，一个完整的HB协议包括$r$轮，其中$r$是一个安全参数，Reader与Tag共享$k$比特密钥$x$。Tag拥有一个噪声发生器，以$\eta\in (0,\frac{1}{2})$的概率产生噪声 $v = \{0,1|prob[v=1]=\eta\}$。

在一轮协议中：

1. Reader随机产生$k$比特序列$a$发给Tag
2. Tag收到后计算$z = (a\cdot x)\oplus v$,响应给Reader
3. Reader检验是否有$z = a\cdot x$

在r轮之后，如果Tag响应的错误论述小于$\eta r$ ,则判断Tag通过了认证，否则没有通过认证。该协议对于Tag来说，只需要完成AND和XOR操作以及产生噪声这两部分操作，因此在硬件上来说，该协议比较容易实现。

HB协议在抗被动攻击方面的安全性可以归约到困难问题LPN问题，因此**HB协议可以抵抗在线窃听等被动攻击，但是它并不能抵抗主动攻击。**

---

### HB$^{+}$协议

为了**解决HB协议中的不能抵御主动攻击的问题**，Juels和Weis设计了[HB$^{+}$协议$^{[3]}$](##参考文献)。该协议所采用模式与前者类似，但是与HB协议不同的是，该协议中的Reader与Tag之间除了原本共享的$k$比特密钥$x$外还增加了一个共享的$k$比特密钥$y$，相应地，$z$的计算方式也发生了改变；另外，该协议需要Tag首先产生$k$比特盲因子$b$发给 Reader。

一轮HB$^{+}$协议过程如下：

1. Tag产生$k$比特盲因子$b$，将其发送给Reader
2. Reader将随机产生的$k$比特序列$a$发给Tag
3. Tag计算$z = (a\cdot x)\oplus (b\cdot y) \oplus v$，并将其发送给Reader
4. Reader检验是否有$z = (a\cdot x)\oplus (b\cdot y)$

在r轮之后，如果Tag响应的错误论述小于$\eta r$ ,则判断Tag通过了认证，否则没有通过认证。与HB协议相比，该协议需要Tag生成一个$k$比特忙因子$b$，同时所需要的AND和XOR操作为2$r$次。

#### HB$^{+}$协议的改进

##### HB$^{+}$协议安全性（MITM）

尽管协议作者证明了该协议在抗主动攻击方面的安全性，但是Henri Gilbert等发现了一个简单的中间人攻击。

该攻击过程如下：

1. 攻击者截获比特序列$a$，计算$a^{'} = a \oplus \delta$,其中$\delta$ 在每一轮都不改变，将比特序列$a^{'}$发送给Tag
2. 如果r轮过后，Tag通过认证测试，则大概率可以判断$\delta\cdot x =0$,否则可以判断$\delta \cdot x = 1$大概率成立
3. 选择k个恰当的$\delta$可以恢复出密钥$x$的每个比特 

---

### HB$^{\#}$ 协议

#### 第一种协议（HB$^{+}$的简单改进、唐&姬提出）

引入一个非线性函数$f$,假设k比特矢量$a = a_1,a_2,...,a_{k-1},a_k, f(a) = f(a_1,a_2,a_3,...,a_{k-1},a_k) = (a_1a_2,a_2a_3,...,a_{k-1}a_k,a_ka_1)$。Tag计算$z=(f(a)\cdot x)\oplus(b\cdot y)\oplus v$。

具体一轮协议过程如下:

1. Tag产生$k$比特盲因子$b$，将其发送给Reader
2. Reader将随机产生的$k$比特序列$a$发给Tag
3. Tag计算$z=(f(a)\cdot x)\oplus(b\cdot y)\oplus v$，并将其发送给Reader
4. Reader检验是否有$z = (f(a)\cdot x)\oplus (b\cdot y)$

因为$f(a\oplus\delta) \neq f(a)\oplus f(\delta)$，所以该方式可以抵御前文所描述的简单MITM。

#### 第二种协议(Henri Gilbert等人提出)

RANDOM-HB$^{\#}$协议与HB$^{\#}$协议相比HB$^{+}$协议，提供了更优的错误率以及减少了通信负载。在文章中，Gilbert等人还提出了HB$^{+}$协议所存在的一些问题，除了一些合法的Tag会被拒绝认证之外，复杂且频繁的Tag与Reader之间的通信使得HB$^{+}$协议使用起来更加困难。

RANDOM-HB$^{\#}$协议解决了HB$^{+}$协议存在的一些问题，该协议可以保证在detection-base (DET)模型下的安全以及抵御包括GRS攻击（GRS-MIM-model）在内的一类攻击。但是RANDOM-HB$^{\#}$协议允许一个主动攻击者以他们希望的方式改变从reader获取的任何信息，并且观察reader是否接受或者拒绝这一次认证过程。

而HB$^{\#}$协议则在保持HB$^{+}$协议计算效率的前提下，与RANDOM-HB$^{\#}$协议相比拥有不那么大的存储要求，因为RANDOM-HB$^{\#}$协议中的Tag需要保存两个随机的二进制矩阵，而其中矩阵参数都是三位数，因此RANDOM-HB$^{\#}$协议下Tag的存储花销会变得非常的难以处理。

##### RANDOM-HB$^{\#}$协议

RANDOM-HB$^{\#}$协议在HB$^{+}$协议的基础上将$k$比特密钥$x$与$y$，改变成$k_X \times m$以及$k_Y \times m$大小的比特矩阵密钥$X$和$Y$

该协议一轮过程如下:

1. Tag产生$k_Y$比特盲因子$b$，将其发送给Reader
2. Reader将随机产生的$k_X$比特序列$a$发给Tag
3. Tag计算$z=(a \cdot X)\oplus(b\cdot Y)\oplus v$，并将其发送给Reader
4. Reader检验是否有$Hwt[(a \cdot X)\oplus(b\cdot Y)\oplus z] \leq um$, 其中$um$为认证成功阈值参数，$u \in (\eta, \frac{1}{2})$

##### HB$^{\#}$协议

该协议与RANDOM的不同之处在于，两个比特矩阵密钥$X$和$Y$ 变成了两个随机的$k_X \times m$以及$k_Y \times m$大小的托普利兹矩阵。

托普利兹矩阵

1. 托普利兹矩阵完全由其第1行和第1列的2n-1个元素确定。
2. 托普利兹矩阵沿平行主对角线的每一对角线上的元素是相等的，是关于交叉对角线对称的。
3. 除第一行第一列外，其他每个元素都与左上角的元素相同。
4. 矩阵中的各元素关于次对角线对称，即T型矩阵为次对称矩阵。

## 参考文献

[1] 《基于LPN问题的RFID安全协议设计与分析》.唐静，姬东耀

[2]《Secure human identification protocols》. Hopper N J and Blum M.

[3]《Authenticating pervasive devices with human protocols》. Juels A and Weis S.

[4]《HB$^{\#}$: Increasing the security and Efficiency of HB$^{+}$》.Henri Gilbert, Matthew J.B. Robshaw and Yannick Seurin