#### [SPOJ - REPEATS](https://www.spoj.com/problems/REPEATS/en/)

@[后缀数组, rmq]

&ensp; 题目大意: 求字符串中连续子串最多出现的次数

&ensp; 枚举子串长度 len 并将字符串按 len 划分, 枚举每个位置 i, 则以 i 起始的长度为 len 的子串一共会重复出现 cnt = (lcp(i, i+len) + len)/len 次, 根据分块的思想, 需要暴力出两端可能多出的部分

&ensp; 关于两端多出的部分,  合起来的长度不会超过 len, 否则会和上一次的位置 i 重复, 所以如果两端要多凑出一次, 需要利用右端多出的 lcp %len, 因为子串内从任何位置开始都具有周期性, 所以相当于将 [i, i+len*cnt - 1]右移到 [i + lcp %len, i + len + lcp -1], 剩下[i - (len - len %lcp), i + lcp %len-1], 即验证从 i - (len - len %lcp) 开始是否能构成周期字符串

&ensp; 关于求两个位置的lcp, 用rmq预处理height数组, 结合rank数组回答即可

```cpp
#include <iostream>
#include <cstdio>
#include <vector>
#include <queue>
#include <string>
#include <cstring>
#include <algorithm>
using namespace std;
const int MAXN = 5e4+10;

int sa[MAXN], Rank[MAXN], height[MAXN];

int temp_arr1[MAXN], temp_arr2[MAXN], cc[MAXN];
void getSA(vector<int>& s, int m=128)
{
    s.push_back(0);

    int *x = temp_arr1, *y = temp_arr2;
    int n = s.size();

    for(int i = 0; i < m; i++) cc[i] = 0;
    for(int i = 0; i < n; i++) cc[x[i] = s[i]]++;
    for(int i = 1; i < m; i++) cc[i] += cc[i - 1];
    for(int i = n-1; i >= 0; i--) sa[--cc[x[i]]] = i;

    for(int j = 1, p = 1; j < n && p < n && !(p=0); j <<=1, m = p)
    {
        for(int i = n-j; i < n; i++) y[p++] = i;

        for(int i = 0; i < n; i++)
            if(sa[i] >= j) y[p++] = sa[i] - j;

        for(int i = 0; i < m; i++) cc[i] = 0;
        for(int i = 0; i < n; i++) cc[x[y[i]]]++;
        for(int i = 1; i < m; i++) cc[i] += cc[i - 1];
        for(int i = n-1; i >= 0; i--) sa[--cc[x[y[i]]]] = y[i];

        swap(x, y); p = 1, x[sa[0]] = 0;
        for(int i = 1; i < n; i++)
        {
            x[sa[i]] = y[sa[i-1]] == y[sa[i]] &&
                    y[sa[i-1]+j] == y[sa[i]+j] ? p-1: p++;
        }
    }
    s.pop_back();
}

void getHeight(const vector<int>& s)
{
    int n = s.size();
    for(int i = 0; i <= n; i++ ) Rank[sa[i]] = i;
    for(int i = 0, k = 0; i < n; i++)
    {
        k? k-- :0;
        int j = sa[Rank[i]-1];
        while( i+k < n && j +k < n &&
            s[i+k] == s[j+k]) k ++;
        height[Rank[i]] = k;
    }
}

int N;

int dp[MAXN][20];
void initRMQ()
{
    for(int i = 1; i <= N; i++) 
        dp[i][0] = height[i];
    for(int j = 1; (1 << j) <= N; j++)
        for(int i = 1; i + (1 << j) - 1 <= N; i++)
            dp[i][j] = min(dp[i][j - 1], dp[i + (1 << (j - 1))][j - 1]);
}
 
int getLCP(int a, int b)
{
    int ra = Rank[a], rb = Rank[b];
    if(ra > rb) swap(ra, rb);
    int k = 0;
    while((1 << (k + 1)) <= rb - ra) k++;
    return min(dp[ra + 1][k], dp[rb - (1 << k) + 1][k]);
}

int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    int T;  cin >> T;
    while(T--)
    {
        cin >> N;
        vector<int> s;
        for( int i = 0; i < N; i++ )
        {
            char ch;  cin >> ch;
            s.push_back(ch);
        }

        getSA(s);
        getHeight(s);

        initRMQ();

        int ans = 0;
        for( int len = 1; len <= N; len++ )
        {
            for( int i = 0; i+len < N; i+=len)
            {
                int lcp = getLCP(i, i+len);
                int times = lcp /len +1;
                int pos = i - (len - lcp %len);
                if( pos >= 0 && lcp %len) 
                    if( getLCP(pos, pos+len) >= lcp) times ++;
                ans = max(ans, times);    
            }
        }
        cout << ans << endl;
    }
    return 0;
}
```

&nbsp;

***

#### [POJ 3693	Maximum repetition substring](http://poj.org/problem?id=3693)

&ensp; 题目大意: 求字符串中最多出现次数并且字典序最小的连续子串

&ensp; 上一题, 因为是要凑出一个周期len, 所以直接验证左端位置, 这里因为需要字典序最小, 有可能在同一出现次数下, 周期串可以左移 len - len %lcp 位, 所以对于每一位都要暴力, 结合rank数组来查看最小字典序

```cpp
#include <iostream>
#include <cstdio>
#include <vector>
#include <queue>
#include <string>
#include <cstring>
#include <algorithm>
using namespace std;
const int MAXN = 1e5+10;

int sa[MAXN], Rank[MAXN], height[MAXN];

int temp_arr1[MAXN], temp_arr2[MAXN], cc[MAXN];
void getSA(vector<int>& s, int m=128)
{
    s.push_back(0);

    int *x = temp_arr1, *y = temp_arr2;
    int n = s.size();

    for(int i = 0; i < m; i++) cc[i] = 0;
    for(int i = 0; i < n; i++) cc[x[i] = s[i]]++;
    for(int i = 1; i < m; i++) cc[i] += cc[i - 1];
    for(int i = n-1; i >= 0; i--) sa[--cc[x[i]]] = i;

    for(int j = 1, p = 1; j < n && p < n && !(p=0); j <<=1, m = p)
    {
        for(int i = n-j; i < n; i++) y[p++] = i;

        for(int i = 0; i < n; i++)
            if(sa[i] >= j) y[p++] = sa[i] - j;

        for(int i = 0; i < m; i++) cc[i] = 0;
        for(int i = 0; i < n; i++) cc[x[y[i]]]++;
        for(int i = 1; i < m; i++) cc[i] += cc[i - 1];
        for(int i = n-1; i >= 0; i--) sa[--cc[x[y[i]]]] = y[i];

        swap(x, y); p = 1, x[sa[0]] = 0;
        for(int i = 1; i < n; i++)
        {
            x[sa[i]] = y[sa[i-1]] == y[sa[i]] &&
                    y[sa[i-1]+j] == y[sa[i]+j] ? p-1: p++;
        }
    }
    s.pop_back();
}

void getHeight(const vector<int>& s)
{
    int n = s.size();
    for(int i = 0; i <= n; i++ ) Rank[sa[i]] = i;
    for(int i = 0, k = 0; i < n; i++)
    {
        k? k-- :0;
        int j = sa[Rank[i]-1];
        while( i+k < n && j +k < n &&
            s[i+k] == s[j+k]) k ++;
        height[Rank[i]] = k;
    }
}

string str;

int dp[MAXN][20];
void initRMQ()
{
    int N = str.size();
    for(int i = 1; i <= N; i++)
        dp[i][0] = height[i];
    for(int j = 1; (1 << j) <= N; j++)
        for(int i = 1; i + (1<<j) -1 <= N; i++)
            dp[i][j] = min(dp[i][j-1], dp[i + (1 <<(j-1))][j-1]);
}

int getLCP(int a, int b)
{
    int ra = Rank[a], rb = Rank[b];
    if(ra > rb) swap(ra, rb);
    int k = 0;
    while((1 << (k + 1)) <= rb - ra) k++;
    return min(dp[ra + 1][k], dp[rb - (1 << k) + 1][k]);
}


int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    int Case = 0;
    while(cin >> str && str[0] != '#')
    {
        vector<int> s(str.begin(), str.end());

        getSA(s);
        getHeight(s);

        initRMQ();

        int ans = 0, l = 0, r = 0;
        for( int len = 1; len <= s.size(); len ++)
        {
            for( int i = 0; i+len < s.size(); i +=len)
            {
                int pin = i + len + getLCP(i, i+len) -1;

                for( int k = 0; k < len; k++ )
                {
                    int j = i - k;
                    if( j < 0 || s[j] != s[j+len]) break;
                    int cnt = (pin - j +1) /len;

                    if( cnt > ans || (cnt == ans && Rank[j]<Rank[l]))
                        ans = cnt, l = j, r = j+cnt*len-1;
                }
            }
        }

        cout << "Case " << ++Case << ": ";
        if( ans) for( int i = l; i <= r; i++ )
            cout << str[i];
        else cout << str[sa[1]];

        cout << endl;
    }
    return 0;
}

```

