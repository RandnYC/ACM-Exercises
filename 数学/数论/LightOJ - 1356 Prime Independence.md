#### [LightOJ - 1356 Prime Independence ](http://lightoj.com/login_main.php?url=volume_showproblem.php?problem=1356)

@[二分图, 最大匹配, 数论, 素数]

&ensp; 题目大意: 给定一个序列, 求最大的一个子集满足集合的任意一个元素与任意一个素数之和不能在这个集合里找到

&ensp; 预处理素数, 将序列中只差一个质因子的两个元素连边双向边, 问题就等价于求二分图中最大独立集的大小, 使用HK算法加速匹配, 其中最大独立集=|定点数| - 最大匹配数


```cpp
#include<bits/stdc++.h>
using namespace std;
typedef long long LL;
const int MAXN = 4e4+10;
const int INF = 0x3f3f3f3f;

struct Node{
    int x,y;
}p[MAXN], it[MAXN];

struct Edge{
	int v;
	int next;
}edge[int(1e6)];

int nx, ny, tot;
int dist;

int head[MAXN];
int xlink[MAXN], ylink[MAXN];
int dx[MAXN], dy[MAXN];
int vis[MAXN];

void init(){
	tot = 0;
	memset(head, -1, sizeof(head));
	memset(xlink, -1, sizeof(xlink));
	memset(ylink, -1, sizeof(ylink));
}

void addEdge(int u, int v)
{
	edge[tot].v = v;
	edge[tot].next = head[u], head[u] = tot++;
}

int bfs()
{
	queue<int> q;
	dist = INF;
	memset(dx, -1, sizeof(dx));
	memset(dy, -1, sizeof(dy));
	for(int i = 1; i <= nx; i++){
		if(xlink[i] == -1){
			q.push(i);
			dx[i] = 0;
		}
	}
	while(!q.empty()){
		int u = q.front(); q.pop();
		if(dx[u] > dist) break;
		for(int e = head[u]; e != -1; e = edge[e].next){
			int v = edge[e].v;
			if(dy[v] == -1){
				dy[v] = dx[u] + 1;
				if(ylink[v] == -1) dist = dy[v];
				else{
					dx[ylink[v]] = dy[v]+1;
					q.push(ylink[v]);
				}
			}
		}
	}
	return dist != INF;
}

int find(int u)
{
	for(int e = head[u]; e != -1; e = edge[e].next){
		int v = edge[e].v;
		if(!vis[v] && dy[v] == dx[u]+1){
			vis[v] = 1;
			if(ylink[v] != -1 && dy[v] == dist) continue;
			if(ylink[v] == -1 || find(ylink[v])){
				xlink[u] = v, ylink[v] = u;
				return 1;
			}
		}
	}
	return 0;
}

int maxMatch()
{
	int ans = 0;
	while(bfs()){
		memset(vis, 0, sizeof(vis));
		for(int i = 1; i <= nx; i++)
            if(xlink[i] == -1)
                ans += find(i);
	}
	return ans;
}

vector<int> prime;

bool isPrime(int x)
{
    for(int i = 2; i*i <= x; i++ )
        if( x %i == 0) return 0;
    return 1;
}

void preDeal()
{
    for( int i = 2; i <= 5e5; i++ )
        if( isPrime(i) ) prime.push_back(i);
}

int N, arr[MAXN];
int pos[int(5e5+10)];

int main()
{
    int T;  cin >> T;
    int Case = 0;
    preDeal();
    while(T--)
    {
        memset(pos, 0, sizeof pos);

        scanf("%d", &N);
        init();  nx = ny = N;
        for( int i = 1; i <= N; i++ )
        {
            scanf("%d", arr+i);
            pos[arr[i]] = i;
        }

        for( int i = 1; i <= N; i++ )
        {
            for( int k = 0; k < prime.size(); k++ )
            {
                LL tmp = (LL)prime[k] *arr[i];
                if( tmp > 5e5 ) break;

                if( pos[tmp] ) {
                    addEdge(i, pos[tmp]);
                    addEdge(pos[tmp], i);
                }
            }
        }

        printf("Case %d: %d\n", ++Case, N-maxMatch()/2);
    }
    return 0;
}

```