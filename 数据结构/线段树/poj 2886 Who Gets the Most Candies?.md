#### [poj 2886 Who Gets the Most Candies?](http://poj.org/problem?id=2886)
@[线段树, 全局第k大]

&ensp; 题目大意: N个人坐成一圈, 每人手持一个数字, 从第K个人开始, 每次点到的人出列, 并以那个数字点下个人, 问第几个出列的人的序号因数最多。

&ensp; 有点像是约瑟夫环，预处理所有序号因数的个数，用线段树来维护人数个数即查询第k大是序号几

```cpp
#include <iostream>
#include <cstdio>
#include <cstring>
#include <cmath>
#include <algorithm>
using namespace std;
const int MAXN = 5e5+10;

int N, K;
struct Player
{
    char name[15];
    int dt;
}p[MAXN];

struct STree
{
    int L, R;
    int idx, sum;
}t[MAXN*4];

void build( int root , int L, int R)
{
    t[root].L = L , t[root].R = R;

    if( L == R )
    {
        t[root].idx = L;
        t[root].sum = 1;
    }
    else
    {
        t[root].idx = 0, t[root].sum = R-L+1;

        int mid = L + (R-L)/2;
        build(root*2, L, mid );
        build(root*2+1, mid+1, R);
    }
}

void update(int root, int index, bool del)
{
    int L = t[root].L, R = t[root].R;
    if( L == R )
    {
        if( !del )
        {
            t[root].idx = index;
            t[root].sum = 1;
        }
        else t[root].idx = t[root].sum = 0;
        return ;
    }

    int mid = L + (R-L)/2;
    if( index <= mid ) update(root*2, index, del);
    else update(root*2+1, index, del);

    t[root].sum = t[root*2].sum + t[root*2+1].sum;
}

int query(int root, int a, int b)
{
    int L = t[root].L, R = t[root].R;
    if( a == L && b== R ) return t[root].sum;

    int mid = L + (R-L)/2;
    if( b <= mid ) return query(root*2, a, b);
    else if( a > mid ) return query(root*2+1, a, b);
    else
    {
        return query(root*2, a, mid)
            + query(root*2+1, mid+1, b);
    }
}

int query(int root, int k)
{
    int L = t[root].L, R = t[root].R;
    if( L == R ) return t[root].idx;

    if( t[root*2].sum >= k ) return query(root*2, k);
    else return query(root*2+1, k-t[root*2].sum);
}

int getNext(const int& x)
{
    register int dt = p[x].dt;
    register int k = query(1, 1, x)-1;
    if( dt > 0 ) k--;
    update(1, x, 1);

    if( !t[1].sum ) return 0;

    if( k+dt < 0)
        k += (-(k+dt)/t[1].sum+1)*t[1].sum;
    k = (k+dt) %t[1].sum +1;

    return query(1, k);
}

int subfac[MAXN];
void preDeal()
{
    memset( subfac , 0 , sizeof subfac );
    for( int i = 1 ; i < MAXN; i++ )
    {
        for( int j = i; j < MAXN ; j += i)
            subfac[j] ++;
    }
}

int main()
{
    preDeal();
    while(~scanf("%d%d", &N, &K ))
    {
        if( !N ) continue;
        build(1, 1, N);

        int Max = 0;
        for( int i = 1 ; i <= N ; i++ )
        {
            scanf("%s%d", p[i].name, &p[i].dt);
            Max = max( Max, subfac[i] );
        }

        int x = K, cnt = 1;
        while(x)
        {
            if( subfac[cnt] == Max ) break;
            x = getNext(x), cnt++;
        }

        printf("%s %d\n", p[x].name, Max);
    }

    return 0;
}
```