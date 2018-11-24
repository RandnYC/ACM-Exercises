#### [2018 上海大都会赛J Beautiful Numbers](https://www.nowcoder.com/acm/contest/163/J)

@[dp, 数位dp]

&ensp; 题目大意: 求N以内, 数位和能整除自身的数的个数

&ensp; 数位dp, 由于N很大, 数组开不下N, 转而枚举模数再数位dp, dp[i][j][k]分别表示数位为i, 0到i的数位和是j, 模上某个枚举的数之后余k。

&ensp; 本来应该多开一维模数, 而空间不够, 所以枚举模数dp数组要重置

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
const int MAXN = 20;

LL N;
vector<int> digit;
int mod;

LL dp[MAXN][MAXN*10][MAXN*10];
LL dfs(int pos, bool limit, int sum, int x)
{
    if( pos < 0 ) return !x && sum == mod;
    if( sum > mod ) return 0;
    if( !limit && ~dp[pos][sum][x] )
        return dp[pos][sum][x];

    int cl = limit? digit[pos] :9;

    LL ret = 0;
    for( int i = 0; i <= cl; i++ )
    {
        ret += dfs(pos-1, limit && i == cl,
                sum+i, (x*10+i) %mod);
    }
    return !limit? dp[pos][sum][x] = ret: ret;
}

LL solve(LL x)
{
    digit.clear();
    while(x)
    {
        digit.push_back(x %10);
        x /= 10;
    }

    LL ret = 0;
    for( mod = 1; mod <= 9*12; mod++)
    {
        memset(dp, -1, sizeof dp);
        ret += dfs(digit.size()-1, 1, 0, 0);
    }
    return ret;
}

int main()
{
    int T; cin >> T;
    int Case = 0;

    while(T--)
    {
        cin >> N;
        printf("Case %d: %lld\n",
                ++Case, solve(N));
    }
    return 0;
}
```