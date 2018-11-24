#### [HDU 5661 Claris and XOR](http://acm.hdu.edu.cn/showproblem.php?pid=5661)

@[贪心, 异或]

&ensp; 题目大意: 给定a, b, c, d, 求x在ab范围, y在cd范围内x异或y的最大值

&ensp; 参考01字典树的思想, 对于当前二进制位:
- 若ab和cd各不相同, 则从此开始的二进制位都可以凑出1
- 若ab和cd其中一组相同, 另一组不同, 则选法唯一并当前位可以凑出1, 若看作是字典树中, 不同组对应的指针需要指向对应的子树中, 若映射到数字上, 进行相应的二进制调整即可
- 若ab和cd每组内都相同, 则选法唯一

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;

int main()
{
    int T;  cin >> T;
    while(T--)
    {
        LL a, b, c, d;
        cin >> a >> b >> c >> d;

        LL ans = 0;
        for( int i = 62; i >= 0; i-- )
        {
            bool t[] = {
                (a>>i) &1, (b>>i) &1,
                (c>>i) &1, (d>>i) &1
            };

            if( t[0] != t[1] && t[2] != t[3] )
            {
                while(i >= 0)
                    ans += (1LL <<i--);
                break;
            }
            else if( t[0] == t[1] && t[2] != t[3] )
            {
                ans += (1LL <<i);
                if( t[0]) d = (1LL <<i) -1;
                else c = 0;
            }
            else if( t[0] != t[1] && t[2] == t[3])
            {
                ans += (1LL <<i);
                if( t[2]) b = (1LL <<i) -1;
                else a = 0;
            }
            else if( t[0] != t[2] ) ans += (1LL <<i);
        }

        cout << ans << endl;
    }
    return 0;
}

```