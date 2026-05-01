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

## 辅助角公式

$$
\begin{aligned}

a\cos x + b\sin x 
&= \sqrt{a^2 + b^2}\,\sin(x + \varphi), 
\quad \tan \varphi = \frac{a}{b} \\

a\cos x + b\sin x 
&= \sqrt{a^2 + b^2}\,\cos(x - \varphi), 
\quad \tan \varphi = \frac{b}{a}

\end{aligned}
$$

## 等价无穷小代换

$$
\begin{eqnarray}
 \lim_{x \to 0} \\
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
\begin{aligned}

\left[ f(x) \pm g(x) \right]' 
&= f'(x) \pm g'(x) \\

\left[ f(x)\cdot g(x) \right]' 
&= f'(x)\cdot g(x) + f(x)\cdot g'(x) \\

\left[ \frac{f(x)}{g(x)} \right]' 
&= \frac{f'(x)g(x) - f(x)g'(x)}{g^2(x)} \\

\left[ f(g(x)) \right]' 
&= f'(g(x))\cdot g'(x)

\end{aligned}
$$

## 洛必达法则
$$
\begin{equation}
 \lim_{\varDelta x \to a} \frac{f(x)}{g(x)} = \lim_{\varDelta x \to a} \frac{f’(x)}{g’(x)} 
\end{equation}
$$

## 泰勒公式

$$
\begin{aligned}

e^x 
&= 1 + \frac{x}{1!} + \frac{x^2}{2!} + \frac{x^3}{3!} + \cdots \\

\sin x 
&= x - \frac{x^3}{3!} + \frac{x^5}{5!} - \cdots \\

\cos x 
&= 1 - \frac{x^2}{2!} + \frac{x^4}{4!} - \cdots \\

\tan x 
&= x + \frac{x^3}{3} + \frac{2x^5}{15} + \cdots \\

\frac{1}{1-x} 
&= 1 + x + x^2 + x^3 + \cdots \quad (|x|<1) \\

\frac{1}{1+x} 
&= 1 - x + x^2 - x^3 + \cdots \quad (|x|<1) \\

\ln(1+x) 
&= x - \frac{x^2}{2} + \frac{x^3}{3} - \frac{x^4}{4} + \cdots \quad (-1<x\le1) \\[10pt]


\text{泰勒展开（一般式）：} \\

f(x) 
&= f(x_0) 
+ f'(x_0)(x-x_0) 
+ \frac{f''(x_0)}{2!}(x-x_0)^2 \\
&\quad + \frac{f^{(3)}(x_0)}{3!}(x-x_0)^3 
+ \cdots

\end{aligned}
$$

## 不定积分

$$
\begin{aligned}

\int k \, dx 
&= kx + C \\

\int x^a \, dx 
&= \frac{x^{a+1}}{a+1} + C \quad (a \ne -1) \\

\int \frac{1}{x} \, dx 
&= \ln|x| + C \\

\int a^x \, dx 
&= \frac{a^x}{\ln a} + C \quad (a>0,\ a\ne1) \\

\int e^x \, dx 
&= e^x + C \\[8pt]


\int \sin x \, dx 
&= -\cos x + C \\

\int \cos x \, dx 
&= \sin x + C \\

\int \sec^2 x \, dx 
&= \tan x + C \\

\int \csc^2 x \, dx 
&= -\cot x + C \\[8pt]


\int \frac{1}{1+x^2} \, dx 
&= \arctan x + C \\

\int \frac{1}{\sqrt{1-x^2}} \, dx 
&= \arcsin x + C

\end{aligned}
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
\int_{0}^{\frac{\pi}{2}} \cos^n dx = \int_{0}^{\frac{\pi}{2}} \sin^n dx \\
n = 奇数 \\
\frac{n-1}{n} \times \frac{n-3}{n-2} \ldots \frac{2}{3} \times 1 \\
n = 偶数 \\
\frac{n-1}{n} \times \frac{n-3}{n-2} \ldots \frac{1}{2} \times \frac{\pi}{2} \\
\int_{0}^{\frac{\pi}{2}} \cos^5 xdx = \frac{4}{5} \times \frac{2}{3} \times 1 = \frac{8}{15} \\
\int_{0}^{\frac{\pi}{2}} \sin^6 xdx = \frac{5}{6} \times \frac{3}{4} \times \frac{1}{2} \times \frac{\pi}{2} = \frac{5\pi}{32}

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
 \int f(x)dx = 关于x的函数 \\
 \int_{a}^{b} f(x)dx = 数字 \\
 \int_{a}^{b} = F(a) - F(a) = 0 \\
 \int_{a}^{b} f(x)dx= - \int_{b}^{a} f(x)dx \\
 \int_{a}^{b} f(x)dx = \int_{a}^{c} f(x)dx + \int_{c}^{b} f(x)dx
\end{array}
$$

## 定积分应用

$$
\begin{array}
 y=x^2-2x x在[1,2)之间的曲线与x轴 x=1围成的图形 求该图形绕Y轴旋转一周形成的体积 \\
 横切法 \\
 y + (x-1)^2 - 1 \\
 y + 1 = (x - 1)^2 \\
 x = 1 \pm \sqrt{y+1} \\
 x = 1 + \sqrt{y+1} \\
 \int_{-1}{0} (1+\sqrt{y+1})^2
 \int_{-1}^{0}x^2\pi dy \\
 \pi int_{-1}{0}(1+y+1+2\sqrt{y+1})dy \\
 \pi int_{-1}{0}(2+y+2\sqrt{y+1})dy \\
 \pi (2y + \frac{y^2}{2} + (\sqrt{y+1})^\frac{3}{2}\frac{2}{3}2) \\
 \frac{17}{6} \pi \\
 V = \frac{17}{6} \pi - \pi = \frac{11}{6} \pi
 竖切法 \\
 体积 = \lvert y \rvert 2 \times \pi \times x \\
 \int_{1}{2} \lvert y \rvert 2 \pi x \\
 \int_{1}{2} (2x - x^2) 2 \pi x \\
 2x 2\pi x - 2\pi x^3 \\
 2\pi (2x^2 - x^3) \\
 2\pi int_{2}{1}(2x^2 - x^3)dx \\
 2\pi (\frac{2}{3} x^3 - \frac{x^4}{4})
\end{array}
$$

## 偏微分方程

$$
\begin{equation}
  y’=x 解 y=\frac{x^2}{2} + C
\end{equation}
$$

$$
\begin{array}
  y’= x 当x = 0 时 y = 1 \\
  解出 y=\frac{x^2}{2} + C \\
  带入 x = 0 \\
  1 = 0 + C \\
  y = \frac{x^2}{2} + 1
\end{array}
$$

$$
\begin{array}
  y’’= x \\
  解 y’ = \frac{x^2}{2} + c_1 \\
  y’’ = \frac{x^2}{x} +c_1x + c_2 \\
  每一次求原函数，需要加一个c
\end{array}
$$

## 一阶微分方程

### 分离变量法

$$
\begin{array}

  y’= 2xy \\
  y’=\frac{dy}{dx} \\
  \frac{dy}{dx} = 2xy \\
  \frac{dy}{y} = 2xdx \\
  \int \frac{dy}{y} = \int 2xdx \\
  ln\lvert y \rvert = x^2 +C \\
  隐函数，左右两边e的指数 \\
  e^{ln\lvert y \rvert} = e^{x^2+c} \\
  \lvert y \rvert = e^{x^2+c} \\
  \lvert y \rvert = e^{x^2} \times e^c \\
  因为 e^c 可以取到所有实数 写成C \\
  y=Ce^{x^2} \\
\end{array}
$$

### 通解

$$
\begin{array}
y’=p(x)y \\
y=ce^{\int p(x)dx}
\end{array}
$$


## 偏导数

$$
\begin{aligned}

\text{偏导定义：} \\
\frac{\partial z}{\partial x}
&= \lim_{\Delta x \to 0}
\frac{z(x+\Delta x,y)-z(x,y)}{\Delta x} \\

\frac{\partial z}{\partial y}
&= \lim_{\Delta y \to 0}
\frac{z(x,y+\Delta y)-z(x,y)}{\Delta y} \\[8pt]


\text{例1：} \\
z &= x^2 + 3xy + y^2 \\

\frac{\partial z}{\partial x}
&= 2x + 3y \\

\frac{\partial z}{\partial y}
&= 3x + 2y \\

\text{代入 }(1,2)： \\
\frac{\partial z}{\partial x}(1,2)
&= 8 \\
\frac{\partial z}{\partial y}(1,2)
&= 7 \\[10pt]


\text{第一题：} \\
S &= \frac{u^2 + v^2}{uv}
= \frac{u}{v} + \frac{v}{u} \\

\frac{\partial S}{\partial u}
&= \frac{1}{v} - \frac{v}{u^2} \\

\frac{\partial S}{\partial v}
&= -\frac{u}{v^2} + \frac{1}{u} \\[10pt]


\text{第二题：} \\
Z &= (1+xy)^y \\

\frac{\partial Z}{\partial x}
&= y^2 (1+xy)^{y-1} \\

Z &= e^{y\ln(1+xy)} \\

\frac{\partial Z}{\partial y}
&= e^{y\ln(1+xy)}
\left(
\ln(1+xy) + \frac{xy}{1+xy}
\right) \\[10pt]


\text{第三题：} \\
U &= x^{\frac{y}{z}} \\

\frac{\partial U}{\partial x}
&= \frac{y}{z} x^{\frac{y}{z}-1} \\

\frac{\partial U}{\partial y}
&= x^{\frac{y}{z}} \ln x \cdot \frac{1}{z} \\

\frac{\partial U}{\partial z}
&= x^{\frac{y}{z}} \ln x \left(-\frac{y}{z^2}\right) \\[10pt]


\text{第四题：} \\
Z &= x + (y-1)\arcsin\left(\sqrt{\frac{x}{y}}\right) \\

y=1 \Rightarrow\quad
Z &= x \\

\frac{\partial z}{\partial x}(0,1)
&= 1

\end{aligned}
$$

## 点到线公式

$$
\begin{aligned}

\text{2D} \\
d = \frac{Ax+By}{\sqrt{A^2+B^2}} \\

\text{3D} \\
\text{平面} \\
Ax+By+Cz+D=0 \\
d = \frac{Ax+By+Cz+D}{\sqrt{A^2+B^2+C^2}}

\end{aligned}
$$

## 全微分,方向导数,梯度

$$
\begin{aligned}

\textbf{一、全微分} \\[4pt]

dz 
&= \frac{\partial z}{\partial x} dx + \frac{\partial z}{\partial y} dy \\

\frac{\partial z}{\partial x} &: \text{x方向变化率} \\
\frac{\partial z}{\partial y} &: \text{y方向变化率} \\
dx &: \text{x的变化} \\
dy &: \text{y的变化} \\
dz &: \text{z的变化} \\[8pt]


\textbf{可全微分判定：} \\

\lim_{(dx,dy)\to(0,0)}
\frac{
z(x+dx,y+dy)-z(x,y)
- \left(
\frac{\partial z}{\partial x}dx
+ \frac{\partial z}{\partial y}dy
\right)
}{
\sqrt{dx^2+dy^2}
}
&= 0 \\[12pt]


\textbf{例1：} \quad z = e^{xy} \\

dz 
&= y e^{xy} dx + x e^{xy} dy \\[10pt]


\textbf{例2（近似计算）：} \\

f(x,y) &= x^y,\quad f(2,3)=8 \\

\frac{\partial z}{\partial x} 
&= y x^{y-1} \\
\frac{\partial z}{\partial y} 
&= x^y \ln x \\

df 
&= y x^{y-1} dx + x^y \ln x \, dy \\

df(0.002,0.001) 
&= 12 \cdot 0.002 + 5.6 \cdot 0.001 \\[12pt]


\textbf{二、方向导数} \\[4pt]

\frac{\partial z}{\partial \vec{l}} 
&= \frac{\partial z}{\partial x}\cos\theta 
+ \frac{\partial z}{\partial y}\sin\theta \\

&= \frac{
\frac{\partial z}{\partial x} \, \Delta x
+ \frac{\partial z}{\partial y} \, \Delta y
}{
\sqrt{\Delta x^2 + \Delta y^2}
} \\

\cos\alpha 
&= \frac{\Delta x}{\sqrt{\Delta x^2 + \Delta y^2}} \\
\cos\beta 
&= \frac{\Delta y}{\sqrt{\Delta x^2 + \Delta y^2}} \\[10pt]


\textbf{例1：} \\

z &= x e^{2y},\quad P(1,0)\rightarrow Q(2,-1) \\

\frac{\partial z}{\partial x} 
&= e^{2y} \\
\frac{\partial z}{\partial y} 
&= 2x e^{2y} \\

\frac{\partial z}{\partial x}(1,0) &= 1 \\
\frac{\partial z}{\partial y}(1,0) &= 2 \\

\vec{l} &= (1,-1) \\
|\vec{l}| &= \sqrt{2} \\

\cos\alpha &= \frac{1}{\sqrt{2}},\quad
\cos\beta = -\frac{1}{\sqrt{2}} \\

D_{\vec{l}} z 
&= 1 \cdot \frac{1}{\sqrt{2}} 
+ 2 \cdot \left(-\frac{1}{\sqrt{2}}\right) \\
&= -\frac{1}{\sqrt{2}} \\[12pt]


\textbf{例2：} \\

z &= x^2 - 2xy + y \\

\frac{\partial z}{\partial x} 
&= 2x - 2y \\
\frac{\partial z}{\partial y} 
&= -2x + 1 \\

(2,1) \Rightarrow 
\frac{\partial z}{\partial x} = 2,\quad
\frac{\partial z}{\partial y} = -3 \\

D_{\vec{l}} z 
&= 2\cos\theta - 3\sin\theta \\

&= (2,-3)\cdot(\cos\theta,\sin\theta) \\

\text{最大：} 
&\quad \theta = \arctan\!\left(-\frac{3}{2}\right) \\

\text{最小：} 
&\quad \theta = \arctan\!\left(-\frac{3}{2}\right)+\pi \\

\text{为0：} 
&\quad \theta = \arctan\!\left(-\frac{3}{2}\right)\pm\frac{\pi}{2} \\[12pt]


\textbf{例3（梯度性质）：} \\

f(x,y) &= \tfrac{1}{2}(x^2+y^2) \\

\nabla f(2,1) &= (2,1) \\

\text{最大方向：} &(2,1) \\
\text{最小方向：} &(-2,-1) \\
\text{为0方向：} &(1,-2),\ (-1,2) \\[10pt]


\textbf{梯度：} \\

\nabla z &= \left(\frac{\partial z}{\partial x},\frac{\partial z}{\partial y}\right)

\end{aligned}
$$




