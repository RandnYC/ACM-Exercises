#### [codeforces 242E XOR on Segment ](https://vjudge.net/problem/34357/origin)

@[线段树]

&ensp; 题目大意: 给定长度为N的序列, Q个询问,  每次询问可以为区间中的每个数逐个异或一个给定的数, 或者询问区间求和

&ensp; 分解成二进制, 按每一位分别操作

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
const int MAXN = 1e5+10;

int N, Q;
int arr[MAXN];

int cnt[MAXN<<2][27];
bool dt[MAXN<<2][27];

void pushup(int root)
{
    for( int i = 0; i <= 20; i++ ){
        cnt[root][i] = cnt[root*2][i] +
                cnt[root*2+1][i];
    }
}
void build(int root=1, int L=1, int R=N)
{
    if( L == R) for( int i = 0; i <= 20; i++ )
    {
        int& v = arr[L];
        cnt[root][i] += (v >>i) &1;
    }
    else {
        int mid = L + (R-L>>1);
        build(root*2, L, mid);
        build(root*2+1, mid+1, R);

        pushup(root);
    }
}

void pushdown(int root, int L, int R)
{
    for( int i = 0; i <= 20; i++ ) if( dt[root][i])
    {
        cnt[root][i] = R-L+1 - cnt[root][i];

        if( L != R) {
            dt[root*2][i] ^= 1;
            dt[root*2+1][i] ^= 1;
        }
        dt[root][i] = 0;
    }
}

void update(int root, int a, int b, int& v, int L=1, int R=N)
{
    if( a == L && b == R) {
        for( int i = 0; i <= 20; i++ )
            dt[root][i] ^= (v >>i) &1;
        return ;
    }

    pushdown(root, L, R);

    int mid = L + (R-L>>1);
    if( b <= mid) update(root*2, a, b, v, L, mid);
    else if( a > mid) update(root*2+1, a, b, v, mid+1, R);
    else {
        update(root*2, a, mid, v, L, mid);
        update(root*2+1, mid+1, b, v, mid+1, R);
    }

    pushdown(root*2, L, mid);
    pushdown(root*2+1, mid+1, R);

    pushup(root);
}

LL query(int root, int a, int b, int L=1, int R=N)
{
    pushdown(root, L, R);

    if( a <= L && b >= R) {
        LL res = 0;
        for( int i = 0; i <= 20; i++ )
            res += (LL)cnt[root][i] *(1<<i);
        return res;
    }

    int mid = L + (R-L>>1); LL res = 0;
    if( a <= mid) {
        res += query(root*2, a, b, L, mid);
    }
    if( b > mid) {
        res += query(root*2+1, a, b, mid+1, R);
    }
    return res;
}

int main()
{
    scanf("%d", &N);
    for( int i = 1; i <= N; i++ )
        scanf("%d", arr+i);

    build();

    scanf("%d", &Q);
    while(Q--)
    {
        int op;  scanf("%d", &op);

        if( op == 1)
        {
            int a, b;  scanf("%d%d", &a, &b);
            printf("%lld\n", query(1, a, b));
        }
        else {
            int a, b, v;
            scanf("%d%d%d", &a, &b, &v);
            update(1, a, b, v);
        }
    }
    return 0;
}
```
