#### [SPOJ - ZQUERY](https://vjudge.net/problem/52867/origin)

@[分块, 前缀和]

&ensp; 题目大意: 给出一个元素只有-1和1的序列N, 和Q个询问, 每次询问要求区间中最长的和为0个序列长度

&ensp; 先预处理前缀和以及每个前缀和对应的下标, 再分块预处理两两块之间最长序列, 复杂度$O(N^{1.5})$, 对于每个询问, 根据前缀和二分暴力出不在块中的区间两端间隙, 复杂度$O(Q \cdot \sqrt N \cdot log^{N})$

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 5e4+10;

int N, Q;
int arr[MAXN], sum[MAXN];
vector<int> sumpos[MAXN<<1];

int searchRight(int i, int rbound)
{
    int& sx = sum[--i];
    return *--upper_bound(
        sumpos[sx].begin(),
        sumpos[sx].end(), rbound
    ) - i;
}
int searchLeft(int j, int lbound)
{
    int& sx = sum[j];
    return -*lower_bound(
        sumpos[sx].begin(),
        sumpos[sx].end(), lbound-1
    ) + j;
}

int main()
{
    scanf("%d%d", &N, &Q);
    for( int i = 1; i <= N; i++ )
        scanf("%d", arr+i);

    sum[0] = MAXN, sumpos[sum[0]].push_back(0);
    for( int i = 1; i <= N; i++) {
        sum[i] = sum[i-1] + arr[i];
        sumpos[sum[i]].push_back(i);
    }

    int sz = sqrt(N);
    int units = (N-1)/sz +1;

    static int bk_ans[MAXN/10][MAXN/10];
    for( int ibk = 0; ibk < units; ibk++ )
    {
        static int firstcur[MAXN<<1];
        memset(firstcur, -1, sizeof firstcur);

        int maxlen = 0, i = ibk *sz +1;
        firstcur[sum[i-1]] = i-1;
        for( int j = i; j <= N; j++ )
        {
            int jbk = (j-1)/sz;
            if( !((j-1) %sz) && jbk > ibk)
                bk_ans[ibk][jbk-1] = maxlen;

            int& fs = firstcur[sum[j]];
            ~fs? maxlen = max(maxlen, j-fs): fs = j;
        }
        bk_ans[ibk][units-1] = maxlen;
    }

    while(Q--)
    {
        int a, b;
        scanf("%d%d", &a, &b);
        //cerr << a << " " << b << endl;

        int bl = (a-1)/sz+1, br = (b-1)/sz-1;
        int ans = br >= bl? bk_ans[bl][br] :0;

        int tmp = 0;
        for( int i = a; i < min(bl*sz+1, b); i++ )
        {
            tmp = max(tmp, searchLeft(i, a));
            tmp = max(tmp, searchRight(i, b));
        }
        for( int i = max(a, (br+1)*sz+1); i <= b; i++ )
        {
            tmp = max(tmp, searchLeft(i, a));
            tmp = max(tmp, searchRight(i, b));
        }
        printf("%d\n", ans = max(ans, tmp));
    }

    return 0;
}
```