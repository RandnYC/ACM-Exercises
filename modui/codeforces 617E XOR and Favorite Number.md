#### [codeforces 617E XOR and Favorite Number](http://codeforces.com/problemset/problem/617/E)

@[莫队, 前缀和]

&ensp; 题目大意: 给出一个长度为N的序列, 和Q个询问, 以及数K, 对于每个询问要求回答区间内有多少个连续子区间的异或和为K

&ensp; 预处理异或前缀和, O(1)转移, 每次在区间两头增删元素x, 所对答案的贡献为对应的区间内有多少个 x ^k的连续异或和

&ensp; 因为如果要求[l, r]区间的异或, 需要求前缀r - 前缀l-1, 所以需要把每个询问的l-1端点也算进来


```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
const int MAXN = 1e5+10;

int N, Q, K;
int arr[MAXN];

struct Query {
    int l, r, id;
    static int sz;
};
bool operator< (Query& x, Query& y)
{
    int& sz = Query::sz;
    if( x.l /sz == y.l /sz) {
        return x.l /sz &1?
            x.r < y.r: x.r > y.r;
    }
    return x.l < y.l;
}

int sum[MAXN];
int Query::sz;

LL cnt[1<<21], g_res = 0;
inline void add(int& x) {
    g_res += cnt[x ^K], cnt[x] ++;
}
inline void del(int& x) {
    cnt[x] --, g_res -= cnt[x ^K];
}

int main()
{
    scanf("%d%d%d", &N, &Q, &K);

    for( int i = 1; i <= N; i++ )
    {
        scanf("%d", arr+i);
        sum[i] = sum[i-1] ^arr[i];
    }

    vector<Query> qy;
    for( int q = 1; q <= Q; q++ )
    {
        int l, r;  scanf("%d%d", &l, &r);
        qy.push_back({--l, r, q});
    }
    Query::sz = ceil(N /sqrt(Q));
    sort(qy.begin(), qy.end());

    static LL ans[MAXN];
    int pl = 1, pr = 0;
    for( auto &o : qy)
    {
        int &l = o.l, &r = o.r;

        while(pr < r) add(sum[++pr]);
        while(pr > r) del(sum[pr--]);
        while(pl < l) del(sum[pl++]);
        while(pl > l) add(sum[--pl]);

        ans[o.id] = g_res;
    }

    for( int q = 1; q <= Q; q++ )
        printf("%lld\n", ans[q]);

    return 0;
}

```
