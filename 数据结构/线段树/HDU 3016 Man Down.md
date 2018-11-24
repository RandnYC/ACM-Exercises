#### [HDU 3016 Man Down](http://acm.hdu.edu.cn/showproblem.php?pid=3016)

@[线段树]

&ensp; 题目大意: 是男人就下一百层, 初始100血量, 落在最高的那个板上, 每次可以从板的左边或右边跳下去, 每个板都可以加血或扣血, 问到地面最多多少血?

&ensp; 用线段树来维护但前这层的挡板序号, 边更新边查询来获取每个挡板从左和从右落下的下个挡板的序号, 最后dp

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 1e5+10;

struct Blocks
{
    int L, R;
    int lnxt, rnxt;
    int height, value;
    bool operator < (const Blocks& o) const 
    { return height < o.height; }
}bk[MAXN];

struct STree
{
    int L, R;
    int idx, dt;
}t[MAXN*4];

void build(int root, int L, int R)
{
    t[root].L = L, t[root].R = R;
    t[root].idx = t[root].dt = 0;

    if( L == R ) return ;
    int mid = L + (R-L)/2;
    build(root*2, L, mid);
    build(root*2+1, mid+1, R);
}

void pushdown(int root)
{
    if( t[root].dt )
    {
        int L = t[root].L, R = t[root].R;
        t[root].idx = t[root].dt;
        if( L != R )
        {
            t[root*2].dt = t[root*2+1].dt
                = t[root].dt;
        }
        t[root].dt = 0;
    }
}

void update(int root, int a, int b , int index)
{
    int L = t[root].L, R = t[root].R;
    if( a == L && b == R )
    {
        t[root].dt = index;
        return ;
    }

    pushdown(root);
    int mid = L+ (R-L)/2;
    if( b <= mid ) update(root*2, a, b, index);
    else if( a > mid ) update(root*2+1, a, b, index);
    else
    {
        update(root*2,a , mid, index);
        update(root*2+1, mid+1, b, index);
    }
}

int query(int root, int x)
{
    int L = t[root].L, R = t[root].R;
    pushdown(root);

    if( L == R ) return t[root].idx;

    int mid = L + (R-L)/2;
    if( x <= mid ) return query(root*2, x);
    else return query(root*2+1, x);
}

int N;
int dp[MAXN];

int main()
{
    while(~scanf("%d", &N))
    {
        int l = 1 << 30, r = 0;
        for( int i = 1; i <= N ; i++)
        {
            scanf("%d%d%d%d", &bk[i].height, 
                &bk[i].L, &bk[i].R, &bk[i].value);
            l = min(l, bk[i].L);
            r = max(r, bk[i].R);
        }
        sort( bk+1, bk+1+N );
        build(1, l, r);
        for( int i = 1; i <= N ; i++)
        {
            bk[i].lnxt = query(1, bk[i].L);
            bk[i].rnxt = query(1, bk[i].R);
            update(1, bk[i].L, bk[i].R, i);
        }
        memset( dp, 0 , sizeof dp);
        dp[N] = 100+bk[N].value;
        for( int i = N; i >= 1; i--)
        {
            const int& lnxt = bk[i].lnxt;
            const int& rnxt = bk[i].rnxt;
            dp[lnxt] = max(dp[lnxt], 
                dp[i]+bk[lnxt].value);
            dp[rnxt] = max(dp[rnxt],
                dp[i]+bk[rnxt].value);
        }
        printf("%d\n", dp[0]>0 ? dp[0] : -1);
    }
    return 0;
}
```