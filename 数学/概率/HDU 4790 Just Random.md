#### [HDU 4790 Just Random](http://acm.hdu.edu.cn/showproblem.php?pid=4790)

@[数学]

&ensp; 题目大意: 给出两个区间[a, b], [c, d] , [a, b] 取x; [c, d]取y, 求形成(x+y) %p = m的x与y的概率

&ensp; 转化为二维平面, a, b在x轴上, c, d在y轴上, 题目也就是求矩形内满足(x+y) %p = m的点数

&ensp; 用 y = -x + k 可将矩形划分为两个三角形和一个平行四边形, 分别统计点数

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;

int main()
{
	int T;  cin >> T;
	int Case = 0;
	while(T--)
	{
		LL a, b, c, d, p, m;
		cin >> a >> b >> c >> d >> p >> m;

		if( b-a > d-c ) swap(a, c), swap(b, d);

		LL k1, k2, ans = 0;

		k1 = ceil((a+c-m)*1.0/p);
		k2 = floor((b+c-1-m)*1.0/p);
		ans += ((k1*p+m -a-c+1)+(k2*p+m -a-c+1)) *(k2-k1+1)/2;

		k1 = ceil((b+c-m)*1.0/p);
		k2 = floor((a+d-1-m)*1.0/p);
		ans += (k2-k1+1) *(b-a+1);

		k1 = ceil((a+d-m)*1.0/p);
		k2 = floor((b+d-m)*1.0/p);
		ans += ((b+d-(k1*p+m)+1)+(b+d-(k2*p+m)+1)) *(k2-k1+1)/2;

		LL frac = LL(b-a+1) *(d-c+1);
		LL f = __gcd(ans, frac);
		frac /= f, ans /= f;
		printf("Case #%d: %lld/%lld\n", ++Case, ans, frac);
	}
	return 0;
}
```