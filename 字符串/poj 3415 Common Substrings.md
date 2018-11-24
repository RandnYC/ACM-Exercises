#### [poj 3415 Common Substrings](http://poj.org/problem?id=3415)

@[后缀数组, 单调栈]

&ensp; 题目大意: 统计两个串中公共子串长度超过给定的K的公共子串的个数

&ensp; 拼接两个字符串求height数组, 对于A串的每个位置求B串中height[rank[i]] >= K的位置, 再对B串每个位置统计A串, 复杂度O(n^2)

&ensp; 使用单调栈来优化: 假设先统计A串, 对height >= K 分组, 栈中记录自底而上递增的height值, 栈同时记录 cnt 表示如果以当前height为最小值时当前位之前在这组中有多少个sa[i-1] 处于A串

&ensp; 假设先统计A串,  如果当前位的height比栈顶要小, 并且但前表示以当前height作为最小值, 那么需要将原来栈中大的height扣掉对应cnt个, 并加上这cnt个height

```cpp
#include <iostream>
#include <cstdio>
#include <vector>
#include <string>
#include <cstring>
#include <stack>
#include <algorithm>
using namespace std;
typedef long long LL;
const int MAXN = 2e5+10;

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

int K;
struct Point { int val, cnt; };
Point stk[MAXN];

int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    while(cin >> K && K)
    {
        string str1, str2; cin >> str1 >> str2;
        string str = str1 + "#" + str2;

        vector<int> s(str.begin(), str.end());
        getSA(s), getHeight(s);

        LL ans = 0, tmp = 0;
        int top = 0;
        for( int i = 1; i <= s.size(); i++ )
        {
            if( height[i] < K) top = tmp = 0;
            else
            {
                int cnt = 0;
                if( sa[i-1] < str1.size())
                    cnt = 1, tmp += height[i]-K+1;
                while(top > 0 && height[i] <= stk[top-1].val)
                {
                    top --;
                    tmp -= stk[top].cnt *(stk[top].val - height[i]);
                    cnt += stk[top].cnt;
                }
                stk[top++] = {height[i], cnt};
                if(sa[i] > str1.size()) ans += tmp;
            }
        }

        top = tmp = 0;

        for( int i = 1; i <= s.size(); i++ )
        {
            if( height[i] < K) top = tmp = 0;
            else
            {
                int cnt = 0;
                if( sa[i-1] > str1.size())
                    cnt = 1, tmp += height[i]-K+1;
                while(top > 0 && height[i] <= stk[top-1].val)
                {
                    top --;
                    tmp -= stk[top].cnt *(stk[top].val - height[i]);
                    cnt += stk[top].cnt;
                }
                stk[top++] = {height[i], cnt};
                if(sa[i] < str1.size()) ans += tmp;
            }
        }
        cout << ans << endl;
    }
    return 0;
}

```