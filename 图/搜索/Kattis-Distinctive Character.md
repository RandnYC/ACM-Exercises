#### [Kattis-Distinctive Character](https://open.kattis.com/problems/distinctivecharacter)
@[bfs]

&ensp; 给出N个01串, 求出任意一个串使其与所有源串中的最大相似度尽可能的小

&ensp; 问题可转化为目标串与所有源串的最小不相似度尽可能的大。对所有源串bfs，bfs的步长就是但前串到源串的最小不相似度，记录一个最大值即可

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 1e5+10;

int M, N;
bool visited[(1<<20)+10];

int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    string input;
    while(cin >> M >> N )
    {
        memset(visited, 0, sizeof visited);
        queue<pair<int, int> > que;
        for( int m = 0 ; m < M ; m++ )
        {
            int s = 0;
            cin >> input;
            for( int i = 0 ; i < N ; i++ )
                if( input[N-1-i] - '0') s |= 1<<i;
            que.push({s, 0});
        }

        int ans, Max = -1;
        while(!que.empty())
        {
            int s = que.front().first;
            int step = que.front().second;
            que.pop();

            if(visited[s]) continue;
            visited[s] = 1;

            if( step > Max )
                Max=step, ans=s;

            for( int i = 0 ; i < N ; i++ )
            {
                int ss = s^(1<<i);
                que.push({ss, step+1});
            }
        }

        for( int i = N-1 ; i>= 0 ; i--)
            cout << ((ans>>i) & 1);
        cout << endl;
    }
    return 0;
}

```