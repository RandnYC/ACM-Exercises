#### [HDU 6301 Distinct Values](http://acm.hdu.edu.cn/showproblem.php?pid=6301)

@[莫队, 离线, 思想]

&ensp; 题目大意: 给定一个区间的长度和操作的次数, 每次操作选定一个区间使得这个区间的元素两两不同且区间元素满足字典序最小

&ensp; 将区间排序, 删除被覆盖的小区间, 结合莫队思想用两个左右区间端点的指针, 维护一个装有当前区间答案候选的set

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 1e5+10;

int N, M;

int ans[MAXN];

struct Segment
{
    int L, R;
    bool operator< (const Segment& o)const
    {
        if( L == o.L ) return R > o.R;
        return L < o.L;
    }
    bool operator== (const Segment& o)const
    {
        if( L <= o.L && R >= o.R ) return 1;
        return 0;
    }
};

int main()
{
    int T; cin >> T;
    while(T--)
    {
        memset(ans, 0, sizeof ans);
        scanf("%d%d", &N, &M);

        set<int> cho;
        vector<Segment> ln;

        while(M--)
        {
            register int a, b;
            scanf("%d%d", &a, &b);
            ln.push_back(Segment({a, b}));
        }
        sort(ln.begin(), ln.end());
        ln.erase(unique(ln.begin(), ln.end()), ln.end());

        for( int i = 1; i <= N; i++ )
        {
            cho.insert(i);
            ans[i] = 1;
        }

        int L = 1, R = 0;
        int prel = 0, prer = 0;
        for( int i = 0; i < ln.size(); i++ )
        {
            if( prer < ln[i].L )
            {
                while(L <= prer )
                    cho.insert(ans[L++]);
                R = ln[i].L-1;
            }
            else
            {
                while(L < ln[i].L)
                    cho.insert(ans[L++]);
            }

            while(R < ln[i].R)
            {
                ans[++R] = *cho.begin();
                cho.erase(cho.begin());
            }

            prel = ln[i].L, prer = ln[i].R;
        }

        for( int i = 1; i <= N ; i++ )
            printf("%d%c", ans[i], (i==N? '\n' :' '));
    }
    return 0;
}
```