#### [codeforce 17 E. Palisection](http://codeforces.com/problemset/problem/17/E)

@[回文自动机]

&ensp; 题目大意: 给出一个字符串, 求有多少对回文串互相有重叠部分

&ensp; 问题可以转化成所有回文串两两组合的情况 - 两两不相交的情况

&ensp; 因为在回文自动机中, 一个节点的fail指针指向的回文后缀和这个节点具有一定长度相同的后缀, 也就是fail节点的回文后缀和当前节点有着一定相同的模式。

&ensp; 记num[i] 为以i节点为后缀的最长回文串的长度, 于是num[i] = num[fail[i]] + 1, 加的1表示加入新字符后形成的新的回文后缀

&ensp; 于是可以统计出对应字符串每个下标之前的所有回文子串的数量, 再倒过来逐个添加字符, 便可以形成与不相交的回文子串的情况

&ensp; 用前向星来优化节点next指针的空间

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
const int MAXN = 2e6+10;
const int MOD = 51123987;

struct Edge {
    static int stkptr;
    int u, v, w;
    int next;
} e[MAXN <<1];
int Edge::stkptr, head[MAXN];

int S[MAXN], sp = 0;
struct Node {
    static int lastptr;
    int cnt, num, len;
    int fail;
}ss[MAXN];
int Node::lastptr, stkptr = 0;

int newNode(int len=1)
{
    Node& o = ss[stkptr];
    o.fail = 0, o.len = len;
    o.cnt = o.num = 0;
    return stkptr ++;
}

void init() {
    stkptr = sp = 0;
    Node::lastptr = 0;
    newNode(0), newNode(-1);
    S[0] = -1;  ss[0].fail = 1;

    Edge::stkptr = 0;
    memset(head, -1, sizeof head);
}   // 0 -fail-> 1

int getFail(int o) {
    while(S[sp-ss[o].len-1] != S[sp])
        o = ss[o].fail;
    return o;
}

int getNext(int p, int ch)
{
    for( int i = head[p]; ~i; i = e[i].next)
        if( e[i].w == ch ) return e[i].v;
    return 0;
}
inline void addEdge(int p, int q, int ch)
{
    int &n = Edge::stkptr;
    e[n] = {p, q, ch, head[p]};
    head[p] = n ++;
}

void add(int ch)
{
    int& lastptr = Node::lastptr;

    S[++sp] = (ch-='a');
    int p = getFail(lastptr);
    if( !getNext(p, ch))
    {
        int q = newNode(ss[p].len+2);
        ss[q].fail = getNext(getFail(ss[p].fail), ch);
        ss[q].num = ss[ss[q].fail].num + 1;
        addEdge(p, q, ch);
    }

    lastptr = getNext(p, ch);
    ss[lastptr].cnt ++;
}

void getCount() {
    const int n = stkptr;
    for( int i = n-1; i >= 0; i--)
        ss[ss[i].fail].cnt += ss[i].cnt;
}
void insert(string& str) {
    for( int i = 0; i < str.size(); i++)
        add(str[i]);
}

int N;
int sum[MAXN];

int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    cin >> N;

    string str;  cin >> str;

    init();

    for( int i = 0; i < str.size(); i++)
    {
        add(str[i]);
        int p = Node::lastptr;
        sum[i] = ((i - 1 < 0? 0:
            sum[i-1]) + ss[p].num) %MOD;
    }

    int ans = 0;
    init();

    for( int i = str.size()-1; i >= 1; i-- ) {
        add(str[i]);
        int p = Node::lastptr;
        (ans += (LL)ss[p].num *sum[i-1] %MOD) %=MOD;
    }

    int tot = sum[str.size()-1];
    tot = (LL)tot *(tot-1)/2 %MOD;

    cout << (tot-ans +MOD) %MOD << endl;

    return 0;
}
```