#### [codeforce Gym - 101550B Bless You Autocorrect! ](http://codeforces.com/gym/101550/attachments/download/6031/20162017-acmicpc-nordic-collegiate-programming-contest-ncpc-2016-en.pdf)

@[字典树, bfs]

&ensp; 题目大意: 手机码字会提示, 选择一个提示需要按tab, 删除需要按delete, 给出一系列后台字典, 和需要输入的字符, 问对于每个询问的字符最少按几次可以输入
&ensp; 建立字典树, 对字典树bfs记录从根节点到这个节点的最短步长, 最后直接查询最大相同长度

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 1<<20;

int N, M;
string str;

struct Trie
{
    Trie* next[26];
    Trie* skip, *parent;
    int step;
    Trie() : step(0), skip(0), parent(0)
    { memset(next, 0 , sizeof next); }
}*root=0;

void insert( Trie* p, int k, Trie*& ed )
{
    if( k == str.size() ) return ;

    Trie*& t = p->next[str[k]-'a'];
    if( !t ) t = new Trie;
    ed = t, t->parent = p;

    insert(t, k+1, ed);
    if(!t->skip) t->skip = ed;
}

unordered_map<Trie*, bool> visited;
void bfs()
{
    visited.clear();
    queue<pair<Trie*, int> > que;
    que.push({root, 0});

    while(!que.empty())
    {
        Trie* p = que.front().first;
        int step = que.front().second;
        que.pop();

        if( !p ) continue;
        if( visited[p] ) continue;
        visited[p] = 1;
        p->step = step;

        for(int i = 0; i < 26 ; i++ )
            que.push({p->next[i], step+1});
        que.push({p->skip, step+1});
        que.push({p->parent, step+1});
    }
}

int query()
{
    int ans = 0;
    Trie* p = root; int i = 0;
    for( i = 0; i < str.size() ; i++)
    {
        p = p->next[str[i]-'a'];
        if( !p ) break;
        else ans = p->step;
    }

    return ans+str.size()-i;
}

void clear(Trie *p)
{
    if( !p ) return ;
    for( int i = 0 ; i < 26; i++ )
        clear(p->next[i]);
    delete(p);
}

int main()
{
    ios::sync_with_stdio(false);
    cin.tie(0);

    while(cin >> N >> M)
    {
        clear(root);
        Trie *tmp = 0; root = new Trie;
        while(N--)
        {
            cin >> str;
            insert(root, 0, tmp);
        }

        bfs();
        while(M--)
        {
            cin >> str;
            cout << query() << "\n";
        }
    }
    return 0;
}

```