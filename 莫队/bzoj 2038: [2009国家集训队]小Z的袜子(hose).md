#### [bzoj 2038: [2009国家集训队]小Z的袜子(hose)](https://www.lydsy.com/JudgeOnline/problem.php?id=2038)

@[莫队]

&ensp; 题目大意: 给出一个序列, 任取两个元素, 求取到相同的概率, 输出最简分数

&ensp; 莫队, 区间移动时统计元素个数, 维护加减一个元素后的组合数

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
typedef pair<LL, LL> PLL;
const int MAXN = 5e4+10;

LL gcd(LL a, LL b) { return !b? a :gcd(b, a%b); }

int N, Q;
int arr[MAXN];

int unit;
LL g_tmp;
int cnt[MAXN];
PLL ans[MAXN];

struct Query
{
    int L, R;
    int idx;
    bool operator< (const Query& o)const
    {
        if( L/unit == o.L /unit)
            return R < o.R;
        return L/unit < o.L/unit;
    }
}qy[MAXN];

inline void add(int x){
    g_tmp -= (LL)cnt[x]*(cnt[x]-1); cnt[x]++;
    g_tmp += (LL)cnt[x]*(cnt[x]-1);
}
inline void del(int x){
    g_tmp -= (LL)cnt[x]*(cnt[x]-1); cnt[x]--;
    g_tmp += (LL)cnt[x]*(cnt[x]-1);
}

int main()
{
    while(~scanf("%d%d", &N , &Q))
    {
        memset( cnt , 0, sizeof cnt );
        g_tmp = 0;

        for( int i = 1; i <= N ; i++ )
            scanf("%d", arr+i);

        unit = sqrt(N);
        for( int i = 1; i <= Q ; i++ )
        {
            scanf("%d%d", &qy[i].L, &qy[i].R);
            qy[i].idx = i;
        }
        sort(qy+1, qy+Q+1);

        int L = 1, R = 0; //若L从0开始组合数会计算出错
        for( int i = 1; i <= Q; i++ )
        {
            while(R < qy[i].R) add(arr[++R]);
            while(R > qy[i].R) del(arr[R--]);
            while(L < qy[i].L) del(arr[L++]);
            while(L > qy[i].L) add(arr[--L]);

            ans[qy[i].idx] =
                make_pair(g_tmp, (LL)(R-L)*(R-L+1));
        }

        for( int i = 1; i <= Q; i++)
        {
            LL a = ans[i].first, b = ans[i].second;
            if( !a ) puts("0/1");
            else
            {
                LL f = gcd(a, b);
                printf("%lld/%lld\n", a/f, b/f);
            }
        }
    }
    return 0;
}
```