---
title: 微分積分の学習
author: zhangyile
date: 2026-2-21 13:42:00 +0800
categories: [math]
tags: [math,Development]
comments: false
math: true
img_path: /assets/img/
image:
  path: company_without/isogashii_man.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

## 三角函数半角公式

$$
\begin{eqnarray}
  \cos (\frac{\pi}{2} - \alpha) = \sin \alpha \\
  \cos (\frac{\pi}{2} - \alpha) = \sin \alpha
\end{eqnarray}
$$

## 三角函数二倍角公式

$$
\begin{eqnarray}
  \sin 2\theta = 2\sin \theta \times \cos \theta \\
  \cos 2\theta = \cos^2 \theta - \sin^2 \theta \\
  \cos 2\theta = 2\cos^2 \theta - 1 \\
  \cos 2\theta = 1 - 2\sin^ \theta \\
  \sin^2 \theta + \cos^2 \theta = 1 \\
  \tan^2 \theta = \frac{2\tan \theta}{1 - \tan^2 \theta}
\end{eqnarray}
$$

## 降次公式

$$
\begin{eqnarray}
  \cos^2 (\frac{\alpha}{2}) = \frac{1 + \cos \alpha}{2} \\
  \sin^2 (\frac{\alpha}{2}) = \frac{1 - \cos \alpha}{2} \\
  \cos^2 (\alpha) = \frac{1 + \cos 2\alpha}{2} \\
  \sin^2 (\alpha) = \frac{1 - \cos 2\alpha}{2} 
\end{eqnarray}
$$

## 等价无穷小代换

$$
\begin{eqnarray}
 \lim_{varDelta x \to 0} \\
 \sin x \to x  \\
 \tan x \to x  \\
 \arcsin x\to x \\
 \arctan x \to x \\
 \ln (x+1) \to x \\
 e^x -1 \to x  \\
 a^x -1 \to x \ln a  \\
 1 - \cos x \to \frac{x^2}{2} \\
 [\left(1 + x\right)^a - 1] \to ax
\end{eqnarray}
$$

## 无穷大

$$
\begin{equation}
\ln x \leq x^a \leq x^b \leq c^x \leq d^x \leq x! \leq x^x
\end{equation}
$$

## 常用导数

$$
\begin{array}{l}
C’ = 0 \\
(x^a)’ = ax^{a-1} \\
(\sin x)’ = \cos x \\
(\cos x)’ = -\sin x \\
(\tan x)’ = \sec^2 x \\
(\cot x)’ = -\csc^2 x \\
(\ln |x|)’ = \frac{1}{x} \\
(\log_{a}(x))’ = \frac{1}{x \ln a} \\
(e^x)’ = e^x \\
(a^x)’ = a^x \ln a \\
(\arcsin x)’ = \frac{1}{\sqrt{1-x^2}} \\
(\arctan x)’ = \frac{1}{1+x^2}
\end{array}
$$

## 导数四则运算

$$
\begin{array}
[f(x)\pm g(x)]’ = f’(x) \pm g’(x) \\
[f(x)\times g(x)]’ = f’(x) \times g(x) + f(x) \times g’(x) \\
[\frac{f(x)}{g(x)}] = \frac{f’(x) \times g(x) + f(x) \times g’(x)} {g^2(x)} \\
[f(g(x))]’ = f’(g(x)) \times g’(x)
\end{array}
$$

## 洛必达法则
$$
\begin{equation}
 \lim_{varDelta x \to a} \frac{f(x)}{g(x)} = \lim_{varDelta x \to a} \frac{f’(x)}{g’(x)} 
\end{equation}
$$

## 泰勒公式

$$
\begin{array}
e^x = 1 + \frac{1}{1!}x + \frac{1}{2!}x^2 + \frac{1}{3!}x^3+\ldots \\
sinx = x - \frac{1}{3!}x^3 + \frac{1}{5!}x^5 + \ldots \\
cosx = 1 - \frac{1}{2!}x^2 + \frac{1}{4!}x^4 + \ldots \\
tanx = x + \frac{1}{3}x^3 + \frac{2}{15}x^5 + \ldots \\
\frac{1}{1-x} = 1 + x + x^2 + x^3 + x^4 + \ldots \\
\frac{1}{1+x} = 1 - x + x^2 - x^3 + x^4 + \ldots \\
\ln(1+x) = x - \frac{x^2}{2} + \frac{x^3}{3} - \frac{x^4}{4} + \frac{x^5}{5} + \ldots \\
f(x) = f(x_0) + f’(x- x_0) + \frac{f’’(x_0)}{2!}x^2 + \ldots 
\end{array}
$$

## 不定积分

$$
\begin{array}
 \int kdx = kx + c \\
 \int \frac{1}{\sqrt{1-x^2}}dx = \arcsin (x) + c \\
 \int x^adx = \frac{x^{a+1}}{a+1} + c \\
 \int a^xdx = \frac{a^x}{\lna} + c \\
 \int \frac{1}{x} = \ln (|x|) + c \\
 \int \cos xdx = \sin x + c \\
 \int \sin xdx = -\cos x + c \\
 \int \frac{1}{1+x^2} = \arctanx x + c \\
 \int \frac{1}{\cos^2 x} = \int \sec^2 x = \tan x + c \\
 \int \frac{1}{\sin^2 x} = \int \csc^2 x = -\cot x + c \\
 \int e^xdx = e^x + c
\end{array}
$$


## 不定积分运算法则

$$
\begin{array}
 \int [f(x) \pm g(x)]dx= \int f(x)dx + \int g(x)dx \\
 \int k \times f(x) dx = k \times \int f(x)dx (k \neq 0)
\end{array}
$$

## 华莱士公式 点火公式
$$
\begin{array}
\int_{0}{\frac{\pi}{2}} \cos^n dx = \int_{0}{\frac{\pi}{2}} \sin^n dx \\
n = 奇数 \frac{n-1}{n} \times \frac{n-3}{n-2} \ldots \frac{2}{3} \times 1 \\
n = 偶数 \frac{n-1}{n} \times \frac{n-3}{n-2} \ldots \frac{1}{2} \times \frac{\pi}{2} \\

\int_{0}{\frac{\pi}{2}} \cos^5 xdx = \frac{4}{5} \times \frac{2}{3} \times 1 = \frac{8}{15} \\
\int_{0}{\frac{\pi}{2}} \sin^6 xdx = \frac{5}{6} \times \frac{3}{4} \times \frac{1}{2} \times \frac{\pi}{2} = \frac{5\pi}{32}
\end{array}
$$


## 不定积分根号处理

$$
\begin{array}
 \sqrt{a^2 - x^2} \\
 设 x = a \sin t \\
 \sqrt{a^2 - x^2} = a \times \cos t \\
 dx = a \times \cos t dt
\end{array}
$$

$$
\begin{array}
 \sqrt{a^2 + x^2} \\
 设 x = a \tan t \\
 \sqrt{a^2 + x^2} = a \times \sec t \\
 dx = a \times \sec^2 t dt
\end{array}
$$

$$
\begin{array}
 \sqrt{x^2 - a^2} \\
 设 x = a \sec t \\
 \sqrt{x^2 - a^2} = a \times \tan t \\
 dx = a \times \sec t \times \tan t dt
\end{array}
$$

## 分布积分

$$
\begin{array}
 \int udv = u \times v - \int vdu \\
 d(uv) = udv + vdu \\
 \int udv = \int d(uv) - \int vdu
\end{array}
$$

## 有理分式积分

$$
\begin{array}
 \int f(x)dx 关于x的函数 \\
 \int_{a}^{b} f(x) dx 数字 \\
 \int_{a}^{b} = 0 = F(a) - F(a) \\
 \int_{a}^{b} f(x)dx= - \int_{b}^{a} f(x)dx \\
 \int_{a}^{b} f(x)dx = \int_{a}^{c} f(x)dx + \int_{c}^{b} f(x)dx
\end{array}
$$







