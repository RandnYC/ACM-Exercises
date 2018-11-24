#### [codeforce 919D Substring](http://codeforces.com/contest/919/problem/D)

@[拓扑排序, dp]

&ensp; 题目大意: 给出一张有向图, 图的每个节点代表一种颜色, 问若沿着一条路径走完能得到的最多相同的颜色有多少

&ensp; 图中可能存在有向环, 需要拓扑排序判环, 并且当无环时因为拓扑排序会有序的访问到所有节点一次, 可以用dp数组记录记录当前拓扑序对应节点每个颜色的最大值

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef pair<int, int> PII;
const int MAXN = 3e5+10;

int N, M;
char str[MAXN];
vector<int> G[MAXN];
int degree[MAXN];

int dp[MAXN][30];

int main()
{
    scanf("%d%d", &N, &M);
    scanf("%s", str+1);

    while(M--)
    {
        int a, b;  scanf("%d%d", &a, &b);
        G[a].push_back(b);
        degree[b] ++;
    }

    queue<int> que;
    for( int i = 1; i <= N; i ++)
    {
        if( !degree[i] )
        {
            que.push(i);
            dp[i][str[i]-'a']++;
        }
    }

    int ans = 1;
    while(!que.empty())
    {
        int u = que.front();
        que.pop();

        for( int i = 0; i < G[u].size(); i++ )
        {
            int v = G[u][i];
            if( !--degree[v] ) que.push(v);

            for( int k = 0; k < 26; k++ )
            {
                dp[v][k] = max(dp[v][k],
                    dp[u][k] + (str[v]-'a' == k)
                );
                ans = max(ans, dp[v][k]);
            }
        }
    }

    for( int i = 1; i <= N; i++ )
        if( degree[i] ) { ans = 0; break; }

    cout << (!ans? -1: ans) << endl;

    return 0;
}
```