#### [codeforce 55D Beautiful numbers](http://codeforces.com/problemset/problem/55/D)

@[dp, 数位dp, lcm, 离散化]

&ensp; 题目大意: 统计一个区间上的数每个数位上除了0以外都能整除自己的数

&ensp; 数位dp记录每次的lcm, 因为1到9的lcm是2520, 所以每次x模2520可以记录在dp数组里, 因为数的范围是long long, 所以dp大小就是20*2520*2520, 会爆内存。1到9的所有lcm其实并不多，所以可以离散化来节约空间

```cpp
#include <bits/stdc++.h>
typedef long long LL;
using namespace std;
const int MOD = 2520;

LL getGCD(LL a, LL b)
{ return !b? a: getGCD(b, a %b); }
LL getLCM(LL a, LL b)
{ return a/getGCD(a, b)*b; }

vector<int> bak;
int getid(int x)
{
    return lower_bound(
        bak.begin(), bak.end(),
      x ) - bak.begin();
}

vector<int> digit;

LL dp[20][100][2600];
LL dfs(int pos, bool limit, int lcm, int x)
{
    if( pos < 0 ) return x %lcm == 0;

    LL &alias = dp[pos][getid(lcm)][x];
    if( !limit && ~alias) return alias;

    int cl = limit? digit[pos] :9;
    LL ret = 0;
    for( int i = 0; i <= cl; i++ )
    {
        int __lcm = !i? lcm :getLCM(lcm, i);
        ret += dfs(pos-1, limit && i==cl,
                __lcm, (x*10 + i) %MOD);
    }
    return limit? ret: alias = ret;
}

LL solve(LL x)
{
    digit.clear();
    do{
        digit.push_back(x %10);
    }while(x /= 10);

    return dfs(digit.size()-1, 1, 1, 0);
}

int main()
{
    memset(dp, -1, sizeof dp);
    for( int s = 0; s < (1<<10); s++ )
    {
        int lcm = 1;
        for( int i = 1; i < 10; i++ )
            if( s &(1<<i)) lcm = getLCM(lcm, i);
        bak.push_back(lcm);
    }
    sort(bak.begin(), bak.end());
    bak.erase(unique(bak.begin(), bak.end()), bak.end());

    int T;  cin >> T;
    while(T--)
    {
        LL a, b;  cin >> a >> b;
        cout << solve(b)-solve(a-1) << endl;
    }
    return 0;
}
```