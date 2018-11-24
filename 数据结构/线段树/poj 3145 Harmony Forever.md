#### [poj 3145 Harmony Forever](http://poj.org/problem?id=3145)

@[线段树, 权值线段树]

&ensp; 题目大意: 给出M个操作, 可以为集合添加元素以及询问模某个数余数最小且优先靠后的元素的下标

&ensp; 把但前序列元素按模数大小划分区间, 用权值线段树来查询下标; 同时用暴力直接遍历来优化模数比较小的情况

```cpp
#include <iostream>
#include <cstdio>
#include <cmath>
#include <cstring>
#include <algorithm>
using namespace std;
const int INF = 1 << 30;
const int MAXN = 5e5+10;

int N, M, Max;
int a[MAXN];

struct STree
{
    int L, R;
    int idx;
}t[MAXN*4];

void build(int root, int L, int R)
{
    t[root].L = L, t[root].R = R;
    t[root].idx = 0;

    if( L == R ) return ;
    int mid = L + (R-L)/2;
    build(root*2, L , mid);
    build(root*2+1, mid+1, R);
}

void update(int root, int value, int idx)
{
    int L = t[root].L, R = t[root].R;

    if( L == R )
    {
        t[root].idx = idx;
        return ;
    }

    int mid = L + (R-L)/2;
    if( value <= mid ) update(root*2, value, idx);
    else update(root*2+1, value, idx);

    if( !t[root*2].idx ) t[root].idx = t[root*2+1].idx;
    else t[root].idx = t[root*2].idx;
}

int query(int root, int a, int b)
{
    int L= t[root].L, R = t[root].R;
    if( !t[root].idx ) return 0;
    if( a == L && b == R ) return t[root].idx;

    int mid = L + (R-L)/2;
    if( b <= mid ) return query(root*2, a ,b);
    else if( a > mid) return query(root*2+1, a, b);
    else
    {
        int ans1 = query(root*2, a, mid);
        if( ans1 ) return ans1;
        int ans2 = query(root*2+1, mid+1, b);
        return ans2;
    }
}

int query(int mod)
{
    int ans = -1, Min = INF;
    if( !N ) ;
    else if( mod <= 5e3 )
    {
        for( int i = N; i >= 1 ; i--)
        {
            register int tmp = a[i] %mod;
            if( tmp < Min )
                Min = tmp , ans = i;
            if( !Min ) break;
        }
    }
    else
    {
        for( int i = 0; i <= Max/mod ; i++)
        {
            register int tmp =
                query(1, max(i*mod, 1), min((i+1)*mod-1, Max));
            if( !tmp ) continue;

            if( a[tmp] %mod < Min) Min = a[tmp] %mod, ans = tmp;
            else if( a[tmp] %mod == Min && tmp > ans ) ans = tmp;
        }
    }
    return ans;
}


int main()
{
    int Case = 0;
    while( ~scanf("%d", &M) && M )
    {
        if( Case ) puts("");
        printf("Case %d:\n", ++Case);

        N = Max = 0;
        build(1, 1, MAXN);
        for( int i = 1 ; i <= M ; i++ )
        {
            register char op; register int x;
            scanf(" %c%d", &op, &x);

            if( op == 'B')
            {
                a[++N] = x, Max = max(Max, a[N]);
                update(1, x, N);
            }
            else printf("%d\n", query(x));
        }
    }
    return 0;
}
```