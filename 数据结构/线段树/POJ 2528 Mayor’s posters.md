#### [POJ 2528 Mayor's posters](http://poj.org/problem?id=2528)

@[线段树, 离散化]

&ensp; 题目大意: 每次选定一个区间染上不同的颜色, 问最后能看到多少种颜色。
&ensp; 区间很大，需要离散化来压缩区间，但注意压缩区间时，如[1, 10], [1, 4], [6, 10] 压缩会把可见的[5, 6]压缩掉, 所以离散化时需要再记录大于1的两个端点间的任意值。

&ensp; 线段树节点tag标记-1为区间中含多种颜色，0为没有颜色，>0 为对应的颜色标号

```cpp
#include <iostream>
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <vector>
#include <set>
using namespace std;
typedef long long LL;
typedef set<int> SI;
const int MAXN = 4e4+10;

struct STree
{
    int L, R;
    int tag, dt;
}t[MAXN*4];

void build(int root, int L, int R)
{
    t[root].L = L, t[root].R = R;
    t[root].tag = t[root].dt = 0;

    if( L == R ) return ;
    int mid = L + (R-L)/2;
    build(root*2, L, mid);
    build(root*2+1, mid+1, R);
} 

void pushdown(int root)
{
    if( t[root].dt)
    {
        int L = t[root].L, R = t[root].R;
        t[root].tag = t[root].dt;

        if( L != R)
        {
            t[root*2].dt = t[root*2+1].dt
                = t[root].dt;
        }
        t[root].dt = 0;
    }
}

void update(int root, int a, int b, int c)
{
    int L = t[root].L, R = t[root].R;
    if( a== L && b == R)
    {
        t[root].dt = c;
        return ;
    }
    
    pushdown(root);
    int mid = L + (R-L)/2;
    if( b <= mid ) update(root*2, a, b, c);
    else if( a > mid ) update(root*2+1, a, b, c);
    else
    {
        update(root*2, a, mid, c);
        update(root*2+1, mid+1, b, c);
    }
    
    pushdown(root*2), pushdown(root*2+1);
    int t1 = t[root*2].tag, t2 = t[root*2+1].tag;
    if( t1 < 0 || t2 < 0 || (t1>0 && t2>0))
        t[root].tag = -1;
    else if( !t1 && t2) t[root].tag = t2;
    else if( !t2 && t1) t[root].tag = t1;
    else t[root].tag = 0;
}


SI ans;
void query(int root, int a, int b)
{
    int L = t[root].L, R = t[root].R;
    int mid = L +(R-L)/2;

    if( t[root].dt ) 
    {
        ans.insert(t[root].dt);
        return ;
    }
    
    if( a == L && b == R)
    {
        if( t[root].tag < 0 )
        {
            query(root*2, a, mid);
            query(root*2+1, mid+1, b);
        }
        else if( t[root].tag ) 
            ans.insert(t[root].tag);
        return ;
    }
    
    pushdown(root);
    if( b <= mid ) query(root*2, a, b);
    else if( a > mid ) query(root*2+1 , a, b);
    else
    {
        query(root*2, a, mid);
        query(root*2+1, mid+1, b);
    }
}

struct Line
{
    int L, R;
}ln[MAXN];

int M;

vector<int> h;
int getid(LL x)
{
    return (
        lower_bound(h.begin(),
          h.end(), x) - h.begin()
    );
}

int main()
{
    int T; cin >> T;
    while(T--)
    {
        h.clear(), ans.clear();

        scanf("%d", &M);
        for( int i = 0 ; i < M ; i++)
        {
            int a, b; scanf("%d%d", &a, &b);
            ln[i].L = a, ln[i].R = b;
            h.push_back(a), h.push_back(b);
        }
        h.push_back(0);
        sort(h.begin(), h.end());
    
        int sz = h.size();
        for( int i = 1 ; i < sz ; i++)
        {
            if( h[i] - h[i-1] > 1 )
                h.push_back(h[i-1]+1);
        }
        sort(h.begin(), h.end());


        build(1, 1, h.size());
        for( int i = 0 ; i < M ; i++)
        {
            int a= ln[i].L , b= ln[i].R;
            update(1, getid(a), getid(b), i+1);
        }
        query(1, 1, h.size());
        printf("%d\n", int(ans.size()));
    }
    return 0;
}
```