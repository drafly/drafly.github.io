---
layout: post
title: "R语言——数值分析"
description: ""
category: "Statistical Computing with R"
analytics: true
tags: [R]
---
{% include JB/setup %}
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script src="https://google-code-prettify.googlecode.com/svn/loader/run_prettify.js"></script>

### 1. 数值分析	
	数值分析(Wiki)：是指在数学分析（区别于离散数学）问题中，对使用数值近似（相对于一般化的符号运算）算法的研究。数值分析的目的是设计及分析一些计算的方式，可针对一些问题得到近似但够精确的结果。数值分析方法分为两种：直接法和迭代法。直接法利用固定次数的步骤求出问题的解，包括求解线性方程组的高斯消去法和解析法，求解线性规划的单纯形法等；迭代法是通过从一个初始估计出发寻找一系列近似解来解决问题的数学过程。和直接法不同，用迭代法求解问题时，其步骤没有固定的次数，而且只能求得问题的近似解，所找到的一系列近似解会收敛到问题的精确解，然后利用审敛法来判别所得到的近似解是否会收敛，包括牛顿法、二分法、梯度下降方法等。一般我们采用迭代法对问题进行求解。
	
[数值分析](http://zh.wikipedia.org/wiki/%E6%95%B0%E5%80%BC%E5%88%86%E6%9E%90#.E7.9B.B4.E6.8E.A5.E6.B3.95.E5.92.8C.E8.BF.AD.E4.BB.A3.E6.B3.95)应用领域包括函数求值，曲线拟合（回归），求解方程和方程组，求解特征值，积分微分运算和最优化。其中，最优化是数值分析研究中的重点，机器学习和人工智能科学把最优化作为一个重要的领域来研究，R中常用的优化算法或函数包括最大似然估计`mle()`、`optimize()`、`uniroot()`牛顿或梯度下降`optim()`和线性规划的*simplex*方法`simplex()`等，这部分内容将会在接下来的《机器学习》篇章中做详细介绍。因为，机器学习最优化算法基本上利用迭代法进行求解，所以本篇博客则作为最优化算法和机器学习算法的开端，重点引出数值分析中求解方程或方程组常用的迭代法。

### 2. 方程求根
求根算法是求解非线性方程$$f(x)=c$$或$$g(x)=f(x)-c=0$$解的算法，函数的根就是使其值为零的点。根据是否利用函数导数可将求根算法分为两类：若函数本身可微且其导数是已知的，可以用牛顿法求解；若函数不可微或不利用导数，可以用二分法或割线法等求解；线性规划则是另一种求解非线性方程的方法。

#### 1. 二分法
二分法(Bisection method)，是一种方程式根的近似值求法，该方法的最基本思想是：如果能够将方程根所在的范围尽量缩小，那么在一定精度下，我们可以得到根的近似值。其数学表达是：对于区间$$[a,b]$$上连续且满足$$f(a)\cdot f(b)<0$$的函数$$y=f(x)$$，通过不断把函数$$y=f(x)=0$$的点（即方程的根）所在区间一分为二，使区间两个端点$$a和b$$逐步逼近方程的根，进而得到方程$$f(x)=0$$根的近似值的方法。

该方法成立的理论基础是数值分析中的介值定理(intermediate value theorem)，该定理描述了连续函数在两点间的连续性。如果连续函数$$f(x)$$通过$$(a,f(a))$$与$$(b,f(b))$$两点，它也必定通过$$[a,b]$$区间内的任意一点$$(c,f(c)),a<c<b$$。该表述等价于：假设$$f:I\rightarrow\mathbb{R}$$是连续函数，且实数$$u$$满足$$f(a)<u<f(b)$$或$$f(a)>u>f(b)$$，则存在$$c\in(a,b)$$使得$$f(c)=u$$。

根据介值定理的描述，我们可以利用区间$$[a,b]$$的中点将区间一分为二，将方程的根所在范围缩小到$$[\frac{a+b}{2},b]$$或者$$[a,\frac{a+b}{2}]$$；然后通过多次迭代，将区间无限缩小至一点。但是，如果区间$$[a,b]$$中包含一个以上的方程的根，二分法可以获得其中一个根，所以该方法复杂度是线性的。

若要求已知连续函数$$f(x)=0$$的根（方程的解），二分法具体实现步骤如下：

1. 先找到区间$$[a,b]$$，使得$$f(a)\times f(b)<0$$，根据介值定理，这个区间一定包含方程的根；
1. 求该区间$$[a,b]$$的中点$$m=\frac{a+b}{2}$$，并计算$$f(m)$$的值；
1. 若$$f(m)\times f(a)>0$$，则取$$[m,b]$$为新的区间，否则取$$[a,m]$$；
1. 判断是否达到$$\varepsilon$$，即若$$\|a-b\|<\varepsilon$$，则获得方程根的近似值$$a$$或$$b$$，否则重复第2和第3步直至达到精度$$\varepsilon$$。

#### 2. 布伦特(Brent)法
布伦特（Brent）方法是在二分法或试位法的基础上，借助二次插值方法进行加速，又利用反插值方法来简化计算而形成的一种方法，计算效率要高于二分法。但是如果生成了不能接受的结果（例如根的估计值超出了边界），就转而使用较保守的二分法。

假如存在函数上的三个点，分别是$$(a,f(a)),(b,f(b)),(c,f(c))$$，其中$$a<b<c$$，且点$$(b,f(b))$$是当前迭代下区间$$[a,c]$$内方程的近似解，则布伦特法利用拉格朗日差值法，通过这三个点计算下一次迭代的估计值。计算表达式为：

$$x=\frac{[y-f(a)][y-f(b)]c}{[f(c)-f(a)][f(c)-f(b)]} + \frac{[y-f(b)][y-f(c)]a}{[f(a)-f(b)][f(a)-f(c)]} + \frac{[y-f(c)][y-f(a)]b}{[f(b)-f(c)][f(b)-f(a)]}$$

其中，$$y=0$$，如果计算的值$$x$$不在区间[a,c]范围内，则重新采用二分法计算；否则，根据$$x$$和$$b$$间的关系，构成新区间$$[x,b]$$或$$[b,x]$$，然后再判断$$f(x)\cdot f(b)$$的符号确定下一次迭代的区间。在R中，可采用`uniroot()`函数实现Brent法求解一元方程的根。

#### 3. 实例
本实例分别采用直接法、二分法和布伦特法对方程$$a^2+y^2+\frac{2ay}{n-1}=n-2$$求解，其中$$a$$为常数，$$n$$为大于2的整数。该方程为一元二次方程，我们可以通过直接法，利用根的判别式获得该方程的[解析解](http://zh.wikipedia.org/wiki/%E8%A7%A3%E6%9E%90%E8%A7%A3)，即：

$$y=\frac{-a}{n-1}\pm\sqrt{n-2+a^2+(\frac{a}{n-1})^2}$$

对于上述方程根的求解，R中二分法、布伦特法和直接法的代码如下：

<pre class="prettyprint">
#将待求解方程改写为函数形式
f <- function(y, a, n){
  a^2+y^2+2*a*y/(n-1) - (n-2)
}

#设定初始值
a <- 0.5
n <- 20
##区间初始值
b0 <- 0
b1 <- 5*n

#二分法
iter <- 0
##设定迭代停止条件epsilon
eps <- .Machine$double.eps^0.25
##计算区间及区间中值
r <- seq(b0, b1, length=3)
##计算区间端点对应的函数值
y <- c(f(r[1], a, n), f(r[2], a, n), f(r[3], a, n))
##判断初始区间是否满足二分法f(a)*f(b)<0的条件
if (y[1]*y[3] > 0){
  ##执行出现错误后的提示
  stop("函数在区间两端具有相同符号的函数值")
}
##执行二分法
while (iter < 1000 && abs(y[2]) > eps){
  iter <- iter + 1
  if (y[1]*y[2] <0){
    r[3] <- r[2]
    y[3] <- y[2]
  }
  else{
    r[1] <- r[2]
    y[1] <- y[2]
  }
  r[2] <- (r[1]+r[3]) / 2
  y[2] <- f(r[2], a, n)
}
##输出最终方程根的近似解和近似解对应的函数值
print(sprintf("二分法近似解为%g,其对应函数f的值为%g",r[1], y[1]))

#Brent方法
brent <- uniroot(function(y){
  a <- 0.5
  n <- 20
  a^2+y^2+2*a*y/(n-1) - (n-2)}, 
  lower = 0, upper = n*5)
print(sprintf("Brent法近似解为%g",brent$root))

##根的判别式->方程解析解
jx <- c(0, 0)
jx[1] <- -a/(n-1) + sqrt(n-2+a^2+(a/(n-1))^2)
jx[2] <- -a/(n-1) - sqrt(n-2+a^2+(a/(n-1))^2)
print(sprintf("方程解析解为%g和%g", jx[1], jx[2]))
</pre>

![二分法和直接法](/img/R/numerical/bisection.jpg)

实验结果表明，二分法、Brent方法和直接法的结果近似相等，分别为4.187和4.245。但二分法和Brent方法只能找到方程的某一个根，无法获得方程所有可能解。