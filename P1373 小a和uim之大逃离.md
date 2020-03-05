[传送门 Teleport: 1373 Lil-A and uim: Great Escape](https://www.luogu.com.cn/problem/P1373)

I don't know why I'm translating the problem titles every time; it just feel cringy but anyways.

题目意思花里胡哨的，其实就是：n*m的矩阵，每格的数的范围是0..k。只能往右和下走。一条合法的路径是从任意一格开始，到任意一格结束，长度为偶数，
并且奇数步的和%(k+1)等于偶数步的和%(k+1).求所有合法路径的数量%1 000 000 007. n,m <= 800. k <= 15.

The problem description is a bit confusing. Actually it goes roughly as follows: given an n*m matrix, the value of each block is within 0..k.
You can only go towards the right or bottom. A legal path starts from an arbitrary block and ends at an arbitrary block and is of even-number length.
Its odd-number step sum % (k+1) equals even-number step sum % (k+1). Find the number of all legal paths % 1 000 000 007. n,m <= 800. k <= 15.

要求是从任何一点出发，任何一点结束，所以简单的方格取数DP并不能解决。如果考虑f(P1, P2, w)表示[从点P1到点P2的奇数步和偶数步差值为w的方法数]的话，那就要O((NM)^2 *K)并不可以。
但是记录奇偶的差值的想法应该没问题：奇数步格子的和-偶数步格子的和=0就说明符合要求。然后想到，试试只存一个点，看看能不能递推？于是想到f(P,w)表示[任何一点开始到点P的差值为w的方法数]，便可以直接从头递推到尾。

每个格子都贡献f(P,0)到答案。但是要考虑一个细节：走奇数步可不可以？题目要求是偶数步，但这样可能会把[到这一格走了奇数步，并且差值为0]的情况算进去。

所以要加一个条件：f(P,w,0/1)。0表示目前走了偶数步，1表示奇数步。那么f(P,v[P],1)就要天然+1，因为一定存在一种情况：第一步从这格开始，目前差值就是这格的值。

The requirement is to start from any point and ending from any point, so the simple DP for picking numbers wouldn't do (you prolly know about that simple problem). 
If you consider f(P1, P2, w) to be [# of ways to start from block P1 and end at block P2 with the difference between odd and even steps being w],
then it goes O((NM)^2 *K) which wouldn't work. But the idea to keep record of the difference between odd and even steps should be useful: 
to satisfy the requirement, let the sum of odd-number step blocks - sum of even-number step blocks = 0. Then I thought, try saving only one point, and see if we could do it iteratively?
So I thought of f(P,w) denoting [# of ways from any block to block P with the difference being w, so you can iterate from the beginning to the end.

Each block contributes f(P,0) to the answer. But one detail should be taken into account: can you do it with an odd number of steps? The problem requires the path should be an odd number of steps, 
but this way you could count in cases where [it takes an odd number of steps to reach this block, and the difference is 0].

So another condition added: f(P,w,0/1). 0 denotes that it's taken an even number of steps so far, 1 denotes odd. 
Then f(P,v[P],1) gets +1 automatically, since there must be a case where the first step starts on this block and the difference until now is the value of this block.

由此写出状态转移方程：

Following the idea I had the state-transition formula:

f(P, w, 0) = f(P->up,(w-p+k)%k,1) + f(P->left,(w-p+k)%k,1) 

f(P, w, 1) = f(P->up,(w-p+k)%k,0) + f(P->left,(w-p+k)%k,0) + (1 if w == v[P]).

然而是错的。事实上数值范围是0..k，所以应该对k+1取余。

然而改了以后还是错的。推前一格的值的时候，奇数位是减，偶数位是+

我又因为各种原因交了几次，发现的错误有（见代码）：每次更新f都要取余，不能ans再取余。因为可能出现f加了两次接近2*10^10，ans再加就会超出int32.

如果太阔绰，数组把800开到1000，就会MLE。

还有一个细节：理论上差值可能是负的，但是不用管它，也不用为了它强行把所有差值都+(k+1）来满足数组下标（不能是负数）。因为比如差值-2和差值15其实是一样(k+1=17)


However, it was wrong. In fact the range of the values is 0..k, so I should have taken remainder by (k+1).

However, it was still wrong after I corrected it. When inferring the value for the previous block, do - on odd-number blocks and + on even-number blocks.

I submitted another few times for various reasons, and found bugs including (see code): take remainder each time you update f, instead of do it on ans. Since it's possible f gets added two times and nears 10^7, and ans adds up to overflowing int32.

If you're too generous and allocate the arrays at 1000 for 800, you get MLE.

Another detail: theoretically the difference could be negative, but you don't have to care, or add (k+1) to all difference values for legal array indices (can't be negative).
Because e.g. difference = -2 is the same as difference = 15 (k+1 being 17).

代码长这样：
Code looks like this:
```C++
#include <iostream>
#include <cstring>
#include <algorithm>
#include <cstdio>
const int MOD = 1000000007;
using namespace std;
int n, m, k;
int v[801][801], f[801][801][20][2], ans = 0;
void dp(){
	//f(P, w, 1) = f(P-up, (w+k-v[P])%k, 0) + f(P-left, (w+k-v[P]%k), 0) + (1 if w == v[P])
	//f(P, w, 0) = f(P-up, (w+k-v[P])%k, 1) + f(P-left, (w+k-v[P]%k), 1)
	for(int i=0; i<n; i++)
		for(int j=0; j<m; j++){
			f[i][j][v[i][j]][1] = 1;
			if(i > 0)
				for(int w=0; w<=k; w++){
					f[i][j][w][1] += f[i-1][j][(w+(k+1)-v[i][j])%(k+1)][0];
					f[i][j][w][0] += f[i-1][j][(w+(k+1)+v[i][j])%(k+1)][1];
					f[i][j][w][0] %= MOD;
					f[i][j][w][1] %= MOD;
				}
			if(j > 0)
				for(int w=0; w<=k; w++){
					f[i][j][w][1] += f[i][j-1][(w+(k+1)-v[i][j])%(k+1)][0];
					f[i][j][w][0] += f[i][j-1][(w+(k+1)+v[i][j])%(k+1)][1];
					f[i][j][w][0] %= MOD;
					f[i][j][w][1] %= MOD;
				}
			
			
			ans = (ans+f[i][j][0][0])%MOD;
		}
}

int main(){
	scanf("%d%d%d", &n, &m, &k);
	for(int i=0; i<n; i++)
		for(int j=0; j<m; j++)
			scanf("%d", &v[i][j]);
	
	dp();
	printf("%d\n", ans);
	
	return 0;
}
```
