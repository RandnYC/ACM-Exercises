#### [EOJ 3631 Delivery Service](https://acm.ecnu.edu.cn/problem/3631/)

@[树链剖分, 贪心, 离线]

&ensp; 题目大意: 给出一棵带边权的树, 和Q和询问, 并且可以任意交换树上的边权, 求所有询问路径之和的最小值

&ensp; 因为边权确定, 不难想到只要求出树上边的路径覆盖次数, 排序后贪心赋予边权即为答案。

&ensp; 使用树链剖分来记录询问的路径覆盖每条边的次数, 最后遍历一遍整棵树对边的次数排序

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef pair<int, int> PII;
typedef long long LL;

const int MAXN = 2e5 +10;
int depth[MAXN], fa[MAXN], son[MAXN], sz[MAXN];
//节点深度, 父节点, 重儿子, 子节点树大小
int idx[MAXN], top[MAXN], weight[MAXN];
//重新编号, 重链上头节点

int N, Q, mod=1e9+7;
vector<int> G[MAXN];

//找重儿子
int dfs1( int u , int pre , int d)
{
    depth[u] = d, fa[u] = pre, sz[u] = 1;

    int Max = 0;
    for( int i  = 0 ; i< G[u].size() ; i++ )
    {
        int v = G[u][i];
        if( v == pre ) continue;    //成环
        sz[u] += dfs1( v , u , d+1);
        if( sz[v] > Max ) Max = sz[v], son[u] = v;
    }
    return sz[u];
}

//节点重新编号并找重链顶
int clk = 0;
void dfs2(int u , int t )    //t为重链顶
{
    idx[u] = ++clk , top[u] = t;

    if( !son[u] ) return ;
    else dfs2( son[u] , t); //优先递归重儿子

    for( int i = 0 ; i< G[u].size() ; i++ )
    {
        int v = G[u][i];
        if( !idx[v])    dfs2( v, v );    //剩下轻儿子
    }
}

struct STree { LL sum, dt; } t[MAXN*4];
void build(int root=1, int L=1, int R=N)
{
    t[root].sum = t[root].dt = 0;
    if( L == R) return ;
    int mid = L + (R-L)/2;
    build(root*2, L, mid);
    build(root*2+1, mid+1, R);
}

void pushdown(int root, int L, int R)
{
    if( t[root].dt )
    {
        (t[root].sum += (R-L+1) *t[root].dt) %=mod;
        if( L != R)
        {
            (t[root*2].dt += t[root].dt) %=mod;
            (t[root*2+1].dt += t[root].dt) %=mod;
        }
        t[root].dt = 0;
    }
}

void update(int root, int a, int b, LL val, int L=1, int R=N)
{
    if( a == L && b == R ) return void((t[root].dt += val) %=mod);

    pushdown(root, L, R);

    int mid = L + (R-L)/2;
    if( b <= mid ) update(root*2, a, b, val , L, mid);
    else if(a > mid ) update(root*2+1, a, b, val, mid+1, R);
    else
    {
        update(root*2, a, mid, val, L, mid);
        update(root*2+1, mid+1, b, val, mid+1, R);
    }

    pushdown(root*2, L, mid), pushdown(root*2+1, mid+1, R);
    t[root].sum = (t[root*2].sum + t[root*2+1].sum) %mod;
}

LL query(int root, int a, int b, int L=1, int R=N)
{
    pushdown(root, L, R);
    if( a == L && b == R ) return t[root].sum;

    int mid = L + (R-L)/2;
    if( b <= mid ) return query(root*2, a, b, L, mid);
    else if( a > mid ) return query(root*2+1, a, b, mid+1, R);
    else
    {
        return (query(root*2, a, mid, L, mid)
            + query(root*2+1, mid+1, b, mid+1, R ) ) %mod;
    }
}

LL treeQuery(int a, int b)
{
    LL ans = 0;
    while( top[a] != top[b])  //重链跳跃
    {
        if( depth[ top[a] ] < depth[ top[b]] ) swap(a ,b);
        ans = (ans+ query(1, idx[top[a]], idx[a]) ) % mod;
        a = fa[top[a]];     //深度深的上浮
    }

    if( depth[a] > depth[b]) swap(a ,b);    //同一条重链下
    ans = (ans + query( 1, idx[a] , idx[b]) )% mod;    //重新编号的根必为1
    return ans;
}

void treeUpdate( int a , int b , int v)
{
    v %= mod;
    while( top[a] != top[b] )
    {
        if( depth[ top[a]] < depth[ top[b]]) swap( a ,b);
        update( 1 , idx[top[a]] , idx[a] ,v ) ;
        a = fa[top[a]];
    }
    if( depth[a] > depth[b]) swap(a, b);
    update(1, idx[a], idx[b], v);
    update(1, idx[a], idx[a], -v);
}

void init( int root=1 )
{
    dfs1(root, 0 , 1);
    dfs2(root, root );
    /*build();

    for( int i = 1 ; i <= N ; i++ )
        update(1, idx[i], idx[i], weight[i] %mod);*/
}

priority_queue<int, vector<int>, less<int> > que;
void dfs(int u=1, int pre=0)
{
    for( int i = 0; i < G[u].size(); i++ )
    {
        int v = G[u][i];
        if( v == pre ) continue;
        dfs(v, u);
        que.push(treeQuery(v, v));
    }
}

int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0), cout.tie(0);

    cin >> N;

    for( int i = 1; i < N; i++ )
    {
        int a, b, w;  cin >> a >> b >> w;
        G[a].push_back(b);
        G[b].push_back(a);
        weight[i] = w;
    }

    init();

    cin >> Q;
    while(Q--)
    {
        int a, b;  cin >> a >> b;
        treeUpdate(a, b, 1);
    }

    dfs();
    sort(weight+1, weight+N);

    LL ans = 0;  int k = 1;
    while(!que.empty())
    {
        int tp = que.top();  que.pop();
        ans += weight[k++] *tp;
    }
    cout << ans << endl;

    return 0;
}

```