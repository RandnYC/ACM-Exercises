#### [HDU 4757 Tree](http://acm.hdu.edu.cn/showproblem.php?pid=4757)

@[lca, 异或, 字典树, 可持久化]

&ensp; 题目大意: 给出一棵树, 树上节点对应权值, 每次询问给出一个数k, 查询树上两个编号最短距离节点中和k异或的最大值

&ensp; 参考主席树的思想, 将01字典树可持久化, 维护的是每个节点前缀的个数, 每个节点拷贝父亲节点的版本。

&ensp; 查询时, 记区间为[a, b], 当前节点的信息也就是
sum(roota) + sum(rootb) - sum(lca(a, b)) - sum(pre[lca(a, b)]), 因为要异或最大, 所以根据查询给出的k的当前位的相反值来递归

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
const int MAXN = 1e5+10;
const short MAXD = 18;

int N, Q;
vector<int> G[MAXN];
LL weight[MAXN];

vector<int> root(1);
int stkptr = 0;

struct Trie
{
    int cnt;
    int next[2];
}t[MAXN*3*10];


int insert(int preroot, LL& x, int i=MAXD)
{
    int root = stkptr ++;
    t[root] = t[preroot];
    t[root].cnt ++;

    if( !i ) return root;

    bool bit = x &(1LL<<(i-1));
    t[root].next[bit] = insert(
        t[preroot].next[bit], x, i-1
    );

    return root;
}

LL query(int aroot, int broot, int lroot, int plroot, LL &k, int i=MAXD)
{
    if( !i ) return 0;
    bool bit = !(k &(1LL<<(i-1)));
    int sum = t[t[aroot].next[bit]].cnt + t[t[broot].next[bit]].cnt
        - t[t[lroot].next[bit]].cnt - t[t[plroot].next[bit]].cnt;

    if( sum ) return ((LL)bit<<(i-1)) + query(t[aroot].next[bit], t[broot].next[bit],
                t[lroot].next[bit], t[plroot].next[bit], k, i-1);

    else return ((LL)(!bit)<<(i-1)) + query(t[aroot].next[!bit], t[broot].next[!bit],
            t[lroot].next[!bit], t[plroot].next[!bit], k, i-1);
}

int seq[MAXN], rmq[MAXN*4][20];
int rseq[MAXN*4], rdepth[MAXN*4];
int ck, ck2;

int pre[MAXN], idx[MAXN];
bool vis[MAXN];
void dfs(int u=1, int fa=0, int d=0)
{
    if( vis[u] ) return ;

    seq[u] = ++ck;
    rseq[ck] = u, rdepth[ck] = d;
    vis[u] = 1;

    idx[u] = ++ck2;  pre[u] = fa;
    root.push_back(insert(root[idx[fa]], weight[u]));

    for(int i = 0; i < G[u].size(); i++ )
    {
        int v = G[u][i];
        dfs(v, u, d+1);
        rseq[++ck] = u, rdepth[ck] = d;
    }
}

void init()
{
    for( int i = 1; i <= ck; i++ ) rmq[i][0] = i;

    for( int k = 1; (1<<k) <= ck; k++ )
    {
        for( int i = 1; i+(1<<k)-1 <= ck; i++ )
        {
            int x = rmq[i][k-1];
            int y = rmq[i+(1<<(k-1))][k-1];
            rmq[i][k] = rdepth[x] < rdepth[y]? x :y;
        }
    }
}

int getLCA(int x, int y)
{
    x = seq[x], y = seq[y];
    if( x > y ) swap(x, y);

    int k = log(y-x+1)/log(2);
    x = rmq[x][k], y = rmq[y-(1<<k)+1][k];

    return rdepth[x] < rdepth[y]? rseq[x] :rseq[y];
}


void clear()
{
    ck = ck2 = 0, stkptr = 1;
    root.clear();  root.push_back(0);
    memset(pre, 0, sizeof pre);
    memset(vis, 0, sizeof vis);
    for( int i = 0; i <= N; i++ ) G[i].clear();
}

int main()
{
    while(~scanf("%d%d", &N, &Q))
    {
        clear();
        for( int i =1 ; i <= N; i++ )
            scanf("%lld\n", weight+i);
        for( int i = 1; i < N ; i++ )
        {
            int a, b;  scanf("%d%d", &a, &b);
            G[a].push_back(b), G[b].push_back(a);
        }

        dfs();  init();

        while(Q--)
        {
            int a, b;  LL k;
            scanf("%d%d%lld", &a, &b, &k);

            int lca = getLCA(a, b);
            int pl = pre[lca];
            a = idx[a], b = idx[b];
            lca = idx[lca], pl = idx[pl];

            printf("%lld\n", k ^query(
                root[a], root[b],
                root[lca], root[pl], k
            ));
        }
    }
    return 0;
}
```