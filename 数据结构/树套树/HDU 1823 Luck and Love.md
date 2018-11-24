#### [HDU 1823 Luck and Love](http://acm.hdu.edu.cn/showproblem.php?pid=1823)
@[线段树, 二维]
&ensp; 题目大意: 给出M个操作, 每次登记一个人的身高和活泼值和匹配度, 或者查询给定范围内身高和活泼值的最大匹配度

&ensp; 二维线段树维护最大值

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 1e3+10;

struct STree
{
    int L, R;
    struct SubSTree
    {
        int L, R;
        int Max;
    }t[1000*4];

    void build(int root, int L, int R)
    {
        t[root].L = L, t[root].R = R;
        t[root].Max = -1;

        if( L == R ) return ;
        int mid = L + (R-L)/2;
        build(root*2, L, mid);
        build(root*2+1, mid+1, R);
    }

    void update(int root, int pos , int val)
    {
        int L = t[root].L , R= t[root].R;
        if( L == R )
        {
            t[root].Max = max(t[root].Max, val);
            return ;
        }

        int mid = L+ (R-L)/2;
        if( pos <= mid ) update(root*2, pos, val);
        else update(root*2+1, pos, val);

        t[root].Max = max(t[root*2].Max, t[root*2+1].Max);
    }

    int query(int root, int a, int b)
    {
        int L = t[root].L , R= t[root].R;
        if( a == L && b == R ) return t[root].Max;

        int mid = L + (R-L)/2;
        if( b <= mid ) return query(root*2, a, b);
        else if( a > mid ) return query(root*2+1, a, b);
        else
        {
            return max( query(root*2, a, mid),
                query(root*2+1, mid+1, b));
        }
    }

}t[100*4];

void build(int root, int L, int R)
{
    t[root].L = L , t[root].R = R;
    t[root].build(1, 0, 1000);

    if( L == R ) return ;
    int mid = L + (R-L)/2;
    build(root*2, L, mid);
    build(root*2+1, mid+1, R);
}

void update(int root, int pos1, int pos2, int val)
{
    int L = t[root].L , R= t[root].R;
    t[root].update(1, pos2, val);

    if( L == R ) return ;

    int mid = L + (R-L)/2;
    if( pos1 <= mid ) update(root*2, pos1, pos2, val);
    else update(root*2+1, pos1, pos2, val);
}

int query(int root, int a1, int b1, int a2, int b2)
{
    int L = t[root].L , R= t[root].R;
    if( a1 == L && b1 == R )
        return t[root].query(1, a2, b2 );

    int mid = L + (R-L)/2;
    if( b1 <= mid ) return query(root*2, a1, b1, a2, b2);
    else if( a1 > mid ) return query(root*2+1, a1, b1, a2, b2);
    else
    {
        return max(
            query(root*2, a1, mid, a2, b2),
            query(root*2+1, mid+1, b1, a2, b2)
        );
    }
}

int M;

int main()
{
    while(~scanf("%d", &M) && M )
    {
        build(1, 100, 200);
        while(M--)
        {
            register char op; scanf(" %c", &op);

            if( op == 'I')
            {
                register int h, act, mh;
                register double tmp1, tmp2;
                scanf("%d%lf%lf", &h, &tmp1, &tmp2);
                act = tmp1*10, mh = tmp2*10;

                update(1, h, act, mh);
            }
            else
            {
                register int h1, h2, act1, act2;
                register double tmp1, tmp2;
                scanf("%d%d%lf%lf", &h1, &h2, &tmp1, &tmp2);
                act1 = tmp1*10, act2 = tmp2*10;

                if( h1 > h2 ) swap(h1, h2);
                if( act1 > act2 ) swap(act1, act2);
                int ans = query(1, h1, h2, act1, act2);
                if( ans < 0 ) puts("-1");
                else printf("%.1f\n", ans/10.0);
            }
        }
    }
    return 0;
}

```