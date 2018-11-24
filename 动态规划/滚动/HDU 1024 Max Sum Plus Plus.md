####  [HDU 1024 Max Sum Plus Plus](http://acm.hdu.edu.cn/showproblem.php?pid=1024)

@[dp, 滚动, 经典]

&ensp; 题目大意: 求M个子段之和的最大值

&ensp; dp[i][j] 表示以i为结尾, 选取了j段的最大和

转移方程: 
$$
dp[i][j] = max\left\{\begin{matrix}
dp[i-1][j] + a[i] & (i-1\ge j) \\
max\{ dp[k][j-1] \} + a[i] & (k \in [1, i-1] \bigcap k >= j-1)
\end{matrix}\right. 
$$


```cpp
#include <bits/stdc++.h>
using namespace std;
const int INF = 1 << 30;
const int MAXN = 1e6+10;

int N, M;
int arr[MAXN];
int dp[MAXN], last[MAXN];

int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    while(cin >> M >> N)
    {
        memset(dp, 0, sizeof dp);
        memset(last, 0, sizeof last);

        for( int i = 1; i <= N; i++ )
            cin >> arr[i];

        int tmp = 0;
        for( int j = 1; j <= M; j++ )
        {
            tmp = -INF;
            for( int i = j; i <= N; i++ )
            {
                dp[i] = max(dp[i-1], last[i-1]) + arr[i];
                last[i-1] = tmp;
                tmp = max(dp[i], tmp);
            }
        }
        cout << tmp << endl;
    }
    return 0;
}
```
