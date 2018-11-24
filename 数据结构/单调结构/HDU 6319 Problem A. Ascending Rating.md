#### [HDU 6319 Problem A. Ascending Rating](http://acm.hdu.edu.cn/showproblem.php?pid=6319)

@[单调队列]

&ensp; 题目大意: 计算每个滑动窗口的最大值和以区间第一个元素为起点的最长上升子序列长度

&ensp; 正向无法维护个数, 所以倒跑维护单调双端队列

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
typedef pair<LL, LL> PLL;
const int MAXN = 1e7+10;

int N, M;
LL arr[MAXN];
int k, p, q, r, mod;

void generate()
{
    for( int i = k+1; i <= N; i++ )
    {
        arr[i] = ((LL)p *arr[i-1] %mod
            + (LL)q *i %mod + r %mod) %mod;
    }
}

PLL getAns()
{
    LL ans[2] = {0, 0};
    deque<int> que;  que.push_back(N);
    for( int i = N-1; i >= N-M+1; i-- )
    {
        if( arr[i] < arr[que.front()] )
            que.push_front(i);
        else
        {
            while(!que.empty() && arr[i] >= arr[que.front()] )
                que.pop_front();
            que.push_front(i);
        }
    }
    ans[0] = arr[que.back()] ^(N-M+1);
    ans[1] = que.size() ^(N-M+1);

    for( int i = N-M; i >= 1; i-- )
    {
        if( que.back() == i+M ) que.pop_back();

        if( arr[i] < arr[que.front()] )
            que.push_front(i);
        else
        {
            while(!que.empty() && arr[i] >= arr[que.front()] )
                que.pop_front();
            que.push_front(i);
        }

        ans[0] += arr[que.back()] ^i;
        ans[1] += que.size() ^i;
    }
    return make_pair(ans[0], ans[1]);
}

int main()
{
    int T;  cin >> T;
    while(T--)
    {
        scanf("%d%d", &N, &M);
        scanf("%d%d%d%d%d", &k,
            &p, &q, &r, &mod);

        for( int i = 1; i <= k; i++ )
            scanf("%lld", arr+i);
        generate();

        PLL ans = getAns();
        printf("%lld %lld\n", ans.first, ans.second);
    }
    return 0;
}

```