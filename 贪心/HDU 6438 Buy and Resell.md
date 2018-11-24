#### [HDU 6438 Buy and Resell](http://acm.hdu.edu.cn/showproblem.php?pid=6438)

@[贪心，最小堆]

&ensp; 题目大意: 给定N天的股票价格, 每天可以买入或卖出或什么都不做, 每天只能买卖一支股票, 问最终最大收益和最少操作次数

&ensp; 用最小堆来维护当前序列的最小股票价格
- 如果当天的股票$x$ 比堆顶t 还要小, 则直接买入，贡献为 $-x$
- 否则，把当天股票价格入堆两次
 	- 一次表示直接买进当天股票，贡献为 $-x$
	- 另一次表示用堆中的最小价值的股票在当天卖出, 则收益为$+x-t$, 但可能不是最终解, 所以需要抵消则此操作。假设之后某天价格为$y$, 并在堆中访问到$x$, 这时本来因该卖出之前的$t$, 此时变成了$x$, 则收益为 $(-t + x) + ( -x + y) = -t + y$ 不变
	
&ensp; 由于同时需要求最少操作次数，所以把左右的交易数量 - 抵消的次数就是最终结果

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
const int MAXN = 1e5+10;

int N;
unordered_map<int, int> vis;

int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    int T;  cin >> T;
    while(T--)
    {
        vis.clear();

        cin >> N;
        LL ans = 0, cnt = 0;
        priority_queue<int, vector<int>, greater<int> > que;
        for( int i = 1; i <= N; i++ )
        {
            int x;  cin >> x;
            if( que.empty() || que.top() > x)
                que.push(x);
            else
            {
                int tp = que.top();  que.pop();
                ans += -tp + x;
                cnt ++,  vis[x] ++;

                que.push(x), que.push(x);
                if( vis[tp] > 0 )
                    cnt --, vis[tp] --;
            }
        }
        cout << ans << " " << cnt*2 << endl;
    }
    return 0;
}
```
