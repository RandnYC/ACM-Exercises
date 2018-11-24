#### [ZOJ 3624 Count Path Pair](http://acm.zju.edu.cn/onlinejudge/showProblem.do?problemCode=3624)

@[组合数学]

&ensp; 给出四个点A(0,0),B(p,0),C(m,q),D(m,n). 要求从A到D, 和B到C的整数端点的路径上不相交的有多少条

&ensp; 总路径为: $A->D: C(M+N, N);\ \  B->C: C(M+Q-P, Q)$乘法原理相乘。 考虑相交路径可以认为是从A到C再从C到D，同理从B到D再D到C，于是$A->C: C(M+Q, Q); \ \ B->D: C(M+N-P, N)$, 最后总路径-相交路径即可

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
const int MAXN = 2e5 +10;
const int MOD = 100000007;

LL fac[MAXN];
void preDeal()
{
    fac[0] = 1; 
    for( int i = 1; i < MAXN ; i++)
        fac[i] = fac[i-1]*i %MOD;
}

LL extendGCD(LL a, LL b, LL &x, LL &y)
{
    LL ret = 0;
    if( !a && !b) return -1;
    else if( !b )
    {
        x = 1, y = 0;
        return a;
    }
    else
    {
        ret = extendGCD(b, a%b, y, x);
        y -= a/b *x;
    }
    return ret;
}


LL modInverse(LL a, LL mod)
{
    LL x= 0 , y = 0;
    extendGCD(a, mod, x, y);
    return (x %mod +mod ) %mod;
}


LL C(LL n, LL m)
{
    return fac[n]*modInverse(fac[n-m], MOD)%MOD
        *modInverse(fac[m], MOD)%MOD;
}

int N, M, P, Q;
int main()
{
    preDeal();
    while(~scanf("%d%d%d%d", &M, &N, &P, &Q))
    {
        printf("%lld\n", (C(M+N, M)* C(M-P+Q, Q)%MOD-
                        C(M+Q, Q)* C(M+N-P, N)%MOD+MOD)%MOD);
    }
    return 0;
}
```