####  [HDU 5145 NPY and girls](http://acm.hdu.edu.cn/showproblem.php?pid=5145)

@[莫队, 逆元]

&ensp; 题目大意: 给定一个序列, 每次询问一个区间内的数有多少种排列

&ensp; 答案即 $\frac{(R-L+1)!}{\sum{(cnt(x)}!)}$ , 所以只要统计区间内相同元素的个数即可。因为阶乘结果很大, 又有模数, 再加上莫队算法需要在O(1)内完成移动, 所以需要预处理出1e4范围内的乘法逆元。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
const int MAXN = 3e4+10;
const int MOD = 1e9+7;

LL qpow(LL base, LL index=MOD-2)
{
    LL ans = 1;
    while(index)
    {
        if( index & 1 )
            ans = (ans *base) %MOD;
        index /= 2;
        base = (base *base) %MOD;
    }
    return ans;
}

LL fac[MAXN], inv_f[MAXN];
void preDeal()
{
    fac[0] = inv_f[0] = 1;
    for( int i = 1 ; i < MAXN; i++ )
    {
        fac[i] = (LL)fac[i-1]*i %MOD;
        inv_f[i] = qpow(fac[i]);
    }
}


int N, Q;
int arr[MAXN];

int unit, cnt[MAXN];
LL g_tmp, ans[MAXN];

struct Query
{
    int L, R;
    int idx;
    bool operator< (const Query& o)const
    {
        if( L/unit == o.L/unit)
            return R < o.R;
        return L/unit < o.L/unit;
    }
}qy[MAXN];


inline void move(int x, int dt){
    g_tmp = (g_tmp *inv_f[cnt[x]]) %MOD;
    cnt[x]+=dt; g_tmp = (g_tmp *fac[cnt[x]]) %MOD;
}


int main()
{
    preDeal();

    int T; cin >> T;
    while(T--)
    {
        scanf("%d%d", &N, &Q);

        memset(cnt , 0 , sizeof cnt);
        unit = sqrt(N), g_tmp = 1;

        for( int i = 1 ; i <= N ; i++ )
            scanf("%d", arr+i);

        for( int i = 1; i <= Q; i++ )
        {
            scanf("%d%d", &qy[i].L, &qy[i].R);
            qy[i].idx = i;
        }
        sort( qy+1, qy+Q+1 );

        int L = 1, R = 0;
        for( int i =1 ; i <= Q ; i++ )
        {
            while(R < qy[i].R) move(arr[++R], 1);
            while(R > qy[i].R) move(arr[R--], -1);
            while(L < qy[i].L) move(arr[L++], -1);
            while(L > qy[i].L) move(arr[--L], 1);

            ans[qy[i].idx] = fac[R-L+1] *qpow(g_tmp) %MOD;
        }

        for( int i = 1; i <= Q; i++ )
            printf("%lld\n", ans[i]);
    }
    return 0;
}

```