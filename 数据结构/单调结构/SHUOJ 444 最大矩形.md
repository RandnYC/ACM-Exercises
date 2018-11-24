#### [SHUOJ 444 最大矩形](http://acmoj.shu.edu.cn/problem/444/)

@[单调栈]

&ensp; 题目大意: 求01矩阵的最大1矩阵面积

&ensp; 单调栈, 维护直方图递增序列下标

```cpp
#include <bits/stdc++.h>
using namespace std;

int N, M;
bool Map[207][207];

int main()
{
    while(cin >> N >> M)
    {
        for( int i = 1; i <= N; i++ ) {
            for( int j = 1; j <= M; j++ ) {
                char ch;  cin >> ch;
                Map[i][j] = ch - '0';
            }
        }

        static int h[207];
        memset(h, 0, sizeof h);

        int ans = 0;  h[0] = -1;
        for( int i = 1; i <= N; i++ )
        {
            for( int j = 1; j <= M; j++ )
                h[j] = Map[i][j]? h[j]+1: 0;

            stack<int> stk;
            for( int j = 0; j <= M+1; j++ )
            {
                while(!stk.empty() && h[j] <= h[stk.top()])
                {
                    int tp = stk.top();  stk.pop();
                    int L = stk.top()+1, R = j - 1;

                    ans = max(ans, (R-L+1) *h[tp]);
                }
                stk.push(j);
            }
        }
        cout << ans << endl;
    }
    return 0;
}
```