####[cf 988F Rain and Umbrellas](https://codeforces.com/problemset/problem/988/F)

@[dp, 普通]

&ensp; 题目大意: x坐标轴上, 给出N片下雨的区域[L, R], M把伞, 要求不能淋湿, 带着伞消耗的体力为$伞重\times路程$, 求到终点时消耗的最少体力

$$
dp[i] =
	\left\{\begin{matrix} 
			dp[i-1] & (rain[i-1]!=1)\\
			min\{dp[j]+(i-j)\times umb[j]\} & (rain[i-1]==1 \bigcap 0\leq j < i)\\
	\end{matrix}\right.
$$

&nbsp;
***
```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 2e3+10;
const int INF = 1 << 30;

int tml, N , M;
bool rain[MAXN];
int umb[MAXN], dp[MAXN];

int main()
{
    while(~scanf("%d%d%d", &tml, &N ,&M))
    {
        memset(rain , 0 , sizeof rain);
        memset(umb, 0 , sizeof umb);
        for( int i = 0 ; i < N ; i ++)
        {
            int a, b;   scanf("%d%d", &a, &b);
            while(a < b) rain[a++] = 1;
        }
            
        for( int i = 0 ; i < M ; i++ )
        {
            int a, b;   scanf("%d%d", &a, &b);
            if( umb[a] ) umb[a] = min(umb[a], b);
            else umb[a] = b;
        }

        dp[0] = 0;
        for( int i = 1 ; i <= tml ; i++)
        {
            if(!rain[i-1]) dp[i] = dp[i-1];
            else
            {
                dp[i] = INF;
                for( int j = 0 ; j < i ; j++)
                {
                    if( umb[j])
                        dp[i] = min(dp[i], dp[j]+(i-j)*umb[j]);
                }
            }
        }
        printf("%d\n", dp[tml]==INF ? -1 : dp[tml]);
    }
    return 0;
}
```
