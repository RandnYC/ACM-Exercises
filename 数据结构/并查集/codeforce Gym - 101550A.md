#### [codeforce Gym - 101550A](https://vjudge.net/contest/237465#problem/A)

@[并查集]

&ensp;  题目大意: 给出N*M白格子, 每次操作涂黑一定长度的的方块, 问每次操作有多少联通白块

&ensp; 读入所有操作, 从后往前反演, 第一次需要dfs计算出联通块个数, 每消去一条黑块, 运用并查集对每块查询四个方向是否和自己联通。

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 1e3+10;
const int ways[][2] = {
    {1, 0}, {-1, 0},
    {0, 1}, {0, -1}
};

int N, M, Q;

int num[MAXN][MAXN];
bool visited[MAXN][MAXN];

int getid(int x, int y)
{ return x*M + y; }

struct Query
{ int x1, x2, y1, y2; };
vector<Query> qy;

int pre[MAXN*MAXN];
void init()
{
    memset( num , 0 , sizeof num );
    memset( pre , 0 , sizeof pre);
    memset(visited, 0 , sizeof visited );
    qy.clear();
    for( int i = 0; i <= N*M; i++ )
        pre[i] = i;
}

int find(int x)
{ return pre[x] = x==pre[x]? x: find(pre[x]); }

bool merge(int x, int y)
{
    x = find(x), y = find(y);
    if( x == y ) return 0;

    pre[y] = x;
    return 1;
}

bool inRange(int x, int y)
{
    if( x < 0 || x >= N
        || y < 0 || y >= M ) return 0;
    return 1;
}

void dfs(int x, int y)
{
    for(int k = 0 ; k < 4 ; k++ )
    {
        int xx = x+ways[k][0];
        int yy = y+ways[k][1];
        if( inRange(xx, yy)
            && !visited[xx][yy] && !num[xx][yy])
        {
            merge(getid(x, y), getid(xx, yy));
            visited[xx][yy] = 1;
            dfs(xx, yy);
        }
    }
}

int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    while( cin >> N >> M >> Q )
    {
        init();

        for( int q = 0 ; q < Q ; q++ )
        {
            register int x1, x2, y1, y2;
            cin >> x1 >> y1 >> x2 >> y2;
            qy.push_back({--x1, --x2, --y1, --y2});

            for( int x = x1; x <= x2 ; x++)
            {
                for( int y = y1; y <= y2; y++)
                    num[x][y]++;
            }
        }

        vector<int> ans; int tmp = 0;
        for( int i = 0 ; i < N ; i++ )
        {
            for( int j = 0 ; j < M ; j++ )
            {
                if( !visited[i][j] && !num[i][j])
                    dfs(i, j), tmp++;
            }
        }
        ans.push_back(tmp);

        for( int q = Q-1 ; q >= 1 ; q-- )
        {
            const int &x1 = qy[q].x1, &x2 = qy[q].x2,
                    &y1 = qy[q].y1, &y2 = qy[q].y2;

            for( int x = x1 ; x <= x2 ; x++ )
            {
                for( int y = y1; y <= y2 ; y++)
                {
                    if( !--num[x][y] )
                    {
                        tmp ++;
                        for( int k = 0 ; k < 4 ; k++ )
                        {
                            register int xx = x+ways[k][0],
                                        yy = y+ways[k][1];
                            if( !inRange(xx, yy) ||
                                num[xx][yy] ) continue;

                            if( merge(getid(x, y),
                                    getid(xx, yy)) ) tmp --;
                        }
                    }
                }
            }
            ans.push_back(tmp);
        }

        for( int q = Q-1; q >= 0; q-- )
            cout << ans[q] << endl;
    }
    return 0;
}

```