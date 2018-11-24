#### [HDU 2795 Billboard](http://acm.hdu.edu.cn/showproblem.php?pid=2795)

@[线段树]

&ensp; 题目大意: 给出一个H*W的板, 可以粘帖N张纸, 纸的宽度是w高度1, 优先从上面和左边开始贴, 问每个贴纸在哪一行。

&ensp; 线段树以列作为下标，对应数字是每行的宽度， 维护一个最大值.

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 2e5 +10;

int H, W, M;

struct STree
{
    int L, R;
    int Max;
}t[MAXN*4];

void build(int root, int L, int R)
{
    t[root].L = L, t[root].R = R;
    t[root].Max = W;
    if( L == R ) return ;
    
    int mid = L + (R-L)/2;
    build(root*2, L , mid );
    build(root*2+1, mid+1, R);
}

void update(int root, int v)
{
    int L = t[root].L, R = t[root].R;
    if( L == R )
    {
        t[root].Max -= v;
        printf("%d\n", L);
        return ;  
    }
    
    int mid = L + (R-L)/2;
    if( t[root*2].Max >= v) update(root*2, v);
    else update(root*2+1, v);

    t[root].Max = max( t[root*2].Max, t[root*2+1].Max);
}

int main()
{
    while(~scanf("%d%d%d", &H, &W, &M))
    {
        build(1, 1, min(H, M));
        while(M--)
        {
            int w; scanf("%d", &w);
            if( w > t[1].Max ) puts("-1"); 
            else update(1, w);
        }
    }
    return 0;
}
```