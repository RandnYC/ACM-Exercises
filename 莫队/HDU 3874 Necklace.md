#### [HDU 3874 Necklace](http://acm.hdu.edu.cn/showproblem.php?pid=3874)

@[莫队]

&ensp; 题目大意: 给定一个序列, 每次询问回答某个区间内不同的数的个数之和

&ensp; 离线处理: 将问题按右端点排序, 每次扫过去的时候维护一个数最后一次出现的下标, 如果重复则删除在树状数组之前的位置, 更新维护区间和

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
const int MAXN = 5e4+10;

struct Query
{
    int L, R;
    int idx;

    bool operator< (const Query& o)const
    { return R < o.R; }
}qy[MAXN*4];

int N, Q;
int lastpos[MAXN*20], arr[MAXN];
LL ans[MAXN*4];

LL c[MAXN];
inline int lowbit(int& x) { return x & -x; }
void update(int pos, int val)
{
    for( ; pos <= N ; pos += lowbit(pos) )
        c[pos] += val;
}
LL getSum(int pos)
{
    LL ret = 0;
    for( ; pos > 0; pos -= lowbit(pos) )
        ret += c[pos];
    return ret;
}

void clear()
{
    memset(c, 0, sizeof c);
    memset(lastpos, 0, sizeof lastpos);
}

int main()
{
    int T; cin >> T;
    while(T--)
    {
        scanf("%d", &N);

        clear();
        for( int i = 1; i <= N; i++)
            scanf("%d", arr+i);
        scanf("%d", &Q);

        for( int i = 1; i <= Q ; i++ )
        {
            scanf("%d%d", &qy[i].L, &qy[i].R);
            qy[i].idx = i;
        }
        sort(qy+1, qy+Q+1);

        for( int i = 1, k = 1; i <= N ; i++ )
        {
            if( lastpos[arr[i]] )
                update(lastpos[arr[i]], -arr[i]);
            update(i, arr[i]);
            lastpos[arr[i]] = i;

            while(k <= Q && qy[k].R == i )
            {
                const int &L = qy[k].L, &R = qy[k].R;
                ans[qy[k++].idx] = getSum(R)-getSum(L-1);
            }
        }

        for( int i = 1; i <= Q; i++ )
            printf("%lld\n", ans[i]);
    }
    return 0;
}
```
&nbsp;

&ensp; 也可以将线段树可持续化, 同样维护某个数最后一次出现的下标位置, 记录每次操作后的线段树版本, 所以对于版本r的线段树, 记录的就是下标从1到r的只出现过一次的数字的和。因此每次查询只要在右端点版本的线段树中查询区间和即可。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
const int MAXN = 5e4+10;

int N, Q;

struct Node {
    int next[2];
    LL sum;
}t[MAXN *30];

int stkptr = 0;
vector<int> root(1);

int update(int preroot, int pos, int val, int L=1, int R=N)
{
    int root = stkptr ++;
    t[root] = t[preroot];

    if( L == R) return t[root].sum +=val, root;
    int mid = L + (R-L)/2;
    if( pos <= mid) {
        t[root].next[0] = update(
            t[preroot].next[0],
            pos, val, L, mid
        );
    }
    else {
        t[root].next[1] = update(
            t[preroot].next[1],
            pos, val, mid+1, R
        );
    }

    t[root].sum = t[t[root].next[0]].sum +
                t[t[root].next[1]].sum;
    return root;
}
LL query( int root, int a, int b, int L=1, int R=N)
{
    if( a <= L && b >= R) return t[root].sum;

    int mid = L + (R-L)/2;  LL res = 0;
    if( a <= mid) res += query(t[root].next[0], a, b, L, mid);
    if( b > mid) res += query(t[root].next[1], a, b, mid+1, R);
    return res;
}

int main()
{
    int T;  cin >> T;
    while(T--)
    {
        stkptr = 1;
        root.clear(), root.push_back(0);

        static int lastpos[MAXN*20];
        memset(lastpos, 0, sizeof lastpos);

        scanf("%d", &N);
        for( int i = 1; i <= N; i++)
        {
            int x;  scanf("%d", &x);

            if( lastpos[x]) {
                root.push_back(update(
                    root.back(),
                    lastpos[x], -x
                ));
            }
            else root.push_back(root.back());

            lastpos[x] = i;
            root.back() = update(
                root.back(), i, x
            );
        }

        scanf("%d", &Q);
        while(Q-- )
        {
            int a, b;
            scanf("%d%d", &a, &b);
            printf("%lld\n", query(root[b], a, b));
        }
    }
    return 0;
}
```

&nbsp;

&ensp; 或者用莫队直接分块统计出每个区间元素第一次出现的值
```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
const int MAXN = 5e4+10;

int N, Q;
int arr[MAXN], cnt[MAXN*20];
LL ans[MAXN*4], g_res;
int unit;

struct Query
{
    int L, R;
    int idx;
    bool operator< (const Query& o)const
    {
        if( L/unit == o.L/unit )
            return  R < o.R;
        return L/unit < o.L/unit;
    }
}qy[MAXN*4];


inline void add(int x)
{ if(!cnt[x]++) g_res += x; }

inline void del(int x)
{ if(!--cnt[x]) g_res -= x; }


int main()
{
    int T; cin >> T;
    while(T--)
    {
        g_res = 0;
        memset( cnt, 0, sizeof cnt);

        scanf("%d", &N);
        for( int i = 1; i <= N; i++ )
            scanf("%d", arr+i);
        scanf("%d", &Q);
        for(int i = 1; i <= Q ; i++ )
        {
            scanf("%d%d", &qy[i].L, &qy[i].R);
            qy[i].idx = i;
        }

        unit = sqrt(N);
        sort(qy+1, qy+Q+1);

        int L = 0, R = 0;
        for( int i = 1 ; i <= Q ; i++ )
        {
            while(R < qy[i].R) add(arr[++R]);
            while(R > qy[i].R) del(arr[R--]);
            while(L < qy[i].L) del(arr[L++]);
            while(L > qy[i].L) add(arr[--L]);
            ans[qy[i].idx] = g_res;
        }

        for( int i = 1; i <= Q; i++)
            printf("%lld\n", ans[i]);
    }
    return 0;
}
```