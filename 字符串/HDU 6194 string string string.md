#### [HDU 6194 string string string](http://acm.hdu.edu.cn/showproblem.php?pid=6194)

@[后缀数组, RMQ, 容斥, 单调队列, 单调栈]

&ensp; 题目大意: 求重复正好k次的不同子串的数量

&ensp; 方法一: 求正好k次即至少k次 - 至少k+1次的数量, 因为重复子串对应的后缀排名一定是连续的, 如果按长度k分组, 则同一组内至少出现k次的前缀子串数即组内的公共LCP长度

&ensp; 枚举下标i, 计算从排名i开始的k个后缀出现至少k次前缀子串数为LCP(i, i+k-1), 减去和排名i之前所有相关的子串数LCP(i-1, i+k-1), 减去至少k+1次前缀子串数LCP(i, i+k), 因为LCP(i-1, i+k)个子串是多减的容斥加上即可

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
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

string str;  int K;

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
    if( a == b ) return str.size() - a;

    int ra = Rank[a], rb = Rank[b];
    if(ra > rb) swap(ra, rb);
    int k = 31 - __builtin_clz(rb-ra);
    return min(dp[ra+1][k], dp[rb-(1<<k)+1][k]);
}


int main()
{
	ios::sync_with_stdio(0);
	cin.tie(0);

	int T;  cin >> T;
	while(T--)
	{
		cin >> K >> str;

		vector<int> s(str.begin(), str.end());
		getSA(s), getHeight(s);
		initRMQ();

		LL ans = 0;

		for( int i = 1; i + K -1 <= s.size(); i++ )
		{
			ans += getLCP(sa[i], sa[i+K-1]);
			if( i -1 > 0 ) ans -= getLCP(sa[i-1], sa[i+K-1]);
			if( i + K <= s.size()) ans -= getLCP(sa[i], sa[i+K]);
			if( i -1 > 0 && i + K <= s.size() )
				ans += getLCP(sa[i-1], sa[i+K]);
		}
		cout << ans << endl;
	}
	return 0;
}
```

&nbsp;

&ensp; 方法二: 这题也可以单调队列维护窗口大小为k-1的height数组的滑动窗口最小值来计算至少为k次的子串数, 当滑动一格最小值变大时说明有新的前缀加入, 否则不变

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
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

string str;  int K;

LL solve(int k)
{
    LL res = 0;
    if( !k) for( int i = 1; i <= str.size(); i++ )
        res += str.size() - sa[i] - height[i];
    if( !k) return res;

    deque<int> que;
    for( int i = 1; i <= k; i++ )
    {
        while(!que.empty() && height[i] <= height[que.back()])
            que.pop_back();
        que.push_back(i);
    }

    for( int i = k+1; i <= str.size(); i++ )
    {
        int prev = height[que.front()];
        while(!que.empty() && height[i] <= height[que.back()])
            que.pop_back();
        que.push_back(i);
        if( que.front() <= i-k) que.pop_front();

        int curv = height[que.front()];
        res += max(curv - prev, 0);
    }
    return res;
}

int main()
{
	ios::sync_with_stdio(0);
	cin.tie(0);

    int T;  cin >> T;
    while(T--)
    {
        cin >> K >> str;
        vector<int> s(str.begin(), str.end());
        getSA(s), getHeight(s);

        cout << solve(K-1) - solve(K) << endl;
    }
	return 0;
}

```

&nbsp;

&ensp; 方法三: 同样求至少为k的子串数, 使用单调栈来维护每一个以height[i]为最小值的最大区间, 若区间长度不少于k-1, 则当前height对答案的贡献为height[i] - max(height[L-1], height[R+1]), 因为这两端的height必然比当前height小, 且会算入一个重复的LCP。主要是计算一个区间的增量

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
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

string str;  int K;

LL solve(int k)
{
    LL res = 0;
    if( k == 1) for( int i = 1; i <= str.size(); i++ )
        res += str.size() - sa[i] - height[i];
    if( k == 1) return res;

    stack<int> stk;
    height[str.size()+1] = 0;
    for( int i = 1; i <= str.size()+1; i++ )
    {
        while(!stk.empty() && height[i] <= height[stk.top()])
        {
            int tp = stk.top();  stk.pop();
            int L = stk.empty()? 1: stk.top()+1, R = i-1;

            if( R-L+1 +1 >= k)
            {
                res += height[tp] - max(
                    height[L-1], height[R+1]
                );
            }
        }
        stk.push(i);
    }

    return res;
}

int main()
{
	ios::sync_with_stdio(0);
	cin.tie(0);

    int T;  cin >> T;
    while(T--)
    {
        cin >> K >> str;
        vector<int> s(str.begin(), str.end());
        getSA(s), getHeight(s);

        cout << solve(K) - solve(K+1) << endl;
    }
	return 0;
}

```

&nbsp;
***
#### [ACM-ICPC 2018 焦作赛区网络预赛 H. String and Times](https://nanti.jisuanke.com/t/31717)

&ensp; 题目大意: 统计出现次数在A到B之间的不同子串数

```
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
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
            dp[i][j] = min(dp[i][j-1], dp[i+(1<<(j-1))][j-1]);
}

int getLCP(int a, int b)
{
	if( a == b ) return str.size()-a;

    int ra = Rank[a], rb = Rank[b];
    if(ra > rb) swap(ra, rb);
    int k = 31 - __builtin_clz(rb-ra);
    return min(dp[ra+1][k], dp[rb-(1<<k)+1][k]);
}


int main()
{
	ios::sync_with_stdio(0);
	cin.tie(0);

	while(cin >> str)
	{
		int a, b;  cin >> a >> b;

		vector<int> s(str.begin(), str.end());
		getSA(s), getHeight(s);
		initRMQ();

		LL ans = 0;
		for( int i = 1; i+a-1 <= s.size(); i++ )
		{
			ans += getLCP(sa[i], sa[i+a-1]);
			if( i-1 >= 0) ans -= getLCP(sa[i-1], sa[i+a-1]);
			if( i+b <= s.size()) ans -= getLCP(sa[i], sa[i+b]);
			if( i-1 >= 0 && i+b <= s.size())
				ans += getLCP(sa[i-1], sa[i+b]);
		}
		cout << ans << endl;
	}
	return 0;
}

```