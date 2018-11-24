#### [2018上海大都会赛F  Color it](https://www.nowcoder.com/acm/contest/163/F)

@[暴力, 扫描线]

&ensp; 题目大意: 给出一个矩阵, 和一系列圆的信息, 求没有被圆覆盖的点的个数

&ensp; 由于圆的个数比较少, 根据扫描线的思想枚举每一行和所有圆相交的区间, 求每一行的区间交集之和, 最后从总个数里扣掉即可

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 2e4+10;

int N, M, Q;

struct Circle{ int x, y, r; } c[MAXN];
struct Line
{
    int L, R;
    bool operator< (const Line& o)const
    {
        if( L == o.L ) return R < o.R;
        return L < o.L;
    }
    bool operator== (const Line& o)const
    { return L <= o.L && R >= o.R; }
};

int main()
{
    int T; cin >> T;
    while(T--)
    {
        scanf("%d%d%d", &N, &M, &Q);

        for( int i = 0; i < Q; i++ )
            scanf("%d%d%d", &c[i].x, &c[i].y, &c[i].r);

        int ans = N *M;
        for( int i = 0; i < N; i++ )
        {
            vector<Line> v;
            for( int j = 0; j < Q; j++ )
            {
                int x = c[j].x, y = c[j].y, r = c[j].r;
                if( abs(i-x) > r ) continue;

                double d = sqrt(r*r - (x-i)*(x-i));
                int L = ceil(y-d), R = floor(y+d);
                v.push_back({max(0, L), min(R, M-1)});
            }
            sort(v.begin(), v.end());
            v.erase(unique(v.begin(), v.end()), v.end());

            if(v.empty()) continue;

            int len = v[0].R - v[0].L +1;
            for( int i = 1; i < v.size(); i++ )
            {
                if(v[i].L > v[i-1].R)
                    len += v[i].R - v[i].L +1;
                else len += v[i].R - v[i-1].R;
            }
            ans -= len;
        }
        printf("%d\n", ans);
    }
    return 0;
}

```