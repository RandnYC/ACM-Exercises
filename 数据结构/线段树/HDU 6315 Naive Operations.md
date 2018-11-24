#### [HDU 6315 Naive Operations](http://acm.hdu.edu.cn/showproblem.php?pid=6315)

@[线段树]

&ensp; 题目大意: 给出一个序列$b_i$, 一系列操作, 对于另一个初始为0的序列$a_i$, 每次可以为一段区间加1, 也可以询问一段区间内的$\sum{⌊\frac{a_i}{b_i}⌋}$的结果

&ensp; 考虑到只有每次当$a_i$达到$b_i$的倍数时才对结果有贡献, 所以考虑每次修改操作可以反过来, 把区间内的数-1, 减到0重置为对应$b_i$, 对结果+1。
&ensp; 本来因为是非线性操作，若用线段树只能单点更新来维护结果，这里可以转化为维护$b_i$序列的区间最小值，当区间最小值-1>0 时对结果没有影响, 因此可以打上懒惰标记直接退出 来优化单点更新, 否则则必须更新到叶子节点

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 1e5+10;

int N, Q;
int barr[MAXN];

struct STree
{
    int Minb;
    int dt, sum;
}t[MAXN*4];

inline void pushdown(int root)
{
    if( t[root].dt )
    {
        t[root].Minb -= t[root].dt;

        t[root*2].dt += t[root].dt;
        t[root*2+1].dt += t[root].dt;

        t[root].dt = 0;
    }
}

inline void pushup(int root)
{
    pushdown(root*2), pushdown(root*2+1);

    t[root].Minb = min(
        t[root*2].Minb,
        t[root*2+1].Minb
    );

    t[root].sum =
        t[root*2].sum
        + t[root*2+1].sum;
}

void build( int root, int L=1, int R=N)
{
    t[root].dt = t[root].sum = 0;

    if( L == R )
    {
        t[root].Minb = barr[L];
        return ;
    }

    int mid = L + (R-L)/2;
    build(root*2, L, mid);
    build(root*2+1, mid+1, R);

    pushup(root);
}

void update(int root, int a, int b, int L=1, int R=N)
{
    pushdown(root);

    if( a == L && b == R )
    {
        if( L == R )
        {
            if( !--t[root].Minb )
            {
                t[root].Minb = barr[L];
                t[root].sum ++;
            }
            return ;
        }

        if( t[root].Minb-1 > 0 )
        {
            t[root].dt++;
            return ;
        }
    }

    int mid = L + (R-L)/2;
    if( b <= mid ) update(root*2, a, b, L, mid);
    else if( a > mid ) update(root*2+1, a, b, mid+1, R);
    else
    {
        update(root*2, a, mid, L, mid);
        update(root*2+1, mid+1, b, mid+1, R);
    }

    pushup(root);
}

int query(int root, int a, int b, int L=1, int R=N)
{
    if( a == L && b == R ) return t[root].sum;

    int mid = L + (R-L)/2;
    if( b <= mid ) return query(root*2, a, b, L, mid);
    else if( a > mid ) return query(root*2+1, a, b, mid+1, R);
    else
    {
        return query(root*2, a, mid, L, mid)
            + query(root*2+1, mid+1, b, mid+1, R);
    }
}

int main()
{
    while(~scanf("%d%d", &N, &Q))
    {
        for( int i = 1; i <= N ; i++ )
            scanf("%d", barr+i);

        build(1);

        char op[7];
        while(Q--)
        {
            register int a, b;
            scanf("%s%d%d", op, &a, &b);

            if(op[0] == 'a') update(1, a, b);
            else printf("%d\n", query(1, a, b));
        }
    }
    return 0;
}
```