#### [CodeForces 786B 	Legacy](https://vjudge.net/contest/243934#problem/M)

@[线段树, 最短路]

&ensp; 求起点到所有点的最短路, 建图时一个点和多个点相连。

&ensp; 线段树建图跑迪杰斯特拉.
&ensp; 建图时构造两棵线段树, 一个自下而上一个自下而上

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
typedef pair<int, LL> PIL;
const LL INF = 1LL << 62;
const int MAXN = 1e5+10;

int N, M, st;
vector<PIL> G[MAXN*4];

int stkptr;
int re[MAXN*4][2];

int build(int root, int fg, int L=1, int R=N)
{
    if( L == R ) return re[root][fg] = L;
    int ptr = re[root][fg] = stkptr ++;

    int mid = L + (R-L)/2;

    int lnxt = build(root*2, fg, L, mid );
    int rnxt = build(root*2+1, fg, mid+1, R);

    if( fg )
    {
        G[ptr].push_back({lnxt, 0});
        G[ptr].push_back({rnxt, 0});
    }
    else
    {
        G[lnxt].push_back({ptr, 0});
        G[rnxt].push_back({ptr, 0});
    }
    return ptr;
}

void update(int root, int v, int a, int b, int w, int fg, int L=1, int R=N)
{
    if( a == L && b == R )
    {
        if( fg ) G[v].push_back({re[root][fg], w});
        else G[re[root][fg]].push_back({v, w});
        return ;
    }

    int mid = L + (R-L)/2;
    if( b <= mid ) update(root*2, v, a, b, w, fg, L, mid);
    else if(a > mid ) update(root*2+1, v, a, b, w, fg, mid+1, R);
    else
    {
        update(root*2, v, a, mid, w, fg, L, mid);
        update(root*2+1, v, mid+1, b, w, fg, mid+1, R);
    }
}

struct comp {
    bool operator() (PIL a, PIL b)
    { return a.second > b.second; }
};

LL dist[MAXN];
void dijkstra()
{
    fill(dist+1, dist+ stkptr, INF), dist[st] = 0;

    priority_queue<PIL, vector<PIL>, comp> que;
    que.push({st, 0});

    while(!que.empty())
    {
        PIL tp = que.top();
        que.pop();

        int& u = tp.first;
        for( int i = 0; i < G[u].size(); i++)
        {
            int& v = G[u][i].first;
            LL& w = G[u][i].second;

            if( dist[v] > dist[u] + w )
            {
                dist[v] = dist[u] + w;
                que.push({v, dist[v]});
            }
        }
    }
}

int main()
{
    scanf("%d%d%d", &N, &M, &st);

    stkptr = N+1;
    build(1, 1), build(1, 0);

    while(M--)
    {
        int op; scanf("%d", &op);
        if( op == 1 )
        {
            int a, b, w;
            scanf("%d%d%d", &a, &b, &w);
            G[a].push_back({b, w});
        }
        else
        {
            int v, a, b, w;
            scanf("%d%d%d%d", &v, &a, &b, &w);
            if( op == 2 ) update(1, v, a, b, w, 1);
            else update(1, v, a, b, w, 0);
        }
    }

    dijkstra();
    for( int i = 1; i <= N; i++ )
    {
        printf("%lld%c", dist[i]==INF? -1: dist[i],
                        i == N? '\n': ' ');
    }

    return 0;
}
```