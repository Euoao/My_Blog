---
title: hello hexo
tags: '-a'
sticky: 1
abbrlink: 4e1ab0e7
date: 2022-06-29 14:37:46
---

### 约数

- 唯一分解定理：$$n=\prod_{i=1}^kp_i^{a_i}=p_1^{a_1}p_2^{a_2}p_3^{a_3}...p_s^{a_k}$$

- 约数个数定理：$f(n)=\prod_{i=1}^k{(a_i+1)}=(a_1+1)(a_2+1)(a_3+1)...(a_k+1)$

- 约数和定理：

  $f(n)=\prod_{i=1}^k \sum_{j=0}^{a_i}p_i^j=(1+p_1+...+p_1^{a_1})(1+p_2+...+p_2^{a_2})...(1+p_k+...+p_k^{a_k})=\frac{p_1^{a_1+1}-1}{p_1-1}\times\frac{p_2^{a_2+1}-1}{p_2-1}\times...\times\frac{p_k^{a_k+1}-1}{p_k-1}$



### 开方

> 牛顿迭代法

``` cpp
ll mySqrt(ll x) {
    if(x==0)return 0;
    ll ans=x;
    while(ans>x/ans){
        ans=(ans+x/ans)/2;
    }
    return ans;
}
```

