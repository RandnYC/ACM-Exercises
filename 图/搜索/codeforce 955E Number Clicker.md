#### [codeforce 955E Number Clicker](http://codeforces.com/problemset/problem/995/E)

@[双向bfs]

&ensp; 题目大意: 给定一个数字和一个目标数字以及一个素数模数, 可以对数字执行+1或-1或取逆元的操作, 问如何到达目标并打印其中一种策略

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;

int st, ed, mod;
struct Point { int x, step, pre, op; };

int qpow(LL base, int index=mod-2)
{
    LL ans = 1;
    while(index)
    {
        if( index &1)
            (ans *= base) %= mod;
        index /= 2;
        (base *= base) %= mod;
    }
    return ans;
}

map<int, bool> vis[2];
map<int, pair<int, int> > path[2];

int bfs()
{
    queue<Point> que[2];
    que[0].push({st, 0, -1, -1}), que[1].push({ed, 0, -1, -1});

    while(!que[0].empty() || !que[1].empty())
    {
        for( int i = 0; i <= 1; i ++ )
        {
            if( !que[i].empty())
            {
                Point tp = que[i].front();  que[i].pop();
                int &x = tp.x, &step = tp.step,
                    &pre = tp.pre, &op = tp.op;

                if( vis[i][x]) continue;
                vis[i][x] = 1;

                if( ~pre ) path[i][x] = make_pair(pre, i? (op==3? 3: op^3): op);

                if( vis[i ^1][x]) return x;

                que[i].push({(x+1) %mod, step+1, x, 1});
                que[i].push({(x+mod-1) %mod, step+1, x, 2});
                que[i].push({qpow(x), step+1, x, 3});
            }
        }
    }
}

int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    cin >> st >> ed >> mod;

    int p = bfs();
    path[0][st] = path[1][ed] = make_pair(-1, -1);

    deque<int> ans;

    int x[2] = {path[0][p].first, path[1][p].first};
    int op[2] = {path[0][p].second, path[1][p].second};

    while(~x[0] || ~x[1])
    {
        for( int i = 0; i <= 1; i++ )
        {
            if( !~x[i]) continue;

            if(!i) ans.push_front(op[i]);
            else ans.push_back(op[i]);

            op[i] = path[i][x[i]].second;
            x[i] = path[i][x[i]].first;
        }
    }

    cout << ans.size() << endl;
    while(!ans.empty())
    {
        cout << ans.front();
        ans.pop_front();
        cout << (ans.empty()? "\n": " ");
    }
    //cout << endl;

    return 0;
}
```