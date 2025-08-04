---
title: vector function
description: 
author: doxawahebi
date: 2025-08-04 15:39:00 +0900
categories: [math]
tags: [math, calculus, vector-function]     # TAG names should always be lowercase
pin: false
math: false
mermaid: false
---



## Vector function
---
벡터함수(vector function)는 정의역이 실수의 집합이고 치역이 벡터의 집합인 함수이다.

다음과 같은 벡터함수 $r(t)$가 있다. 
$$r(t)=<f(t), g(t), r(t)>=f(t)i+g(t)j+h(t)k$$
f, g, h는 성분함수(component function)라고 한다. 
참고로 함수이기 때문에 모든 t에 대하여 벡터$V_3$가 유일하게 존재한다.

(벡터함수 그림 예시)

## Space curve
---
f, g, h를 구간 I에서 연속인 실숫값 함수라고 할 때 t가 구간 I 전체에서 변한다면 다음 식을 만족하는 공간의 모든 점 (x, y, z)의 집합 C를 공간곡선(Space curve)라 한다.
$$x = f(t), y=g(t), z=h(t)$$
(공간곡선 그림 예시)

이 방정식을 C의 매개변수방정식(parameter equations of C), t를 매개변수(parameter)라 한다.

매개변수방정식에 대응하는 벡터방정식은 다음과 같다. 이 방정식을 곡선 C의 매개변수화(parametrization)이라 한다.

$$r(t)=f(t)i+g(t)j+h(t)k$$

## derivatives
---

$$\frac{dr}{dt}=r'(t)=\lim_{h \to 0}\frac{r(t+h)-r(t)}{h}$$

$r'(t)$가 존재하고 $r'(t)\neq0$이면 곡선의 점 P에서 접선벡터(tangent vector)가 존재한다.

(tangent vector 그림)

접선벡터와 방향이 같은 unit vector를 단위 접선 벡터(unit tangent vector)라고 한다.
$$T(t)=\frac{r'(t)}{|r'(t)|}$$


다음 Theorem은 중요하니까. 머리 속에 넣어두자.

> $|r(t)|$=$c$ (constant)이면 $r'(t)$는 모든 t에 대해 $r(t)$와 orthogonal한다.

따라서 $r(t) \cdot r'(t)=0$가 성립힌다

($r'(t)$와 $r(t)$와 orthogonal한 그림, 화살표 2개가 직교)

## Integral
---
연속된 벡터함수 $r(t)$의 정적분은 실수함수와 비슷하지만 결과값은 벡터이다.

다음과 같이 정의할 수 있다.

$$
\int_{a}^{b}r(t)dt = (\int_{a}^{b}f(t)dt)i+(\int_{a}^{b}g(t)dt)j+
(\int_{a}^{b}h(t)dt)k
$$

## Arc Length
---
공간곡선의 길이는 다음 공식으로 구할 수 있다.

$$L=\int_{a}^{b}\sqrt{[f'(t)]^2+[g'(t)]^2+[h'(t)]^2}dt=\int_{a}^{b}\sqrt{(\frac{dx}{dt})^2+(\frac{dy}{dt})^2+(\frac{dz}{dt})^2}dt$$

r(t)에 대하여 다음이 성립하므로
$$|r'(t)|=|f'(t)i+g'(t)j+h'(t)k|=\sqrt{[f'(t)]^2+[g'(t)]^2+[h'(t)]^2}$$

다음과 같이 작성할 수도 있다.
$$L=\int_{a}^{b}|r'(t)|dt$$

### Arc Length Function
---
$a \leq t \leq b$일 때 Arc Length Function s를 다음과 같이 정의한다.
$$s(t)=\int_{a}^{t}|r'(u)|du=\int_{a}^{t}\sqrt{(\frac{dx}{du})^2+(\frac{dy}{du})^2+(\frac{dz}{du})^2}du$$

미적분학의 기본정리 1을 사용하여 양변을 미분하면 다음과 같이 된다.
$$\frac{ds}{dt}=|r'(t)|$$

(r(a)와 r(t), s(t)의 관계를 표현한 그림)

## Curvature
---
구간 $I$에서 $r'$이 연속이고 $r'(t) \neq 0$이라면 parametrization $r(t)$은 I에서 매끄럽다(smooth)라고 한다. smooth curve는 첨점(cusp)이 없고 tangent vector가 방향을 바꿀 때 연속적으로 방향을 바꾼다.

이때 unit tangent vector $T(t)$는 다음과 같이 주어진다.
$$T(t)=\frac{r'(t)}{|r'(t)|}$$
(이해가 되지 않는다면 $r'(t)$가 $r(t)$와 직교인 것과 |v|가 길이라는 것을 생각하자.)

unit tangent vector $T$에 대해 곡률 Curvature은 다음과 같이 정의한다.

$$k=|\frac{dT}{ds}|$$
$$k(t)=|\frac{T'(t)}{r'(t)}|=\frac{|r'(t) \times  r''(t)|}{|r'(t)|^3}$$

곡률은 그 점에서 곡선이 얼마나 빠르게 방향을 바꾸는 가에 대한 척도(measure)이다. 또한 호의 길이에 대한 unit tangent vector의 변화율의 크기로 정의할 수 있다.

(곡선 C에서 많은 점에서 단위접선벡터이 있는 그림, 곡선이 얼마나 빠르게 바뀌는 지 대충 보인다.)

특별히 $r(x)=xi+f(x)j$이면 
$$k(x)=\frac{|f''(x)|}{[1+(f'(x)^2)]^{3/2}}$$
가 성립한다.


## The Normal and Binormal Vectors
---
임의의 점에서 주단위법선벡터(principal unit normal vector) N(t)를 다음과 같이 정의할 수 있다.

$$N(t)=\frac{T'(t)}{|T'(t)|}$$

(T'(t)는 T(t)와 직교한다는 점을 참고하자.)

vector $B(t) = T(t) \times N(t)$를 종법선벡터(binormal vector)라고 한다.

(벡터 N과 벡터 T, vector B가 직교하는 그림)


곡선 C 위의 점 P에서 normal vector N과
 binormal vector B에 의해 결정되는 plane을 P에서 C의 법평면(normal plane)이라고 한다.

벡터 T와 N에 의해 결정되는 평면을 P에서 C의 접촉평면(osculating plane)이라한다.

(normal plane과 osculating plane을 보여주는 그림)

P에서 C의 곡률원(circle of curvature) 또는 접촉원(osculating circle)은 벡터 N을 따라서 P로부터 $1/k$ 떨어진 거리에 중심이 있으며 반지름이 $1/k$이고 P를 지나는 접촉평면에 있는 원이다. 이때 원의 중심을 P에서 C의 곡률중심(center of curvature)이라 한다.

(곡률원에 대한 그림)

다음을 만족하는 스칼라 $\tau$ 가 존재한다.
$$\frac{dB}{ds}=- \tau N$$

곡선 C의 비틀림율(torsion)은 다음과 같다.

$$\tau = - \frac{dB}{ds} \cdot N$$


$$\tau(t)=-\frac{B'(t) \cdot N(t)}{|r'(t)|}=\frac{[r'(t) \times r''(t)] \cdot r'''(t)}{|r'(t) \times r''(t)|^2}$$

## Reference
---
Calculus 9E - James Stewart

## appendix 
---
helix, toroidal spiral, trefoil knot, Twisted cubic($t, t^2, t^3$)

Differentiation Rules

differential geometry
Frenet-Serret formulas
evolute
