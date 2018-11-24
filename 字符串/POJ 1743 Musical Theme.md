#### [POJ 1743 Musical Theme](http://poj.org/problem?id=1743)

@[后缀数组, 二分]

&ensp; 题目大意:  给定一个序列, 求最长不相交的两个子串满足LCP长度大于5并且LCP可以被定义为两个子串每位对应相差一个常数

&ensp;  若是用后缀数组求最长不相交的公共子串, 则需要二分长度len, 验证时找到一段下标内最小的height >= len(即后缀LCP>=len) , 并且满足下标距离>=len

&ensp; 由于公共子串可以相差一个常数, 所以把原数列中arr[i+1] - arr[i] 来构造一个新的数列, 这个数列的长度为N-1, 然后用后缀数组跑一遍最长不交公共子串

```cpp
#include <iostream>
#include <cstdio>
#include <cstring>
#include <vector>
#include <string>
#include <cmath>
#include <algorithm>
using namespace std;
const int MAXN = 2e4+10;

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
vector<int> s;

bool check(int len)
{
    int mx = sa[1], mi = sa[1];
    for( int i = 2; i <= N; i++ )
    {
        if( mx - mi +1 >= len ) return 1;

        if( height[i] < len ) mx = mi = sa[i];
        else
        {
            mx = max(mx, sa[i]);
            mi = min(mi, sa[i]);
        }
    }
    return 0;
}

int main()
{
    s.reserve(MAXN);

    while(scanf("%d", &N), N)
    {
        s.clear();
        for( int i = 0; i < N; i++ )
        {
            int x;  scanf("%d", &x);
            s.push_back(x);
        }

        for( int i = 0; i < N-1; i++ )
            s[i] = s[i+1] - s[i] + 100;

        getSA(s, 200);
        getHeight(s);

        int L = 4, R = N-1;
        int ans = 0;
        while(L <= R)
        {
            int mid = L + (R-L)/2;
            if( check(mid))
            {
                ans = mid;
                L = mid+1;
            }
            else R = mid-1;
        }
        if( ans < 4) puts("0");
        else printf("%d\n", ans+1);
    }
    return 0;
}

```