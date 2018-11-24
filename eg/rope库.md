### rope库
@[stl]

> - head: `<ext/rope>`
> - namespace: `__gnu_cxx` 

&ensp; *内部实现: 基于块状链表*, 复杂度O($\sqrt{n}$)

&nbsp;

#### 成员函数
函数 | 说明
---| ---
push_back(Type elem) | 尾部添加
append(Type* arr, int len) / append(Type elem)  | 尾部添加
erase(int pos, int len) | 删除
insert(int pos, Type elem) | 在pos光标**前**插入
replace(int pos, Type elem) | 替换
substr(int pos, int len) | 子序列
operator[] | 索引
operator+ | 拼接

&nbsp;

注意点: 
- *rope不能直接由索引[]修改值, 需要通过replace*
- **rope一个个append或push_back, 以及replace会多消耗掉不少额外空间和时间**

****
#### 字符串处理
```cpp
#include <bits/stdc++.h>
#include <ext/rope>
using namespace std;
using namespace __gnu_cxx;

int main()
{
    rope<char> str;
    str.append("python"+4); //on
    str.erase(0);  //n
    str.insert(10, "3");  //n3
    cout << str << endl;
    return 0;
}
```

&ensp; 

#### 可持久化平衡树

&ensp; 因为rope对象之间copy速度很快, 于是可以很方便实现可持久化, 并可以实现可持久化平衡树的基本职能


##### [BZOJ 3674 可持久化并查集加强版](https://www.lydsy.com/JudgeOnline/problem.php?id=3674)

&ensp; 题目大意: 强制在线, 可持久化并查集

```cpp
#include <bits/stdc++.h>
#include <ext/rope>
using namespace std;
using namespace __gnu_cxx;
const int MAXN = 2e5+10;

int N, Q;
vector<rope<int> > pre(1);

int find(int i)
{
    if( i == pre.back()[i] ) return i;

    int anc = find(pre.back()[i]);
    if( pre.back().at(i) == anc ) return anc;

    pre.back().replace(i, anc);   // replace消耗空间比较严重
    return anc;
}
void merge(int a, int b)
{
    a = find(a), b = find(b);
    if( a != b ) pre.back().replace(b, a);
}

int main()
{
    ios::sync_with_stdio(0);
    cin.tie(0), cout.tie(0);

    cin >> N >> Q;
    for( int i = 0; i <= N; i++ ) //最好先装临时数组 再初始化rope
        pre[0].push_back(i);  //push_back 也很消耗空间

    int ans = 0;
    for( int i = 1; i <= Q; i++)
    {
        int op;  cin >> op;
        if( op == 1 )
        {
            pre.push_back(pre.back());
            int a, b;  cin >> a >> b;
            merge(a ^ans, b ^ans);
        }
        else if( op == 2 )
        {
            int k;  cin >> k;
            pre.push_back(pre[k ^ans]);
        }
        else
        {
            pre.push_back(pre.back());
            int a, b;  cin >> a >> b;
            a ^= ans, b ^= ans;
            cout << (ans = find(a)==find(b)? 1: 0) << endl;
        }
    }

    return 0;
}
```