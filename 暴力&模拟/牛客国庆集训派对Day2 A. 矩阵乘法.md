#### [牛客国庆集训派对Day2 A. 矩阵乘法](https://www.nowcoder.com/acm/contest/202/A)

@[分块, 状压]

&ensp; 题目大意: 给出一个十六进制A矩阵和二进制B矩阵, 两个矩阵相乘求结果矩阵所有元素的异或和

&ensp; A矩阵规格为4096*64, B矩阵规格为64*4096

&ensp; 因为矩阵的维度比较大, 直接4096*64*4096可能会超时, 将A按列分块, B按行分块, 因为分块后二进制矩阵B每块中每列一共只有0-255种情况, 预处理A矩阵对应块中的元素和这256情况的结果, 复杂度变为4096*256*8, 即分为8块, 每块预处理为4096*256

&ensp; 随后将B矩阵每列每块对应的状态查找出相应的结果, 并计入最后答案

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;

int N, P, M;
int A[4100][70], B[70][4100];
int Abk[4100][10][260]; //ans of blocks by status
int Bs[10][4100];   //status of B
LL res[4100][4100];

int main()
{
    cin >> N >> P >> M;

    for( int i = 0; i < N; i++ )
        for( int j = 0; j < P; j++ )
            scanf("%x", &A[i][j]);

    for( int j = 0; j < M; j++ )
    {
        string str;  cin >> str;
        for( int i = 0; i < P; i++ )
            B[i][j] = str[i]-'0';
    }

    int blocks = (P-1)/8+1, sz = P /blocks;

    for( int i = 0; i < N; i++ )
    {
        for( int s = 0; s < (1<<sz); s++)
        {
            for( int j = 0; j < blocks; j++ )
            {
                for( int k = 0; k < sz; k++ )
                    if( s &(1 <<(sz-1-k))) {
                        Abk[i][j][s] += A[i][j*sz+k];
                    }
            }
        }
    }

    for( int j = 0; j < M; j++ ) {
        for( int i = 0; i < blocks; i++ ) {
            for( int k = 0; k < sz; k++ ) {
                Bs[i][j] += B[i*sz+k][j] <<(sz-k-1);
            }
        }
    }

    for( int k = 0; k < blocks; k++ ) {
        for( int i = 0; i < N; i++ ) {
            for( int j = 0; j < M; j++ ) {
                res[i][j] += Abk[i][k][Bs[k][j]];
            }
        }
    }

    LL ans = 0;
    for( int i = 0; i < N; i++ ) {
        for( int j = 0; j < M; j++ ) {
            ans ^= res[i][j];
        }
    }
    cout << ans << endl;

    return 0;
}

```