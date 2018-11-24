#### [BZOJ 2588 Count on a tree](https://www.lydsy.com/JudgeOnline/problem.php?id=2588)
@[LCA, 主席树]

&ensp; 强制在线求树上静态第k大

&ensp;rmq预处理lca, 为节点先序遍历打上序号, 于是就子节点就可以根据父亲节点来构造可持久化权值线段树, 由于节点权值未知, 需要离散化处理, 记$sum(a, b)$为从a到b的和, 最后查询的答案为$sum(root, a)+sum(root, b)-sum(root, lca(a, b))-sum(root, pre(lca(a, b)))$, 节点标号需要转成先序序列处理


```cpp
#include <bits/stdc++.h>
using namespace std;
const int INF = 1 << 30;
const int MAXN = 1e5+10;

int N, Q;

vector<int> G[MAXN], weight;
int pre[MAXN];
int bak[MAXN];
int getid(int x)
{
    return lower_bound(
        weight.begin(), weight.end(),
    x) - weight.begin();
}

int ridx[MAXN], idx[MAXN], dfn[MAXN*3], rmq[MAXN*3][18];
int seq[MAXN*3], idepth[MAXN*3];
bool vis[MAXN*3];
int clk = 0, clk2 = 0;

void dfs(int u, int dep)
{
    dfn[u] = ++clk, seq[clk] = u;
    idepth[clk] = dep;
    ridx[++clk2] = u, idx[u] = clk2;

    vis[u] = 1;

    for(int i = 0; i < G[u].size(); i++ )
    {
        int v = G[u][i];
        if( !vis[v] )
        {
            dfs(v, dep+1); pre[v] = u;
            seq[++clk] = u, idepth[clk] = dep;
        }
    }
}

void init()
{
    dfs(1, 0);

    for( int i = 1; i <= clk; i++ ) rmq[i][0] = i;

    for( int k = 1; (1<<k) <= clk; k++ )
    {
        for( int i = 1; i+(1<<k)-1 <= clk; i++ )
        {
            int x = rmq[i][k-1];
            int y = rmq[i+(1<<(k-1))][k-1];
            rmq[i][k] = idepth[x] < idepth[y]? x :y;
        }
    }
}

int getLCA(int x, int y)
{
    x = dfn[x], y = dfn[y];
    if( x > y ) swap(x, y);

    int k = log2(y-x+1);
    x = rmq[x][k], y = rmq[y-(1<<k)+1][k];

    return idepth[x] < idepth[y]? seq[x] :seq[y];
}


int stkptr = 0; vector<int> root;
struct STree { int sum, lnxt, rnxt; } t[MAXN*3*10];

int build(int L=1, int R=N)
{
    int root = stkptr++;
    t[root].sum = 0;
    if( L == R ) return root;

    int mid = L + (R-L)/2;
    t[root].lnxt = build(L, mid);
    t[root].rnxt = build(mid+1, R);
    return root;
}

int update(int preroot, int pos, int L=1, int R=N)
{
    int root = stkptr++;
    t[root] = t[preroot];

    t[root].sum ++;
    if( L == R ) return root;

    int mid = L + (R-L)/2;
    if( pos <= mid )
        t[root].lnxt = update(t[preroot].lnxt, pos, L, mid);
    else t[root].rnxt = update(t[preroot].rnxt, pos, mid+1, R);
    return root;
}

int query(int aroot, int broot, int lroot, int plroot,
            int k, int L=1, int R=N)
{
    if( L == R ) return weight[L];
    int sum = t[t[aroot].lnxt].sum + t[t[broot].lnxt].sum
        - t[t[lroot].lnxt].sum - t[t[plroot].lnxt].sum;

    int mid = L + (R-L)/2;
    if( k <= sum)
    {
        return query(
            t[aroot].lnxt, t[broot].lnxt,
            t[lroot].lnxt, t[plroot].lnxt,
            k, L, mid
        );
    }
    else
    {
        return query(
            t[aroot].rnxt, t[broot].rnxt,
            t[lroot].rnxt, t[plroot].rnxt,
            k-sum, mid+1, R
        );
    }
}

void clear()
{
    clk = clk2 = stkptr = 0;
    for( int i = 1; i <= N ; i++)
        G[i].clear();
    root.clear(), weight.clear();
    weight.push_back(-INF);
}

int main()
{
    while(~scanf("%d%d", &N, &Q))
    {
        clear();
        root.push_back(build());

        for( int i = 1; i <= N; i++ )
        {
            scanf("%d", bak+i);
            weight.push_back(bak[i]);
        }

        for( int i = 1; i < N ; i++ )
        {
            register int a, b;
            scanf("%d%d", &a, &b);
            G[a].push_back(b), G[b].push_back(a);
        }
        init();

        sort(weight.begin(), weight.end());
        weight.erase(unique(
            weight.begin(), weight.end()
            ), weight.end()
        );

        for( int i = 1; i <= N; i++ )
        {
            root.push_back(update(
                root[idx[pre[ridx[i]]]],
                getid(bak[ridx[i]])
            ));
        }

        int ans = 0;
        while(Q--)
        {
            register int a, b, k, l, pl;
            scanf("%d%d%d", &a, &b, &k), a ^= ans;

            l = getLCA(a, b), pl = pre[l];

            ans = query(
                root[idx[a]], root[idx[b]],
                root[idx[l]], root[idx[pl]], k
            );
            printf("%d\n", ans);
        }
    }
    return 0;
}

```