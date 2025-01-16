[首页](/README.md)

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
