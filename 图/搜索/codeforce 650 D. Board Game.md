#### [codeforce 650 D. Board Game](http://codeforces.com/contest/605/problem/D)

@[树套树, BFS]

&ensp; 题目大意: 给出一个序列, 序列的每个元素有四种属性a, b, c, d。只有当某个元素的c, d属性都大于等于另一个元素的a, b属性时，可以转移，开始属性为0, 0, 0, 0, 求到达第N个元素的最小步长并打印路径

&ensp; 以 a 为下标建立树状数组，树状数组的每个节点维护一个集合记录 b 的大小，bfs 边搜边找路，因为bfs的特点，已经被搜到过的点可以在树状数组里删除来加快搜索

```cpp
#include <iostream>
#include <cstdio>
#include <cstring>
#include <vector>
#include <queue>
#include <set>
#include <stack>
#include <algorithm>
using namespace std;
const int INF = 1 << 30;
const int MAXN = 4e5+10;

int N;
struct Card {
    int a, b, c, d;
}arr[MAXN];

vector<int> bak;
int getid(int x) {
    return lower_bound(
        bak.begin(), bak.end(),
      x) - bak.begin();
}

struct Node { int v, idx; };
bool operator< (const Node& x, const Node& y)
{ return x.v < y.v || (x.v == y.v && x.idx < y.idx); }

set<Node> bitc[MAXN];
inline int lowbit(int &x) { return x & -x; }
void update(int pos, int val, int idx) {
    for( ; pos < bak.size(); pos += lowbit(pos))
        bitc[pos].insert({val, idx});
}
void query(int c, int d, set<int> &st)
{
    Node o = Node({d, INF});
    for( ; c > 0; c -=lowbit(c)) {
        auto iter = bitc[c].upper_bound(o);
        if( iter == bitc[c].begin()) continue;

        for( auto p = bitc[c].begin(); p != iter; p++)
            st.insert(p->idx);
        bitc[c].erase(bitc[c].begin(), iter);
    }
}

struct Point { int u, step, fa; };
int path[MAXN];
int bfs()
{
    queue<Point> que;
    que.push({0, 0, -1});

    static bool vis[MAXN];
    while(!que.empty()) 
    {
        auto tp = que.front();  que.pop();
        const int &u = tp.u, &fa = tp.fa;
        int step = tp.step;

        if(vis[u]) continue;
        vis[u] = 1;  path[u] = fa;

        if( u == N ) return step;

        set<int> g;
        query(getid(arr[u].c), getid(arr[u].d), g);

        for( int v :g) que.push({v, step+1, u});
    }
    return -1;
}

int main()
{
    scanf("%d", &N);

    bak.push_back(-1);
    for( int i = 1; i <= N; i++ )
    {
        scanf("%d%d%d%d", &arr[i].a, &arr[i].b,
                        &arr[i].c, &arr[i].d);
        bak.push_back(++ arr[i].a);
        bak.push_back(++ arr[i].b);
        bak.push_back(++ arr[i].c);
        bak.push_back(++ arr[i].d);
    }
    arr[0].a = arr[0].b = arr[0].c = arr[0].d = 1;
    bak.push_back(1);

    sort(bak.begin(), bak.end());
    bak.erase(unique(bak.begin(), bak.end()), bak.end());

    for( int i = 1; i <= N; i++ )
        update(getid(arr[i].a), getid(arr[i].b), i);

    int ans = bfs();
    printf("%d\n", ans);
    if( ~ans) {
        stack<int> stk;
        for(int p = N; p; p = path[p]) {
            stk.push(p); 
        }
        while(!stk.empty()) {
            int tp = stk.top(); stk.pop();
            printf("%d ", tp);
        }
        puts("");
    }

    return 0;
}
```