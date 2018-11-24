#### [HDU 4814 Golden Radio Base](http://acm.hdu.edu.cn/showproblem.php?pid=4814)

@[模拟, 数学]

&ensp; 题目大意: 将一个十进制数转化为黄金比例进制数, 要求不能出现连续的1

&ensp; 已知公式 $\phi ^i = \phi ^{i-1} + \phi ^{i-2} $ 和 $2 \cdot \phi ^{i}  = \phi ^{i+1} + \phi ^{i-2}$, 由第一个公式可以将两个连续的一化为进一位的一个一; 并且根据第二个公式可以将进制位上大于1的数降到0或1


```cpp
#include <bits/stdc++.h>
using namespace std;

int N;
int arr[107];

int main()
{
	while(cin >> N)
	{
		memset(arr, 0, sizeof arr);
		arr[50] = N;

		bool fg = 0;
		do{
			fg = 0;
			for( int i = 0; i < 100; i++ )
			{
				int &x = arr[i];
				if( x > 1) {
					arr[i+1] += x/2;
					arr[i-2] += x/2;
					x &= 1, fg = 1;
				}
			}

			for( int i = 0; i < 100; i++ )
			{
				if( arr[i] && arr[i+1])
				{
					int t = min(arr[i], arr[i+1]);
					arr[i] -= t, arr[i+1] -= t;
					arr[i+2] += t, fg = 1;
				}
			}
		}while(fg);

		int st = 0, ed = 0;
		for( int i = 100; i >= 0; i-- )
			if( arr[i]) { st = i;  break; }
		for( int i = 0; i < 100; i++ )
			if( arr[i]) { ed = i;  break; }

		for( int i = st; i >= ed; i-- )
		{
			cout << arr[i];
			if( i == 50 && ed < 50)
				cout << ".";
		}
		cout << endl;
	}
	return 0;
}
```