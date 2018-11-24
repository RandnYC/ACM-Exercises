####[HDU 5748 Bellovin](http://acm.hdu.edu.cn/showproblem.php?pid=5748)

@[LIS, 树状数组, 二分]

&ensp; 题目大意: 求以每个位置为结尾的最长上升子序列

&ensp; 方法一: 树状数组


```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 1e5+10;

int N, arr[MAXN];
int lis[MAXN];

vector<int> bak;
int getid(int x)
{
    return lower_bound(
        bak.begin(), bak.end(),
      x ) - bak.begin();
}

int bitc[MAXN];
inline int lowbit(int &x) { return x & -x; }
void update(int pos, int val)
{
    for( ; pos <= N; pos += lowbit(pos))
        bitc[pos] = max(bitc[pos], val);
}
int getMax(int pos)
{
    int ret = 0;
    for( ; pos > 0; pos -= lowbit(pos))
        ret = max(ret, bitc[pos]);
    return ret;
}

int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    int T;  cin >> T;
    while(T--)
    {
        bak.clear();  bak.push_back(0);
        memset(bitc, 0, sizeof bitc);

        cin >> N;
        for( int i = 1; i <= N; i++ )
            cin >> arr[i];

        bak.insert(bak.end(), arr+1, arr+N+1);
        sort(bak.begin(), bak.end());
        bak.erase(unique(bak.begin(), bak.end()), bak.end());

        for( int i = 1; i <= N; i++ )
        {
            int x = getid(arr[i]);
            lis[i] = getMax(x-1) +1;
            update(x, lis[i]);
        }

        for( int i = 1; i <= N; i++ )
            cout << lis[i] << ( i == N? "\n" :" ");
        //cout << endl;
    }

    return 0;
}
```

&nbsp; 
***
&ensp; 方法二: 二分

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 1e5+10;

int N, arr[MAXN];
int lis[MAXN];

vector<int> bak;
int getid(int x)
{
    return lower_bound(
        bak.begin(), bak.end(),
      x ) - bak.begin();
}

int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    int T;  cin >> T;
    while(T--)
    {
        bak.clear();  bak.push_back(0);

        cin >> N;
        for( int i = 1; i <= N; i++ )
            cin >> arr[i];

        bak.insert(bak.end(), arr+1, arr+N+1);
        sort(bak.begin(), bak.end());
        bak.erase(unique(bak.begin(), bak.end()), bak.end());

        vector<int> v(1);
        for( int i = 1; i <= N; i++ )
        {
            int x = getid(arr[i]);
            if( v.back() < x) {
                v.push_back(x);
                lis[i] = v.size()-1;
            }
            else {
                int p = lower_bound(
                    v.begin(), v.end(),
                  x ) - v.begin();
                v[p] = x, lis[i] = p;
            }
        }

        for( int i = 1; i <= N; i++ )
            cout << lis[i] << ( i == N? "\n" :" ");
        //cout << endl;
    }

    return 0;
}
```