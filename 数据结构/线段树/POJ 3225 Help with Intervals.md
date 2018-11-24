#### [POJ 3225 Help with Intervals](http://poj.org/problem?id=3225)

@[线段树]

&ensp; 题目大意: 给出几轮交并差异或操作和连续区间, 求最后区间

Command | Semantics
--- | ---
U T	| S ← S ∪ T
I T	|S ← S ∩ T
D T|	S ← S − T
C T|	S ← T − S
S T|	S ← S ⊕ T

&nbsp;

- 操作转换:
	- U相当于给定区间[a, b]置1
	- I相当于[-inf, a)和(b, inf)置0
	- D相当于[a,b]置0
	- C相当于[-inf,a)和(b,inf]置0, [a,b]取反
	- S相当于[a,b]取反
- 线段树处理区间开闭采用区间倍增法: 坐标*2,
	-	(l, r) --> [2*l+1, 2*r-1] 
	-	(l, r] --> [2*r+1, 2*r]
	-	[l, r) -->  [2*l, 2*r-1)
	-	[l, r] --> [2*l, 2*r]
	-  最后还原时只需倍增法的逆过程, 区间开闭由倍增区间的奇偶决定

- 在处理取反时, 懒惰标记若遇到两次取反直接消去即可


```cpp
#include <iostream>
#include <cstdio>
#include <cstring>
#include <string>
#include <set>
#include <sstream>
#include <algorithm>
using namespace std;
const int MAXN = 14e4 +10;
bool visited[MAXN];

struct STree
{
    int L, R;
    int dt, cover;
}t[MAXN*4];

void build(int root, int L, int R)
{
    t[root].L = L, t[root].R = R;
    t[root].cover = t[root].dt = 0;

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

        if( t[root].dt == 1 )
            t[root].cover = 1;
        else if( t[root].dt == 2 )
            t[root].cover = 0;
        else if( t[root].dt == -1 )
        {
            if( t[root].cover >= 0 )
                t[root].cover = !t[root].cover;
        }

        if( L != R )
        {
            if( t[root].dt < 0 )
            {
                if( t[root*2].dt < 0 )
                    t[root*2].dt = 0;
                else if( t[root*2].dt > 0 )
                    t[root*2].dt = t[root*2].dt == 1? 2:1;
                else t[root*2].dt = t[root].dt;

                if( t[root*2+1].dt < 0)
                    t[root*2+1].dt = 0;
                else if( t[root*2+1].dt > 0 )
                    t[root*2+1].dt = t[root*2+1].dt == 1 ? 2:1;
                else t[root*2+1].dt = t[root].dt;
            }
            else
            {
                t[root*2].dt = t[root*2+1].dt
                    = t[root].dt;
            }
        }
        t[root].dt = 0;
    }
}

void pushup(int root)
{
    int c1 = t[root*2].cover, c2 = t[root*2+1].cover;
    if( c1 < 0 || c2 < 0 )
        t[root].cover = -1;
    else if( !c1 && !c2 )
        t[root].cover = 0;
    else if( c1 == 1 && c2 == 1)
        t[root].cover = 1;
    else t[root].cover = -1;
}

void update(int root, int a, int b, int fg)
{
    if( a > b ) return ;

    int L = t[root].L, R = t[root].R;
    if( a== L && b== R )
    {
        if( fg < 0 && t[root].dt < 0 )
            t[root].dt = 0;
        else if( fg < 0 && t[root].dt > 0)
            t[root].dt = t[root].dt == 1? 2:1;
        else t[root].dt = fg;
        return ;
    }

    pushdown(root);
    int mid = L + (R-L)/2;
    if( b <= mid ) update(root*2, a, b, fg);
    else if( a > mid ) update(root*2+1, a, b, fg);
    else
    {
        update(root*2, a, mid, fg);
        update(root*2+1, mid+1, b, fg);
    }
    pushdown(root*2), pushdown(root*2+1);
    pushup(root);
}

void query(int root)
{
    int L = t[root].L, R = t[root].R;
    pushdown(root);
    if( t[root].cover == 1 )
    {
        for(int i = L ; i <= R ; i++ )
            visited[i] = 1;
        return ;
    }
    else if( !t[root].cover ) return ;

    query(root*2);
    query(root*2+1);
}

void Union(int a, int b)
{
    update(1, a, b, 1);
}

void Intersect(int a, int b)
{
    update(1, 1, a-1, 2);
    update(1, b+1, MAXN, 2);
}

void Minus1(int a, int b)
{
    update(1, a, b, 2);
}

void Minus2(int a, int b)
{
    Intersect(a, b);
    update(1, a, b, -1);
}

void Symmertic(int a, int b)
{
    update(1, a, b, -1);
}

void show()
{
    query(1);

    int st = 0;
    bool first = 1, empty = 1;

    for( int x = 1; x <= MAXN ; x++)
    {
        if( !st && visited[x] )
            st = x;
        else if( st && !visited[x] )
        {
            int ed = x-1;
            if(!first) printf(" ");
            first = 0, empty = 0;

            printf("%c%d,%d%c", st&1 ? '(': '[',
                st/2-1, (ed+1)/2-1, ed&1? ')':']');
            st = 0;
        }
    }

    if( empty ) puts("empty set");
    else puts("");
}

int main()
{
    char op, lch, mch, rch;
    int a, b;

    build(1, 1, MAXN);
    while(~scanf(" %c %c%d %c%d %c", &op,
            &lch, &a, &mch, &b, &rch))
    {
        a++, a*=2;
        if( lch == '(') a++;
        b++, b*=2;
        if( rch == ')') b--;

        switch(op)
        {
            case 'U':
                Union(a, b);
                break;
            case 'I':
                Intersect(a, b);
                break;
            case 'D':
                Minus1(a, b);
                break;
            case 'C':
                Minus2(a, b);
                break;
            case 'S':
                Symmertic(a, b);
                break;
        }
    }
    show();

    return 0;
}
```