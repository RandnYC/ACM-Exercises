#### [poj 2991 Crane](http://poj.org/problem?id=2991)
@[线段树]

&ensp; 题目大意: 给出一个机械臂, 可以按关节变换角度, 给出机械臂的关节数N和M个操作, 每次操作把机械臂这一节和前一节的夹角变为指定角度, 每次输出机械臂最后一节的坐标 。

&ensp; 考虑到改变某一节的角度并不影响除了这一节的相对夹角, 因此可以用线段树来给但前节和后面所有节加上一个变化角度。

```cpp
#include <iostream>
#include <cstdio>
#include <cstring>
#include <cmath>
#include <algorithm>
using namespace std;
const double PI = acos(-1);
const int MAXN = 1e4+10;

inline double rad2deg(double rad) { return rad/PI*180; }
inline double deg2rad(double deg) { return deg/180*PI; }

int len[MAXN];

struct Vec2
{
    double x, y;
    double getDegree() const
    {
        if( !x ) return y > 0? 90: 270;
        if( !y ) return x > 0? 0: 180;

        if( x > 0 && y > 0 ) return rad2deg(atan(y/x));
        else if( x < 0 && y > 0 ) return rad2deg(PI-atan(abs(y/x)));
        else if( x < 0 && y < 0 ) return rad2deg(PI+atan(y/x));
        else if( x > 0 && y < 0 ) return rad2deg(2*PI-atan(abs(y/x)));
    }
    double getLen()const { return sqrt(x*x + y*y); }

    Vec2 operator+ (const double deg) const
    {
        double nowdeg = fmod(getDegree()+deg+360, 360);
        double nowlen = getLen();
        return Vec2({nowlen* cos(deg2rad(nowdeg)), nowlen* sin(deg2rad(nowdeg))});
    }
    Vec2 operator+ (const Vec2& o) const
    { return Vec2({x+o.x, y+o.y}); }
};

struct STree
{
    int L, R;
    Vec2 tp;
    int dt;
}t[MAXN*4];

void pushup( int root)
{
    t[root].tp = t[root*2].tp+t[root*2+1].tp;
}

void pushdown(int root)
{
    if( t[root].dt )
    {
        int L= t[root].L, R = t[root].R;

        t[root].tp = t[root].tp + t[root].dt;
        if( L != R )
        {
            t[root*2].dt = (t[root*2].dt+t[root].dt+360) %360;
            t[root*2+1].dt = (t[root*2+1].dt+t[root].dt+360) %360;
        }
        t[root].dt = 0;
    }
}

void build(int root, int L, int R)
{
    t[root].L = L, t[root].R = R;
    t[root].dt = 0;

    if( L == R )
    {
        t[root].tp = Vec2({0, len[L]});
        return ;
    }

    int mid = L + (R-L)/2;
    build(root*2, L, mid);
    build(root*2+1, mid+1, R);

    pushup(root);
}

void update(int root, int a, int b, int deg)
{
    int L= t[root].L, R = t[root].R;
    if( a == L && b== R )
    {
        t[root].dt = (t[root].dt+deg+360) %360;
        return ;
    }

    pushdown(root);
    int mid = L + (R-L)/2;
    if( b <= mid ) update(root*2, a, b, deg);
    else if( a > mid ) update(root*2+1, a, b, deg);
    else
    {
        update(root*2, a, mid, deg);
        update(root*2+1, mid+1, b, deg);
    }

    pushdown(root*2), pushdown(root*2+1);
    pushup(root);
}

int N, M;
int predeg[MAXN];

int main()
{
    bool first = 1;
    while( ~scanf("%d%d", &N, &M))
    {
        if( !first ) puts(""), first = 0;
        for( int i = 1; i <= N ; i++ )
        {
            scanf("%d", len+i);
            predeg[i] = 180;
        }

        build(1, 1, N);

        while(M--)
        {
            register int index, deg;
            scanf("%d%d", &index, &deg);

            update(1, index+1, N, deg-predeg[index+1]);
            predeg[index+1] = deg;

            printf("%.2f %.2f\n", t[1].tp.x, t[1].tp.y);
        }
    }
    return 0;
}
```