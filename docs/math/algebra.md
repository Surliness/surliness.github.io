
### [各种花体](https://blog.csdn.net/Annie_0321/article/details/115764983)

`\scr{}`、`\cal{}`会将后续的所有字母都变为该字体，不建议使用
$$
\begin{array}{c|c}
\text{黑板粗体(Blackboard Bold) }&\mathbb{F\;L\;Z}\\
\text{书法(Calligraphic) }&\mathcal{F\;L\;Z}\;\text{or}\;\symcal{F\;L\;Z}\\
\text{手写体(Script) }&\mathscr{F\;L\;Z}\;\text{or}\;\symscr{F\;L\;Z}\\
\text{(德文)折角体(Fraktur)}&\mathfrak{F\;L\;Z}\\
\\
&\mathrm{F\;L\;Z}\\
&\mathbf{F\;L\;Z}\\
&\mathbfit{F\;L\;Z}\\
\text{符号粗体}&\boldsymbol{FLZ\alpha}\\
\text{伪粗体(poor man's bold)}&\pmb{F\;L\;Z\sum}\\
\end{array}
$$

### 集合

#### 补集

全集$S$补集可以表示记为$\complement_S A$或者$\overline{A}$，建议用`\setminus`
$$
S\setminus A\\
S\smallsetminus A\\
S\backslash A\\
$$
