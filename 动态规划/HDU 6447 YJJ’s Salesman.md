#### [HDU 6447 YJJ's Salesman](http://acm.hdu.edu.cn/showproblem.php?pid=6447)

@[dp, 离散化, 树状数组]

&ensp; 题目大意: 在一个二维平面内给出N个点, 每个点有个权值, 只能往右或上或右上走, 只有走右上的时候才能得到这个点的权值, 问最多能收获多少

&ensp; 因为坐标分布的范围xy都是1e5, 所以需要离散化处理。很容易想到是二维dp, 表示到达该坐标最多能获得多少。

转移方程:
$$
dp[i][j] = max\{dp[i-1][j], dp[i][j-1], max\{dp[x][y]\} (\ x \in [1, i), y \in [1, j) \ ) \}
$$

&ensp; 由于二维dp需要二重循环内再多一重循环来寻找左下方的最大值, 需要进行优化：根据01背包的思想, 可以将dp的一维 i 滚动掉, 并且从下而上, 从右到左来维护每行对应的信息。

&ensp; 因为同一行或同一列的点到当前这个点并没有贡献，并且等于从某个左下方的点直接斜着过来，所以转移方程对于每行 i 优化：

$$
dp[j] = max\{ dp[k]\} , (\ k \in [1, j) \ )
$$

&ensp; 对于寻找最大值可以使用线段树或树状数组来维护, 把内层循环降至 log

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef pair<int, int> PII;
const int MAXN = 1e5+10;

int N;
int dp[MAXN];

vector<int> ay, ax;
vector<PII> ayx[MAXN];
int getidy(int y)
{
    return lower_bound(
        ay.begin(), ay.end(),
      y ) - ay.begin();
}
int getidx(int x)
{
    return lower_bound(
        ax.begin(), ax.end(),
      x ) - ax.begin();
}

int bitc[MAXN];
inline int lowbit(int x) { return x & -x; }
void update(int pos, int val)
{
    for( ; pos <= ax.size(); pos += lowbit(pos) )
        bitc[pos] = max(bitc[pos], val);
}
int getMax(int pos)
{
    int ret = 0;
    for( ; pos > 0; pos -= lowbit(pos))
        ret = max(ret, bitc[pos]);
    return ret;
}

int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    int T;  cin >> T;
    while(T--)
    {
        cin >> N;

        memset(bitc, 0, sizeof bitc);
        memset(dp, 0, sizeof dp);
        ax.clear(), ax.push_back(-1);
        ay.clear(), ay.push_back(-1);
        for( int i = 0; i <= N; i++ ) ayx[i].clear();

        vector<pair<int, PII> > p;
        for( int i = 1; i <= N; i++ )
        {
            int x, y, w;  cin >> x >> y >> w;
            p.push_back(make_pair(x, make_pair(y, w)));
            ay.push_back(y), ax.push_back(x);
        }
        sort(ay.begin(), ay.end());
        ay.erase(unique(ay.begin(), ay.end()), ay.end());
        sort(ax.begin(), ax.end());
        ax.erase(unique(ax.begin(), ax.end()), ax.end());

        for( int i = 0; i < p.size(); i++ )
        {
            int x = p[i].first, y = p[i].second.first;
            int w = p[i].second.second;

            y = getidy(y);
            ayx[y].push_back({x, w});
        }

        int ans = 0;
        for( int i = 1; i < ay.size(); i++ )
        {
            int y = getidy(ay[i]);
            vector<PII>& arr = ayx[y];

            for( int j = arr.size()-1; j >= 0; j -- )
            {
                int x = arr[j].first, w = arr[j].second;
                x = getidx(x);

                dp[x] = max(dp[x], getMax(x-1) + w);
                ans = max(dp[x], ans);
                update(x, dp[x]);
            }
        }
        cout << ans << endl;
    }
    return 0;
}

```