# CMU 离散微分几何2021 作业0

**写在前面**：受笔者水平所限，文章可能会有错误，答案仅供参考，欢迎交流讨论。

## Written

### 1.Euler’s Polyhedral Formula

欧拉特征 $\chi = V−E + F$ 是一个拓扑不变量：即使我们使对连通性进行小的局部更改（例如在多边形的中间插入一个新顶点）。它仅在拓扑发生全局更改时才会更改，例如添加额外的部分，或者更改额外的处理。特别地，证明对于任何具有 V 个顶点、E 条边和 F 个面的多边形盘，以下关系成立：

$V− E + F = 1$

然后，解释为什么 $V−E + F = 2$ 适用于任何多边形球。

提示：使用归纳法。请注意，如果从给定对象开始并进行分解，归纳通常会更容易把它分成更小的部分，而不是把它变大，因为要考虑的情况更少。



假设E=3，即一个有3条边的多边形盘形成一个三角形。它有3个顶点和1个面。因此，V − E + F = 3 − 3 + 1 = 1，满足该公式。

假设对于任意具有 k 条边的多边形盘，欧拉公式成立。现在，考虑一种具有 k+1 条边的多边形盘。

当我们向现有的多边形盘添加一条边时，我们引入一个新的顶点，并且不改变面的数量（因为我们是在现有面的边界上添加）。因此，V′=V+1，E′=E+1，且 F′=F。

将这些值代入欧拉公式：

$$\begin{aligned}
&V'-E'+F'=(V+1)-(E+1)+F \\
&=(V-E+F)+(1-1) \\
&=1\quad(\text{因为根据归纳假设,对于具有}k\text{ 条边的多边形盘,}V-E+F=1)
\end{aligned}$$

因此，该公式对于 k+1 条边成立。通过归纳法，欧拉公式 $V− E + F = 1$ 对于任意多边形盘成立。



从多面体去掉一面，通过把去掉的面的边互相拉远，把所有剩下的面变成点和曲线的平面网络。不失一般性，可以假设变形的边继续保持为直线段。正常的面不再是正常的多边形即使开始的时候它们是正常的。但是，点，边和面的个数保持不变，和给定多面体的一样。

重复一系列可以简化网络却不改变其欧拉数（也是欧拉示性数）F − E + V的额外变换。

1. 若有一个多边形面有3条边以上，我们划一个对角线。这增加一条边和一个面。继续增加边直到所有面都是三角形。
2. 除掉只有一条边和外部相邻的三角形。这把边和面的个数各减一而保持顶点数不变。
3. （逐个）除去所有和网络外部共享两条边的三角形。这会减少一个顶点、两条边和一个面。

重复使用第2步和第3步直到只剩一个三角形。对于一个三角形F = 2（把外部数在内），E = 3，V = 3。所以F − E + V = 2。证毕。

![689px-V-E+F=2_Proof](https://cdn.jsdelivr.net/gh/s1san/asusual@img/summer2024/689px-V-E+F=2_Proof.png)

------

![euler](https://cdn.jsdelivr.net/gh/s1san/asusual@img/summer2024/euler.png)

显然不是所有的表面看起来都像圆盘或球体。有些表面有额外的 $handles$ 从拓扑上区分它们；handles 的数量 $g$ 被称为曲面的 $genus$（参见上面的插图）。事实上，在所有没有边界的曲面中连接的（意思是单一的），紧凑的（意思是封闭的，包含在一个有限的球中大小）和可定向（有两个不同的侧面）， $genus$ 是唯一区分两者的东西表面。一个更一般的公式适用于这样的曲面，即

$V−E + F = 2−2g$，

也就是众所周知的欧拉-庞加莱公式。

------

![regular](https://cdn.jsdelivr.net/gh/s1san/asusual@img/summer2024/regular.png)

组合曲面中顶点的 $valence$ 是包含该顶点的边的数目。当一个单纯形曲面的顶点的价（valence）等于6时，我们就说它是规则的。许多数值算法（如细分）只有在常规情况下才表现出理想的行为，而且通常是这样当不规则价顶点数量较少时，性能更好。接下来的几个练习探讨关于组合表面中价的一些有用事实。

![valence](https://cdn.jsdelivr.net/gh/s1san/asusual@img/summer2024/valence.png)

### 2.Platonic Solids

即使是古希腊人也对规则网格感兴趣。特别是，他们知道只有五个零亏格多面体，每个面都是相同的，即五个柏拉图立体：四面体，二十面体，八面体，十二面体，立方体。表明这个列表确实是详尽的。

提示:你不需要使用任何关于长度或角度的事实；仅连通性。



**拓扑证明**

观察正多面体的边，一条边由两个点确定，又连接了两个面。假设正多面体是p-边形（p > 2），每个顶点连接q条边（q > 2），则$pE=2E=qV$。由欧拉公式$V-E+F=2$，可联立求解得：

$$V=\frac{4p}{4-(p-2)(q-2)};\quad E=\frac{2pq}{4-(p-2)(q-2)};\quad F=\frac{4q}{4-(p-2)(q-2)}$$

可以得出如下解:

p=3，q=3，对应正四面体；

p=3，q=4，对应正八面体；

p=3，q=5，对应正二十面体；

p=4，q=3，对应正六面体；

p=5，q=3，对应正十二面体。

或者，将 pF=2E=qV 带入欧拉公式 V-E+F=2，得关系式 $\frac{2E}q-E+\frac{2E}p=2$，进一步地有 $\frac1q+\frac1p=\frac12+\frac1E>\frac12$。因此，p,q的组合只有(3,3), (4,3), (3,4), (5,3), (3,5)这五种可能。

**几何证明**
下面的几何讨论和欧几里得在几何原本中给出的证明非常相似：

1. 多面体的每个顶点至少在三个面上。
2. 这些相交的面处的角（也就是顶点发出的角）的和必须小于 360°。
3. 正多面体的顶点发出的角是相等的，所以这个角必须小于 360°/3 = 120°。
4. 正六边形及边更多的正多边形的角大于等于 120°，所以正多面体上的面只能是正三角形，正方形或正五边形。于是：
   * 正三角形：每个角是 60°，所以正多面体每个顶点发出的角数目小于 360°/60° = 6，也就是每个顶点只能在三、四、五个面上，这分别对应于正四面体、正八面体、正二十面体；
   * 正方形：每个角是 90°，所以正多面体每个顶点发出的角数目小于 360°/90° = 4，也就是每个顶点只能在三个面上，这对应于正方体；
   * 正五边形：每个角是 108°，所以正多面体每个顶点发出的角数目小于 360°/108° = 10/3，也就是每个顶点只能在三个面上，这对应于正十二面体。

### 3.Regular Valence

证明唯一的（连通的，可定向的）每个顶点都有规则的单纯形曲面价是一个环面（$g$ = 1）。你可以假设这个表面有有限多个面。

提示:使用欧拉-庞加莱公式。



现在假设我们有一个连通的、每个顶点都有规则的单纯形曲面，记作 M。根据题目条件，我们知道每个顶点都有规则的单纯形曲面价。因此，每个顶点都有相同数量的面围绕着它。

我们将每个面围绕每个顶点视为一个“环”。由于每个顶点都有相同数量的环，我们可以将这个曲面划分为一些环，并将每个环上的面归类到对应的顶点。

因为曲面是连通的，所以每个面都至少与另一个面共享一条边。因此，我们可以将曲面划分为一些环，并且每个环上至少有一个顶点。这意味着我们没有划分出独立的环，每个环都与另一个环相连。

由于曲面是可定向的，我们可以确保每个环都有一个内部和一个外部。因此，每个环都有一个内部面和一个外部面。

根据欧拉-庞加莱公式，对于一个连通的、可定向的曲面，其欧拉特征为 2 - 2g，其中 g 是亏格数。对于一个环面，g 等于 1。因此，对于一个环面，其欧拉特征为 0。

现在让我们应用欧拉-庞加莱公式到我们的情况。考虑到每个顶点都有相同数量的环，我们可以将每个顶点周围的环数乘以顶点数得到总的环数。因此，我们有：

$V−E+F=0$

根据我们之前的讨论，我们知道 *V* 是顶点数，E 是边数，F 是面数。因此，我们得到的方程与欧拉特征公式完全相同。

因此我们可以得出结论，唯一的每个顶点都有规则的单纯形曲面价的曲面是一个环面，其亏格数为 1。

### 4.Minimum Irregular Valence

证明在一个（连通的，可定向的）$g$ 亏格的单纯曲面K由

$$m(K)=\begin{cases}4,&g=0\\0,&g=1\\1,&g\geq2,\end{cases}$$

假设所有顶点至少有3个价并且有有限多个面。注意：你不需要构造最小三角剖分；只要根据欧拉-庞加莱公式。



每个三角形有3条边，每条边属于两个三角形。所以，边数 E 与面数 F 的关系为： $2E=\frac{3F}{2}$

假设曲面 K 中有 V 个顶点，每个顶点至少有3个邻接边，这意味着顶点的平均度数至少为3。

所有顶点的度数和为 2E，因此： $2E≥3V$ 根据 $2E=\frac{3F}{2}$，我们可以得到：

 $\begin{aligned}&3F\geq6V\\&F\geq2V\end{aligned}$

当亏格 *g* 为0时（球面）：

对于$g=0$：

$χ=2$

结合欧拉公式 $V−E+F=0$ 和 $E=\frac32F$：
$$\begin{aligned}\\&V-\frac{3F}2+F=2\\&V-\frac F2=2\\&V=\frac F2+2\end{aligned}$$

利用$F≥2V$：

$$\begin{aligned}&F\geq2\left(\frac F2+2\right)\\&F\geq F+4\end{aligned}$$

这是矛盾的，除非 F 非常小。因此，对于球面，最小的符合条件的剖分是由4个面构成。即 $m(K)=4$。

当亏格 *g* 为1时（环面）：

对于$g=1$：

$χ=0$

结合欧拉公式 $V−E+F=0$ 和 $E=\frac32F$：
$$\begin{aligned}\\&V-\frac{3F}2+F=0\\&V-\frac F2=0\\&V=\frac F2\end{aligned}$$

利用$F≥2V$：

$$\begin{aligned}&F\geq2\left(\frac F2\right)\\&F\geq F\end{aligned}$$

这总是成立的，因此对于亏格为1的环面，没有附加的边界条件，所有的剖分均满足条件。即 $m(K)=0$。

当亏格 *g* 大于等于2时：

对于$g≥2$：

$χ=2-2g$

结合欧拉公式 $V−E+F=2-2g$ 和 $E=\frac32F$：
$$\begin{aligned}\\&V-\frac{3F}2+F=2-2g\\&V-\frac F2=2-2g\\&V=\frac F2+2-2g\end{aligned}$$

利用$F≥2V$：

$$\begin{aligned}
&F\geq2\left(\frac F2+2-2g\right) \\
&F\geq F+4-4g \\
&4g\geq4 \\
&g\geq1
\end{aligned}$$

这对于 $2g≥2$ 始终成立。最小的符合条件的剖分有一个剖分。即 $m(K)=1$。

### 5.Mean Valence (Triangle Mesh)

表明在一个（连通的，可定向的）单纯曲面趋于无穷大，顶点与边与三角形的比值因此趋于

$$V : E : F = 1:3:2$$

你可以假设 $g$ 亏格随着顶点数量的增加而保持固定。提示：欧拉-庞加莱公式！



根据欧拉-庞加莱公式：$χ=V−E+F$，且$χ=2−2g$

在一个三角剖分的单纯曲面上，每个面都是一个三角形，并且每条边都是共享两面的。所以： $3F=2E$ ，即$E=\frac32F$

将$E=\frac32F$代入欧拉公式：

$$\begin{aligned}&\chi=V-\frac32F+F\\&\chi=V-\frac12F\end{aligned}$$

由于$χ=2−2g$，我们得到：

$$2-2g=V-\frac12F\\V=2-2g+\frac12F$$

我们现在考虑顶点数 V 和面数 F 都趋于无穷大时，*g* 作为一个固定值， V 和 F 都很大，所以 2−2*g* 可以忽略不计。这时可以写成： $V≈\frac12F$

现在我们有：

$$\begin{aligned}&E=\frac32F\\&V\approx\frac12F\end{aligned}$$

所以：

$V:E:F=\frac12F:\frac32F:F$

化简得：

$V:E:F=1:3:2$

### 6.Mean Valence (Quad Mesh)

与前面的练习类似，考虑一个四边形网格，即一个完全的组合曲面由四面四边形而不是三面三角形构成。设 Q 表示的个数四边形，给出比率的表达式
$$
V: E: Q
$$
在顶点数趋于无穷时的极限。你可以再次假设亏格仍然是固定的。



同上题：根据欧拉-庞加莱公式：$χ=V−E+F$，且$χ=2−2g$

在这里，四边形 Q 替代了三角形 F，所以公式变成： $χ=V−E+Q$

在一个由四边形组成的单纯曲面上，每个四边形有4条边，但每条边被两个四边形共享。所以： $E=2Q$

将$E=2Q$代入欧拉公式：

$$\begin{aligned}&\chi=V-2Q+Q\\&\chi=V-Q\end{aligned}$$

由于$χ=2−2g$，我们得到：

$$2-2g=V-Q\\V=2-2g+Q$$

我们现在考虑顶点数 V 和四变形数 Q 都趋于无穷大时，*g* 作为一个固定值， V 和 Q 都很大，所以 2−2*g* 可以忽略不计。这时可以写成： $V≈Q$

现在我们有：

$$\begin{aligned}&E=2Q\\&V\approx Q \end{aligned}$$

所以：

$V:E:Q=Q:2Q:Q$

化简得：

$V:E:Q=1:2:1$

------

了解网格元素的近似比例在制定算法设计决策时很有用（例如，在边上存储一个数量的成本大约是在顶点上的三倍），并且简化了关于渐近增长的讨论（由于不同元素类型的数量基本上是由一个常数关系）。类似的比例可以针对四面体网格进行计算，尽管在这里需要更加粗略一些：

### 7.Mean Valence (Tetrahedral)

考虑一个流形单纯 3-复形，其中 V、E、F 和 T 分别表示顶点、边、三角形和四面体的数量。当元素数量趋向无穷大时，可以得出以下粗略估计的比例：

$$V:E:F:T$$

对于四面体网格，存在类似于欧拉多面体公式的公式：$V−E+F−T=c$，其中 c 是一个只依赖于全局拓扑（handles数量等）的常数。为了得到一个粗略估计，我们可以假设每个顶点 i 的链接 $Lk(i)$ 都是一个组合二十面体。由于你关心的是渐近行为，可以安全地忽略边界顶点。提示：哪些比率可以轻松计算？



假设每个顶点 i 的链接 Lk(i) 都是一个组合二十面体。一个组合二十面体有 20 个三角形面、30 条边和 12 个顶点。对于每个顶点 i，链接 Lk(i) 满足： 

$三角形:边:顶点=20:30:12=10:15:6$

在一个四面体网格中：

* 每个四面体有 4 个顶点、6 条边和 4 个三角形。
* 每条边被两个四面体共享。
* 每个三角形被两个四面体共享。

得：

2E=6T，E=3T，2F=4T，F=2T

我们将 E=3T 和 F*=2*T 代入类欧拉公式：

$$\begin{aligned}
&V-3T+2T-T=c \\
&V-2T=c \\
&V=2T+c
\end{aligned}$$

当顶点 V、边 E、三角形 F 和四面体 T 的数量趋于无穷大时，常数 c 可以忽略不计。因此，我们可以认为： $V≈2T$

综合上述得：$V:E:F:T=2T:3T:2T:T=2:3:2:1$

------

这里是一些来自真实数据的大型四面体网格的统计数据。它们大致与您估算的比率相符吗？（如果它们不完全匹配，您不需要担心！）

![large-ish](https://cdn.jsdelivr.net/gh/s1san/asusual@img/summer2024/large-ish.png)

### 8.Star, Closure, and Link

对于下面用深蓝色表示的子集 S（由三个顶点、三条边和两个三角形组成），给出星星 St(S)、闭包 Cl(S) 和链接 Lk(S)，可以通过绘制图片或提供每个集合中单纯形的列表来完成。

![scl](https://cdn.jsdelivr.net/gh/s1san/asusual@img/summer2024/scl.png)



$\mathrm{St}(S)=\{k,o,p,ko,op,pk,oj,ol,pq,kop,koq,lop,olp,poj\}$

$\mathrm{Cl}(S)=\{k,o,p,ko,op,pk,kop\}$

$\mathrm{Lk}(S)=\{jo,lo,qo,po,lp,qk,lop,koq,olp,poj\}$

### 9.Boundary and Interior

对于上面用深蓝色表示的子集 $\mathcal{K}^{\prime}$（由12个顶点、23条边和12个三角形组成），给出边界 bd($\mathcal{K}^{\prime}$) 和内部 int($\mathcal{K}^{\prime}$)，可以通过绘制图片或提供每个集合中单纯形的列表来完成。



边界$\operatorname{bd}(\mathcal{K}^\prime)$

* 边：$ab,ae,bf,fg,gh,hk,kl,lm,mp,pq$

内部$\operatorname{int}(\mathcal{K}^\prime)$

三角形：$koq,elp,poa,hpm,ekh,kbl,lmo,opq,koq,elp,poa,hpm$

### 10.Surface as Permutation

对于下图的组合面，通过填写以下表格，给出对边和下一条边 $\eta$ 和 $\rho$ ：

![surfaceaspermutation](https://cdn.jsdelivr.net/gh/s1san/asusual@img/summer2024/surfaceaspermutation.png)



| *h*      | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    |
| -------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| *η*(*h*) | 4    | 2    | 1    | 5    | 0    | 3    | 7    | 6    | 9    | 8    |

| *h*      | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    |
| -------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| *ρ*(*h*) | 1    | 2    | 0    | 4    | 0    | 9    | 3    | 6    | 7    | 8    |

### 11.Permutation as Surface

对于下面给出的排列  $\rho$，描述它所描述的组合曲面——或者用文字，或者画幅图。您应该假设 $\eta$ 是按照第2.5节的描述确定的，即：偶数半边h的对边是 h + 1；奇数半边h的对边是h - 1。

![permutationassurface](https://cdn.jsdelivr.net/gh/s1san/asusual@img/summer2024/permutationassurface.png)



通过给出的$\rho$，可以得出：

| *h*      | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    | 10   | 11   | 12   | 13   | 14   | 15   |
| -------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| *η*(*h*) | 1    | 0    | 3    | 2    | 5    | 4    | 7    | 6    | 9    | 8    | 11   | 10   | 13   | 12   | 15   | 14   |

$\{0,8,7\},\{1,2,14\},\{3,4,12\},\{5,6,10\}$ 分别组成一个三角形，$\{9,15,13,11\}$ 组成一个四边形。我们可以看出这是一个底面为四边形的五面体（四角锥）。

### 12.Surface as Matrices

给出下图所示的单纯盘的邻接矩阵$A_0$和$A_1$。

<img src="https://cdn.jsdelivr.net/gh/s1san/asusual@img/summer2024/surfaceasmatrices.png" alt="surfaceasmatrices" style="zoom:50%;" />



图中有五个顶点，八条边。按照**边**的连接关系构建**顶点**的邻接矩阵$A_0$，按照**面**的连接关系构建**边**的邻接矩阵$A_1$

$$A_0=
\begin{pmatrix}
1&1&0&0&0\\
1&0&1&0&0\\
1&0&0&1&0\\
1&0&0&0&1\\
0&1&0&0&1\\
0&1&1&0&0\\
0&0&1&1&0\\
0&0&0&1&1
\end{pmatrix}\quad 
A_1=
\begin{pmatrix}
1&0&0&1&1&0&0&0\\
1&1&0&0&0&1&0&0\\
0&1&1&0&0&0&1&0\\
0&0&1&1&0&0&0&1
\end{pmatrix}$$

------

接下来的三个练习，你可以严格或随意，只要你喜欢正确地传达每个陈述是正确的核心原因。

### 13.Classification of Simplicial 1-Manifolds

解释为什么每个单纯 1-流形（可能带边界）不能包含除边的路径和闭合边环之外的任何东西。



单纯 1-流形是由顶点和边组成的几何对象，其中每条边连接两个顶点。每个顶点的邻域看起来像实数轴 R 的一个子集（局部性质），即每个顶点最多与两个边相连。

路径是由一系列相连的边组成的。每条路径有两个端点，且每个端点处的顶点要么是路径的起点或终点，要么是路径中的中间顶点（与两条边相连）。闭合边环是由边构成的环形结构，其中所有顶点都恰好连接两条边，使得路径首尾相连形成环。

若某顶点连接超过两条边，则在该点的局部结构不再像实数轴的子集，而像图论中的“节点”，这违反了 1-流形的定义。因此，一个单纯 1-流形只能包含路径（包括开路径和闭路径即环）。任何超过两条边连接到同一个顶点的情况或分支结构都违背了 1-流形的定义，因而不能存在于单纯 1-流形中。

### 14.Boundary Loops

解释为什么任何单纯表面（即任何单纯 2-流形）的边界总是一个闭合环的集合。



单纯 2-流形的边界边必须连接形成连续路径。拓扑结构要求这些路径闭合以形成完整的边界。每个单纯 2-流形的边界边在局部连接上必须形成闭合的环形结构，以满足边界定义。

### 15.Boundary Has No Boundary

解释为什么单纯流形的边界 bd($\mathcal{K}$) 没有边界。换句话说，为什么 bd(bd($\mathcal{K}$)) = $\varnothing$？



边界是由低维单纯形组成，这些单纯形在形成边界后已经是闭合结构，没有悬挂部分。拓扑性质确保了边界的结构没有进一步的边界存在，形成了一个完整的闭合系统。

## Coding

### 1.实现`assignElementIndices`方法

```cpp
void SimplicialComplexOperators::assignElementIndices() {

    geometry->requireVertexIndices();
    geometry->requireEdgeIndices();
    geometry->requireFaceIndices();

    size_t idx = 0;
    for (Vertex v : mesh->vertices()) {
        geometry->vertexIndices[v] = idx++;
    }

    idx = 0;
    for (Edge e : mesh->edges()) {
        geometry->edgeIndices[e] = idx++;
    }

    idx = 0;
    for (Face f : mesh->faces()) {
        geometry->faceIndices[f] = idx++;
    }
}
```



### 2.实现`buildVertexEdgeAdjacencyMatrix  `方法

```c++
SparseMatrix<size_t> SimplicialComplexOperators::buildVertexEdgeAdjacencyMatrix() const {
    geometry->requireVertexIndices();
    geometry->requireEdgeIndices();

    std::vector<Tri> coefficients;
    SpMat A0(mesh->nEdges(), mesh->nVertices());

    for (Edge e : mesh->edges()) {
        size_t idx = geometry->edgeIndices[e];
        coefficients.push_back(Tri(idx, geometry->vertexIndices[e.firstVertex()], 1));
        coefficients.push_back(Tri(idx, geometry->vertexIndices[e.secondVertex()], 1));
    }

    A0.setFromTriplets(coefficients.begin(), coefficients.end());

    return A0
}
```

构建一个稀疏矩阵，表示顶点-边的邻接关系。

### 3.实现`buildEdgeFaceAdjacencyMatrix  `方法

```c++
SparseMatrix<size_t> SimplicialComplexOperators::buildFaceEdgeAdjacencyMatrix() const {

    geometry->requireFaceIndices();
    geometry->requireEdgeIndices();

    std::vector<Tri> coefficients;
    SpMat A1(mesh->nFaces(), mesh->nEdges());

    for (Face f : mesh->faces()) {
        size_t idx = geometry->faceIndices[f];
        for (Edge e : f.adjacentEdges()) coefficients.push_back(Tri(idx, geometry->edgeIndices[e], 1));
    }

    A1.setFromTriplets(coefficients.begin(), coefficients.end());

    return A1;
}
```

构建一个面-边邻接矩阵。

### 4.实现`buildVertexVector`，`buildVector`，`buildVector`方法

```c++
Vector<size_t> SimplicialComplexOperators::buildVertexVector(const MeshSubset& subset) const {
    Vector<size_t> v = Vector<size_t>::Zero(mesh->nVertices());

    for (size_t idx : subset.vertices) v[idx] = 1;

    return v;
}
```



```c++
Vector<size_t> SimplicialComplexOperators::buildEdgeVector(const MeshSubset& subset) const {
    Vector<size_t> e = Vector<size_t>::Zero(mesh->nEdges());

    for (size_t idx : subset.edges) e[idx] = 1;

    return e;
}
```



```c++
Vector<size_t> SimplicialComplexOperators::buildFaceVector(const MeshSubset& subset) const {
    Vector<size_t> f = Vector<size_t>::Zero(mesh->nFaces());

    for (size_t idx : subset.faces) f[idx] = 1;

    return f;
}
```

构建顶点、边和面指示向量，反映给定子集中的元素，代码都差不多。

对于剩下的方法，记得**必须**使用邻接矩阵（如上所述）；而不是直接使用半边数据结构实现这些方法。

### 5.实现`star`方法

```c++
MeshSubset SimplicialComplexOperators::star(const MeshSubset& subset) const {
    SpMat vertexEdgeSpMat = buildVertexEdgeAdjacencyMatrix();
    SpMat faceEdgeSpMat = buildFaceEdgeAdjacencyMatrix();

    MeshSubset star = subset.deepCopy();

    for (size_t idx : subset.vertices)
        for (int k = 0; k < vertexEdgeSpMat.outerSize(); k++)
            for (SpMat::InnerIterator it(vertexEdgeSpMat, k); it; ++it)
                if (it.col() == int(idx)) star.addEdge(it.row());

    for (size_t idx : star.edges)
        for (int k = 0; k < faceEdgeSpMat.outerSize(); k++)
            for (SpMat::InnerIterator it(faceEdgeSpMat, k); it; ++it)
                if (it.col() == int(idx)) star.addFace(it.row());

    return star;
}

```

对于子集中的每个顶点 `idx`，使用 `vertexEdgeSpMat` 的 `InnerIterator` 来遍历与该顶点相邻的所有边，并将这些边添加到 `star` 中。

对于 `star` 中的每条边 `idx`，使用 `faceEdgeSpMat` 的 `InnerIterator` 来遍历与该边相邻的所有面，并将这些面添加到 `star` 中。

### 6.实现`closure`方法

```c++
MeshSubset SimplicialComplexOperators::closure(const MeshSubset& subset) const {
    SpMat vertexEdgeSpMat = buildVertexEdgeAdjacencyMatrix();
    SpMat faceEdgeSpMat = buildFaceEdgeAdjacencyMatrix();

    MeshSubset closure = subset.deepCopy();

    for (size_t idx : subset.faces)
        for (int k = 0; k < faceEdgeSpMat.outerSize(); k++)
            for (SpMat::InnerIterator it(faceEdgeSpMat, k); it; ++it)
                if (it.row() == int(idx)) closure.addEdge(it.col());

    for (size_t idx : closure.edges)
        for (int k = 0; k < vertexEdgeSpMat.outerSize(); k++)
            for (SpMat::InnerIterator it(vertexEdgeSpMat, k); it; ++it)
                if (it.row() == int(idx)) closure.addVertex(it.col());

    return closure;
}
```

闭包操作要求包括所有选定的单纯形及其所有面。例如，如果一个面被选中，那么它的所有边和顶点也必须被包括在内。如果一条边被选中，那么它的两个顶点也必须被包括在内。

### 7.实现`link`方法

```c++
MeshSubset SimplicialComplexOperators::link(const MeshSubset& subset) const {
    MeshSubset link;
    link = closure(star(subset));
    link.deleteSubset(star(closure(subset)));

    return link;
}
```

计算给定子集的链接，即所有与该子集邻接但不属于该子集或其闭包的单纯形。

### 8.实现`isComplex`，`isPureComplex`方法

```c++
bool SimplicialComplexOperators::isComplex(const MeshSubset& subset) const {
    return closure(subset).equals(subset);
}
```

如果一个子集的闭包等于它本身，则它是单纯复形

```c++
int SimplicialComplexOperators::isPureComplex(const MeshSubset& subset) const {
    SpMat vertexEdgeSpMat = buildVertexEdgeAdjacencyMatrix();
    SpMat faceEdgeSpMat = buildFaceEdgeAdjacencyMatrix();   

    MeshSubset isPure;

    if (!isComplex(subset)) {
        return -1;
    } else if (!subset.faces.empty()) {
        isPure = subset;
        isPure.deleteEdges(subset.edges);
        isPure.deleteVertices(subset.vertices);
        isPure = closure(isPure);

        if (isPure.edges == subset.edges && isPure.vertices == subset.vertices)
            return 2;
        else
            return 1;
    } else if (!subset.edges.empty()) {
        isPure = subset;
        isPure.deleteVertices(subset.vertices);
        isPure = closure(isPure);

        if (isPure.vertices == subset.vertices)
            return 1;
        else
            return -1;
    } else if (!subset.vertices.empty())
        return 0;

    return -1;
}
```

首先检查给定的子集是否是一个单纯复形。如果不是，返回 -1。

然后根据子集中包含的最高维单纯形（面、边、顶点）来判断它是否是纯净单纯复形。

如果包含面，检查是否是纯 2-复形；如果包含边，检查是否是纯 1-复形；如果只有顶点，则是纯 0-复形。

在每个检查过程中，通过删除低维单纯形并计算闭包来验证纯净性。如果计算结果与原始子集一致，则是纯净的。

### 9.实现`boundary`方法

```c++
MeshSubset SimplicialComplexOperators::boundary(const MeshSubset& subset) const {
    SpMat vertexEdgeSpMat = buildVertexEdgeAdjacencyMatrix();
    SpMat faceEdgeSpMat = buildFaceEdgeAdjacencyMatrix();

    MeshSubset boundary;

    if (isPureComplex(subset) == 2) {
        std::vector<size_t> allEdge;
        std::set<size_t> boundaryEdge;

        // 收集所有属于面的边
        for (size_t idx : subset.faces)
            for (int k = 0; k < faceEdgeSpMat.outerSize(); k++)
                for (SpMat::InnerIterator it(faceEdgeSpMat, k); it; ++it)
                    if (it.row() == int(idx)) allEdge.push_back(it.col());

        // 通过计算每条边出现的次数来确定边界边
        for (size_t idx : subset.edges)
            if (std::count(allEdge.begin(), allEdge.end(), idx) == 1) boundaryEdge.insert(idx);

        boundary.addEdges(boundaryEdge);
        boundary = closure(boundary);
        return boundary;
    } else if (isPureComplex(subset) == 1) {    // 如果子集是一个纯净的 1-复形
        std::vector<size_t> allVertex;
        std::set<size_t> boundaryVertex;

        // 收集所有属于边的顶点
        for (size_t idx : subset.edges)
            for (int k = 0; k < vertexEdgeSpMat.outerSize(); k++)
                for (SpMat::InnerIterator it(vertexEdgeSpMat, k); it; ++it)
                    if (it.row() == int(idx)) allVertex.push_back(it.col());

        // 通过计算每个顶点出现的次数来确定边界顶点
        for (size_t idx : subset.edges)
            if (std::count(allVertex.begin(), allVertex.end(), idx) == 1) boundaryVertex.insert(idx);

        // 将边界顶点添加到边界集合中
        boundary.addEdges(boundaryVertex);
        // 计算闭包
        boundary = closure(boundary);
        return boundary;
    }

    // 如果既不是纯净的 1-复形，也不是 2-复形，返回空边界
    return boundary;
}
```

完整代码在[这里](https://github.com/s1san/cmu-ddg2021)

## References

Wiki，[欧拉示性数](https://zh.wikipedia.org/wiki/%E6%AC%A7%E6%8B%89%E7%A4%BA%E6%80%A7%E6%95%B0#)

Wiki，[柏拉图立体](https://zh.wikipedia.org/wiki/%E6%9F%8F%E6%8B%89%E5%9C%96%E7%AB%8B%E9%AB%94#)

数立方，[数理史上的绝妙证明：柏拉图多面体只有五种](https://mathcubic.org/article/article/index/id/478.html)

Wiki，[单纯形](https://zh.wikipedia.org/wiki/%E5%8D%95%E7%BA%AF%E5%BD%A2)

知乎，Steven，[CMU DDG 离散微分几何 编程作业0：单纯复形操作](https://zhuanlan.zhihu.com/p/563081925)