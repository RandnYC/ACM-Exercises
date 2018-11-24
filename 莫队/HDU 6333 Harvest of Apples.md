#### [HDU 6333 Harvest of Apples](http://acm.hdu.edu.cn/showproblem.php?pid=6333)
@[莫队]

&ensp; 题目大意: 给出最多1e5组N和M, 求 $\sum_{i=0}^{M}{C_N^i}$ 

&ensp;  莫队, O(1)转移。

&ensp; 由组合数递推公式: $C_N^M = C_{N-1}^M+C_{N-1}^{M-1}$
&ensp; 得: 
$$
\left\{\begin{matrix}
\sum_{i=0}^{M}{C_N^i}=2 \times \sum_{i=0}^{M}{C_{N-1}^i}-C_N^M \\
\sum_{i=0}^{M}{C_N^i} = \sum_{i=0}^{M-1}{C_N^i} + C_N^M\\
\end{matrix}\right.\\$$

$$记 \ \ f(n, m) = \sum_{i=0}^{M}{C_N^i}$$
$$\therefore \left\{\begin{matrix}
\ \ f(n, m) = 2 \cdot f(n-1,m )-C_N^M\\
f(n, m) = f(n, m-1)+C_N^M\\
\end{matrix}\right.\\$$

&ensp; 莫队区间转移时要保证 m <= n

&nbsp;

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
const int MAXN = 1e5+10;
const int MOD = 1e9+7;

int N, M;
LL fac[MAXN], inv[MAXN];

LL qpow(LL base, LL index)
{
    LL ans = 1;
    while(index)
    {
        if( index & 1)
            ans = (ans *base) %MOD;
        index /=2 ;
        base = (base *base) %MOD;
    }
    return ans;
}

void preDeal()
{
    fac[0] = 1;
    for( int i = 1; i < MAXN; i++ )
    	fac[i] = fac[i-1] *i %MOD;

    inv[MAXN-1] = qpow(fac[MAXN-1], MOD-2);
    for(int i = MAXN-2; i >= 0; i--)
        inv[i]=inv[i+1]*(i+1)%MOD;
}

LL C(LL n, LL m)
{
    return fac[n] *inv[m] %MOD
		*inv[n-m] %MOD;
}

int unit;
LL g_res = 0;
LL ans[MAXN];

struct Query
{
    int n, m;
    int idx;
    bool operator< (const Query& o) const
    {
        if( n/unit == o.n/unit)
			return m < o.m;
        return n < o.n;
    }
}qy[MAXN];

inline void add(int n, int m, bool tag)
{
    if( tag ) g_res = ((g_res *2) - C(n-1, m) + MOD) %MOD;
    else g_res = (g_res + C(n, m)) %MOD;
}
inline void del(int n, int m, bool tag)
{
    if( tag ) g_res = (g_res + C(n-1, m)) %MOD* inv[2] %MOD;
    else g_res = (g_res - C(n, m) + MOD) %MOD;
}

int main()
{
    int T;  cin >> T;

    preDeal();

    unit = sqrt(MAXN);
    for( int Case = 0; Case < T; Case++)
    {
        scanf("%d%d", &N, &M);
        qy[Case] = {N, M, Case};
    }
    sort(qy, qy+T);

    int m = 1, n = 1;  g_res = 2;
    for( int i = 0; i < T; i++ )
    {
        while(n < qy[i].n) add(++n, m, 1);
        while(n > qy[i].n) del(n--, m, 1);
        while(m < qy[i].m) add(n, ++m, 0);
        while(m > qy[i].m) del(n, m--, 0);

        ans[qy[i].idx] = g_res;
    }

    for( int i = 0; i < T; i++ )
        printf("%lld\n", ans[i]);

    return 0;
}
```
