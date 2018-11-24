#### [codeforce Gym - 101490J](http://codeforces.com/gym/101490/attachments/download/5853/2016-benelux-algorithm-programming-contest-bapc-16-en.pdf)

@[二分答案, 匈牙利算法]

&ensp; 有N个学生和N个导师, 距离是曼哈顿距离, 需要学生导师一一匹配, 找出需要走最多路的学生最小的距离

&ensp; 二分最大距离, 跑匈牙利

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef pair<int, int> PII;
const int INF = 1 << 30;
const int MAXN = 1e2+10;

int N;
PII stu[MAXN], thr[MAXN];
int G[MAXN][MAXN];
int vis[MAXN], link[MAXN];

bool path(int u, const int &limit)
{
    for( int v = 1; v <= N; v++ )
    {
        if( G[u][v] <= limit && !vis[v] )
        {
            vis[v] = 1;
            if( link[v] == 0 || path(link[v], limit))
            {
                link[v] = u;
                return 1;
            }
        }
    }
    return 0;
}

bool check(const int& d)
{
    memset(link, 0, sizeof link);

    int ret = 0;
    for( int i = 1; i <= N; i++ )
    {
        memset(vis, 0, sizeof vis);
        if( path(i, d)) ret++;
    }
    return ret == N;
}

int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    cin >> N;
    for( int i = 1; i <= N; i++ )
        cin >> stu[i].first >> stu[i].second;
    for( int i = 1; i <= N; i++ )
        cin >> thr[i].first >> thr[i].second;

    int L = INF, R = 0;
    for( int i = 1; i <= N; i++ )
    {
        for( int j = 1; j <= N; j++ )
        {
            G[i][j] = abs(stu[i].first - thr[i].first)
                    + abs(stu[i].second - thr[i].second);

            L = min(G[i][j], L), R = max(G[i][j], R);
        }
    }

    int ans = R;
    while(L <= R)
    {
        int mid = L + (R-L)/2;
        if( check(mid) )
        {
            ans = mid;
            R = mid-1;
        }
        else L = mid+1;
    }

    cout << ans << endl;

    return 0;
}

```