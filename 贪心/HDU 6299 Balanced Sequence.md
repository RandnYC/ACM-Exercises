#### [HDU 6299 Balanced Sequence](http://acm.hdu.edu.cn/showproblem.php?pid=6299)

@[贪心]

&ensp; 题目大意: 给定括号平衡的定义, 给出N个括号序列, 任意相接, 求所能组成的最长平衡括号子序列

&ensp; 先把每个序列看作独立的序列, 求取各自的子序列长度并去除这些平衡括号, 统计剩下左括号和右括号的数量。
&ensp; 根据贪心思想，需要金尽量把左括号放前面，右括号放后面，所以把左括号相对右括号多的序列排序靠前，之前删除的平衡括号并不影响后续拼接的括号的平衡性, 所以最后再遍历一遍所有的序列, 计算剩余括号相接的平衡括号数

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 1e5+10;

int N;
char str[MAXN];

struct Mystr
{
    int nl, nr;
    bool operator< (const Mystr& o)const
    {
        int attr = nl-nr, oattr = o.nl-o.nr;
        if( attr >= 0 && oattr < 0 ) return 1;
        else if( attr < 0 && oattr >= 0 ) return 0;
        else if( attr >= 0 && oattr >= 0 ) return nr < o.nr;
        else return nl > o.nl;
    }
}p[MAXN];

int main()
{
    int T; cin >> T;
    while(T--)
    {
        scanf("%d", &N);
        memset(p, 0, sizeof p);

        int ans = 0;
        for( int i = 1; i <= N ; i++)
        {
            scanf("%s", str);
            int sz = strlen(str);
            for( int j = 0; j < sz; j++ )
            {
                if( str[j] == '(') p[i].nl++;
                else
                {
                    if( p[i].nl > 0 )
                    {
                        p[i].nl--;
                        ans ++;
                    }
                    else p[i].nr++;
                }
            }
        }
        sort(p+1, p+N+1);

        int tmp = 0;
        for( int i = 1; i <= N; i++ )
        {
            ans += min(tmp, p[i].nr);
            tmp -= min(tmp, p[i].nr);
            tmp += p[i].nl;
        }
        printf("%d\n", ans*2);
    }
    return 0;
}
```

