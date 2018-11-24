#### [HDU 2871 Memory Control](http://acm.hdu.edu.cn/showproblem.php?pid=2871)

@[线段树, 区间合并]

&ensp; 题目大意: 给出一个内存区间, New操作来申请一块指定长度的空间, 左边优先输出左端点; Free 释放一个指定下标所在的内存空间; Reset操作回收所有的内存; Get操作查询指定下标所申请空间左端点

&ensp; 线段树维护空闲区间的可合并区间, 同时用vector来记录已申请的内存。

&nbsp;

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 5e4 +10;

struct STree
{
    int L, R;
    int maxlen, dt;
    int llen, rlen;
}t[MAXN*4];

void build(int root, int L, int R)
{
    t[root].L = L, t[root].R = R;
    t[root].llen = t[root].rlen
        = t[root].maxlen = R-L+1;   
    t[root].dt = 0;

    if( L == R ) return ;
    int mid = L + (R-L)/2;
    build(root*2, L, mid);
    build(root*2+1, mid+1, R);
}

void pushdown(int root)
{
    if( t[root].dt )
    {
        int L = t[root].L, R = t[root].R;
        t[root].llen = t[root].rlen 
            = t[root].maxlen 
            = t[root].dt>0 ? R-L+1 : 0;
        if( L != R )
        {
            t[root*2].dt = t[root*2+1].dt
                = t[root].dt;
        }
        t[root].dt = 0; 
    }
}

void pushup(int root)
{
    int L = t[root].L, R = t[root].R;
    int mid = L + (R-L)/2;

    t[root].llen = t[root*2].llen;
    t[root].rlen = t[root*2+1].rlen;

    if( t[root*2].llen >= mid-L+1)
        t[root].llen += t[root*2+1].llen;
    if( t[root*2+1].rlen >= R-(mid+1)+1 )
        t[root].rlen += t[root*2].rlen;
    
    t[root].maxlen = max(
        max(t[root*2].maxlen, t[root*2+1].maxlen),
        t[root*2].rlen + t[root*2+1].llen
    );
}

void update(int root, int a, int b, bool apply)
{
    int L = t[root].L, R = t[root].R;
    if( a == L && b == R )
    {
        t[root].dt = apply ? -1 : 1;
        return ;
    }
    pushdown(root);

    int mid = L + (R-L)/2;
    if( b <= mid ) update(root*2, a, b, apply);
    else if( a > mid ) update(root*2+1, a, b, apply);
    else
    {
        update(root*2, a, mid, apply);
        update(root*2+1, mid+1, b, apply);
    }

    pushdown(root*2), pushdown(root*2+1);
    pushup(root);
}

int query(int root, int len)
{
    int L = t[root].L, R = t[root].R;
    pushdown(root);

    if( t[root].maxlen < len ) return 0;
    if( R-L+1 == len ) return L;

    pushdown(root*2), pushdown(root*2+1);

    int mid = L + (R-L)/2;
    if( t[root*2].maxlen >= len )
        return query(root*2, len);
    else if( t[root*2].rlen + t[root*2+1].llen >= len )
        return mid-(t[root*2].rlen)+1;
    else if( t[root*2+1].maxlen >= len )
        return query(root*2+1, len);
    else return 0;
}

struct MBlocks
{
    int L, R;
    bool operator < (const MBlocks& o) const
    { return L < o.L; }
    bool operator < (const int &x) const
    { return R < x; }
};

vector<MBlocks> v;
int N, Q;

int main()
{
    while(~scanf("%d%d", &N, &Q))
    {
        char op[7];
        build(1, 1, N), v.clear();
        while(Q--)
        {
            scanf("%s", op);
            if( !strcmp(op, "New") )
            {
                int len; scanf("%d", &len);
                int a = query(1, len);
                if( a )
                {
                    update(1, a, a+len-1, 1);
                    printf("New at %d\n", a);
                    MBlocks tmp = MBlocks( {a, a+len-1} );
                    v.insert(
                        upper_bound(v.begin(), v.end(), tmp),
                        tmp );
                }
                else puts("Reject New");
            }
            else if( !strcmp(op, "Reset") )
            {
                update(1, 1, N, 0);
                v.clear();
                puts("Reset Now");
            }
            else if ( !strcmp(op, "Free") )
            {
                int x; scanf("%d", &x);
                int i = lower_bound( v.begin(), 
                    v.end(), x ) - v.begin();

                if( i == v.size() || v[i].L > x )
                    puts("Reject Free");
                else
                {
                    printf("Free from %d to %d\n", v[i].L, v[i].R);
                    update(1, v[i].L , v[i].R, 0);
                    v.erase(v.begin()+i);
                }
            }
            else
            {
                int i; scanf("%d", &i);
                if( i-1 >= v.size() ) puts("Reject Get");
                else printf("Get at %d\n", v[i-1].L);
            }
        }
        puts("");
    }  
    return 0;
}

```