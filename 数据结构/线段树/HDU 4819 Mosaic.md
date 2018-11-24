#### [HDU 4819 Mosaic](http://acm.hdu.edu.cn/showproblem.php?pid=4819)
@[线段树, 二维]

&ensp; 题目大意: 给出一个N维方阵, 每次给出一个坐标点, 和范围求这个范围内的最大和最小值的平均值, 并以此替换这个坐标

&ensp; 二维线段树维护最大最小值, 更新的时候需要区分横向x是否是单行(外部叶子), 若是则直接修改y区间的那个值; 反之x需要对比提取子区间中y区间的最值


```cpp
#include <bits/stdc++.h>
using namespace std;
const int INF = 2e9+10;
const int MAXN = 8e2+10;

int N, M;

struct STree
{
    int L, R;
    struct SubSTree
    {
        int L, R;
        int Min, Max;
    }t[MAXN*4];

    void build(int root, int L, int R)
    {
        t[root].L = L, t[root].R = R;
        t[root].Min = INF, t[root].Max = 0;

        if( L == R ) return ;
        int mid = L + (R-L)/2;
        build(root*2, L, mid);
        build(root*2+1, mid+1, R);
    }

    void update(int root, int pos, int val)
    {
        int L = t[root].L, R = t[root].R;
        if( L == R )
        {
            t[root].Min = t[root].Max = val;
            return ;
        }

        int mid = L + (R-L)/2;
        if( pos <= mid ) update(root*2, pos, val);
        else update(root*2+1, pos, val);

        t[root].Min = min(t[root*2].Min, t[root*2+1].Min);
        t[root].Max = max(t[root*2].Max, t[root*2+1].Max);
    }

    pair<int, int> query(int root, int a, int b)
    {
        int L = t[root].L, R = t[root].R;
        if( a == L && b == R )
            return make_pair(t[root].Min, t[root].Max);

        int mid = L + (R-L)/2;
        if( b <= mid ) return query(root*2, a, b);
        else if( a > mid ) return query(root*2+1, a, b);
        else
        {
            pair<int, int> ans1 = query(root*2, a, mid),
                        ans2 = query(root*2+1, mid+1, b);
            return make_pair(
                min(ans1.first, ans2.first),
                max(ans1.second, ans2.second)
            );
        }
    }
}t[MAXN*4];

void build( int root, int L, int R)
{
    t[root].L = L , t[root].R = R;
    t[root].build(1, 1, N);

    if( L == R ) return ;
    int mid = L + (R-L)/2;
    build(root*2, L, mid);
    build(root*2+1, mid+1, R);
}

void pushup(int root, int subrt, int pos)
{
    int subL = t[root].t[subrt].L, subR = t[root].t[subrt].R;

    t[root].t[subrt].Max = max(
        t[root*2].t[subrt].Max,
        t[root*2+1].t[subrt].Max
    );

    t[root].t[subrt].Min = min(
        t[root*2].t[subrt].Min,
        t[root*2+1].t[subrt].Min
    );

    if( subL == subR ) return ;

    int subm = subL + (subR-subL)/2;
    if( pos <= subm ) pushup(root, subrt*2, pos);
    else pushup(root, subrt*2+1, pos);
}

void update(int root, int pos1, int pos2, int val)
{
    int L = t[root].L, R = t[root].R;

    if( L == R )
    {
        t[root].update(1, pos2, val);
        return ;
    }

    int mid = L + (R-L)/2;
    if( pos1 <= mid ) update(root*2, pos1, pos2, val);
    else update(root*2+1, pos1, pos2, val);

    pushup(root, 1, pos2);
}

pair<int, int> query(int root, int a1, int b1, int a2, int b2)
{
    int L = t[root].L, R = t[root].R;
    if( a1 == L && b1 == R )
        return t[root].query(1, a2, b2);

    int mid = L + (R-L)/2;
    if( b1 <= mid ) return query(root*2, a1, b1, a2, b2);
    else if( a1 > mid ) return query(root*2+1, a1, b1, a2, b2);
    else
    {
        pair<int, int> ans1 = query(root*2, a1, mid, a2, b2),
                    ans2 = query(root*2+1, mid+1, b1, a2, b2);
        return make_pair(
            min(ans1.first, ans2.first),
            max(ans1.second, ans2.second)
        );
    }
}

int main()
{
    int T; cin >> T;
    int Case = 0;
    while(T--)
    {
        printf("Case #%d:\n", ++Case);

        scanf("%d", &N);
        build(1, 1, N);
        for( int i = 1; i <= N; i++ )
        {
            for( int j = 1; j <= N; j++ )
            {
                register int x; scanf("%d", &x);
                update(1, i, j, x);
            }
        }

        scanf("%d", &M);
        while(M--)
        {
            register int x, y, l;
            scanf("%d%d%d", &x, &y, &l);

            register int x1, x2, y1, y2;
            x1 = x-l/2, x2 = x+l/2;
            y1 = y-l/2, y2 = y+l/2;

            x1 = x1 < 1? 1: x1; x2 = x2 > N? N: x2;
            y1 = y1 < 1? 1: y1; y2 = y2 > N? N: y2;

            pair<int, int> ans
                = query(1, x1, x2, y1, y2);
            printf("%d\n", (ans.first+ans.second)/2);
            update(1, x, y, (ans.first+ans.second)/2);
        }
    }
    return 0;
}
```