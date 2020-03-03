3/3/2020

[传送门 Teleport: Mayan Game @Luogu](https://www.luogu.com.cn/problem/P1312)

中文要求及图见原题。
Click the link for descriptions and pics.

游戏大概是：7行5列，有点像对对碰，但不一定填满整个board。只能左右移，如果移到其他方块就交换位置，移到空气就换到空气，但是任何时候悬空的块都会掉下。。
和对对碰一样，一个方向3个以上即消除，但注意可能一个chunk可能有横竖两个方向，比如十字和T型。

目标是找到N步之内的方法使得整个board清空。如果有多个则输出字典序最小的：x优先，其次y。右移小于左移。找不到输出-1.

So the game is played like this: on a 5 rows * 7 columns board, you can move a tile to the left or right. If the place it is moved towards already has a tile, then they are swapped. 
Otherwise it simply moves to an empty block. Any tile that floats in the air at any moment should drop instantly. 

A chunk with 3+ same color tiles on the same direction will be removed. Likewise any tile that floats in the air this way gets dropped.

Note that a chunk can have 3+ same color tiles on both directions (horizontal, vertical).

The objective is to find if there's such a way that within N steps, all tiles are removed. 
If there are multiple ways, output the lexicographically smallest way (x first, then y, and right before left).
If no such way exists, output -1.

刚观察的时候感觉是BFS，找个方法存状态用来比较是否达到过。找不到方法（并不会hash这个），但是发现既然有步数限制，最多N<=6步，只要到时候停就可以了

并且有一个重要发现。大多数时候不需要左移，因为把左边的右移一定更优。只有当前块左边没有块时才尝试左移，并且也要在尝试右移之后。

移之后清理chunks。注意对于两个方向的chunk，清理完一个方向另一个方向就检测不出来了。我的方法是把横向的标为负数，这样既知道这些块需要清理，也保留了value。

So as I observed it, I thought of BFS. Somehow you store the states and check if a state has been visited before. I couldn't find a way 
(I mean I don't know how to hash this thing). But since the steps are at most N <= 6, you just stop at N steps (don't have to check if replicate). Since they ask for a lexicographical order, you just search in this fashion. Loop from x and then y.

Now there's an important observation. Most of the cases you don't move left, since if you move [the tile to its left] right, they are effectively the same but the latter has more priority.
Try moving left only if there is no block on its left and only after moving right.

Now we have made the move, time to clean the chunks out. Note that for the chunks that go in both directions, when you clean the tiles in one direction first, the other direction can't be detected anymore (think of a X or T shape).
The trick I came up with is, mark the horizontal tiles as negative. This way you know they are to be cleaned (< 0) while retaining the value (abs).

```C++
//rightward: mark as negative.
for(int x=0; x+2<5; x++)
	for(int y=0; y<7; y++)
		//if at least 3 blocks the same, the first block not marked rightward yet, mark them negative for reference
		if(new_state.board[x][y] > 0 && abs(new_state.board[x+1][y]) == abs(new_state.board[x][y]) && abs(new_state.board[x+2][y]) == new_state.board[x][y]){
			need_clean = 1;	//you'll have to check again to see if there's new chunks to be cleaned
			for(int i=1; x+i<5 && abs(new_state.board[x+i][y]) == abs(new_state.board[x][y]); i++)
				new_state.board[x+i][y] = -abs(new_state.board[x+i][y]);
			new_state.board[x][y] = -new_state.board[x][y];	//you can't mark the starting tile first, since the following ones are compared to it.			}
//upward: mark as zero
for(int x=0; x<5; x++)
	for(int y=0; y+2<7; y++)
		//if at least 3 blocks the same, the first block not marked upward yet, mark them zero directly
		if (new_state.board[x][y] != 0 && abs(new_state.board[x][y]) == abs(new_state.board[x][y+1]) && abs(new_state.board[x][y]) == abs(new_state.board[x][y+2])){
			need_clean = 1;
			for(int i=1; y+i < 7 && abs(new_state.board[x][y+i]) == abs(new_state.board[x][y]); i++)
				new_state.board[x][y+i] = 0;
			new_state.board[x][y] = 0;
		}
```

想到了一个舒服的方法清理每列：把这列的非零值记下来，按顺序贴回去即可。

I also came up with a clean() function to clean up the columns. Just look through the column and record all the nonzero values. Then paste it to the column board in order.


```C++
void clean(int board[5][7], int x){
	int new_col[7] = {0,0,0,0,0,0,0}, idx = 0;
	for(int y=0; y<7; y++)
		if(board[x][y])	//if not blank
			new_col[idx++] = board[x][y];
	
	for(int y=0; y<7; y++)
		board[x][y] = new_col[y];
}
```

并没有过样例。很快发现清一次chunks不够，还要继续找并清理掉，直到找不到新的chunk为止。

然后提交了，好几个MLE，一个T。看题解都在用DFS，发现的确在有步数限制时DFS可能更快，而且省内存。
之前T的点没T，但是另一个点T了。想了想把限制从N>6退出改成[N>6退出 然后确认完之后N=6也退出]
这样就。。。过了

Didn't pass the sample. Shortly I realized that cleaning the chunks once was not enough, you need to keep finding the chunks, clean them and do it over again until there's no new chunk to clean.

Then I submitted. BFS Turned out to be many MLEs and TLE for one case. Looked at solutions, which all used DFS. I realized that it could be faster now that there's a limit on number of steps. Also it saves memory.
The case where I TLE'd didn't TLE but another case did. Then I decided to change the constraint [quit when N>6] into [N>6 and then N=6 after checking].

And then I AC'd...

Complete code:

``` C++
#include <iostream>
#include <cstdio>
#include <algorithm>
#include <cstring>
#include <queue>
#include <vector>
using namespace std;
struct Move{
	int x, y, dir;
};
struct State{
	int board[5][7];
	vector<Move> moves;
	State(){
		memset(board, 0, sizeof(board));
		moves.clear();
	}
}init_state;

int N;

void clean(int board[5][7], int x){
	int new_col[7] = {0,0,0,0,0,0,0}, idx = 0;
	for(int y=0; y<7; y++)
		if(board[x][y])	//if not blank
			new_col[idx++] = board[x][y];
	
	for(int y=0; y<7; y++)
		board[x][y] = new_col[y];
}

void process(State &new_state){
	
	bool need_clean = 1;
	while(need_clean){
		need_clean = 0;
		//find all the chunks to be cleaned, first rightward
		//rightward: mark as negative.
		for(int x=0; x+2<5; x++)
			for(int y=0; y<7; y++)
				//if at least 3 blocks the same, the first block not marked rightward yet, mark them negative for reference
				if(new_state.board[x][y] > 0 && abs(new_state.board[x+1][y]) == abs(new_state.board[x][y]) && abs(new_state.board[x+2][y]) == new_state.board[x][y]){
					need_clean = 1;
					for(int i=1; x+i<5 && abs(new_state.board[x+i][y]) == abs(new_state.board[x][y]); i++)
						new_state.board[x+i][y] = -abs(new_state.board[x+i][y]);
					new_state.board[x][y] = -new_state.board[x][y];
				}
		//upward: mark as zero
		for(int x=0; x<5; x++)
			for(int y=0; y+2<7; y++)
				//if at least 3 blocks the same, the first block not marked upward yet, mark them zero directly
				if (new_state.board[x][y] != 0 && abs(new_state.board[x][y]) == abs(new_state.board[x][y+1]) && abs(new_state.board[x][y]) == abs(new_state.board[x][y+2])){
					need_clean = 1;
					for(int i=1; y+i < 7 && abs(new_state.board[x][y+i]) == abs(new_state.board[x][y]); i++)
						new_state.board[x][y+i] = 0;
					new_state.board[x][y] = 0;
				}
		
		//clean up rightward marks
		for(int x=0; x<5; x++)
			for(int y=0; y<7; y++)
				if(new_state.board[x][y] < 0)
					new_state.board[x][y] = 0;
	
		
		//clean the blanks
		for(int x=0; x<5; x++)
			clean(new_state.board, x);
	}
}

void print_board(const State &cur_state){
	for(int y=6; y>=0; y--){
		for(int x=0; x<5; x++)
			printf("%d ", cur_state.board[x][y]);
		printf("\n");
	}
}

void dfs(State cur_state, vector<Move> &ans, bool &finished){
	
	if(finished)	return;
	if(cur_state.moves.size() > N)	return;
	int sum = 0;
	for(int x=0; x<5; x++)
		sum += cur_state.board[x][0];
	if(sum == 0){
		ans = cur_state.moves;
		finished = 1;
		return;
	}	
	if(cur_state.moves.size() == N)	return;
	
	for(int X=0; X<5; X++)
		for(int Y=0; Y<7; Y++)
			//if not blank
			if(cur_state.board[X][Y])	{
				//try moving right
				if(X+1 < 5 && cur_state.board[X+1][Y] != cur_state.board[X][Y]){
//							printf("trying to move (%d,%d): %d to right\n", X, Y, cur_state.board[X][Y]);
					State new_state = cur_state;
					//swap with a block or with blank
					int tmp = new_state.board[X+1][Y];
					new_state.board[X+1][Y] = new_state.board[X][Y];
					new_state.board[X][Y] = tmp;
					//if right is blank, clean airborne blanks
					if(tmp == 0){
						clean(new_state.board, X);
						clean(new_state.board, X+1);
					}
					
					process(new_state);
					
					new_state.moves.push_back({X, Y, 1});
//							print_board(new_state);
					dfs(new_state, ans, finished);
				}
				//if left is blank, try moving left
				if(X-1 >= 0 && cur_state.board[X-1][Y] == 0 && cur_state.board[X-1][Y] != cur_state.board[X][Y]){
//							printf("trying to move (%d,%d): %d to left\n", X, Y, cur_state.board[X][Y]);
					State new_state = cur_state;
					//swap with a block or with blank
					int tmp = new_state.board[X-1][Y];
					new_state.board[X-1][Y] = new_state.board[X][Y];
					new_state.board[X][Y] = tmp;
					//if left is blank (of course), clean airborne blanks
					if(tmp == 0){
						clean(new_state.board, X);
						clean(new_state.board, X-1);
					}
					
					process(new_state);
//							print_board(new_state);
					new_state.moves.push_back({X, Y, -1});
					dfs(new_state, ans, finished);
				}
			}
	
	
}

int main(){
	scanf("%d", &N);
	int t;
	for(int x=0; x<5; x++){
		int y = 0;
		while(scanf("%d", &t)){
			if(t == 0)	break;
			init_state.board[x][y++] = t;
		}
	}
	init_state.moves = vector<Move>();
//	printf("%d\n", init_state.moves.size());

	//test board
//	for(int y=6; y>=0; y--){
//		for(int x=0; x<5; x++)
//			printf("%d ", init_state.board[x][y]);
//		printf("\n");
//	}

	bool finished = 0;
	vector<Move> ans;
	dfs(init_state, ans, finished);
	
	if(ans.size())
		for(int i=0; i<ans.size(); i++)
			printf("%d %d %d\n", ans[i].x, ans[i].y, ans[i].dir);
	
	else
		printf("-1");
	
	return 0;
}
```
