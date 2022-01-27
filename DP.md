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
|                   |               |               |     |                 |
|  :-------------:  |:-------------:|:-------------:|:---:|:---------------:|
| **Weight (W):**   | W<sub>0</sub> | W<sub>1</sub> | ... | W<sub>n-1</sub> |
| **Profit (P):**   | P<sub>0</sub> | P<sub>1</sub> | ... | P<sub>n-1</sub> |
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

**Code:**
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
