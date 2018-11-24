#### [HDU 6415 Rikka with Nash Equilibrium](http://acm.hdu.edu.cn/showproblem.php?pid=6415)

@[dp]

&ensp; 题目大意: 给出矩阵的大小N, M。从1到N*M向其中填数，满足最多只有一个元素比它所在的行和列的元素都大

&ensp; 若把元素从大到小填充到矩阵中，则后来的元素一定填充在已有元素所在的行或列中

&ensp; 设dp[i][j][k]为已经填充了i行, j列, 和k个数，则转移方程：
$$
dp[i][j][k] = \begin{cases} 
dp[i-1][j][k-1] \cdot (n-i+1) \cdot j  & \\
dp[i][j-1][k-1] \cdot (m-j+1) \cdot i & \\
dp[i][j][k-1] \cdot (n\cdot m - k+1) & (n \cdot m >= k-1) 
\end{cases}
$$


```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;

int N, M, mod;
LL dp[100][100][6410];

int main()
{
    int T;  cin >> T;
    while(T--)
    {
        cin >> N >> M >> mod;

        dp[1][1][1] = N*M;
        for( int i = 1; i <= N; i++ )
        {
            for( int j = 1; j <= M; j ++)
            {
                if( i == 1 && j == 1 ) continue;

                for( int k = 1; k <= i*j ; k++ )
                {
                    dp[i][j][k] = dp[i-1][j][k-1] *(N-i+1)*j %mod;
                    (dp[i][j][k] += dp[i][j-1][k-1] *(M-j+1)*i %mod) %=mod;

                    if( i*j-k+1 < 0) break;
                    (dp[i][j][k] += dp[i][j][k-1] *(i*j-k+1) %mod) %=mod;
                }
            }
        }

        cout << dp[N][M][N*M] << endl;
    }
    return 0;
}

```