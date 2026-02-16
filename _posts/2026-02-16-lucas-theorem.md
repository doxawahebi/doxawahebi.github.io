---
title: Lucas Theorem
description: 뤼카의 정리
author: doxawahebi
date: 2026-02-16 23:35:00 +0900
categories: [web]
tags: [lucas_theorem, number_theory]     # TAG names should always be lowercase
pin: false
math: true
mermaid: false
---


## Overview
---
뤼카의 정리는 아주 큰 수의 Combination을 작은 소수로 나눈 소수로 나눈 나머지를 구할 때 사용합니다.

보통 조합 $\binom{n}{r} \pmod p$을 구할 때, $n$과 $r$이 작다면 모듈러 역원(페르마의 소정리로 구함.)을 이용해 팩토리얼로 계산한다. (알고있겠지만 모듈러 역원이 나눗셈 대신에 사용된다.)

```cpp
ll P; // P must be prime
ll fact[2005];

ll power(ll base, ll exp) {
    ll res = 1;
    base %= P;
    while (exp > 0) {
        if (exp % 2 == 1) res = (res * base) % P;
        base = (base * base) % P;
        exp /= 2;
    }
    return res;
}
ll modInverse(ll n) { return power(n, P - 2); // 페르마의 소정리 }

// 미리 팩토리얼을 전처리(Precompute) 해두면 O(1) 혹은 O(log P)에 가능
void prepareFactorial(){
	fact[0] = 1;
	for(int i = 1; i < P; i++){
		fact[i] = (fact[i-1] * i) % P;
	}
}

ll nCr_small(ll n, ll r){
	if (r < 0 || r > n) { return 0; }
	ll num = fact[n];
	ll den = (fact[r] * fact[n-r]) % P;
	return (num * modInverse(den)) % P;
}
```

음이 아닌 정수 $n$과 $r$이 매우 크고(예: $10^{18}$ 이상), $p$가 상대적으로 작은 소수(예: $2000$ 이하)일 때는 일반적인 방법으로 계산할 수 없습니다. 이떄 뤼카의 정리를 사용하면 $O(\log_p n)$의 복잡도로 문제를 해결할 수 있습니다.


## background
---
뤼카의 정리에서 "자릿수별로 nCr을 계산해서 곱하기로 표현할 수 있는 이유"가 궁금한 경우 아래를 읽어보면 됩니다.

### 1학년의 꿈
---
소수 p로 나눈 나머지 세계에서는 다음이 성립합니다. ($GF(p)$)
$$(1+x)^p \equiv 1 + x^p \pmod p$$

이유는 이항정리로 전개해보면 알 수 있습니다.
$$(1+x)^p = 1 + \binom{p}{1}x + \binom{p}{2}x^2 + \dots + \binom{p}{p-1}x^{p-1} + x^p$$

여기서 중간에 있는 계수들 $\binom{p}{1}, \binom{p}{2}, \dots$은 분자에 소수 $p$가 들어있으므로, **모두 $p$의 배수**입니다.
즉, $\pmod p$를 하면 중간 항들은 전부 0이 되어 사라지고 양 끝의 $1$과 $x^p$만 남습니다.

### 식 변형
---
우리가 구하려는 건 $\binom{n}{r}$입니다. 이걸 다르게 보면 $(1+x)^n$을 전개했을 때 $x^r$의 계수가 $\binom{n}{r}$입니다.

이제 n을 p진법이라고 하고 두 자리라고 가정합니다. ($n = n_1p + n_0$)
$$(1+x)^n = (1+x)^{n_1p+n_0}$$
지수 법칙으로 쪼갭니다.
$$= (1+x)^{n_1p} \times (1+x)^{n_0}$$
$$= ((1+x)^p)^{n_1} \times (1+x)^{n_0}$$
여기서 아까 배운 1학년의 꿈을 사용합니다. $(1+x)^p$를 $1+x^p$로 바꿔버리는 겁니다.

$$\equiv (1+x^p)^{n_1} \times (1+x)^{n_0} \pmod p$$
### 계수 비교
---
이제 $x^r$의 계수를 찾습니다. (r역시 p진법으로 $r = r_1 p + r_0$라고 합시다.)

식은 두 덩어리의 곱으로 되어 있습니다.
1. $(1+x^p)^{n_1}$: 여기서는 $x$의 지수가 항상 $p$의 배수($0, p, 2p \dots$)인 항만 나옵니다.
2. $(1+x)^{n_0}$: 여기서는 $x$의 지수가 $p$보다 작은($0, 1, \dots, n_0$) 항만 나옵니다.

이제 $x^r = x^{r_1 p + r_0}$를 만들려면 다음과 같은 선택을 해야합니다.
- $(1+x^p)^{n_1}$에서 $x^p$를 $n_1$중 $r_1$번 선택해서 $x^{r_1 p}$를 꺼내고 (계수는 $\binom{n_1}{r_1}$)
- $(1+x)^{n_0}$에서 $x$를 $n_0$중 $r_0$번 선택해서 $x^{r_0}$를 꺼내서 (계수는 $\binom{n_0}{r_0}$)
- 둘을 곱해서 $x^{r_1 p + r_0}$을 만들어줍니다.

따라서 전체 $x^r$의 계수 $\binom{n}{r}$는 각 덩어리에서 나온 계수의 곱과 같습니다.

$$\binom{n}{r} \equiv \binom{n_1}{r_1} \times \binom{n_0}{r_0} \pmod p$$

이것이 바로 자릿수별로 쪼개서 곱해도 되는 이유입니다.

## 뤼카의 정리
---
n과 r을 p진법으로 전개하여 다음과 같이 나타낼 수 있습니다.
$$n = n_k p^k + \dots + n_1 p + n_0 \quad (0 \le n_i < p)$$
$$r = r_k p^k + \dots + r_1 p + r_0 \quad (0 \le r_i < p)$$

이때, 뤼카의 정리는 다음과 같이 성립합니다.

$$\binom{n}{r} \equiv \prod_{i=0}^k \binom{n_i}{r_i} \pmod p$$
즉, **"$n$과 $r$을 $p$진법으로 나타냈을 때, 각 자릿수끼리의 조합을 모두 곱한 것과 같다"** 는 뜻입니다.
참고로 만약 어떤 $i$에 대해 $r_i > n_i$라면, $\binom{n_i}{r_i} = 0$이 되어 전체 결과는 0이 됩니다. (작은 것에서 큰 것을 뽑을 수 없기 때문입니다.)

### Example
---
$\binom{10}{2} \pmod 3$을 구해봅시다. (실제 값: $\frac{10 \times 9}{2} = 45$, $45 \equiv 0 \pmod 3$)

1. **진법 변환 ($p=3$):**
    - $n = 10 \Rightarrow 1 \cdot 3^2 + 0 \cdot 3^1 + 1 \cdot 3^0 \Rightarrow (1, 0, 1)_3$
    - $r = 2 \Rightarrow 0 \cdot 3^2 + 0 \cdot 3^1 + 2 \cdot 3^0 \Rightarrow (0, 0, 2)_3$
        
2. **각 자리별 조합 계산:**
    - $i=0$: $\binom{n_0}{r_0} = \binom{1}{2} = 0$ (1개에서 2개를 뽑을 수 없음)
    - $i=1$: $\binom{n_1}{r_1} = \binom{0}{0} = 1$
    - $i=2$: $\binom{n_2}{r_2} = \binom{1}{0} = 1$
        
3. **최종 결과:**
    - $1 \times 1 \times 0 = 0$
    - 따라서 $\binom{10}{2} \equiv 0 \pmod 3$입니다.

### Time complexity
---
$O(\log_p n)$

## Code
---
```cpp
ll lucas(ll n, ll r) {
    if (r == 0) return 1;
    // n % P, r % P는 현재 자릿수(n_i, r_i)를 의미
    // n / P, r / P는 다음 자릿수로 넘어가는 과정
    return (lucas(n / P, r / P) * nCr_small(n % P, r % P)) % P;
}
```

`/ P`는 P진법에서 자릿수가 시프트되는것와 같다. 2진법에서 `/ 2`나 `>> 1`을 생각해보자.
