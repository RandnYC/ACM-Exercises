#### [codeforce 712 E. Memory and Casinos](http://codeforces.com/contest/712/problem/E)

@[概率, 极限, 分治, 线段树]

&ensp; 题目大意: 给出一个序列, 每个序列的元素对应一个概率, 表示成功向右移动一格的概率, 给出Q个操作, 每个操作可以单点修改概率, 也可以询问成功从l 移动出 r的概率

&ensp; 记 f(l, r)表示成功从l点移动到r+1点的概率, g(l, r)表示成功从r 移动到 r+1的概率

$$
f(l,r) = f(l, mid) \cdot f(mid+1, r) + f(l, mid) \cdot (1-f(mid+1, r)) \cdot g(l, mid) \cdot f(mid+1, r) + \cdots \\
f(l, r) = f(l, mid) \cdot f(mid+1, r) \cdot \sum_{n=0}^{\infty}((1-f(mid+1, r)) \cdot g(l, mid)) ^n \\ 
f(l, r) = \frac{f(l, mid) \cdot f(mid+1, r)}{1-(1-f(mid+1, r)) \cdot g(l, mid)}
$$

&nbsp;

$$
g(l, r) = g(mid+1, r) + (1 - g(mid+1, r)) \cdot g(l, mid) \cdot f(mid+1, r) + (1-g(mid+1, r)) \cdot g(l, mid) \cdot (1 - f(mid+1,r) ) \cdot g(l, mid) \cdot f(mid+1, r) + \cdots \\ 
g(l, r) = g(mid+1, r) + (1 - g(mid+1, r)) \cdot g(l, mid) \cdot f(mid+1, r) \cdot \sum_{n=0}^{\infty}(g(l, mid) \cdot (1 - f(mid+1,r))^n \\
g(l, r) = g(mid+1, r) + \frac{(1 - g(mid+1, r)) \cdot g(l, mid) \cdot f(mid+1, r)}{1-g(l, mid) \cdot (1 - f(mid+1,r))}
$$


```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 1e5+10;

int N, Q;

struct Node { double f, g; } t[MAXN <<2];
Node pushup(Node t1, Node t2)
{
    double &f1 = t1.f, &g1 = t1.g;
    double &f2 = t2.f, &g2 = t2.g;

    double f = f1 *f2/(1-(1-f2)*g1);
    double g = g2 + (1-g2)*g1*f2/(1+(f2-1)*g1);
    return {f, g};
}
void update(int root, int pos, double val, int L=1, int R=N)
{
    if( L == R ) return void(t[root].f = t[root].g = val);

    int mid = L + (R-L)/2;
    if( pos <= mid ) update(root*2, pos, val, L, mid);
    else update(root*2+1, pos, val, mid+1, R);

    t[root] = pushup(t[root*2], t[root*2+1]);
}
Node query(int root, int a, int b, int L=1, int R=N)
{
    if( a == L && b == R) return t[root];

    int mid = L + (R-L)/2;
    if( b <= mid ) return query(root*2, a, b, L, mid);
    else if( a > mid ) return query(root*2+1, a, b, mid+1, R);
    else
    {
        return pushup(query(root*2, a, mid, L, mid),
                query(root*2+1, mid+1, b, mid+1, R));
    }
}

int main()
{
    scanf("%d%d", &N, &Q);

    for( int i = 1; i <= N; i++ )
    {
        int a, b;  scanf("%d%d", &a, &b);
        update(1, i, a *1.0/b);
    }

    while(Q--)
    {
        int op;  scanf("%d", &op);
        if( op == 1 )
        {
            int pos, a, b;
            scanf("%d%d%d", &pos, &a, &b);
            update(1, pos, a *1.0/b);
        }
        else
        {
            int a, b;  scanf("%d%d", &a, &b);
            printf("%.10f\n", query(1, a, b).f);
        }
    }

    return 0;
}
```