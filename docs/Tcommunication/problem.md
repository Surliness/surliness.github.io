一些脑子混乱的时刻

## 三角函数的正交性

20250815，在0811梳理了傅里叶级数的起源后，突然好奇为什么分解的级数一定三角函数做基底，能不能使用别的正交基比如勒让德基，切比雪夫基？换句话就是，为什么三角函数是正交基？能不能由其级数的定义给出证明？进而有一个疑问：为什么三角函数或者说波动在自然界广泛存在？

[从级数开始的三角函数 （上篇） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/638539407)

[三角函数的级数，为什么能符合它的几何意义？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/416380024)

从级数出发的三角函数，可以使用级数的运算(四则运算、积分、微分等)得到诱导、积化和差等公式。由函数凹凸性与特殊值算得零点，再根据零点定义$\pi$。

## 有关负频率

困惑点：目前(20250712)所学知识中，负频率的产生是便于数学上处理所引入的，但是在频谱搬移的过程中又确确实实会导致混叠的问题，一个数学中引入的概念为什么会约束现实的物理现象？

理解顺序上的问题，应该是负频率的引入刚好也能解释了频率混叠或者混频的现象，并且贴合已有的理论。就像复数的引入刚好解释了三次方程无法表示但是又确确实实存在的根一样。在实际中，两个不同频率比如3 Hz与2 Hz的载波在混频时，相乘后会出现5 Hz和1 Hz的分量，但是当引入负频率后，既符合积化和差的公式的结果，又能够与傅里叶级数的复指数形式贴合得更好。

所以并不应该说是因为存在负频率而需要足够高的载频才能避免混叠，而是因为存在着载频不够高而导致的混频现象，引入了虚构的负频率对频域进行扩充，以方便分析与计算。类似于数域的不断扩充。

> 当然，就像复数一样，或许负频率是真的存在的呢🤔，比如像上课说的是空间观测的角度问题。我们观测$i$的方法是$i^2=-1$，在只有正数的世界观测负数的方法是给它加上一个正数，那么类似的，观测负频率的方法就是频谱搬移。😎

## 传输信号的复指数表达式

20250708

困惑点：常常会看到将信号表示为下面这种形式，$\phi_i$是载波的初相，时间长度为该符号的持续时间，代入计算可以很容易的验证。
$$
s=a+jb\\
s(t)=a\cos(2\pi f_ct+\phi_c)-b\sin(2\pi f_ct+\phi_c)=\real[se^{j\omega_ct+\phi_c}]
$$
下面主要说说自己的理解，顺便解释一下常常出现的"减号是为了便于复指数表达"的缘由。暂时先抛弃上面的写法，实际中传递的信号并不能表示复数，因此所谓的复数符号中，虚部主要是为了代表正交分量，而传输时是使用一对同频正交的三角函数分别传输同相分量和正交分量。如果不考虑脉冲成型与初相，发射的符号一般可以表示为
$$
\vec{s}=(a,b)^T,\vec{\omega}_c=(\cos\omega_ct,\sin\omega_ct)^T\\
s(t)=a\cos(\omega_ct)+b\sin(\omega_ct)=\vec{s}^T\cdot\vec{\omega}_c
$$
上述式子和函数展开成傅里叶级数的表达式很像，不妨把傅里叶级数转为复指数表示的方法直接沿用过来，并记$s=a+jb$，可以得到
$$
\begin{align}
s(t)&=\frac{a-jb}{2}e^{j\omega_ct}+\frac{a+jb}{2}e^{-j\omega_ct}\\
&=\frac{1}{2}(\bar{s}e^{j\omega_ct}+se^{-j\omega_ct})\\
&=\real[\bar{s}e^{j\omega_ct}]\text{ or }\real[se^{-j\omega_ct}]
\end{align}
$$
载波可以用复指数表示的同时，符号也可以用复数表示，对比和一般写法的结果可以知道，如果$s(t)$中写为减号，那么最终结果就可以直接写成$\real[se^{j\omega_ct}]$，类似于符号与载波相乘，表达的含义更为清晰，所以说"减号是为了便于复指数表达"。或者根据上式的前者$\real[\bar{s}e^{j\omega_ct}]$理解为，将符号的共轭调制到复指数载波上。

另一个角度，向量的点乘和复数乘法是有联系的，当将写成$\vec{s}=a+jb,\vec{\omega}_c=\cos\omega_ct+j\sin\omega_ct$时，按照复向量的内积，可以写成$\vec{s}^H\cdot\vec{\omega}_c$，取其实部即$s(t)$。这样就和代数中的复向量内积运算统一，并且保留了载波和符号的表达式都是加号。

> 所以我感觉，所谓的"减号是为了便于复指数表达"并没有做好引入复数的时逻辑上的统一。反而是为了和所认知调制形式，即符号直接乘以载波的表达式相契合。不修正定义，又不修正运算，光修改表达式，不得分😑。

## OFDM的多载波与循环前缀

[The history of orthogonal frequency-division multiplexing [History of Communications] | IEEE Journals & Magazine | IEEE Xplore](https://ieeexplore.ieee.org/document/5307460)

### 数字调制中的多载波体现

困惑点：为什么对序列做离散傅里叶变换与多载波等价？

1. 建议直接参考樊昌信的通信原理中对OFDM原理的讲解。郑君里的信号与系统下册里也有一部分内容讲到。
2. [Fourier transform communication system | Proceedings of the first ACM symposium on Problems in the optimization of data communications systems](https://dl.acm.org/doi/10.1145/800165.805240)
3. [Data Transmission by Frequency-Division Multiplexing Using the Discrete Fourier Transform | IEEE Journals & Magazine | IEEE Xplore](https://ieeexplore.ieee.org/document/1090705)
4. [正交频分复用(OFDM)原理及实现_ofdm原理实现过程-CSDN博客](https://blog.csdn.net/qq_34070723/article/details/88926168)
5. [OFDM原理--使用FFT实现OFDM信号（5） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/32985216077)
6. [InTech-Fourier_transform_based_transmission_systems_for_broadband_wireless_communications.pdf (intechopen.com)](https://cdn.intechopen.com/pdfs/14626/InTech-Fourier_transform_based_transmission_systems_for_broadband_wireless_communications.pdf)

> 2、3是使用DFT实现通信系统的古早论文，6是教程。

## 中断信道容量和各态历经信道容量

[中断信道容量和各态历经信道容量的意义与区别](https://www.zhihu.com/question/55771024/answer/146231527)

> 为什么slow fading channel的情况下我们要考虑outage capacity？
>
> 答：因为对于这个问题，传统的shannon capacity应该是0，也就是不存在一个非0的传输速率可以让差错无限小。
>
> 这个可以参考教材里那幅容量与误码率的图，误码率趋近于零时，容量也趋近于零。

**考虑中断容量和各态历经容量的前提条件：**接收端知道完整的信道信息，而发射端只知道信道的分布。

下面主要参考了《无线通信》(Wireless Communications)(Andrea Goldsmith)

> 带中断的容量(CAPACITY WITH OUTAGE)
>
> 带中断的容量适用于慢变信道，信道的瞬时信噪比在一段传输突发的时间内是恒定的，经过这段时间后依相应的衰落分布变为另一个值。在这种信道中，如果发送端已知此信道突发时间内的信噪比为$\gamma$、就能以速率$B\log_2(1+\gamma)$传输数据而误码率可以做到任意小。由于发送端不知道这个信噪比的数值，所以只能以一个不依赖于瞬时信噪比数值的固定速率进行传输。
>
> 带中断的容量允许在某个突发时段(transmission burst)以一定概率译错所传输的比特。发送端确定一个最小接收信噪比$\gamma_{min}$，再按这个信噪比确定一个速率$C=B\log_2(1+\gamma_{min})$，然后在所有突发中以这个速率传输。如果接收的瞬时信噪比大于或等于$\gamma_{min}$，则能正确译码。若接收信噪比小于$\gamma_{min}$，就**不能以接近1的概率译对**突发中的所有数据比特，此时接收机将指示出现了一次中断。出现中断的概率为$P_{out}= P(\gamma<\gamma_{min})$。在所有突发中，正确传输的概率是$1-P_{out}$，所以平均正确接收的数据速率为$C_{out}=(1-P_{out})B\log_2(1+\gamma_{min})$。数值$\gamma_{min}$**是一个设计参数**，取决于可接受的中断率。



## 波束成形、天线、振子

1. [4G | ShareTechnote](https://www.sharetechnote.com/html/Handbook_LTE_BeamForming.html)
2. [5G大规模天线阵列的振子数，天线数，通道数，流数之间到底是什么关系？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/259402194)
3. [5G怎样实现波束赋形？_波束赋形技术-CSDN博客](https://blog.csdn.net/flypassion/article/details/106453136?ops_request_misc=%7B%22request%5Fid%22%3A%22162842721816780357228367%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fblog.%22%7D&request_id=162842721816780357228367&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-4-106453136.pc_v2_rank_blog_default&utm_term=全向波束与定向波束&spm=1018.2226.3001.4450)

## 关于使用格雷码时BPSK与QPSK误码率

对于BPSK的误码率根据星座图与噪声方差的关系可得
$$
BER_{BPSK}=Q(\frac{A}{\sigma}),\sigma=\sqrt{\frac{N_0}{2}B}
$$
交流信号的幅值与能量的关系为
$$
E_s=\frac{A^2}{2}T\implies A=\sqrt{\frac{2E_s}{T}}=\sqrt{2E_sR_s}
$$
所以
$$
BER_{BPSK}=Q(\frac{A}{\sigma})=Q(\sqrt\frac{4E_sR_s}{N_0B})
$$
传输带宽为符号速率的两倍，故
$$
BER_{BPSK}=Q(\sqrt\frac{2E_s}{N_0})=Q(\frac{2\sqrt{E_s}}{\sqrt{2N_0}})=Q(\frac{d}{\sqrt{2N_0}})
$$
在BPSK中，$d=2\sqrt{E_s}=2\sqrt{E_b}$，在QPSK中，$d=\sqrt{2E_s}=\sqrt{4E_b}$，代入可得两者相同。这里的相同是指在发射端**符号信号能量相同**的条件下，单个符号包含的比特所产生的错误概率相同(如果QPSK的星座点错成相邻星座点时只有一个比特的错误，这其实和编码方式有关)，并不是这种调制方式的错误概率相同。调制方式的错误概率需要根据实际情况此另外计算。比如采用格雷码，QPSK星座点错成相邻点的概率大，而相邻点只相差一个比特，所以误比特率只有误码率的一半。

QPSK的星座点间距变短了，但是一个符号包含的比特多了，弥补了距离减少带来的性能损失。



## 频率分辨率

[辨析改善频谱分辨率问题与栅栏效应——补零、插值、Zoom-FFT算法的影响（附Matlab仿真） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/597110561)

[频谱泄露 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/337431264)

### 频率分辨率



### 频谱泄露



## MATLAB做傅里叶变换分析

### 一般步骤

### fftshift()的使用

[关于MATLAB中fft，fftshift，fftshift(fft(fftshift(x)))，FFT要乘以dx 等问题理解 - ╰追憶似水年華ぃ╮ - 博客园 (cnblogs.com)](https://www.cnblogs.com/zxhyxiao/p/12359243.html)



## 移动通信教材

2024-11-18，第七章里的一个公式证明，这问题23年学的时候就没解决，结果后面查了查就找到了不少资料。
$$
\forall M\in\N_+\\
\int_{0}^{+\infty}Mx(1-e^{-x})^{M-1}e^{-x}\;{\rm d}x\overset{?}{=}\sum_{k=1}^M\frac{1}{k}\\
\sum_{i=1}^{M}C_M^i\frac{(-1)^{i-1}}{i}\overset{?}{=}\sum_{k=1}^M\frac{1}{k}
$$

参考：

1. [calculus - Does the equation hold? - Mathematics Stack Exchange](https://math.stackexchange.com/questions/5000122/does-the-equation-hold)，评论里给出了用$\Beta$函数直接对积分的式子进行证明
2. [Linear diversity combining techniques | IEEE Journals & Magazine | IEEE Xplore](https://ieeexplore.ieee.org/document/1182067)见附录VI，提供了两种方法(包括了使用$\Beta$函数)
3. [sofo12.pdf (emis.de)](https://www.emis.de/journals/JIS/VOL15/Sofo/sofo12.pdf)
4. [A Reciprocal Summation Identity: 10490 on JSTOR](https://www.jstor.org/stable/2589481?origin=crossref&seq=1)，第二个方法的(1)证明了上面组合数的式子(唯一一个比较简洁且我能看得懂的:joy:)
5. [View of A Note on Alternating Sums (combinatorics.org)](https://www.combinatorics.org/ojs/index.php/eljc/article/view/v3i2r7/pdf)

4中第二个方法，所用到恒等式$1=\sum_{k=1}^nC_n^k(-1)^{k-1}$的证明：
$$
0^n=(1-1)^n=\sum_{k=0}^nC_n^k(-1)^{k}=1+\sum_{k=1}^nC_n^k(-1)^{k}\\
\implies1=-\sum_{k=0}^nC_n^k(-1)^{k}=\sum_{k=1}^nC_n^k(-1)^{k-1}
$$
思路是运用数学归纳法：$n=1$时成立，假设$n-1$时成立，对$n>1$有：
$$
\begin{align}
\sum_{k=1}^{n}C_n^k\frac{(-1)^{k-1}}{k}&=\sum_{k=1}^{n-1}(C_{n-1}^k+C_{n-1}^{k-1})\frac{(-1)^{k-1}}{k}+\frac{(-1)^{n-1}}{n}\\
&=\sum_{k=1}^{n-1}C_{n-1}^k\frac{(-1)^{k-1}}{k}+\sum_{k=1}^{n-1}C_{n-1}^{k-1}\frac{n}{k}\frac{(-1)^{k-1}}{n}+\frac{(-1)^{n-1}}{n}\\
\text{运用归纳假设}&=\sum_{k=1}^{n-1}\frac{1}{k}+\sum_{k=1}^{n-1}C_{n}^{k}\frac{(-1)^{k-1}}{n}+\frac{(-1)^{n-1}}{n}\\
&=\sum_{k=1}^{n-1}\frac{1}{k}+\frac{1}{n}\sum_{k=1}^{n}C_{n}^k(-1)^{k-1}\\
\text{利用}1=\sum_{k=1}^nC_n^k(-1)^{k-1}\text{可得}&=\sum_{k=1}^{n}\frac{1}{k}
\end{align}
$$
