---
title: '[USACO21DEC] Convoluted Intervals S'
tags:
  - 差分
categories: 
- ACM
- 杂题
abbrlink: c42198e8
date: 2022-07-03 15:33:02
updated: 2022-07-03 15:33:02
---

<!-- more -->

---

# [[USACO21DEC]  Convoluted Intervals S](https://www.luogu.com.cn/problem/P7992)

昨天训练赛题，感觉挺有意思的

## 题意

有$n$个区间，对于某个值$k$,问满足 $a_i+a_j\le k\le b_i+b_j$ 的有序对$(i,j)$的个数。

输出$0$到$2M$内的每个值$k$的答案

### 数据范围

$1\le N \le 2 \times 10^5$

$ 1\le M \le 5000$

$0\le a_i,b_i \le M$

## 思路

开始因为二维偏序问题，仔细看了看范围发现并不难。

可以发现，对于每一对$(i,j)$,它对$k$的贡献是连续一段的，恰好是$[a_i+a_j,b_i+b_j]$

发现$M$不大，因此我们可以暴力枚举$a_i,b_i$,然后利用差分实现区间加

### 代码

``` cpp
#include<bits/stdc++.h>
#define all(x) begin(x), end(x)
using namespace std;
using ll = long long;
using PII = pair<int,int>;
constexpr int mod = 998244353;
constexpr int INF = 0x3f3f3f3f;
constexpr int N = 1e5+5;
ll a[N],b[N];
ll pre[N];
int main(){
    std::ios::sync_with_stdio(false);
    std::cin.tie(nullptr);
    int n,m;
    cin>>n>>m;
    for(int i=0,x,y;i<n;++i){
        cin>>x>>y;
        ++a[x],++b[y];
    }
    for(int i=0;i<=m;++i){
        for(int j=0;j<=m;++j){
            pre[i+j]+=a[i]*a[j];
            pre[i+j+1]-=b[i]*b[j];
        }
    }
    for(int i=1;i<=2*m;++i)pre[i]+=pre[i-1];
    for(int i=0;i<=2*m;++i)cout<<pre[i]<<'\n';
    return 0;
}
```

<!-- Q.E.D. -->
