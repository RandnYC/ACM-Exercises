#### [HDU 1542 Atlantis](http://acm.hdu.edu.cn/showproblem.php?pid=1542)

@[线段树, 扫描线, 离散化, 模板]

&ensp; 题目大意: 给出矩形左下和右上端点求区间面积并

1. 因为数据是浮点数，所以需要离散化， 处理步骤特殊：
	- 离散化维护左闭右开的区间，将离散化后的坐标右端点r-1 加入线段树，获取长度时hash[r+1] - hash[l] 

2. 扫描线求面积：下端线段+1, 上端-1, 每份面积累加到最近的上面一条扫描线，若有n条线段， 则一共n-1次累加

3. 线段树维护区间被完整覆盖次数num，和每个区间包含的线段实际长度len

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Line
{
	double a, b;
	int bottom;
	double height;
	bool operator< (const Line& o)const
	{ return height < o.height; }
};

int N;
vector<double> vec;
int getid( double x )
{
	return lower_bound(
		vec.begin(), vec.end(),
	  x ) - vec.begin();
}

double len[1000];
int num[1000], dt[1000];
void build(int root=1, int L=1, int R=2*N)
{
	len[root] = num[root] = dt[root] = 0;
	if( L == R ) return ;
	int mid = L + (R-L)/2;
	build(root*2, L, mid);
	build(root*2+1, mid+1, R);
}
void pushdown(int , int , int);
void pushup(int root, int L, int R)
{
	int mid = L + (R-L)/2;
	pushdown(root*2, L, mid);
	pushdown(root*2+1, mid+1, R);
	len[root] = len[root*2] + len[root*2+1];
}
void pushdown(int root, int L, int R)
{
	if(dt[root])
	{
		num[root] += dt[root];
		if( L != R )
		{
			dt[root*2] += dt[root];
			dt[root*2+1] += dt[root];
		}
		dt[root] = 0;

		if( num[root] ) len[root] = vec[R+1] - vec[L];
		else if( L == R ) len[root] = 0;
		else pushup(root, L, R);
	}
}
void update(int root, int a, int b, int val, int L=1, int R=2*N)
{
	if( a == L && b == R ) return void(dt[root] +=val);

	int mid = L + (R-L)/2;
	if( b <= mid ) update(root*2, a, b, val , L, mid);
	else if( a > mid ) update(root*2+1, a, b, val, mid+1, R);
	else
	{
		update(root*2, a, mid, val, L, mid);
		update(root*2+1, mid+1, b, val, mid+1, R);
	}
	pushup(root, L, R);
}
inline double query(int L=1, int R=2*N)
{ pushdown(1, L, R); return len[1]; }

int main()
{
	ios::sync_with_stdio(0);
	cin.tie(0);

	int Case = 0;
	while(cin >> N && N)
	{
		vec.clear(), vec.push_back(-1);

		vector<Line> ln;
		for( int i = 1; i <= N; i++ )
		{
			double a, b, c, d;
			cin >> a >> b >> c >> d;
			ln.push_back({a, c, 1, b});
			ln.push_back({a, c, -1, d});
			vec.push_back(a), vec.push_back(c);
		}

		sort(vec.begin(), vec.end());
		sort(ln.begin(), ln.end());
		vec.erase(unique(vec.begin(), vec.end()), vec.end());

		build();

		double ans = 0;
		for( int i = 0; i < ln.size()-1; i++ )
		{
			double a = ln[i].a, b = ln[i].b;
			update(1, getid(a), getid(b)-1, ln[i].bottom);
			ans += query() *(ln[i+1].height-ln[i].height);
		}
		printf("Test case #%d\nTotal explored area: %.2f\n\n", ++Case, ans);
	}
	return 0;
}
```