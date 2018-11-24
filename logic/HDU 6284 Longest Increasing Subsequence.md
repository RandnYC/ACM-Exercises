#### [HDU 6284 Longest Increasing Subsequence](https://vjudge.net/problem/HDU-6284)

@[最长上升子序列]

&ensp; 题目大意: 给出一个长度为N的序列, 记 f(i) 为将序列中所有0替换为i后的最长上升子序列, 求 $\sum_{i=1}^{N} i \cdot f(i)$ 

&ensp; 记f[i]为以i结尾的LIS, g[i]为以i开始的LIS, 替换后的结果只有LIS和LIS+1两种, 如果在某条LIS的路径上的两点下标范围内(i, j)含有0, 则对应(a[i], a[j])的最长上升子序列长度都为LIS+1, 利用扫描线思想记录dt[a[i]+1]++, dt[a[j]]++, 最后再记录一遍


```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
const int INF = 1 << 30;
const int MAXN = 1e5+10;

int N, arr[MAXN];
int lis, f[MAXN], g[MAXN];

void prepare()
{
    vector<int> v;  v.push_back(-1);
    for( int i = 0; i <= N+1; i++ )
    {
        if( !~arr[i]) continue;
        if( arr[i] > v.back())
        {
            v.push_back(arr[i]);
            f[i] = v.size()-1;
        }
        else
        {
            int p = lower_bound(
                v.begin(), v.end(),
              arr[i]) - v.begin();
            v[p] = arr[i], f[i] = p;
        }
    }

    lis = v.size()-1;
    v.clear(), v.push_back(INF);

    for( int i = N+1; i >= 0; i-- )
    {
        if( !~arr[i]) continue;
        if( arr[i] < v.back())
        {
            v.push_back(arr[i]);
            g[i] = v.size()-1;
        }
        else
        {
            int p = lower_bound(
                v.begin(), v.end(),
                arr[i], greater<int>()
            ) - v.begin();
            v[p] = arr[i], g[i] = p;
        }
    }
}

vector<int> point0;
bool containZero(int x, int y)  // (x, y)
{
    vector<int>::iterator it;
    it = upper_bound(point0.begin(), point0.end(), x);
    return it != point0.end() && *it < y;
}

struct Comp {
    bool operator()(int x, int y)
    { return arr[x] < arr[y]; }
};
set<int, Comp> cindex[MAXN];
int dt[MAXN];    //cal the union if lis+1
void clear()
{
    memset(dt, 0, sizeof dt);
    memset(f, 0, sizeof f);
    memset(g, 0, sizeof g);
    for( int i = 0; i <= N; i++ )
        cindex[i].clear();
    point0.clear();  lis = 0;
}

int main()
{
    while(~scanf("%d", &N))
    {
        clear();

        for( int i = 1; i <= N; i++ )
        {
            scanf("%d", arr+i);
            if( !arr[i]) point0.push_back(i);
            if( !arr[i]) arr[i] = -1;
        }
        arr[N+1] = N+1;

        prepare();

        for( int i = 0; i <= N+1; i++ )
        {
            int &x = arr[i];
            if( !~x) continue;

            if( f[i] + g[i] -1 == lis)
            {
                for( int pos :cindex[f[i]-1])
                {
                    if( arr[pos] >= arr[i]) continue;
                    if( containZero(pos, i))
                    {
                        dt[arr[i]] --;
                        dt[arr[pos]+1] ++;
                        break;
                    }
                }
                cindex[f[i]].insert(i);
            }
        }

        lis -= 2;
        LL ans = 0, sum = 0;
        for( int i = 0; i <= N+1; i++ )
        {
            sum += dt[i];
            if( !i || i == N+1) continue;

            if(sum) ans += (LL)i *(lis+1);
            else ans += (LL)i *lis;
        }

        printf("%lld\n", ans);
    }
    return 0;
}

```