#### [zoj 3057 Beans Game](http://acm.zju.edu.cn/onlinejudge/showProblem.do?problemId=2711)

@[博弈, 递推]

&ensp; 题目大意: 给出三堆石头, 每轮可以选择一堆石头取走或选两堆石头取走相同数目

&ensp; 初始状态为三堆全空先手败, 递推即可。由于必败态按操作递推的所有后继状态必是必胜态；相反必胜态按此规则递推却不一定是必败态

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 3e2 +10;

bool dp[MAXN][MAXN][MAXN];

void preDeal()
{
    dp[0][0][0] = 0;
    for( int i = 0 ; i < MAXN ; i++)
    {
        for( int j = 0 ; j < MAXN ; j++ )
        {
            for( int k = 0 ; k < MAXN ; k++ )
            {
                if( !dp[i][j][k] )
                {
                    for( int x = i+1 ; x < MAXN ; x++)
                        dp[x][j][k] = !dp[i][j][k];
                    for( int x = j+1 ; x < MAXN ; x++)
                        dp[i][x][k] = !dp[i][j][k];
                    for( int x = k+1 ; x < MAXN ; x++)
                        dp[i][j][x] = !dp[i][j][k];
                    
                    for( int x = 1; max(i, j)+x < MAXN ; x++)
                        dp[i+x][j+x][k] = !dp[i][j][k];
                    for( int x = 1; max(j, k)+x < MAXN ; x++)
                        dp[i][j+x][k+x] = !dp[i][j][k];
                    for( int x = 1; max(i, k)+x < MAXN ; x++)
                        dp[i+x][j][k+x] = !dp[i][j][k];
                }
            }
        }
    }
}


int N ,M, P;

int main()
{
    preDeal();
    while(~scanf("%d%d%d", &N, &M, &P))
    {
        if( dp[N][M][P] ) puts("1");
        else puts("0");
    }
    return 0;
}
```