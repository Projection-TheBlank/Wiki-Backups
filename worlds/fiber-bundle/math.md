---
title: ■■
description: 欢迎来到数学至上主义的教室
published: true
date: 2025-03-26T11:33:20.355Z
tags: 
editor: markdown
dateCreated: 2025-03-24T17:53:59.955Z
---

# 指数衰减中衰变常数、半衰期和平均寿命的关系
## 何为指数衰减
自然界有些数量，比如一杯啤酒中泡沫的数量，又比如放射性元素的储量，遵循<span style="color:#ff6060">指数衰减</span>的规律。

指数衰减最大的特征为“半衰期”。“半衰期”是一段特定长度的时间。每过一个半衰期，数量就减少一半。假设这里有一块 1 KG 的镭块，为了计算方便，假设镭的半衰期为4天。四天以后，这块金属中实际上只含有 $\frac{1}{2}$ KG 的镭了，再过 4 天则只剩下 $\frac{1}{4}$ KG 的镭了。

我们认为现实中的指数衰减在每一个时刻的瞬时衰减速率是相同的，从微观的角度来讲就是每一颗粒子每一个瞬间衰变的概率都是相同的。用数学的语言来表达就是

$$
\frac{dN(t)}{dt}=-\lambda N(t)
$$

<span style="color:#ff6060">
  其中 $N$ 为粒子的相对数量，$t$ 为时间，$N(t)$ 粒子数量关于时间的函数，$\lambda$ 是正数且是常数。由于 $\lambda$ 决定了变化的速率， $\lambda$ 也被称为衰变常数。
</span>

## 指数衰减的函数表达式

为了进一步研究指数衰减的性质，我们可以先找到这个函数的表达式。方法如下：

$$
\frac{dN(t)}{dt}=-\lambda N(t)
$$

$$
\frac{dN(t)}{dt} \frac{1}{N(t)} =-\lambda
$$

$$
\int \frac{dN(t)}{dt} \frac{1}{N(t)} dt=\int -\lambda dt
$$

$$
\ln{N(t)}= -\lambda t + C
$$

$$
N(t) = N_0 e ^{-\lambda t}, N_0 = e^C
$$
<span style="color:#ff6060">
  因此，这个函数的表达式是 $N(t) = N_0 e ^{-\lambda t}$。我们可以进一步设 $N_0 = N(0) = 1$，于是表达式进一步化简为 $N(t) = e^{-\lambda t}$。
</span>
请注意，虽然结论是<b>正确</b>的，以上推导过程<b>并不完全</b>，我们会在后面给出完整的推导过程。

## 衰变常数与半衰期的关系

我们首先尝试着推导出半衰期。<span style="color:#ff6060">假设存在 $t_1, t_2$ 两个时刻，$t_2$ 时刻粒子的数量只有 $t_1$ 时刻的一半</span>，即

$$
\frac{N(t_2)}{N(t_1)} = \frac{e^{-\lambda t_2}}{e^{-\lambda t_1}} = \frac{1}{2}
$$

因此，

$$
2 e^{-\lambda t_2} = e^{-\lambda t_1}
$$

$$
e^{\ln{2}} e^{-\lambda t_2} = e^{-\lambda t_1}
$$

$$
e^{\ln{2} - \lambda t_2} = e^{-\lambda t_1}
$$

$$
\ln{2} - \lambda t_2 = -\lambda t_1
$$

$$
t_1 - t_2 = \frac{1}{\lambda} \ln{2}
$$

我们发现，不论 $t_1, t_2$ 具体的值是什么，它们的<span style="color:#ff6060">差是固定的</span>。<span style="color:#ff6060">因为 $t$ 的物理学含义是时间，所以 $t_1 - t_2$ 的物理学含义就是一段固定长度时间，我们把这段时间称为半衰期，记作 $t_{\frac{1}{2}}$</span>。

因此，半衰期的长度为衰变常数的倒数乘以 $\ln{2}$，也就是 <span style="color:#ff6060">$t_{\frac{1}{2}} = \frac{1}{\lambda} \ln{2}$</span>。

## 衰变常数与平均寿命的关系

为了算出寿命的平均值，我们可以找到粒子的<span style="color:#ff6060">寿命的概率分布函数</span>，也就是粒子的相对数量随寿命分布的函数。显然，概率分布函数与指数衰减的函数有很大关系。为了简化计算，我们<span style="color:#ff6060">采用 $N_0 = N(0) = 1$ 时的表达式: $N(t) = e^{-\lambda t}$</span>。假设初始粒子的相对数量是 $1$，也就是 $N(0)$，因此设定义域为 $[0,+\infty)$。设概率分布函数为 P(t)，根据定义 $\int_{a}^{b} P(t)$是随机取一个粒子，寿命落在区间 $(a, b)$ 内的概率，也就是寿命在 $a$ 和 $b$ 之间的粒子的相对数量，也就是 $N(a) - N(b)$。因此，

$$
\int_{a}^{b} P(t) = N(a) - N(b) = - N(t) |_{a}^{b}
$$

根据微积分基本定理，

$$
P(t) = -\frac{dN(t)}{dt}
$$

又因为

$$
\frac{dN(t)}{dt}=-\lambda N(t)
$$

因此

$$
P(t) = \lambda N(t) = \lambda e^{-\lambda t}
$$

平均寿命为

$$
\int_{0}^{+\infty} t P(t) = \int_{0}^{+\infty} \lambda t N(t) = \lambda \int_{0}^{+\infty} t e^{-\lambda t} = \lambda e^{-\lambda t}(\frac{-\lambda t - 1}{\lambda ^ {2}}) | _{0} ^{+\infty} = \frac{1}{\lambda}
$$

或者，我们也可以找到粒子的<span style="color:#ff6060">寿命随相对数量分布的函数</span>并求<span style="color:#ff6060">平均值</span>。粒子的寿命随相对数量分布的函数是粒子的相对数量随寿命分布的函数的反函数，记作 <span style="color:#ff6060">$P^{I}$</span> (用角标 $^{I}$ 表示函数的反函数是这篇文章的作者的特色) 。

通过求函数 $P^{I}$ 的值在 $(0, +\infty)$ 以内的平均值也可以得到寿命的平均值。要求平均值，需要求反函数值在 $(0, +\infty)$ 以内的定积分再除以积分上界和下界的差。但是不需要直接求出定积分，反函数值在 $(0, +\infty)$ 以内的定积分就等于原函数 $P(t)$ 在 $(0, +\infty)$ 上的定积分。因为原函数 $P(t)$ 是概率分布函数且 $(0, +\infty)$ 就是它的定义域，其定积分为 $1$。<span style="color:#ff6060">反函数 $P^{I}$ 的值在 $(0, +\infty)$ 以内的定积分的上界和下界是原函数 $P(t)$ 在 $t = 0, +\infty$ 处的值 $\lambda, 0$</span>。

因此平均寿命为，

$$
\lim_{a \to 0} \frac{1}{\lambda - a} \int_{a}^{\lambda} P^{I}(y) dy
= \frac{1}{\lambda} \int_{0}^{\infty} P(t) dt
= \frac{1}{\lambda}
$$

## 指数衰减的函数表达式的完整推导过程

啊吧啊吧啊吧啊吧

还没学会，之后再写。