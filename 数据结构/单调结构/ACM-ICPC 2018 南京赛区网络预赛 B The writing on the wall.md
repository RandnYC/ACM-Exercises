#### [ACM-ICPC 2018 南京赛区网络预赛 B The writing on the wall](https://nanti.jisuanke.com/t/30991)

&ensp; 题目大意: 给出一个01矩阵, 求其中有多少个子矩阵

&ensp; 最多有1e5行和100列。把矩阵按行压缩为直方图，求矩形个数等于求每行的直方图每列 j，以 j为矩形右端的矩形个数。暴力复杂度 $O(N \cdot M^2)$, 勉强可以卡过

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 1e5+10;

int N, M, K;
bool arr[MAXN][110];
int up[110];

int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    int T;  cin >> T;
    int Case = 0;
    while(T--)
    {
        memset(arr, 0, sizeof arr);
        memset(up, 0, sizeof up);

        cin >> N >> M >> K;
        while(K--)
        {
            int x, y;  cin >> x >> y;
            arr[x][y] = 1;
        }

        long long ans = 0;
        for(int i = 1; i <= N; i++ )
        {
            for(int j = 1; j <= M; j++ )
                up[j] = arr[i][j]? i: up[j];

            for(int j = 1; j <= M; j++ )
            {
                int tmp = 1 << 30;
                for(int k = j; k >= 1; k-- )
                {
                    tmp = min(tmp, i-up[k]);
                    ans += tmp;
                }
            }
        }
        printf("Case #%d: %lld\n", ++Case, ans);
    }
    return 0;
}
```


&nbsp; 
&ensp; 可以看出, 对于直方图的每列 j, 和从右往左扫的最小值有关, 可用单调栈优化

&ensp; 维护一个递增单调栈, 同时维护单调栈对应的面积, 复杂度$O(N \cdot M)$
```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 1e5+10;

int N, M, K;

int arr[MAXN][110];
int height[110];

int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    int T;  cin >> T;
    int Case = 0;
    while(T--)
    {
        memset(arr, 1, sizeof arr);
        memset(height, 0, sizeof height);

        cin >> N >> M >> K;
        while(K--)
        {
            int x, y;  cin >> x >> y;
            arr[x][y] = 0;
        }

        long long ans = 0;
        for( int i = 1; i <= N; i++ )
        {
            stack<int> stk;  long long area = 0;

            for(int j = 1; j <= M; j++ )
            {
                int &h = height[j];
                h = arr[i][j]? h+1 :0;

                while(!stk.empty() && h <= height[stk.top()])
                {
                    int tp = stk.top();  stk.pop();
                    int L = stk.empty()? 1: stk.top()+1, R = tp;
                    area -= (R-L+1) *height[tp];
                }

                int L = stk.empty()? 1: stk.top()+1, R = j;
                stk.push(j);  area += (R-L+1) *h;
                ans += area;
            }
        }

        printf("Case #%d: %lld\n", ++Case, ans);
    }
    return 0;
}
```