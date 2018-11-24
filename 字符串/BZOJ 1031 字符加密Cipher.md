#### [BZOJ 1031 字符加密Cipher](https://www.lydsy.com/JudgeOnline/problem.php?id=1031)

@[后缀数组]

&ensp; 题目大意: 给出一个字符串首位相连, 每个位置转一周形成一个新的字符串并排序, 在每个排序后的字符串取最后一个字符, 求这些字符组成的新字符串

&ensp; 将字符串延伸两倍, 用后缀数组sa可求出每个排名的字符串后缀, 对应排名的字符串的最后一个字符就是s[sa[i]+len-1] 


```cpp
#include <bits/stdc++.h>
using namespace std;
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


string str;

int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    cin >> str;

    int len = str.size();
    for( int i = 0; i < len; i++ )
        str.push_back(str[i]);

    vector<int> s(str.begin(), str.end());

    getSA(s);

    for( int i = 1; i <= 2*len; i++ )
        if( sa[i] < len) cout << str[sa[i]+len-1];
    cout << endl;

    return 0;
}
```