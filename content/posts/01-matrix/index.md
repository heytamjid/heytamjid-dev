+++
title = "01 Matrix on LeetCode: Multi-Source BFS and Two-pass DP Solution"
summary = "In Multi-source BFS, we start from multiple source nodes simultaneously instead of starting from a single source node, In this case, all the 0 cells act as the initial sources, and we expand outward in all directions. This works efficiently because BFS inherently explores all nodes layer by layer, guaranteeing that the first time we reach a cell, it is via the shortest possible path. Unlike the naive approach of running BFS separately for each 1 cell (which would be too slow), multi-source BFS spreads out from all 0s in parallel. Again, BFS processes each cell exactly once so this solution will run in O(m*n). As of the DP solution, **the recursive approach must NOT create cyclic dependencies.** When we are computing the distance for a cell, we are making recursive calls to all its neighbors, right?. But if two cells depend on each other, for example, cell A calls cell B, which in turn calls cell A (or another cell that eventually calls A) before A’s value is finalized, you’ll end up in an infinite loop or incorrect results because the dp value for A is still set to –1 when B checks it. In recursive DP, if we do not **ensure that all dependent subproblems are solved before the current cell’s computation,** the recursion can “chase its tail” or run infinitely and never reach a stable solution."
date = "2025-03-08"
categories = ["problem-solving"]
tags = ["leetcode"]
+++

In this post, I will explore two different approaches to solve the [01 Matrix](https://leetcode.com/problems/01-matrix/description/) problem on Leetcode.

## Breadth-first Search (BFS) Approach

At first, let’s have another glance to the typical BFS algorithm. It finds the shortest path from the source to all the nodes.
```
1. Initialize an empty Queue (say q) and the solution data structure to be returned.
2. Initialize non-sources (for all v ∈ G.V - {S})
    - v.d = infinity (*d* denotes distance)
    - v.color = white or visited [v] = false
    - v.parent = NIL
3. Initialize source(s)
    - s.d = 0
    - s.color = grey or visited [s] = true //grey means the node is on the frontier i.e. being explored currently
    - s.parent = NIL
4. Enqueue the source(s)
5. WHILE the Queue is not empty:
    1. Retrieve the front (say f) of the queue and also dequeue it.
    2. Explore all the adjacent nodes //ensure not out-of-bound while coding
        - For each UNVISITED and VALID nodes of f
            - Update its data (distance, color/visited, parent)
            - Push it to the q
    3. Color it black //meaning this node is now behind the wavefront
```

Here is a more formal presentation from the CLRS book:

![bfs-algo-clrs-textbook](/images/bfs-algo-clrs-book.png)

So in the typical BFS, we are starting from a single source node and explore a graph or grid level by level, calculating the shortest distance from that source to all other nodes. Multi-source BFS, however, starts from multiple source nodes at the same time. Instead of running separate BFS processes for each source (and then taking the minimum of the shortest distances you obtained) , you enqueue all the sources into a queue at the beginning and run a single BFS. This creates a “wavefront” that expands outward from all sources together. This is perfect when you need to find the shortest distance from any of several starting points to other nodes.

Now, let’s come to [the problem](https://leetcode.com/problems/01-matrix/description/). We are tasked with finding the distance of the nearest 0 for each cell. In other words, for all of the cells, we need to find the shortest distance to the 0 cells.

Obviously, the distance of the nearest 0 for the 0 cells is 0. We need to find the distance of the nearest 0 cells for the 1 cells.

So should we apply (single-source/standard) BFS separately for each of the 1 cells of the grid to get the shortest distance to nearest 0 cells? Well. as BFS processes each cell exactly once, running BFS once will cost O(m*n) time. [m*n is the grid dimension remember?] So in the worst case where (almost) all cells are 1, this approach will cost O((mn)^2) time. Nah, we can do better.

> Here is the better approach. We will apply multi-source BFS on the 0 cells, (not on the 1 cells). These 0 cells are the “targets,” and we compute how far each 1 cell is from its nearest 0. Essentially, we are measuring distance to targets (0s) by radiating from those targets.

In Multi-source BFS, we start from multiple source nodes simultaneously instead of starting from a single source node, In this case, all the 0 cells act as the initial sources, and we expand outward in all directions. This works efficiently because BFS inherently explores all nodes layer by layer, guaranteeing that the first time we reach a cell, it is via the shortest possible path.

Unlike the naive approach of running BFS separately for each 1 cell (which would be too slow), multi-source BFS spreads out from all 0s in parallel. Again, BFS processes each cell exactly once so this solution will run in O(m*n).

Here is the code for the approach. (It beats 94% on Leetcode. However, this feature on Leetcode is broken/inconsistent for a while.)

```cpp
class Solution {
public:
    vector<vector<int>> updateMatrix(vector<vector<int>>& mat) {
 
        int m = mat.size();
        int n = mat[0].size(); 
 
        vector <int> dir = {0, 1, 0, -1, 0};
 
        queue <pair<int, int>> q; //the queue will contain the coordinates
 
 
        // let's initialize the sources and the other cells. and enqueue all the sources
        for (int i=0; i<m; i++)
        {
            for (int j =0; j<n; j++) //STEP 1-4 of the algo
            {
                if (mat[i][j] == 1) 
                {
                    mat[i][j] = -1; //essentially, we are modifying the same mat matrix to store the shortest distances (as well as the info wheather a cell is visited : -1 -> unvisited , any number else -> shortest distance i.e. visited)
                }
                 
                else if (mat[i][j] == 0) 
                {
                    q.push(make_pair(i, j)); //the shortest distance of the 0 cells to the nearest 0 cell is 0 :)
                }
            }
        }
 
 
        /*
        After initialization, a cell’s value is:
        0 (if it was originally a 0),
         
        -1 (if it hasn’t been processed yet),
         
        A positive number (if we’ve already calculated its distance).
        */
 
        //running bfs
 
        while (q.size() != 0) //STEP 5 of the algo
        {
            // retrieving and dequeueing the front
            int r = q.front().first; 
            int c = q.front().second; 
            q.pop(); 
 
            // exploring the adjacent nodes / valid directions
            for (int i=0; i<=3; i++) //cool trick. it has been described below.
            {
                int newr = r+dir[i]; //getting the new row coordinate
                int newc = c+dir[i+1]; //getting the new column coordinate
 
                if (newr < 0 || newc < 0 || newr >= m || newc >= n) //out of bound
                {
                    continue; // skips to the next iteration
 
                }
 
                if (mat[newr][newc] != -1) //means the cell isn’t marked as unvisited. we skip it because it’s either a target 0 cell or has its distance already set.
                {
                    continue;
                }
                 
                //if we are here, meaning we have got an unvisited cell. let's set up its shortest distance. 
                mat[newr][newc] = mat[r][c] + 1; //shortest distance calculation. also this marks the cell as visited. note that, each cell is processed only once.
                q.push(make_pair(newr, newc));  //enqueuing the newly visited cell
            }
        }
 
        return mat; 
         
    }
};
```

Noktas/Postscript Thoughts:

- Could we have used multi-source BFS on all the 1 cells simultaneously? Not really. Because each 1 cell needs its own distance to the nearest 0

- Hey have you notice the nice trick with the vector <int> dir = {0, 1, 0, -1, 0} to explore the grid up, down, left and right? The array has 5 elements, but we use pairs of consecutive elements to define direction changes for the row (newr) and column (newc). dir[i] is the change in the row coordinate. dir[i+1] is the change in the column coordinate.

- Also note, in this grid problem, we are dealing with matrix coordinates instead of standard nodes/vertices.

- In BFS, the shortest distance is calculated only once. Once the shortest distance is set, it is not modified or updated later.

## Dynamic Programming (DP) Approach

Initially, I came up with a recursive DP approach where I attempted to calculate the minimum distance for each cell by recursively calling its neighbors (up, down, left, and right) and then storing the result in a memoization table. The code something looked like this (gives WA):

```cpp
class Solution {
private:
    int recursiveAlgo(int i, int j, vector<vector<int>> &mat, vector<vector<int>> &dp, int m, int n) {
        // If answer already exists in dp, return it.
        if (dp[i][j] != -1) return dp[i][j];
 
        // Base case: if the cell is 0, its distance is 0.
        if (mat[i][j] == 0) {
            dp[i][j] = 0;
            return 0;
        }
 
        // Initialize distances for all four directions.
        int up = INT_MAX, down = INT_MAX, left = INT_MAX, right = INT_MAX;
         
        if (i != 0)
            up = recursiveAlgo(i - 1, j, mat, dp, m, n);
        if (i != m - 1)
            down = recursiveAlgo(i + 1, j, mat, dp, m, n);
        if (j != 0)
            left = recursiveAlgo(i, j - 1, mat, dp, m, n);
        if (j != n - 1)
            right = recursiveAlgo(i, j + 1, mat, dp, m, n);
 
        // Set dp for current cell.
        dp[i][j] = min({up, down, left, right}) + 1;
        return dp[i][j];
    }
 
public:
    vector<vector<int>> updateMatrix(vector<vector<int>>& mat) {
        int m = mat.size(), n = mat[0].size();
        vector<vector<int>> dp(m, vector<int>(n, -1));
 
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                dp[i][j] = recursiveAlgo(i, j, mat, dp, m, n);
            }
        }
        return dp;
    }
};
```

The main issue in the approach above is that **the recursive approach creates cyclic dependencies.** When we are computing the distance for a cell, we are making recursive calls to all its neighbors, right?. But if two cells depend on each other, for example, cell A calls cell B, which in turn calls cell A (or another cell that eventually calls A) before A’s value is finalized, you’ll end up in an infinite loop or incorrect results because the dp value for A is still set to –1 when B checks it.

In recursive DP, if we do not **ensure that all dependent subproblems are solved before the current cell’s computation,** the recursion can “chase its tail” or run infinitely and never reach a stable solution.

To overcome these issues, we developed a two-pass DP solution that processes the matrix in a specific order that guarantees that every cell’s value depends only on already computed values. In the first pass from the top-left corner to the bottom-right corner, only the top and left neighbors are taken into consideration and computed. In the second pass, we compute in the reverse order, from the bottom-right corner to the top-left corner, only considering the bottom and the right neighbors and updating the dp table accordingly which stores the shortest distance to the nearest 0 for that cell. This ensures that **when updating a cell, we are only relying on cells whose values are already finalized and also we are covering all the possible directions.**

Here is the AC code.

```cpp
class Solution {
public:
    vector<vector<int>> updateMatrix(vector<vector<int>>& mat) {
 
        int m = mat.size();
        int n = mat[0].size(); 
         
        vector <vector<int>> dp (m, vector<int> (n, INT_MAX - 1)); //initializing distance at INT_MAX will result in overflow while computing (dp[i][j] + 1)
         
        for (int i=0; i<m; i++) //first pass: from top-left to bottom-right corner only considering the left and the top neighbors
        {
            for (int j=0; j<n; j++)
            {
                if (mat[i][j] == 0) // if a 0-cell, shortest distance will be 0
                {
                    dp[i][j] = 0;
                    continue; 
                }
 
                if (i>0) // checking top neighbor
                {
                    dp [i][j] = min (dp[i][j], dp[i-1][j] + 1) ; 
                }   
                 
                if (j>0) //checking left neighbor 
                {
                    dp[i][j] = min(dp[i][j], dp[i][j-1] + 1) ; 
                }
            }
        }
 
        for (int i=m-1; i>=0; i--) //second pass: from bottom-right to top-left corner only considering the neighbors at bottom and the right
        {
            for (int j=n-1; j>=0; j--)
            {
                if (i<m-1)  //checking bottom neighbor 
                {
                    dp [i][j] = min (dp[i][j], dp[i+1][j] + 1) ;
                }   
                 
                if (j<n-1) //checking right neighbor 
                {
                    dp[i][j] = min(dp[i][j], dp[i][j+1] + 1) ; 
                }
            }
        }
 
        return dp; 
 
    }
};
```