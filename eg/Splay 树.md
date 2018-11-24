###  Splay 树

@[数据结构, 模板]

&ensp; splay 是一种平衡二叉树, 节点旋转操作和平衡树相同
&ensp; 特点: 将查询到的节点旋转到树根下, 均摊下的复杂度也和平衡树相同

#### 区间处理

&ensp; splay 是一棵区间树, 若需要处理[l, r]区间, 则将 l-1 旋转到树根, 并将 r+1 旋转到根的右子树上, 则根右子树的左子树就是需要处理的区间

&ensp; 因为每次将访问到的节点旋转到树根下, 由此来更新路径上的节点信息


#### 模板

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
#define Type int
const int MAXN = 1e5+10;
const int INF = 1 << 30;

template <typename T>
struct Node{
    static Node* null;
    bool isNull() const { return this == Node::null; }

    T val;  int sz, scale, cnt;
    Node* next[2], *fa;

    Node(T _val=0, int _sz=1, int _scale=1, int _cnt=1):
        val(_val), sz(_sz), scale(_scale), cnt(_cnt)
    { next[0] = next[1] = fa = null; }
};
template <> Node<Type>* Node<Type>::null = new Node<Type>(Type(), 0, 0, 0);

template <typename T>
class Splay
{
    void release(Node<T>* p)
    {
        if( p->isNull()) return ;
        release(p->next[0]);
        release(p->next[1]);
        delete p;
    }
    void init()
    {
        root = new Node<T>(-INF);
        Node<T>* t = new Node<T>(INF);
        root->next[1] = t, t->fa = root;
        pushup(root);
    }

    bool which(const Node<T>* p) const { return p == p->fa->next[1]; }
    void rotate(Node<T>* p)
    {
        Node<T>* pf = p->fa, *pg = pf->fa;
        if( pf->isNull() || p->isNull() ) return ;

        bool wh = which(p);
        pf->next[wh] = p->next[wh ^1];
        pf->next[wh]->fa = pf;
        p->next[wh ^1] = pf;
        pf->fa = p;  p->fa = pg;

        if( pg) pg->next[pf == pg->next[1]] = p;
        pushup(pf), pushup(p);
    }
    void splay(Node<T>* p, Node<T> *tgt)
    {
        if( p == tgt ) return ;
        for( Node<T>* q; (q = p->fa) != tgt; rotate(p))
            if( q->fa != tgt ) rotate(which(p) == which(q) ?q :p);
        if( tgt->isNull()) this->root = p;
    }

    Node<T>* lower_bound(Node<T>* p, const T& val)
    {
        if( p->isNull() || p->val == val) return p;
        Node<T> *t = lower_bound(p->next[val > p->val], val);
        return val < p->val && t->isNull()? p: t;
    }

protected:
    Node<T>* root;  int tot;

    Node<T>* getNode(int k)
    {
        Node<T> *p = root;
        while(1)
        {
            pushdown(p);

            if( p->next[0]->scale >= k )
                p = p->next[0];
            else
            {
                k -= p->next[0]->scale + p->cnt;
                if( k <= 0 ) return p;
                p = p->next[1];
            }
        }
    }
    pair<Node<T>*, int> getKth(int val)
    {
        Node<T> *p = root;  int ret = 1;
        while(1)
        {
            if( val >= p->val ) ret += p->next[0]->scale + p->cnt;
            if( val == p->val ) return make_pair(p, ret-p->cnt);
            if( p->next[val > p->val]->isNull()) return make_pair(p, ret);

            p = p->next[val > p->val];
        }
    }

public:
    Splay() :tot(0) { init(); }
    ~Splay() { release(root); }

    Node<T>* getRoot()const { return this->root; }
    void clear() { release(root), init(); }
    virtual void pushdown(Node<T>* p) {}
    virtual void pushup(Node<T>* p)
    {
        p->sz = p->next[0]->sz +
                p->next[1]->sz +1;
        p->scale = p->next[0]->scale +
                p->next[1]->scale + p->cnt;
    }

    Node<T>* lower_bound(const T& val)
    {
        Node<T> *p = lower_bound(getRoot(), val);
        splay(p, getRoot());
        return p;
    }

    T find_by_order(int k)
    {
        if( !tot || !k ) return 0;
        k = k > tot? tot: k;
        Node<T>* p = getNode(k+1);
        splay(p, getRoot());
        return p->val;
    }
    int order_by_tag(const T& val)
    {
        pair<Node<T>*, int> res = getKth(val);
        splay(res.first, getRoot());
        return res.second - 1;
    }
    const Node<T>* find(const T& val)
    {
        Node<T> *p = root;
        while(1)
        {
            if( p->val == val) { splay(p, root); return p; }
            Node<T> *t = p->next[val > p->val];
            if( t ->isNull() ) return t;
            else p = t;
        }
    }

    void insert(const T& val)
    {
        tot ++;
        Node<T>* p = lower_bound(getRoot(), val);
        if( p->val == val )
        {
            p->scale ++, p->cnt ++;
            splay(p, getRoot());
        }
        else
        {
            Node<T>* t = p->next[0];
            p->next[0] = new Node<T>(val);
            p->next[0]->fa = p;
            p->next[0]->next[0] = t;
            t->fa = p->next[0];
            pushup(p->next[0]);
            splay(p->next[0], getRoot());
        }
        return pushup(getRoot());
    }
    void erase(const T& val)
    {
        Node<T>* p = lower_bound(getRoot(), val);
        if( p->val != val ) return ;

        if( p->cnt > 1)
        {
            p->cnt--, p->scale--;
            splay(p, getRoot());
            return pushup(getRoot());
        }
        else
        {
            int k = getKth(val).second;
            splay(getNode(k-1), Node<T>::null);
            splay(getNode(k+1), getRoot());
            getRoot()->next[1]->next[0] = Node<T>::null;
            pushup(getRoot()->next[1]), pushup(getRoot());
            return delete p;
        }
    }
};

///////////////////////

void display(Node<int> *p)
{
    if( p->isNull()) return ;
    display(p->next[0]);
    cout << p->val << endl;
    display(p->next[1]);
}

bool debug(Node<int> *p)
{
    if( p->isNull()) return 1;
    if(p->scale != p->next[0]->scale + p->next[1]->scale +p->cnt )
        return 0;
    if(!debug(p->next[0]) || !debug(p->next[1]) ) return 0;
    return 1;
}

int M;

int main()
{
    //freopen("in", "r", stdin);

    ios::sync_with_stdio(0);
    cin.tie(0);

    cin >> M;
    Splay<int> t;

    while(M--)
    {
        int op, x;
        cin >> op >> x;
        if( op == 1) t.insert(x);
        else if( op == 2 ) t.erase(x);
        else if( op == 3 ) cout << t.order_by_tag(x) << endl;
        else if( op == 4 ) cout << t.find_by_order(x) << endl;
        else if( op == 5 )
        {
            int k = t.order_by_tag(x);
            cout << t.find_by_order(k-1) << endl;
        }
        else
        {/*
            int k = t.order_by_tag(x);
            if( !t.find(x)->isNull()) k ++;
            cout << t.find_by_order(k) << endl;*/
            int k = t.order_by_tag(x);
            if( !t.find(x)->isNull()) k ++;
            int ans = 0;
            while( (ans = t.find_by_order(k) ) == x ) k++;
            cout << ans << endl;
        }
    }
    return 0;
}

```