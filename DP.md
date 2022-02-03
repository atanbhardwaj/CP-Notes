# Dynamic Programming

<a name="01ks"></a>
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

<br />
<br />

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

**Code: Python3**
```python
W = [7, 2, 4]
P = [10, 5, 6]
C = 7

def knapsack(W,P, Space, i):
    if i == -1 or Space == 0:
        return 0
    if W[i] > Space: 
        return knapsack(W, P, Space, i - 1)
    else:
        return max(knapsack(W, P, Space, i - 1),                
                   P[i] + knapsack(W, P, Space - W[i], i - 1))


print(knapsack(W, P, C, len(W) - 1))
```
**Complexity Analysis**

- **Time complexity: O(2<sup>n</sup>)** - Size of recursion tree will be 2<sup>n</sup>
- **Space complexity: O(n)** - The depth of the recursion tree can go up to n.

<br />
<br />

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

**Code: Python3**
```python
W = [7, 2, 4]
P = [10, 5, 6]
C = 7

dp = {}
def knapsack(W, P, Space, i):
    if (i, Space) in dp.keys():
        return dp[{i, Space}]
    if i == -1 or Space == 0:
        dp[(i, Space)] = 0
        return dp[(i, Space)]
    if  W[i] > Space:
        dp[(i, Space)] = knapsack(W, P, Space, i - 1)
        return dp[(i, Space)]
    else:
        dp[(i, Space)] = max(knapsack(W, P, Space, i - 1),	
                    P[i] + knapsack(W, P, Space - W[i], i - 1))
    return dp[(i, Space)]
    
print(knapsack(W, P, C, len(W) - 1))
```
**Complexity Analysis**

- **Time complexity: O(n\*C)** - This is because the recursive function will only branch or call other recursive functions if it hasn't cached the query yet. The number of possible queries is n\*C (For each item, we have to explore for all capacities <= C)
- **Space complexity: O(n\*C + size of recursive call stack)** - We are caching n\*C results in our Hash Map.

<br />
<br />

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

<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />






## Atcoder Educational Dp Contest

<a name="frog_1"></a>
### [Frog1](https://atcoder.jp/contests/dp/tasks/dp_a)

#### Brute-Force/Recursive Approach
We know that from given `h[i]`, we can only go to either `h[i+1]` or `h[i+2]`. If we reverse this logic, we can see that from any given `h[i]`, we can only arrive there either from `h[i-1]` or `h[i-2]`. Finally, the cost of arriving at `h1` is 0 (since we are already there when we start) and `h1` is `|h[1] - h[0]|` since we can only come to `h1` from `h0`

**Code: C++**

```C++
#include <bits/stdc++.h>
using namespace std;

int recursive(vector<int> &h, int i)
{
	if (i == 0)
		return 0;
	if (i == 1)
		return abs(h[1] - h[0]);
	return min(abs(h[i] - h[i - 1]) + recursive(h, i - 1), abs(h[i] - h[i - 2]) + recursive(h, i - 2));
}

int main()
{
	int n;
	cin >> n;
	vector<int> h(n);
	for (int i = 0; i < n; ++i)
		cin >> h[i];

	cout << recursive(h, n-1);
	return 0;
}
``` 

**Code: Python3**
```python

n = int(input())
nums = list(map(int, input().strip().split()))

def recursive(nums):
    n = len(nums)

    def helper(index):

        if index == 0:
            return 0
        if index == 1:
            return abs(nums[1] - nums[0])
	    
        return min(abs(nums[index] - nums[index - 1]) + helper(index - 1), abs(nums[index] - nums[index - 2]) + helper(index - 2))
    
    return helper(n - 1)

print(recursive(nums))
```

**Complexity Analysis**

- **Time complexity: O(2<sup>n</sup>)** - Size of recursion tree will be 2<sup>n</sup>
- **Space complexity: O(n)** - The depth of the recursion tree can go up to n.

<br />
<br />

#### Recursive (Top-down) + Memoization

Here, we cache the results of the recursive brute-force approach to reduce time complexity:

**Code: C++**

```C++
#include <bits/stdc++.h>
using namespace std;

int topdown(vector<int> &h, int i)
{
	static vector<int> dp(h.size(), -1);
	if (dp[i] != -1)
		return dp[i];
	if (i == 0)
		return dp[i] = 0;
	if (i == 1)
		return dp[i] = abs(h[1] - h[0]);
	return dp[i] = min(abs(h[i] - h[i - 1]) + topdown(h, i - 1), abs(h[i] - h[i - 2]) + topdown(h, i - 2));
}

int main()
{
	int n;
	cin >> n;
	vector<int> h(n);
	for (int i = 0; i < n; ++i)
		cin >> h[i];

	cout << topdown(h, n-1);
	return 0;
}
```


**Code: Python3**

```python
n = int(input())
nums = list(map(int, input().strip().split()))

def topDown(nums):

    n = len(nums)
    dp = [-1 for x in range(0, n)]

    def helper(index):
        if dp[index] != -1:
            return dp[index]
        if index == 0:
            dp[index] = 0
            return dp[index]
        if index == 1:
            dp[index] = abs(nums[1] - nums[0])
            return dp[index]

        dp[index] = min(abs(nums[index] - nums[index - 1]) + helper(index - 1), abs(nums[index] - nums[index - 2]) + helper(index - 2))
        return dp[index]
    
    return helper(n - 1)

print(topDown(nums))
```
**Complexity Analysis**

- **Time complexity: O(n)** - This is because the recursive function will only branch or call other recursive functions if it hasn't cached the query yet. The number of possible calls is n (We have to explore each stone)

- **Space complexity: O(n + size of recursive call stack)** - We are caching n results in our DP array.

<br />
<br />

#### Bottom-Up (Tabular)

From the recurrence relation, we can come up with the bottom up approach:
```
Base cases:
if (i == 0)
	return dp[i] = 0;
if (i == 1)
	return dp[i] = abs(h[1] - h[0]);

Recurrence Relation:
dp[i] = min(abs(h[i] - h[i - 2]) + dp[i - 2], abs(h[i] - h[i - 1]) + dp[i - 1]);
```


**Code: C++**

```C++
#include <bits/stdc++.h>
using namespace std;

int bottomup(vector<int> &h)
{
	int n = h.size();
	vector<int> dp(n);

	dp[0] = 0, dp[1] = abs(h[1] - h[0]);
	for (int i = 2; i < n; ++i)
		dp[i] = min(abs(h[i] - h[i - 2]) + dp[i - 2], abs(h[i] - h[i - 1]) + dp[i - 1]);

	return dp[n - 1];
}
 
int main()
{
	int n;
	cin >> n;
	vector<int> h(n);
	for (int i = 0; i < n; ++i)
		cin >> h[i];

	cout << bottomup(h);
	return 0;
}
```
**Code: Python3**

```python
n = int(input())
nums = list(map(int, input().strip().split()))

def bottomUp(nums, n):
    dp = [0 for x in range(0, n)]
    for i in range(n-2, -1, -1):
        if i == n - 2:
            dp[i] = abs(nums[i + 1] - nums[i])
        else:
            dp[i] = min(abs(nums[i] - nums[i + 1]) + dp[i + 1], abs(nums[i] - nums[i + 2]) + dp[i + 2])

    return dp[0]
    
print(bottomUp(nums))
```

**Complexity Analysis**

- **Time complexity: O(n)** - We have single for loop that run in total of n times.
- **Space complexity: O(n)** - We are caching n results in our DP array.






<br />
<br />
<br />
<br />







<a name="frog_2"></a>
### [Frog2](https://atcoder.jp/contests/dp/tasks/dp_b)


#### Brute-Force/Recursive Approach
Similar to [Frog1](#frog_1), we know that if a frog can jump `k` stones ahead from a stone `i`, this also implies that a frog can only reach stone `i` by jumping from the previous `k` stones.

**Code: C++**

```C++
#include <bits/stdc++.h>
using namespace std;

int recursive(vector<int> &h, int k, int i)
{
	if (i == 0)
		return 0;

	int res = INT_MAX;
	for (int j = max(0, i - k); j < i; ++j)
		res = min(res, abs(h[i] - h[j]) + recursive(h, k, j));

	return res;
}

int main()
{
	int n, k;
	cin >> n >> k;
	vector<int> h(n);
	for (int i = 0; i < n; ++i)
		cin >> h[i];

	cout << recursive(h, k, n-1);
	return 0;
}
``` 

**Complexity Analysis**

- **Time complexity: O(n<sup>k</sup>)** - Size of recursion tree will be n<sup>k</sup>
- **Space complexity: O(n)** - The depth of the recursion tree can go up to n.

<br />
<br />

#### Recursive (Top-down) + Memoization

Here, we cache the results of the recursive brute-force approach to reduce time complexity:

**Code: C++**

```C++
#include <bits/stdc++.h>
using namespace std;

int topdown(vector<int> &h, int k, int i)
{
	static vector<int> dp(h.size(), -1);
	if (dp[i] != -1)
		return dp[i];
	if (i == 0)
		return dp[i] = 0;

	int res = INT_MAX;
	for (int j = max(0, i - k); j < i; ++j)
		res = min(res, abs(h[i] - h[j]) + topdown(h, k, j));

	return dp[i] = res;
}

int main()
{
	int n, k;
	cin >> n >> k;
	vector<int> h(n);
	for (int i = 0; i < n; ++i)
		cin >> h[i];

	cout << topdown(h, k, n-1);
	return 0;
}
```

**Complexity Analysis**

- **Time complexity: O(n)** - This is because the recursive function will only branch or call other recursive functions if it hasn't cached the query yet. The number of possible calls is n (We have to explore each stone)

- **Space complexity: O(n + size of recursive call stack)** - We are caching n results in our DP array.

<br />
<br />

#### Bottom-Up (Tabular)

From the recurrence relation, we can come up with the bottom up approach:
```
Base Case:
if (i == 0)
	return dp[i] = 0;
	
Recurrence Relation:
	dp[i] = max(dp[i], abs(h[i] - h[j]) + dp[j]) -> for all j in range [i-k, i)
```


**Code: C++**

```C++
#include <bits/stdc++.h>
using namespace std;

int bottomup(vector<int> &h, int k)
{
	int n = h.size();
	vector<int> dp(n, INT_MAX);

	dp[0] = 0;
	for (int i = 1; i < n; ++i)
	{
		for(int j = max(0, i - k); j < i; ++j)
			dp[i] = min(dp[i], abs(h[i] - h[j]) + dp[j]);
	}

	return dp[n - 1];
}
 
int main()
{
	int n, k;
	cin >> n >> k;
	vector<int> h(n);
	for (int i = 0; i < n; ++i)
		cin >> h[i];

	cout << bottomup(h, k);
	return 0;
}
```


**Complexity Analysis**

- **Time complexity: O(n)** - We have single for loop that run in total of n times.
- **Space complexity: O(n)** - We are caching n results in our DP array.






<br />
<br />
<br />
<br />






<a name="vacation"></a>
### [Vacation](https://atcoder.jp/contests/dp/tasks/dp_c)

#### Brute-Force/Recursive Approach
We know that we cannot take any given activity for two or more days in a row. So our decision to choose an activity will depend on what our previous decisions were: 
we will pass two parameters in our recursive function: `i` and `j`. Here, `i` denotes the `i`th day and `j` denotes the activity choosen on the `i+1`th day (`0->A, 1->B, 2->C`). Our basic logic is: we will only consider activities on the `i`th day that don't equal `j` because we already chose that on the next day. Therefore, the maximum points we can gain is by choosing one of the two remaining activities for the `i`th day plus the profit from the previous days (recursively).

**Code: C++**

```C++
#include <bits/stdc++.h>
using namespace std;

int recursive(vector<vector<int>> &act, int i, int j)
{
	int res = 0;
	if (i < 0)
		return res;
	if (i == 0)
	{
		for (int a = 0; a <= 2; ++a)
		{
			if (a == j) continue;
			res = max(res, act[a][0]);
		}
		return res;
	}

	for (int a = 0; a <= 2; ++a)
	{
		if (a == j)
			continue;
		res = max(res, act[a][i] + recursive(act, i - 1, a));
	}
	return res;
}

int main()
{
	int n;
	cin >> n;
	vector<vector<int>> act(3, vector<int>(n));
	for (int i = 0; i < n; ++i)
		cin >> act[0][i] >> act[1][i] >> act[2][i];

	cout << max({recursive(act, n - 2, 0) + act[0][n - 1],
				 recursive(act, n - 2, 1) + act[1][n - 1],
				 recursive(act, n - 2, 2) + act[2][n - 1]});
}
``` 

**Complexity Analysis**

- **Time complexity: O(2<sup>n</sup>)** - Size of recursion tree will be 2<sup>n</sup>
- **Space complexity: O(n)** - The depth of the recursion tree can go up to n.

<br />
<br />

#### Recursive (Top-down) + Memoization

Here, we cache the results of the recursive brute-force approach to reduce time complexity:

**Code: C++**

```C++
#include <bits/stdc++.h>
using namespace std;

int n;
int topdown(vector<vector<int>> &act, int i, int j)
{
	static vector<vector<int>> dp(3, vector<int>(n, -1));
	if(dp[j][i] != -1) return dp[j][i];
	int res = 0;
	if (i < 0)
		return dp[j][i] = res;
	if (i == 0)
	{
		for (int a = 0; a <= 2; ++a)
		{
			if (a == j) continue;
			res = max(res, act[a][0]);
		}
		return dp[j][i] = res;
	}

	for (int a = 0; a <= 2; ++a)
	{
		if (a == j)
			continue;
		res = max(res, act[a][i] + topdown(act, i - 1, a));
	}
	return dp[j][i] = res;
}

int main()
{
	cin >> n;
	vector<vector<int>> act(3, vector<int>(n));
	for (int i = 0; i < n; ++i)
		cin >> act[0][i] >> act[1][i] >> act[2][i];

	cout << max({topdown(act, n - 2, 0) + act[0][n - 1],
				 topdown(act, n - 2, 1) + act[1][n - 1],
				 topdown(act, n - 2, 2) + act[2][n - 1]});
}
```

**Complexity Analysis**

- **Time complexity: O(n)** - This is because the recursive function will only branch or call other recursive functions if it hasn't cached the query yet. The number of possible calls is 3*n (We need to explore the max profit at any day given we take each activity)

- **Space complexity: O(n + size of recursive call stack)** - We are caching n results in our DP array.

<br />
<br />

#### Bottom-Up (Tabular)

From the recurrence relation, we can come up with the bottom up approach:
```
Base cases:
if (i < 0)
	return dp[j][i] = 0;
if (i == 0)
{
	for (int a = 0; a <= 2; ++a)
	{
		if (a == j) continue;
		dp[j][i] = max(dp[j][i], act[a][0]);
	}
	return dp[j][i];
}

Recurrence Relation:

for (int a = 0; a <= 2; ++a)
{
	if (a == j) continue;
	dp[j][i] = max(dp[j][i], act[a][i] + topdown(act, i - 1, a));
}
return dp[j][i];
```


**Code: C++**

```C++
#include <bits/stdc++.h>
using namespace std;

int n;
int bottomup(vector<vector<int>> &act)
{
	vector<vector<int>> dp(3, vector<int>(n + 1));

	for (int i = 1; i <= n; ++i)
	{
		int a = act[0][i-1], b = act[1][i-1], c = act[2][i-1];
		dp[0][i] = max(dp[1][i - 1] + b,
					   dp[2][i - 1] + c);
		dp[1][i] = max(dp[0][i - 1] + a,
					   dp[2][i - 1] + c);
		dp[2][i] = max(dp[0][i - 1] + a,
					   dp[1][i - 1] + b);
	}
	return max({dp[0][n], dp[1][n], dp[2][n]});
}
 
int main()
{
	cin >> n;
	vector<vector<int>> act(3, vector<int>(n));
	for (int i = 0; i < n; ++i)
		cin >> act[0][i] >> act[1][i] >> act[2][i];
		
	cout << bottomup(act);
	return 0;
}
```


**Complexity Analysis**

- **Time complexity: O(n)** - We have single for loop that run in total of n times.
- **Space complexity: O(n)** - We are caching n results in our DP array.
