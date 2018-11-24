#### [HDU 1828 Picture](http://acm.hdu.edu.cn/showproblem.php?pid=1828)

@[线段树, 扫描线]

&ensp; 题目大意: 给出一些矩形, 求裸露部分的周长

- 自下而上扫描一遍, 线段树维护横向长度, 每一段的结果是与前一段长度之差;同理, 从左到右再扫一遍, 线段树维护纵向长度, 最后结果相加即可
- 自下而上扫描一遍, 线段树除了维护长度, 同时维护一个可并区间, 从而统计有多少个区间, 计算横向长度同时用高度差乘区间数的两倍

```cpp
//方法一
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 2e4+10;
const int HASH = 1e4+1;
struct Segment
{
    int L, R;
    int height;
    int bottom;
    bool operator< (const Segment& o) const
    { return height < o.height; }
}lnx[MAXN*2], lny[MAXN*2];

struct STree
{
    int L, R;
    int len, num;
}t[MAXN*2*4];

void build(int root, int L, int R)
{
    t[root].L = L, t[root].R = R;
    t[root].len = t[root].num = 0;

    if( L == R ) return ;
    int mid = L + (R-L)/2;
    build(root*2, L, mid);
    build(root*2+1, mid+1, R);
}

void update(int root, int a, int b, int fg )
{
    int L = t[root].L, R = t[root].R;

    t[root].num += fg;

    if( L == R )
    {
        if( !t[root].num )
            t[root].len = 0;
        else t[root].len = 1;
        return ;
    }

    int mid = L + (R-L)/2;
    if( b <= mid ) update(root*2, a, b, fg);
    else if( a> mid ) update(root*2+1, a, b, fg);
    else
    {
        update(root*2, a, mid, fg);
        update(root*2+1, mid+1, b, fg);
    }

    if( t[root].num )
    {
        t[root].len 
            = t[root*2].len + t[root*2+1].len;
    }
    else t[root].len = 0;
}

int N;

int main()
{
    while(~scanf("%d", &N))
    {
        for( int i = 1; i <= N ; i++ )
        {
            int x1, y1, x2, y2;
            scanf("%d%d%d%d", &x1, &y1, &x2, &y2);
            x1 += HASH, x2 += HASH, y1 += HASH, y2 += HASH;
            lnx[i] = {x1, x2, y1, 1};
            lnx[N+i] = {x1, x2, y2, -1};
            lny[i] = {y1, y2, x1, 1};
            lny[N+i] = {y1, y2, x2, -1};
        }
        sort(lnx+1, lnx+2*N+1), sort(lny+1, lny+2*N+1);
        
        build(1, 1, MAXN);
        int ans = 0, prelen = 0;
        for( int i = 1; i <= 2*N ; i++ )
        {
            update(1, lnx[i].L, lnx[i].R-1, lnx[i].bottom);
            ans += abs(t[1].len-prelen);
            prelen = t[1].len;
        }

        build(1, 1, MAXN);
        prelen = 0;
        for( int i = 1; i <= 2*N ; i++ )
        {
            update(1, lny[i].L, lny[i].R-1, lny[i].bottom);
            ans += abs(t[1].len-prelen);
            prelen = t[1].len;
        }
        printf("%d\n", ans);
    }
    return 0;
}
```

&nbsp;

```cpp
//方法二
#include <bits/stdc++.h>
using namespace std;
const int INF = 1 << 30;
const int MAXN = 5e3+10;

struct Line
{
    int L, R;
    int height, bottom;
    bool operator< (const Line& o) const
    { return height < o.height; }
};

int N;
vector<int> vec;
int getid(int x)
{
    return lower_bound(
        vec.begin(), vec.end(),
      x ) - vec.begin();
}

int len[MAXN*8], times[MAXN*8], dt[MAXN*8];
int cnt[MAXN*8], llen[MAXN*8], rlen[MAXN*8];
void build(int root=1, int L=1, int R=MAXN*2)
{
    len[root] = times[root] = dt[root]
        = cnt[root] = llen[root] = rlen[root] = 0;
    if( L == R ) return ;
    int mid = L + (R-L)/2;
    build(root*2, L, mid);
    build(root*2+1, mid+1, R);
}
void pushdown(int , int , int );
void pushup( int root, int L, int R)
{
    int mid = L + (R-L)/2;
    pushdown(root*2, L, mid);
    pushdown(root*2+1, mid+1, R);

    llen[root] = llen[root*2];
    rlen[root] = rlen[root*2+1];
    cnt[root] = cnt[root*2] + cnt[root*2+1];
    len[root] = len[root*2] + len[root*2+1];

    if( llen[root*2] >= mid-L+1)
        llen[root] += llen[root*2+1];
    if( rlen[root*2+1] >= R-(mid+1)+1)
        rlen[root] += rlen[root*2];

    if( rlen[root*2] && llen[root*2+1]) cnt[root] --;

}
void pushdown(int root, int L, int R)
{
    if( dt[root])
    {
        times[root] += dt[root];
        if( L != R)
        {
            dt[root*2] += dt[root];
            dt[root*2+1] += dt[root];
        }
        dt[root] = 0;

        if( times[root])
        {
            len[root] = vec[R+1] - vec[L];
            llen[root] = rlen[root] = R-L+1;
            cnt[root] = 1;
        }
        else if( L == R )
        {
            len[root] = cnt[root]
                = llen[root] = rlen[root] = 0;
        }
        else pushup(root, L, R);
    }
}
void update(int root, int a, int b, int val, int L=1, int R=2*MAXN)
{
    if( a == L && b == R ) return void(dt[root] += val);

    int mid = L + (R-L)/2;
    if( b <= mid ) update(root*2, a, b, val, L, mid);
    else if( a > mid ) update(root*2+1, a, b, val, mid+1, R);
    else
    {
        update(root*2, a, mid, val, L, mid);
        update(root*2+1, mid+1, b, val, mid+1, R);
    }
    pushup(root, L, R);
}
inline pair<int, int> query(int L=1, int R=2*MAXN)
{
    pushdown(1, L, R);
    return make_pair(len[1], cnt[1]);
}

int main()
{
    while(~scanf("%d", &N))
    {
        vec.clear(), vec.push_back(-INF);

        vector<Line> ln;
        for( int i = 1; i <= N; i++ )
        {
            int a, b, c, d;
            scanf("%d%d%d%d", &a, &b, &c, &d);
            ln.push_back({a, c, b, 1});
            ln.push_back({a, c, d, -1});
            vec.push_back(a), vec.push_back(c);
        }
        sort(ln.begin(), ln.end());
        sort(vec.begin(), vec.end());
        vec.erase(unique(vec.begin(), vec.end()), vec.end());

        build();

        int ans = 0, prelen = 0;
        for( int i = 0; i < ln.size(); i++ )
        {
            int& a = ln[i].L, &b = ln[i].R;
            int& val = ln[i].bottom;

            update(1, getid(a), getid(b)-1, val);
            int len = query().first, cnt = query().second;
            ans += abs(len - prelen);

            if( i < ln.size()-1 )
                ans += 2*cnt *(ln[i+1].height - ln[i].height);

            prelen = len;
        }
        printf("%d\n", ans);
    }
    return 0;
}

```