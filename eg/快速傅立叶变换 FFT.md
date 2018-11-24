### 快速傅立叶变换 FFT

#### 卷积

**离散定义式:**
$$(f \circ g)(n) = \sum f(\tau)g(n-\tau)$$

##### 应用
- **多项式乘积系数**  --> 大数乘法
- **$a_i + a_j  = a_k $ 计数**

&nbsp;

#### 模板
```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
const int MAXN= 2e5+10;
const double PI = acos(-1.0);

struct Complex
{
    double x,y;
    inline Complex operator +(const Complex b)const {return (Complex){x+b.x,y+b.y};}
    inline Complex operator -(const Complex b)const {return (Complex){x-b.x,y-b.y};}
    inline Complex operator *(const Complex b)const {return (Complex){x*b.x-y*b.y,x*b.y+y*b.x};}
}va[MAXN*2+MAXN/2],vb[MAXN*2+MAXN/2];

int lenth=1, rev[MAXN*2+MAXN/2];
int N, M;    //f和g的数量
int f[MAXN], g[MAXN];   //f和g的系数
vector<LL> conv;    //卷积结果
vector<LL> multi;  //大数乘积

void init()
{
    int tim=0; lenth = 1;
    conv.clear(), multi.clear();
    memset( va , 0 , sizeof va);
    memset( vb , 0 , sizeof vb);

    while( lenth <= N+M-2 ) lenth<<=1,tim++;
    for( int i=0;i<lenth;i++)
        rev[i]=(rev[i>>1]>>1)+((i&1)<<(tim-1));
}

void FFT(Complex*A,const int fla)
{
    for( int i=0;i<lenth;i++)
    {
        if(i<rev[i])
        {
            swap(A[i],A[rev[i]]);
        }
    }

    for( int i=1;i<lenth;i<<=1)
    {
        const Complex w = (Complex){cos(PI/i),fla*sin(PI/i)};
        for( int j=0;j<lenth;j+=(i<<1))
        {
            Complex K=(Complex){1,0};
            for( int k=0;k<i;k++,K=K*w)
            {
                const Complex x=A[j+k],y=K*A[j+k+i];
                A[j+k]=x+y;
                A[j+k+i]=x-y;
            }
        }
    }
}

void getConv()
{
    init();
    for( int i = 0 ; i < N; i++ ) 
        va[i].x = f[i];
    for( int i = 0 ; i < M ; i++) 
        vb[i].x = g[i];

    FFT(va,1),FFT(vb,1);

    for( int i=0;i<lenth;i++)
        va[i]=va[i]*vb[i];
    FFT(va,-1);

    for( int i = 0; i <= N+M-2 ; i++)
        conv.push_back((LL)(va[i].x/lenth+0.5));
}


void getMulti()
{
    getConv();

    multi = conv;
    reverse(multi.begin(), multi.end());
    multi.push_back(0);
    for( int i = 0; i < multi.size()-1 ; i++)
    {
        multi[i+1] += multi[i]/10;
        multi[i] %= 10;
    }

    while(!multi.back() && multi.size() > 1)
        multi.pop_back();
    reverse(multi.begin(), multi.end());
}

//事先需要设置系数f和g和数组大小N和M
//卷积结果保存conv, 乘法结果保存multi

```
&nbsp;

***
[HDU 1402 A * B Problem Plus](http://acm.hdu.edu.cn/showproblem.php?pid=1402)
&ensp; 题目大意: 求大数乘积 a*b 的值
&ensp; 由离散形式卷积定义,  可得 $f(x) \cdot g(x)$ 的系数, 令 x = 10 即大数$f(10)$与$g(10)$乘积的结果。
```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
const int MAXN= 1e5+10;
const double PI = acos(-1.0);

struct Complex
{
    double x,y;
    inline Complex operator +(const Complex b)const {return (Complex){x+b.x,y+b.y};}
    inline Complex operator -(const Complex b)const {return (Complex){x-b.x,y-b.y};}
    inline Complex operator *(const Complex b)const {return (Complex){x*b.x-y*b.y,x*b.y+y*b.x};}
}va[MAXN*2+MAXN/2],vb[MAXN*2+MAXN/2];

int lenth=1, rev[MAXN*2+MAXN/2];
int N, M;    //f和g的数量
int f[MAXN], g[MAXN];   //f和g的系数
vector<int> conv;    //卷积结果
vector<int> multi;  //大数乘积

void init()
{
    int tim=0; lenth = 1;
    conv.clear(), multi.clear();
    memset( va , 0 , sizeof va);
    memset( vb , 0 , sizeof vb);

    while( lenth <= N+M-2 ) lenth<<=1,tim++;
    for( int i=0;i<lenth;i++)
        rev[i]=(rev[i>>1]>>1)+((i&1)<<(tim-1));
}

void FFT(Complex*A,const int fla)
{
    for( int i=0;i<lenth;i++)
    {
        if(i<rev[i])
        {
            swap(A[i],A[rev[i]]);
        }
    }

    for( int i=1;i<lenth;i<<=1)
    {
        const Complex w = (Complex){cos(PI/i),fla*sin(PI/i)};
        for( int j=0;j<lenth;j+=(i<<1))
        {
            Complex K=(Complex){1,0};
            for( int k=0;k<i;k++,K=K*w)
            {
                const Complex x=A[j+k],y=K*A[j+k+i];
                A[j+k]=x+y;
                A[j+k+i]=x-y;
            }
        }
    }
}

void getConv()
{
    init();
    for( int i = 0 ; i < N; i++ ) 
        va[i].x = f[i];
    for( int i = 0 ; i < M ; i++) 
        vb[i].x = g[i];

    FFT(va,1),FFT(vb,1);

    for( int i=0;i<lenth;i++)
        va[i]=va[i]*vb[i];
    FFT(va,-1);

    for( int i = 0; i <= N+M-2 ; i++)
        conv.push_back((LL)(va[i].x/lenth+0.5));
}


void getMulti()
{
    getConv();

    multi = conv;
    reverse(multi.begin(), multi.end());
    multi.push_back(0);
    for( int i = 0; i < multi.size()-1 ; i++)
    {
        multi[i+1] += multi[i]/10;
        multi[i] %= 10;
    }

    while(!multi.back() && multi.size() > 1)
        multi.pop_back();
    reverse(multi.begin(), multi.end());
}

//事先需要设置系数f和g和数组大小N和M
//卷积结果保存conv, 乘法结果保存multi

char s1[MAXN], s2[MAXN];
int main()
{
    while(~scanf("%s%s", s1, s2))
    {
        N = strlen(s1) , M = strlen(s2);
        for( int i = 0 ; i < N ; i++ )
            f[i] = s1[i] - '0';
        for( int i = 0 ; i < M ; i++ )
            g[i] = s2[i] - '0';
        
        getMulti();
        for( int i = 0 ; i < multi.size() ; i++)
            printf("%d", multi[i]);
        puts("");
    }
    return 0;
}
```


&nbsp;

[2016 acm香港网络赛 A题 A+B Problem ](https://open.kattis.com/problems/aplusb)
&ensp; 给出一个序列, 求满足 $a_i + a_j  = a_k \ (i \not =j \not = k )$的情况

&ensp; 记 $num(x)$ 为大小为x在序列中出现的次数。
$(num \circ num)(a+b)=\sum num(a)num(b)$,  此时卷积的意义即两条边相加大小为a+b的有多少种。
&ensp; 需要另外扣除情况：
- $a[i] + a[i] = a[k]$
- $a[i] + 0 = a[i]  \ 或 \  0 + a[i] = a[i]$
```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
const int MAXN= 2e5+10;
const double PI = acos(-1.0);

struct Complex
{
    double x,y;
    inline Complex operator +(const Complex b)const {return (Complex){x+b.x,y+b.y};}
    inline Complex operator -(const Complex b)const {return (Complex){x-b.x,y-b.y};}
    inline Complex operator *(const Complex b)const {return (Complex){x*b.x-y*b.y,x*b.y+y*b.x};}
}va[MAXN*2+MAXN/2],vb[MAXN*2+MAXN/2];

int lenth=1, rev[MAXN*2+MAXN/2];
int N, M;    //f和g的数量
int f[MAXN], g[MAXN];   //f和g的系数
vector<LL> conv;    //卷积结果
vector<LL> multi;  //大数乘积

void init()
{
    int tim=0; lenth = 1;
    conv.clear(), multi.clear();
    memset( va , 0 , sizeof va);
    memset( vb , 0 , sizeof vb);

    while( lenth <= N+M-2 ) lenth<<=1,tim++;
    for( int i=0;i<lenth;i++)
        rev[i]=(rev[i>>1]>>1)+((i&1)<<(tim-1));
}

void FFT(Complex*A,const int fla)
{
    for( int i=0;i<lenth;i++)
    {
        if(i<rev[i])
        {
            swap(A[i],A[rev[i]]);
        }
    }

    for( int i=1;i<lenth;i<<=1)
    {
        const Complex w = (Complex){cos(PI/i),fla*sin(PI/i)};
        for( int j=0;j<lenth;j+=(i<<1))
        {
            Complex K=(Complex){1,0};
            for( int k=0;k<i;k++,K=K*w)
            {
                const Complex x=A[j+k],y=K*A[j+k+i];
                A[j+k]=x+y;
                A[j+k+i]=x-y;
            }
        }
    }
}

void getConv()
{
    init();
    for( int i = 0 ; i < N; i++ ) 
        va[i].x = f[i];
    for( int i = 0 ; i < M ; i++) 
        vb[i].x = g[i];

    FFT(va,1),FFT(vb,1);

    for( int i=0;i<lenth;i++)
        va[i]=va[i]*vb[i];
    FFT(va,-1);

    for( int i = 0; i <= N+M-2 ; i++)
        conv.push_back((LL)(va[i].x/lenth+0.5));
}


void getMulti()
{
    getConv();

    multi = conv;
    reverse(multi.begin(), multi.end());
    multi.push_back(0);
    for( int i = 0; i < multi.size()-1 ; i++)
    {
        multi[i+1] += multi[i]/10;
        multi[i] %= 10;
    }

    while(!multi.back() && multi.size() > 1)
        multi.pop_back();
    reverse(multi.begin(), multi.end());
}

//事先需要设置系数f和g和数组大小N和M
//卷积结果保存conv, 乘法结果保存multi

const int HASH = 5e4;
int n, a[MAXN], num[MAXN];

int main()
{
    while(~scanf("%d", &n))
    {
        int cnt0 = 0;
        for( int i = 0 ; i < n ; i++ )
        {
            scanf("%d", a+i);
            num[a[i]+HASH]++;
            if( !a[i] ) cnt0++;
        }

        N = M = HASH*2+1;
        for( int i = 0 ; i < N ; i++ )
            f[i] = g[i] = num[i];
        getConv();

        //扣除自己和自己相加
        for( int i = 0 ; i < n ; i++ )
            conv[(a[i]+HASH)*2]--;
        
        LL ans = 0;
        for( int i = 0 ; i < n ; i++ )
        {
            ans += conv[a[i]+HASH*2];
            if( a[i]) ans -= cnt0*2; //a[i]+0 或 0+a[i]
            else ans -= (cnt0-1)*2;  //0+0
        }
    
        printf("%lld\n", ans);

    }
    return 0;
}
```

&nbsp;

[HDU 4609 3-idiots](http://acm.hdu.edu.cn/showproblem.php?pid=4609)
&ensp; 题目大意: 给出一个序列, 任选三条边, 求能组成一个三角形的概率

&ensp; num意义同上题；同时三角形三边满足: $a+b>c$。
&ensp; 枚举最长边c 的情况：
- 加上所有大于这个数c 的卷积(前缀和)
- 扣除两条边一条 a > c,  一条 b < c 的情况
- 扣除两条边a > c 且 b > c 的情况
- 扣除有一条边 a = c 的情况

因为是求概率, 所以最后情况总数/ C(n, 3)
```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
const int MAXN= 2e5+10;
const double PI = acos(-1.0);

struct Complex
{
    double x,y;
    inline Complex operator +(const Complex b)const {return (Complex){x+b.x,y+b.y};}
    inline Complex operator -(const Complex b)const {return (Complex){x-b.x,y-b.y};}
    inline Complex operator *(const Complex b)const {return (Complex){x*b.x-y*b.y,x*b.y+y*b.x};}
}va[MAXN*2+MAXN/2],vb[MAXN*2+MAXN/2];

int lenth=1, rev[MAXN*2+MAXN/2];
int N, M;    //f和g的数量
int f[MAXN], g[MAXN];   //f和g的系数
vector<LL> conv;    //卷积结果
vector<LL> multi;  //大数乘积

void init()
{
    int tim=0; lenth = 1;
    conv.clear(), multi.clear();
    memset( va , 0 , sizeof va);
    memset( vb , 0 , sizeof vb);

    while( lenth <= N+M-2 ) lenth<<=1,tim++;
    for( int i=0;i<lenth;i++)
        rev[i]=(rev[i>>1]>>1)+((i&1)<<(tim-1));
}

void FFT(Complex*A,const int fla)
{
    for( int i=0;i<lenth;i++)
    {
        if(i<rev[i])
        {
            swap(A[i],A[rev[i]]);
        }
    }

    for( int i=1;i<lenth;i<<=1)
    {
        const Complex w = (Complex){cos(PI/i),fla*sin(PI/i)};
        for( int j=0;j<lenth;j+=(i<<1))
        {
            Complex K=(Complex){1,0};
            for( int k=0;k<i;k++,K=K*w)
            {
                const Complex x=A[j+k],y=K*A[j+k+i];
                A[j+k]=x+y;
                A[j+k+i]=x-y;
            }
        }
    }
}

void getConv()
{
    init();
    for( int i = 0 ; i < N; i++ ) 
        va[i].x = f[i];
    for( int i = 0 ; i < M ; i++) 
        vb[i].x = g[i];

    FFT(va,1),FFT(vb,1);

    for( int i=0;i<lenth;i++)
        va[i]=va[i]*vb[i];
    FFT(va,-1);

    for( int i = 0; i <= N+M-2 ; i++)
        conv.push_back((LL)(va[i].x/lenth+0.5));
}


void getMulti()
{
    getConv();

    multi = conv;
    reverse(multi.begin(), multi.end());
    multi.push_back(0);
    for( int i = 0; i < multi.size()-1 ; i++)
    {
        multi[i+1] += multi[i]/10;
        multi[i] %= 10;
    }

    while(!multi.back() && multi.size() > 1)
        multi.pop_back();
    reverse(multi.begin(), multi.end());
}

//事先需要设置系数f和g和数组大小N和M
//卷积结果保存conv, 乘法结果保存multi

int num[MAXN], a[MAXN];
LL sum[MAXN];

int main()
{
    int T;  cin >> T;
    while(T--)
    {
        memset(num, 0 , sizeof num);
        memset(sum , 0 , sizeof sum);
        int n;  scanf("%d", &n);
        int Max = 0;
        for( int i = 0 ; i < n ; i++)
        {
            scanf("%d", a+i);
            num[a[i]]++, Max = max(Max, a[i]);
        }
        sort(a, a+n);

        N = M = Max+1;  //共有Max+1个数
        for( int i = 0 ; i <= Max ; i++)
            f[i] = g[i] = num[i];
        Max *= 2;   //最大的两个数相加的结果

        getConv();
        //减去自己和自己的组合
        for( int i = 0 ; i < n ; i++)
            conv[a[i]*2]--;
        //选择无序
        for( int i = 0 ; i <= Max ; i++)
            conv[i] /= 2;

        sum[0] = conv[0];
        for( int i = 1 ; i <= Max ; i++ )
            sum[i] = sum[i-1] + conv[i];

        LL ans = 0;
        //枚举最长边a[i]
        for( int i = 0 ; i < n ; i++)
        {
            ans += sum[Max] - sum[a[i]];    //两个边长度和>a[i]
            ans -= (LL) (n-1-i)*i;  //去除比a[i]大的和比a[i]小的
            ans -= (LL) (n-1-i)*(n-1-i-1)/2; //去除两个都比a[i]大的
            ans -= n-1; //去除选择自己的
        }
    
        printf("%.7f\n", (double)ans/((LL)n*(n-1)*(n-2)/6));
        //C(n,3)=n*(n-1)*(n-2)/6 为所有情况
    }
    return 0;
}
```