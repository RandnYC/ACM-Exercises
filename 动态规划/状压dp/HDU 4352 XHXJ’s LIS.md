#### [HDU 4352 XHXJ's LIS](http://acm.hdu.edu.cn/showproblem.php?pid=4352)

@[dp, 数位dp, 状压, LIS]

&ensp; 题目大意; 给出一个范围内的数和K, 问数位最长上升子序列为K的数有多少个

&ensp; 借鉴LIS维护一个单调数组的思想, 最后数组的大小就是LIS的长度。因为K比较小, 可以将这个数组里的数状态压缩, 然后跑数位dp

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;

int K;
vector<int> digit;

int getState(int x, int s)
{
    for( int i = x; i < 10; i++ )
        if( s &(1 <<i)) return (s^(1<<i))|(1<<x);

    return s | (1<<x);
}

int getNum(int s)
{
    int cnt = 0;
    while(s)
    {
        if( s &1) cnt++;
        s /= 2;
    }
    return cnt;
}


LL dp[70][1<<10][12][2];
LL dfs(int pos, bool limit, int s, int pre0)
{
    if( pos < 0 ) return getNum(s) == K;
    if( !limit && ~dp[pos][s][K][pre0])
        return dp[pos][s][K][pre0];

    int cl = limit? digit[pos]: 9;
    LL ret = 0;
    for( int i = 0; i <= cl; i++ )
    {
        bool zero = pre0 && !i;

        ret += dfs(pos-1, limit && i == cl,
                zero? 0: getState(i, s), zero);
    }
    return !limit? dp[pos][s][K][pre0]=ret :ret;
}

LL solve(LL x)
{
    digit.clear();
    do{
        digit.push_back(x %10);
    }while(x /= 10);
    return dfs(digit.size()-1, 1, 0, 1);
}

int main()
{
    memset(dp, -1, sizeof dp);

    int T;  cin >> T;
    int Case = 0;
    while(T--)
    {
        LL a, b;  cin >> a >> b >> K;
        printf("Case #%d: ", ++Case);
        cout << solve(b) - solve(a-1) << endl;
    }
    return 0;
}
```