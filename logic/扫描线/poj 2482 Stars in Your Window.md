#### [poj 2482 Stars in Your Window](http://poj.org/problem?id=2482)

@[线段树, 扫描线]

&ensp; 题目大意, 给你N颗星星和宽W长H的框, 和N颗星星的坐标和亮度, 问能框住的星星亮度和最大多少

&ensp; 以每颗星星的坐标为线段的左端点, 构造权值为此星亮度的长W线段, 同时构造对称线段权值为相反数相距H在上。
&ensp; 从下往上区间更新维护线段树的区间最大值, 记录全过程的最大值即结果。
&ensp; 由于相反数线段需要先处理， 所以当线段高度相同时， 按线段权值排序。

```cpp
#include <iostream>
#include <cstdio>
#include <vector>
#include <cstring>
#include <algorithm>
using namespace std;
typedef long long LL;
const int MAXN = 1e4 +10;

int N, H, W;

struct Segment
{
    LL L, R;
    LL height;
    LL value;
    bool operator< (const Segment& o)const
    {
        if( height == o.height )
            return value < o.value;
        return height < o.height;
    }
}ln[MAXN*2];

vector<LL> cordx;
int getidx(LL x)
{
    return lower_bound(
        cordx.begin(), cordx.end(),
          x) - cordx.begin();
}

struct STree
{
    int L, R;
    LL Max, dt;
}t[MAXN*2*4];

void build( int root, int L, int R)
{
    t[root].L = L, t[root].R = R;
    t[root].Max = t[root].dt = 0;

    if( L == R ) return ;
    int mid = L + (R-L)/2;
    build( root*2 , L, mid );
    build( root*2+1, mid+1, R);
}

void pushdown(int root)
{
    if( t[root].dt )
    {
        int L = t[root].L, R = t[root].R;
        t[root].Max += t[root].dt;

        if( L != R )
        {
            t[root*2].dt += t[root].dt;
            t[root*2+1].dt += t[root].dt;
        }
        t[root].dt = 0;
    }
}

void update(int root, int a, int b, LL v)
{
    int L = t[root].L, R = t[root].R;
    if( a == L && b == R)
    {
        t[root].dt += v;
        return ;
    }

    pushdown(root);

    int mid = L + (R-L)/2;
    if( b <= mid ) update(root*2, a, b, v);
    else if( a > mid ) update(root*2+1, a, b, v);
    else
    {
        update(root*2, a, mid, v);
        update(root*2+1, mid+1, b, v);
    }

    pushdown(root*2), pushdown(root*2+1);
    t[root].Max = max(t[root*2].Max, t[root*2+1].Max);
}

int main()
{
    while(~scanf("%d%d%d", &N, &W, &H))
    {
        cordx.clear(), cordx.push_back(-1);
        for( int i = 1; i <= N; i++ )
        {
            LL x, y, brt;
            scanf("%lld%lld%lld", &x, &y, &brt);
            ln[i] = {x, x+W, y, brt};
            ln[N+i] = {x, x+W, y+H, -brt};
            cordx.push_back(x), cordx.push_back(x+W);
        }
        sort(ln+1, ln+2*N+1), sort(cordx.begin(), cordx.end());
        cordx.erase(unique(cordx.begin(), cordx.end()), cordx.end());

        LL ans = 0;
        build(1, 1, cordx.size());
        for( int i = 1 ; i <= 2*N ; i++)
        {
            const LL a = ln[i].L , b= ln[i].R;
            update(1, getidx(a), getidx(b)-1, ln[i].value);
            ans = max( ans, t[1].Max + t[1].dt );
        }
        printf("%lld\n", ans);
    }
    return 0;
}

```