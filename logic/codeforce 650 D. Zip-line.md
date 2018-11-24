#### [codeforce 650 D. Zip-line](http://codeforces.com/problemset/problem/650/D)

@[可持久化线段树, LIS, 离线, 树状数组]

&ensp; 题目大意: 求单点修改后的最长上升子序列

&ensp; 预处理出原始序列的最长上升子序列LIS, 以及以每个点为结尾以及以每个点为起始的最长上升子序列的长度ll[i], lr[i] (可以通过二分或树状数组求出)

&ensp; 判断原始序列的每个点情况, 如果不算上这个点, 则LIS的情况为: 

种类 | 去掉后的LIS | 替换后的LIS
--- | --- | --- 
必经点 | LIS -1 | LIS 或 LIS -1
可能点 | LIS | LIS +1 或 LIS
无效点 | LIS | LIS


&ensp; 归纳可见如果为必经关键点, 则答案至少为LIS -1, 否则答案至少为LIS; 假设每个问题所替换的点为关键点, 可以算出以这个点为结尾的和以这个点为起始的最长上升子序列长度ll和lr, 最后答案 ans = max(关键点? LIS-1: LIS, ll + lr - 1)

&ensp; 至于计算以每个点为起始的和以每个点为结尾的最长上升子序列, 需要逐步添加元素维护树状数组, 这里为了维护序列每一个版本使用在线可持久化线段树, 需要两棵, 一个从左往右, 另一个从右往左

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 4e5+10;

struct Query {
    int pos, v;
} qy[MAXN];

int N, Q;
int arr[MAXN];

vector<int> bak(1);
int getid(int x)
{
    return lower_bound(
        bak.begin(), bak.end(),
      x ) - bak.begin();
}

struct STree
{
    vector<int> root;  int stkptr;

    int maxlen[MAXN*25];
    int lnxt[MAXN*25], rnxt[MAXN*25];

    STree(): stkptr(1) { root.resize(1); }

    int update(int preroot, int pos, int val, int L=1, int R=bak.size())
    {
        int root = stkptr ++;
        maxlen[root] = maxlen[preroot];
        lnxt[root] = lnxt[preroot], rnxt[root] = rnxt[preroot];

        if( L == R ) return maxlen[root] = max(maxlen[root], val), root;

        int mid = L + (R-L)/2;
        if( pos <= mid )
            lnxt[root] = update(lnxt[preroot], pos, val, L, mid);
        else rnxt[root] = update(rnxt[preroot], pos, val, mid+1, R);

        maxlen[root] = max(
            maxlen[lnxt[root]],
            maxlen[rnxt[root]]
        );
        return root;
    }

    int query(int root, int a, int b, int L=1, int R=bak.size())
    {
        if( a > b ) return 0;
        if( a == L && b == R) return maxlen[root];
        int mid = L + (R-L)/2;
        if( b <= mid ) return query(lnxt[root], a, b, L, mid);
        else if(a > mid) return query(rnxt[root], a, b, mid+1, R);
        else
        {
            return max( query(lnxt[root], a, mid, L, mid),
                    query(rnxt[root], mid+1, b, mid+1, R) );
        }
    }
}tl, tr;

bool crit[MAXN];
int lis;
void getLIS()
{
    static int ll[MAXN], lr[MAXN];

    vector<int> spt(1);
    spt.reserve(MAXN);

    for( int i = 1; i <= N; i++ )
    {
        int x = getid(arr[i]);
        if( x > spt.back())
        {
            spt.push_back(x);
            ll[i] = spt.size() -1;
        }
        else
        {
            int p = lower_bound(
                spt.begin(), spt.end(),
              x ) - spt.begin();
            spt[p] = x, ll[i] = p;
        }
    }
    lis = spt.size()-1;

    spt.clear(), spt.push_back(1<<30);
    for( int i = N; i >= 1; i-- )
    {
        int x = getid(arr[i]);
        if( x < spt.back())
        {
            spt.push_back(x);
            lr[i] = spt.size() -1;
        }
        else
        {
            int p = lower_bound(
                spt.begin(), spt.end(), x,
                [](int xx, int yy) { return xx > yy; }
            ) - spt.begin();
            spt[p] = x, lr[i] = p;
        }
    }

    static int cnt[MAXN];
    for( int i = 1; i <= N; i++ ) {
        if( ll[i] + lr[i] -1 == lis)
            cnt[ll[i]] ++;
    }
    for( int i = 1; i <= N; i++ ) {
        if( ll[i] + lr[i] -1 == lis &&
          cnt[ll[i]] == 1) crit[i] = 1;
    }
}

int main()
{
    bak.reserve(MAXN<<1);

    scanf("%d%d", &N, &Q);

    for( int i = 1; i <= N; i++ ) scanf("%d", arr+i);
    for( int q = 1; q <= Q; q ++ )
    {
        scanf("%d%d", &qy[q].pos, &qy[q].v);
        bak.push_back(qy[q].v);
    }

    bak.insert(bak.end(), arr+1, arr+N+1);
    stable_sort(bak.begin(), bak.end());
    bak.erase(unique(bak.begin(), bak.end()), bak.end());

    for( int i = 1; i <= N; i++ )
    {
        int x = getid(arr[i]);
        tl.root.push_back(tl.update(
            tl.root.back(), x,
            tl.query(tl.root.back(), 1, x-1) +1
        ));
    }

    for( int i = N; i >= 1; i-- )
    {
        int x = getid(arr[i]);
        tr.root.push_back(tr.update(
            tr.root.back(), x,
            tr.query(tr.root.back(), x+1, bak.size()) +1
        ));
    }

    getLIS();

    for( int q = 1; q <= Q; q++ )
    {
        const int& pos = qy[q].pos;
        int x = getid(qy[q].v);

        int ans = crit[pos]? lis-1: lis;

        int ll = tl.query(tl.root[pos-1], 1, x-1) +1;
        int lr = tr.query(tr.root[N-pos], x+1, bak.size()) +1;
        ans = max(ans, ll + lr -1);

        printf("%d\n", ans);
    }

    return 0;
}
```