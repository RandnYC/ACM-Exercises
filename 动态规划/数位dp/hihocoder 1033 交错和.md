#### [hihocoder 1033 交错和](http://hihocoder.com/problemset/problem/1033)

@[dp, 数位dp]

&ensp; 定义数位上的交错函数 $f(x) = a_0-a_1+a_2+(-1)^{n-1}\cdot a_{n-1}$ 求给定区间上交错数位和等于K的数之和

&ensp; 对于每个数位上的贡献为该位的大小乘以该位出现的次数以及该位在那个十进制位上,  所以结果要记录次数来辅助计算和

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
const int MOD = 1e9+7;

LL K;

LL _pow10[20];
vector<int> digit;
struct Node { LL cnt, sum; };

Node dp[20][400][2][2];
Node dfs(int pos, bool limit, bool pre0, bool sgn, int sum)
{
    if( pos < 0 ) return Node({sum == K, 0});

    Node &alias = dp[pos][sum+200][pre0][sgn];

    if( !limit && ~alias.cnt) return alias;

    Node ret({0, 0});
    int cl = limit? digit[pos] :9;
    for( int i = 0; i <= cl; i++ )
    {
        bool zero = pre0 && (!i);
        bool __sgn = zero? 0 :!sgn;
        int __sum = __sgn? sum+i :sum-i;

        Node tmp = dfs(pos-1, limit && i == cl, zero,
                        __sgn, __sum) ;

        (ret.cnt += tmp.cnt) %= MOD;
        ret.sum = (((ret.sum + tmp.cnt *i %MOD*
                    _pow10[pos] %MOD) %MOD) %MOD + tmp.sum) %MOD;
    }
    return limit? ret: alias = ret;
}

LL solve(LL x)
{
    digit.clear();
    do{
        digit.push_back(x %10);
    }while(x /= 10);

    return dfs(digit.size()-1, 1, 1, 0, 0).sum;
}

void preDeal()
{
    _pow10[0] = 1;
    for( int i = 1; i < 20; i++ )
        _pow10[i] = _pow10[i-1] *10 %MOD;
}

int main()
{
    preDeal();
    memset(dp, -1, sizeof dp);

    LL a, b;
    cin >> a >> b >> K;

    cout << (solve(b)-solve(a-1)+MOD) %MOD << endl;
    return 0;
}
```

