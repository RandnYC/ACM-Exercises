#### [HDU 6397 Character Encoding](http://acm.hdu.edu.cn/showproblem.php?pid=6397)

@[组合数学, 容斥]

&ensp; 题目大意: 把K个完全相同的小球放进M个不同的盒子里, 并且每个盒子的球数量不超过N-1

&ensp; 现已知若把K个相同的小球放进M个盒子, 则有$C_{M+K-1}^{K}$种情况.

&ensp; 考虑若有 $i$ 个盒子的球数量超过了N-1, 则假定这些盒子各自的球的数量为N, 剩下的小球有$K-i\cdot N$个, 这些剩下的球在这M个盒子里随意摆放, 因为可能合法的位置摆放后仍然可能超过N-1, 和以后的某个$i$状态重复, 所以需要容斥

最后公式: 
$$C_{M+K-1}^{K} - \sum_{i=1}^{[\frac{K}{N}]}(-1)^{i} \cdot C_{M}^{i} \cdot C_{M+K-N\cdot i - 1}^{K- N\cdot i}$$

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
const int MAXN = 2e5+10;
const int MOD = 998244353;

LL N, M, K;

LL qpow(LL base, LL index)
{
    LL ret = 1;
    while(index)
    {
        if( index &1)
            ret = ret *base %MOD;
        index /= 2;
        base = base *base %MOD;
    }
    return ret;
}

LL fac[MAXN], inv[MAXN];
void preDeal()
{
    fac[0] = 1;
    for( int i = 1; i < MAXN; i++ )
        fac[i] = fac[i-1] *i %MOD;
    inv[MAXN-1] = qpow(fac[MAXN-1], MOD-2);

    for( int i = MAXN-2; i >= 0; i-- )
        inv[i] = inv[i+1] *(i+1) %MOD;
}

LL C(LL n, LL m)
{
    if(n == m) return 1;
    else if(n < m) return 0;
    return fac[n] *inv[m] %MOD *inv[n-m] %MOD;
}

int main()
{
    preDeal();

    int T;  cin >> T;
    while(T--)
    {
        cin >> N >> M >> K;

        LL ans = C(M+K-1, K);

        for( int i = 1; i <= K/N; i++ )
        {
            ans = (ans + MOD - (i &1? 1: -1)
                    *(C(M, i) *C(M+K-N*i-1, K-N*i) %MOD)) %MOD;
        }
        cout << ans << endl;
    }
    return 0;
}

```