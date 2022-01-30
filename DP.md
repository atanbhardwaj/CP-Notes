# Dynamic Programming

## 0/1 Knapsack

### Problem Statement:
We are given an array of `n` items, each with a `weight` (**W<sub>i</sub>**) and a `profit` (**P<sub>i</sub>**), and a bag (aka knapsack) which has a `limited capacity` of **C** 
(where **C** is the maximum amount of weight that the bag can hold). The goal is to find a subset of the items such that their `total weight is <= C` and the sum of their
profits is `maximum`.

Given:

- `C`
- `W : [W0, W1, ... Wn-1]`
- `P : [P0, P1, ... Pn-1]`

<!--
|                 |               |               |       |                 |
| :-------------: | :-----------: | :-----------: | :---: | :-------------: |
| **Weight (W):** | W<sub>0</sub> | W<sub>1</sub> |  ...  | W<sub>n-1</sub> |
| **Profit (P):** | P<sub>0</sub> | P<sub>1</sub> |  ...  | P<sub>n-1</sub> |
-->

### Brute-Force/Recursive Approach
For each item, we can make `2 choices`: to `include` it or to `exclude` it. Therefore, the number of combinations/choices we have is **2<sup>n</sup>**. We can recursively
choose or not choose an item, based on the space available in the bag, and then keep track of the maximum profit attained throught all the subsets/combinations we go through.

The recursive function `knapsack` will take the weight array (`W`), profit array (`P`), current space left in the bag (`Space`) and the current index of the item (`i`).
For each item, it will first check to see if the weight of the item `W[i]` doesn't excede the current capacity (`Space`) of the bag. If it does, the item will be excluded 
(since there is no other choice). Otherwise, we will find the maximum profit between including and excluding the item (because it is possible that the item is in the subset 
of items that result in the maximum profit and it is also possible that it is not in that subset - we need to consider both possibilites).

**0/1 Knapsack Recursive Decision Tree** ([Source](https://youtu.be/mGfK-j9gAQA?list=PLEJXowNB4kPxBwaXtRO1qFLpCzF75DYrS&t=1082)):
![0/1 Knapsack Recursive Decision Tree](https://github.com/AyushKoul00/CP-Notes/blob/main/01_Knapsack_DT.PNG?raw=true)

**Code: (C++)**
```C++
#include <bits/stdc++.h>
using namespace std;
/**
 * @param W array of item weights
 * @param P array of item profits
 * @param Space current capacity of the bag (before choosing item i)
 * @param i current item index (start from n-1)
*/
int knapsack(const vector<int> &W, const vector<int> &P, int Space, int i)
{
    if (i == -1 || Space == 0)
        return 0;
    if (W[i] > Space) 
        return knapsack(W, P, Space, i - 1); //Exclude the item
    else
        return max(knapsack(W, P, Space, i - 1),                //Exclude the item
                   P[i] + knapsack(W, P, Space - W[i], i - 1)); //Include the item
}

int main()
{
    vector<int> W = {7, 2, 4}, P = {10, 5, 6};
    int C = 7;
    cout << knapsack(W, P, C, W.size() - 1);

    return 0;
}
```
**Complexity Analysis**

- **Time complexity: O(2<sup>n</sup>)** - Size of recursion tree will be 2<sup>n</sup>
- **Space complexity: O(n)** - The depth of the recursion tree can go up to n.


### Recursive (Top-down) + Memoization
If we notice, out recursive function only depends on two things: the items we choose (`i`) and the current capacity of the bag (`Space`). The weights and profit arrays remain constant in each call (we could choose to make them global as well). Therefore, if we think of the recursive call as `knapsack(Space, i)` then we can see that out recursive function re-calculates a call with the same parameters multiple times. Example:

```
In the following recursion tree, K(a, b) refers 
to knapSack(Space, i).
The recursion tree is for following sample inputs.
W[] = {1, 1, 1}, P[] = {10, 20, 30}, C = 2

                       K(C, n-1)
                       K(2, 3)  
                   /            \ 
                 /                \               
            K(2, 2)                  K(1, 2)
          /       \                  /    \ 
        /           \              /        \
       K(2, 1)      K(1, 1)        K(1, 1)     K(0, 1)
       /  \         /   \              /        
     /      \     /       \          /            
K(2, 0)  K(1, 0)  K(1, 0)  K(0, 0)  K(1, 0)   
```

We can see that K(1, 1) is calculated multiple times. Eventhough it may seem like a small calculation from the example, it is actually very expensive. Imagine if the tree was very deep and we can to calculate the same sub-problems (aka sub-calls) multiple times, it would make the time complexity exponential (which is seen in the brute-force approach). Caching the result saves this time and only explores new sub-problems/sub-calls that haven't been cached.

**Code: (C++)**
```C++
#include <bits/stdc++.h>
using namespace std;

// A hash function used to hash a pair of any kind
struct hash_pair {
    template <class T1, class T2>
    size_t operator()(const pair<T1, T2>& p) const
    {
        auto hash1 = hash<T1>{}(p.first);
        auto hash2 = hash<T2>{}(p.second);
        return hash1 ^ hash2;
    }
};
unordered_map<pair<int, int>, int, hash_pair> dp;

/**
 * @param W array of item weights
 * @param P array of item profits
 * @param Space current capacity of the bag (before choosing item i)
 * @param i current item index (start from n-1)
*/
int knapsack(const vector<int> &W, const vector<int> &P, int Space, int i)
{
	if(dp.find({i, Space}) != dp.end())
		return dp[{i, Space}];
	if (i == -1 || Space == 0)
		return dp[{i, Space}] = 0;
	if (W[i] > Space)
		return dp[{i, Space}] = knapsack(W, P, Space, i - 1); //Exclude the item
	else
		return dp[{i, Space}] = max(knapsack(W, P, Space, i - 1),		 //Exclude the item
					    P[i] + knapsack(W, P, Space - W[i], i - 1)); //Include the item
}

int main()
{
    vector<int> W = {7, 2, 4}, P = {10, 5, 6};
    int C = 7;
    cout << knapsack(W, P, C, W.size() - 1);

    return 0;
}
```
**Complexity Analysis**

- **Time complexity: O(n\*C)** - This is because the recursive function will only branch or call other recursive functions if it hasn't cached the query yet. The number of possible queries is n\*C (For each item, we have to explore for all capacities <= C)
- **Space complexity: O(n\*C + size of recursive call stack)** - We are caching n\*C results in our Hash Map.


### Bottom-Up (Tabular)
In order to understand this approach, we need to refer back to the recurrence relation (how and when we call the recursive calls) we used in the Top-down method.
```
//Base case:
dp(i, 0) = 0
dp(-1, Space) = 0

//Recurrence Relation
if (W[i] > Space)
    dp(i, Space) = dp(i-1, Space)
else
    dp(i, Space) = max(dp(i-1, Space), P[i] + dp(i-1, Space - W[i]))
```
We know that `i` will be in the range of `[0, n-1]` and Space will be in the range of `[1, C]`. We can use this to change dp into a 2D matrix/array for caching the results. In addition, we will be generating the answer from the bottom up (i.e. calculating and storing the results of smaller sub problems then using those to calculate and store the larger ones). In top down, we start from the top (the answer we need, and recursively call the small sub problems), but here, we start from the bottom.

One more point to note is that, our base case checks to see if i is -1 which we can't use if we have an array (since they only go to index 0). Therefore, we will shift the values of i by +1 (make it 1 indexed) and reserve 0 for the -1 case (basically, in the code, dp[0]\[Space] = 0 since 0 works like -1)


**Code: (C++)**
```C++
#include <bits/stdc++.h>
using namespace std;
/**
 * @param W array of item weights
 * @param P array of item profits
 * @param C capacity of the bag
*/
int knapsack(const vector<int> &W, const vector<int> &P, int C)
{
	int n = W.size();

	vector<vector<int>> dp(n + 1, vector<int>(C + 1));
	//dp[0][Space] = 0 (by default in vector)
	//dp[i][0] = 0 (by default in vector)
	for (int i = 1; i <= n; ++i)
	{
		for (int Space = 1; Space <= C; ++Space)
		{
			if (W[i - 1] > Space)
				dp[i][Space] = dp[i - 1][Space];
			else
				dp[i][Space] = max(dp[i - 1][Space], P[i - 1] + dp[i - 1][Space - W[i - 1]]);
		}
	}

	return dp[n][C];
}
int main()
{
    vector<int> W = {7, 2, 4}, P = {10, 5, 6};
    int C = 7;
    cout << knapsack(W, P, C, W.size() - 1);

    return 0;
}
```
**Complexity Analysis**

- **Time complexity: O(n\*C)** - We have 2 nested for loops that run in total of n\*C times.
- **Space complexity: O(n\*C)** - We are caching n\*C results in our 2D matrix/array.


## Frog 1 [Atcoder Educational Dp Contest]

### Problem Statement
There are `N` stones, numbered `1,2,…,N`. For each `i` (**1≤i≤N**), the height of Stone `i` is **h<sub>i</sup>**. There is a frog who is initially on Stone `1`. He will repeat the following action some number of times to reach Stone `N`: 
If the frog is currently on Stone `i`, jump to Stone `i+1` or Stone `i+2`. Here, a cost of **∣h<sub>i</sub> − h<sub>j</sub>∣** is incurred, where `j` is the stone to land on. 
Find the minimum possible total cost incurred before the frog reaches Stone `N`.


Given:

- `N`
- **h<sub>1</sub> h<sub>2</sub> … h <sub>N</sub>**

### Brute-Force/Recursive Approach
- Note: the values of the given height array is used as height[i], height[i+1] ... height[n-1] and represented as height<sub>1</sub>, height<sub>2</sub> and so on in the explaination below.
  
- The totalCost variable is used to store the minimum possible total cost incurred when the frog reached the last stone. 
  
There are `n` stones given, `1,2,3,...,N`. Given that the frog is initially on stone `1` **say [i]** and it has two possible choices, that is either it can jump on stone `2` or `3` let us say the frog jumps on stone `2` **([i+1])** and the cost incurred is **|height<sub>1</sub> - height<sub>2</sub>|** else if it makes the other choice, that is to jump on stone `3` then the cost incurred is **|height<sub>1</sub> - height<sub>2</sub>|** and now we will have to take the minimum of both these values that is **min(**|h<sub>1</sub> - h<sub>2</sub>|**, **|height<sub>1</sub> - height<sub>2</sub>|**)** and add this value to the totalCost incurred and then explore all the other possibilities similarly from stone 2 and stone 3 all the way to stone n. So, here we get the recurrence relation as, 
- **R<sub>x</sub> = min(|height<sub>x</sub> -  height<sub>(x-1)</sub>| + R<sub>(x-1)</sub>, |height<sub>x</sub> - height <sub>(x-2)</sub>| + R<sub>(x-2)</sub>)**
So, in a brute force manner we can explore all the possible answers and update the totalCost after reaching at the last stone.

**Code: C++**

```C++

#include <bits/stdc++.h>
using namespace std;

void minimumPossibleCostIncurred(int *height, int &totalCost, int cost, int index, int n)
{
    if (index >= n)
        return;

    if (index == n - 1)
    {
        totalCost = min(totalCost, cost);
        return;
    }

    minimumPossibleCostIncurred(height, totalCost, cost + abs(height[index] - height[index + 1]), index + 1, n);
    minimumPossibleCostIncurred(height, totalCost, cost + abs(height[index] - height[index + 2]), index + 2, n);
}

int32_t main()
{
    int n;
    cin >> n;
    int height[n];
    for (int i = 0; i < n; i++)
        cin >> height[i];

    int totalCost = INT_MAX;
    int cost = 0;

    minimumPossibleCostIncurred(height, totalCost, cost, 0, n);

    cout << totalCost;
}
``` 

**Complexity Analysis**

- **Time complexity: O(2<sup>n</sup>)** - Size of recursion tree will be 2<sup>n</sup>
- **Space complexity: O(n)** - The depth of the recursion tree can go up to n.