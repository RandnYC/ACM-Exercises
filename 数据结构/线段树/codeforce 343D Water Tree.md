#### [codeforce 343D Water Tree](http://codeforces.com/problemset/problem/343/D)

@[dfs序, 线段树, 树链剖分]

&ensp; 题目大意: 给出一棵树, 每次操作可以将一个节点及其子节点置1, 或将该节点及其祖先节点置0, 或者询问单个节点的01状态

&ensp; 可以直接上树链剖分, 而这里使用dfs序建线段树。由于祖先的分布在dfs序上并不是连续的，所以线段树只能单点更新，而单点更新会超时。

&ensp; 这里可以只更新当前节点，于是当查询的时候则是查询这个节点的子树下是否有0; 同时当需要把一个节点下置1时, 则需要把这个节点的父亲置0, 以此来维护把路径上的祖先置0的操作

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 5e5+10;

int N, Q;
vector<int> G[MAXN];
int pre[MAXN];

int clk = 0;
int L[MAXN], R[MAXN];
bool vis[MAXN];
void dfs(int u=1, int fa=0)
{
    if( vis[u]) return ;
    vis[u] = 1;

    L[u] = ++ clk;  pre[u] = fa;
    for( int i = 0; i < G[u].size(); i++ )
    {
        int v = G[u][i];
        dfs(v, u);
    }
    R[u] = clk;
}

bool full[MAXN*4], dt[MAXN*4];
inline void pushdown(int root, int L, int R)
{
    if( dt[root] )
    {
        full[root] = dt[root];
        if( L != R )
        {
            dt[root*2] = dt[root*2+1]
                    = dt[root];
        }
        dt[root] = 0;
    }
}
void update(int root, int a, int b, bool fill, int L=1, int R=N)
{
    if( a == L && b == R && fill )
        return void(dt[root] = fill);

    pushdown(root, L, R);

    if( L == R && !fill )
        return void(full[root] = fill);

    int mid = L + (R-L)/2;
    if( b <= mid ) update(root*2, a, b, fill, L, mid);
    else if( a > mid ) update(root*2+1, a, b, fill, mid+1, R);
    else
    {
        update(root*2, a, mid, fill, L, mid);
        update(root*2+1, mid+1, b, fill, mid+1, R);
    }

    pushdown(root*2, L, mid), pushdown(root*2+1, mid+1, R);
    full[root] = full[root*2] && full[root*2+1];
}
bool query(int root, int a, int b, int L=1, int R=N)
{
    pushdown(root, L, R);

    if( a == L && b == R) return full[root];
    int mid = L + (R-L)/2;

    if( b <= mid ) return query(root*2, a, b, L, mid);
    else if( a > mid ) return query(root*2+1, a, b, mid+1, R);
    else
    {
        return query(root*2, a, mid, L, mid) &&
            query(root*2+1, mid+1, b, mid+1, R);
    }
}

int main()
{
    scanf("%d", &N);
    for( int i = 1; i < N; i++ )
    {
        int a, b;  scanf("%d%d", &a, &b);
        G[a].push_back(b);
        G[b].push_back(a);
    }

    dfs();

    scanf("%d", &Q);
    while(Q--)
    {
        int op, u;
        scanf("%d%d", &op, &u);

        if( op == 1 )
        {
            if( u != 1 && !query(1, L[u], R[u]) )
                update(1, L[pre[u]], L[pre[u]], 0);
            update(1, L[u], R[u], 1);
        }
        else if( op == 2) update(1, L[u], L[u], 0);
        else printf("%d\n", query(1, L[u], R[u]));
    }
    return 0;
}

```