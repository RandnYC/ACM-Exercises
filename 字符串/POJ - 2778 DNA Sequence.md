#### [POJ - 2778 DNA Sequence](http://poj.org/problem?id=2778)

@[AC自动机, 矩阵快速幂]


&ensp; 题目大意: 给出N个模式串, 和主串长度, 求不含模式串的所有主串有多少种

&ensp; 在tire树尾节点打上标记, 求出AC自动机的每个状态转移到其他状态的关系矩阵M, 这个矩阵M表示从对应行坐标状态到列坐标状态走一步的可行方法数

&ensp; 设AC自动机有n个状态, 根据离散数学的知识可知, 矩阵M的n次方即状态两两可达的所有情况数, 最后所需求的就是根到所有状态的情况数, 将矩阵的第一行求和

```cpp
#include <iostream>
#include <cstdio>
#include <cstring>
#include <string>
#include <vector>
#include <queue>
#include <cassert>
#include <algorithm>
using namespace std;
typedef long long LL;
const int MOD = 1e5;

void copy(LL a[][107], LL b[][107], int n)
{
    for( int i = 0; i < n; i++ )
        for( int j = 0; j < n; j++ )
            a[i][j] = b[i][j];
}
void clear(LL a[][107], int n)
{
    for( int i = 0; i < n; i ++)
        for( int j = 0; j < n; j++ )
            a[i][j] = 0;
}

void multi(LL a[][107], LL b[][107], int n)
{
    static LL res_multi[107][107];
    clear(res_multi, n);

    for( int i = 0; i < n; i ++)
    {
        for( int j = 0; j < n; j++ )
        {
            for( int k = 0; k < n; k++ )
                res_multi[i][j] += a[i][k] *b[k][j];
            res_multi[i][j] %= MOD;
        }
    }

    copy(a, res_multi, n);
}

LL res_qpow[107][107];
void qpow(LL b[][107], LL index, int n)
{
    static LL base[107][107];

    clear(res_qpow, n);
    copy(base, b, n);

    for( int i = 0; i < 107; i++ )
        res_qpow[i][i] = 1;

    while(index)
    {
        if(index &1)
            multi(res_qpow, base, n);
        index /= 2;
        multi(base, base, n);
    }
}

inline int getid(char& ch)
{
    if( ch == 'A') return 0;
    if( ch == 'C') return 1;
    if( ch == 'G') return 2;
    if( ch == 'T') return 3;
}

struct Node {
    int next[4], fail;
    bool stuck;
}ss[107];

int stkptr = 0;
int newNode()
{
    Node& o = ss[stkptr];
    memset(o.next, -1, sizeof o.next);
    o.fail = -1, o.stuck = 0;
    return stkptr ++;
}

void init()
{
    stkptr = 0;
    newNode();
}
void insert(string &pat)
{
    int p = 0;
    for( int i = 0; i < pat.size(); i++ )
    {
        int k = getid(pat[i]);
        if( !~ss[p].next[k])
            ss[p].next[k] = newNode();
        p = ss[p].next[k];
    }
    ss[p].stuck = 1;
}

void getFail()
{
    queue<int> que;

    ss[0].fail = 0;
    for( int k = 0; k < 4; k++ )
    {
        int &v = ss[0].next[k];
        if( ~v) {
            ss[v].fail = 0;
            que.push(v);
        } else v = 0;
    }

    while(!que.empty())
    {
        int u = que.front();  que.pop();
        ss[u].stuck |= ss[ss[u].fail].stuck;

        for( int k = 0; k < 4; k++ )
        {
            int &v = ss[u].next[k];
            int &f = ss[u].fail;
            if( ~v) {
                ss[v].fail = ss[f].next[k];
                que.push(v);
            }
            else v = ss[f].next[k];
        }
    }
}

int N, M;

LL m[107][107];

int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    cin >> M >> N;


    init();
    for( int i = 1; i <= M; i++ )
    {
        string str;  cin >> str;
        insert(str);
    }

    getFail();

    const int n = stkptr;

    for( int p = 0; p < n; p++ )
    {
        if( ss[p].stuck ) continue;

        for( int k = 0; k < 4; k++ )
        {
            int s = ss[p].next[k];
            if( ss[s].stuck ) continue;
            m[p][s] ++;
        }
    }


    qpow(m, N, n);

    //assert(N < 2000000000);

    LL ans = 0;
    for( int i = 0; i < n; i++ )
        ans += res_qpow[0][i];

    cout << ans %MOD << endl;

    return 0;
}
```

&nbsp;

***
#### [HDU - 2243 F - 考研路茫茫――单词情结](https://vjudge.net/contest/238819#problem/F)

&ensp; 题目大意: 给出M个模式串求长度小于等于N的至少包含一个模式串的目标串有多少种

&ensp; 问题可以转化为所有情况 - 不包含任何一个模式串的主串数, 因为需要求N以内的所有数目, 把关系矩阵多开一维并置那列为1, 可以达成要求

&ensp; 关于总情况数, 因为需要取模, 直接等比求和需要算逆元, 这里仿照之前的处理, 多开一列全1计算快速幂

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef unsigned long long LL;

void multi(LL a[][50], LL b[][50])
{
    static LL res_multi[50][50];
    memset(res_multi, 0, sizeof res_multi);

    for( int i = 0; i < 50; i ++)
    {
        for( int j = 0; j < 50; j++ )
        {
            for( int k = 0; k < 50; k++ )
                res_multi[i][j] += a[i][k] *b[k][j];
        }
    }
    memcpy(a, res_multi, sizeof res_multi);
}

LL res_qpow[50][50];
void qpow(LL b[][50], LL index)
{
    static LL base[50][50];

    memset(res_qpow, 0, sizeof res_qpow);
    memcpy(base, b, sizeof base);

    for( int i = 0; i < 50; i++ )
        res_qpow[i][i] = 1;

    while(index)
    {
        if(index &1)
            multi(res_qpow, base);
        index /= 2;
        multi(base, base);
    }
}


struct Node {
    int next[26], fail;
    bool stuck;
}ss[50];

int stkptr = 0;
int newNode()
{
    Node& o = ss[stkptr];
    memset(o.next, -1, sizeof o.next);
    o.fail = -1, o.stuck = 0;
    return stkptr ++;
}

void init()
{
    stkptr = 0;
    newNode();
}
void insert(string &pat)
{
    int p = 0;
    for( int i = 0; i < pat.size(); i++ )
    {
        int k = pat[i] - 'a';
        if( !~ss[p].next[k])
            ss[p].next[k] = newNode();
        p = ss[p].next[k];
    }
    ss[p].stuck = 1;
}

void getFail()
{
    queue<int> que;

    ss[0].fail = 0;
    for( int k = 0; k < 26; k++ )
    {
        int &v = ss[0].next[k];
        if( ~v) {
            ss[v].fail = 0;
            que.push(v);
        } else v = 0;
    }

    while(!que.empty())
    {
        int u = que.front();  que.pop();

        ss[u].stuck |= ss[ss[u].fail].stuck;

        for( int k = 0; k < 26; k++ )
        {
            int &v = ss[u].next[k];
            int &f = ss[u].fail;
            if( ~v) {
                ss[v].fail = ss[f].next[k];
                que.push(v);
            }
            else v = ss[f].next[k];
        }
    }
}

LL M, N;

int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    while(cin >> M >> N)
    {
        init();
        for( int i = 1; i <= M; i++ )
        {
            string pat;  cin >> pat;
            insert(pat);
        }

        getFail();

        static LL m[50][50];
        memset(m, 0, sizeof m);
        const int n = stkptr;

        for( int u = 0; u < n; u++ )
        {
            //if( ss[u].stuck) continue;
            for( int k = 0; k < 26; k++ )
            {
                int v = ss[u].next[k];
                if( ss[v].stuck) continue;
                m[u][v] ++;
            }
        }
        for( int i = 0; i <= n; i++ ) m[i][n] = 1;

        LL minus = 0;
        qpow(m, N);

        for( int i = 0; i <= n; i ++)
            minus += res_qpow[0][i];

        memset(m, 0, sizeof m);
        m[0][0] = 26;
        m[0][1] = m[1][1] = 1;
        qpow(m, N);
        LL ans = res_qpow[0][0] + res_qpow[0][1] - minus;

        cout << ans << endl;
    }

    return 0;
}
```

&nbsp;

***

#### [POJ - 1625 G - Censored!](https://vjudge.net/contest/238819#problem/G)

&ensp; 题目大意: 给出M个模式串和长度为N的主串, 要求主串不得包含任何一个模式串

&ensp; 因为题目不要求取模, 而大数乘法比较慢, 所以转而dp + 大数


```cpp
#include <iostream>
#include <cstdio>
#include <cstring>
#include <vector>
#include <string>
#include <queue>
#include <algorithm>
using namespace std;

vector<char> table;
int getid(char ch) {
    return lower_bound(
        table.begin(), table.end(),
      ch) - table.begin();
}

struct BigInteger {
    static const int BASE=10000; //高进制
    static const int WIDTH=4; //高进制位数
    vector<int> s;
    BigInteger() {}
    BigInteger(long long num) { // 构造函数
        *this=num;
    }
    //赋值
    BigInteger operator = (long long num) {
        s.clear();
        do {
            s.push_back(num%BASE);
            num/=BASE;
        } while(num>0);
        return *this;
    }
    //+
    BigInteger operator + (BigInteger& b) {
        BigInteger c;
        c.s.resize(max(s.size(),b.s.size())+1);
        for(int i=0; i<c.s.size()-1; i++) {
            int tmp1,tmp2;
            if(i>=s.size())tmp1=0;
            else tmp1=s[i];
            if(i>=b.s.size())tmp2=0;
            else tmp2=b.s[i];
            c.s[i]=tmp1+tmp2;
        }
        for(int i=0; i<c.s.size()-1; i++) {
            c.s[i+1]+=c.s[i]/BASE;
            c.s[i]%=BASE;
        }
        while(c.s.back()==0&&c.s.size()>1)c.s.pop_back();
        return c;
    }
    void operator += (BigInteger& b) {
        *this=*this+b;
    }
};
ostream& operator << (ostream& output,const BigInteger& x) {
    output<<x.s.back();
    for(int i=x.s.size()-2; i>=0; i--) {
        char buf[20];
        sprintf(buf,"%04d",x.s[i]);
        int sz = strlen(buf);
        for(int j=0; j<sz; j++)output<<buf[j];
    }
    return output;
}

struct Node {
    int next[50], fail;
    bool stuck;
}ss[200];

int stkptr = 0;
int newNode()
{
    Node& o = ss[stkptr];
    memset(o.next, -1, sizeof o.next);
    o.fail = -1, o.stuck = 0;
    return stkptr ++;
}

void init()
{
    stkptr = 0;
    newNode();
}
void insert(string &pat)
{
    int p = 0;
    for( int i = 0; i < pat.size(); i++ )
    {
        int k = getid(pat[i]);
        if( !~ss[p].next[k])
            ss[p].next[k] = newNode();
        p = ss[p].next[k];
    }
    ss[p].stuck = 1;
}

void getFail()
{
    queue<int> que;

    ss[0].fail = 0;
    for( int k = 0; k < table.size(); k++ )
    {
        int &v = ss[0].next[k];
        if( ~v) {
            ss[v].fail = 0;
            que.push(v);
        } else v = 0;
    }

    while(!que.empty())
    {
        int u = que.front();  que.pop();

        ss[u].stuck |= ss[ss[u].fail].stuck;

        for( int k = 0; k < table.size(); k++ )
        {
            int &v = ss[u].next[k];
            int &f = ss[u].fail;
            if( ~v) {
                ss[v].fail = ss[f].next[k];
                que.push(v);
            }
            else v = ss[f].next[k];
        }
    }
}

int T, N, M;
BigInteger dp[57][300];

int main()
{
    while(~scanf("%d%d%d", &T, &N, &M))
    {
        table.clear();
        for( int i = 1; i <= T; i++ )
        {
            char ch;  scanf(" %c", &ch);
            table.push_back(ch);
        }
        sort(table.begin(), table.end());

        init();
        while( M-- )
        {
            string str;
            cin >> str;
            insert(str);
        }

        getFail();

        const int n = stkptr;

        for( int i = 0; i <= N; i++ ) {
            for( int u = 0; u < n; u++ ) {
                dp[i][u] = 0;
            }
        }
        dp[0][0] = 1;
        for( int i = 1; i <= N; i++ ) {
            for( int u = 0; u < n; u++ ) {
                for( int k = 0; k < table.size(); k++) {
                    int v = ss[u].next[k];
                    if( ss[v].stuck ) continue;
                    dp[i][v] += dp[i-1][u];
                }
            }
        }

        BigInteger ans;
        for( int i = 0; i < n; i++ )
            ans += dp[N][i];
        cout << ans << endl;
    }
    return 0;
}

```

&nbsp;

***

#### [HDU - 2825 Wireless Password](https://vjudge.net/contest/238819#problem/H)

&ensp; 题目大意: 给出N个模式串和主串的长度, 求至少匹配K个模式串的主串有多少个

&ensp; 因为模式串比较少, 所以状压dp来统计匹配K个模式串的情况

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MOD = 20090717;

struct Node {
    int next[26], fail;
    int s;
}ss[200];

int stkptr = 0;
int newNode()
{
    Node& o = ss[stkptr];
    memset(o.next, -1, sizeof o.next);
    o.fail = -1, o.s = 0;
    return stkptr ++;
}

void init()
{
    stkptr = 0;
    newNode();
}
void insert(string &pat, int idx)
{
    int p = 0;
    for( int i = 0; i < pat.size(); i++ )
    {
        int k = pat[i]-'a';
        if( !~ss[p].next[k])
            ss[p].next[k] = newNode();
        p = ss[p].next[k];
    }
    ss[p].s |= (1<<idx);
}

void getFail()
{
    queue<int> que;

    ss[0].fail = 0;
    for( int k = 0; k < 26; k++ )
    {
        int &v = ss[0].next[k];
        if( ~v) {
            ss[v].fail = 0;
            que.push(v);
        } else v = 0;
    }

    while(!que.empty())
    {
        int u = que.front();  que.pop();

        ss[u].s |= ss[ss[u].fail].s;

        for( int k = 0; k < 26; k++ )
        {
            int &v = ss[u].next[k];
            int &f = ss[u].fail;
            if( ~v) {
                ss[v].fail = ss[f].next[k];
                que.push(v);
            }
            else v = ss[f].next[k];
        }
    }
}

int N, M, K;
int dp[27][200][1<<10];

int main()
{
    while(~scanf("%d%d%d", &N, &M, &K), N)
    {
        init();
        for( int i = 0; i < M; i++ )
        {
            string str;  cin >> str;
            insert(str, i);
        }

        getFail();

        const int n = stkptr;

        for( int i = 0; i <= N; i++ )
            for( int u = 0; u < n; u++ )
                for( int s = 0; s < (1<<M); s++ )
                    dp[i][u][s] = 0;

        dp[0][0][0] = 1;
        for(int i = 1; i <= N; i++ ) {
            for( int u = 0; u < n; u++ ) {
                for( int s = 0; s < (1<<M); s++ )
                {
                    if( !dp[i-1][u][s]) continue;   //prevent TLE
                    for( int k = 0; k < 26; k++ )
                    {
                        int v = ss[u].next[k];
                        (dp[i][v][s |ss[v].s] +=
                            dp[i-1][u][s] ) %=MOD;
                    }
                }
            }
        }

        int ans = 0;
        for( int s = 0; s < (1<<M); s++)
        {
            int cnt = 0;
            for( int k = 0; k < M; k++ )
                if( s &(1 <<k)) cnt ++;
            if( cnt < K) continue;

            for( int i = 0; i < n; i++ )
                (ans += dp[N][i][s]) %=MOD;
        }

        printf("%d\n", ans);
    }
    return 0;
}
```

&nbsp;

***


#### [HDU - 2457 DNA repair](https://vjudge.net/contest/238819#problem/J)

@[AC自动机, dp]

&ensp; 题目大意: 给出N个模式串和一个主串, 要求修改最少的次数使得主串不包含任何一个模式串

&ensp; dp[i][j]表示对于字符串第i位, 在自动机上的第j个状态来说, 主串所需要的最少修改, 所以当dp所形成的字符串与主串对应位置字符不同时, dp+1

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Node {
    int next[4], fail;
    bool stuck;
}ss[1007];

int getid(char ch) {
    if( ch == 'A') return 0;
    if( ch == 'T') return 1;
    if( ch == 'C') return 2;
    if( ch == 'G') return 3;
}

int stkptr = 0;
int newNode()
{
    Node& o = ss[stkptr];
    memset(o.next, -1, sizeof o.next);
    o.fail = -1, o.stuck = 0;
    return stkptr ++;
}

void init()
{
    stkptr = 0;
    newNode();
}
void insert(string &pat, int id)
{
    int p = 0;
    for( int i = 0; i < pat.size(); i++ )
    {
        int k = getid(pat[i]);
        if( !~ss[p].next[k])
            ss[p].next[k] = newNode();
        p = ss[p].next[k];
    }
    ss[p].stuck = 1;
}

void getFail()
{
    queue<int> que;

    ss[0].fail = 0;
    for( int k = 0; k < 4; k++ )
    {
        int &v = ss[0].next[k];
        if( ~v) {
            ss[v].fail = 0;
            que.push(v);
        } else v = 0;
    }

    while(!que.empty())
    {
        int u = que.front();  que.pop();

        ss[u].stuck |= ss[ss[u].fail].stuck;

        for( int k = 0; k < 4; k++ )
        {
            int &v = ss[u].next[k];
            int &f = ss[u].fail;
            if( ~v) {
                ss[v].fail = ss[f].next[k];
                que.push(v);
            }
            else v = ss[f].next[k];
        }
    }
}

int M;
int dp[1007][1007];

int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    int Case = 0;
    while(cin >> M, M)
    {
        init();
        for( int i = 1; i <= M; i ++)
        {
            string pat;  cin >> pat;
            insert(pat, i);
        }

        getFail();

        cout << "Case " << ++ Case << ": ";

        string str;  cin >> str;

        const int n = stkptr;
        const int N = str.size();

        for( int i = 0; i <= N; i ++ ) {
            for( int u = 0; u < n; u++ ) {
                dp[i][u] = 1 << 30;
            }
        }

        dp[0][0] = 0;
        for( int i = 1; i <= N; i++ ) {
            for( int u = 0; u < n; u++ ) {
                for( int k = 0; k < 4; k++ )
                {
                    int v = ss[u].next[k];
                    if( ss[v].stuck ) continue;

                    dp[i][v] = min(dp[i][v],
                        dp[i-1][u] + (getid(str[i-1])!=k)
                    );
                }
            }
        }

        int ans = 1 << 30;
        for( int u = 0; u < n; u++ )
            ans = min(ans, dp[N][u]);
        cout << (ans == 1 << 30? -1: ans) << endl;
    }
    return 0;
}

```

