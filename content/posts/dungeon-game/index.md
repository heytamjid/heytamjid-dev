+++
title = "Dungeon Game on LeetCode : Finding The Right Question To Ask in DP Problems"
date = "2024-03-21"
categories = ["problem-solving"]
tags = ["leetcode"]
+++

I have been solving DP problems for last 2 days and guess what, they are really cooool and fun to solve. Previously I didn’t enjoy this much while solving topic-wise problems on Prefix Sum or Sliding Window. [Dungeon Game](https://leetcode.com/problems/dungeon-game/description/) is my first solved bottom-up DP problem. It’s marked as hard, but I believe it is sort of quite medium level problem.

In DP problems, the main part is to find out **what to remember** [as Errichto said](https://youtu.be/YBSt1jYwVfU?si=VWSg2Xta6vAQrQXk&t=1090). And for that, you have to know **the right question to ask**.

<!--more-->

In this particular problem, first observe that, At princess’s position (bottom-right cell), we need at least / minimum 1 health.

So the question we need to ask at each cell [i][j], what is the minimum health required while entering this cell (aka minimum initial health) so that my final health at the bottom-right cell becomes exactly 1. (Not greater than 1 as we need to find out MINIMUM initial health, not less than 1 as then the knight will die)

That’s what our dp state holds.

Intuitively it seems like bottom-up DP.

Now, we can say, for each cell,
> initialHealth + cost (aka grid[i][j] that could be positive or negative) = finalHealth
We have to calculate the initialHealth for each cell. Cost (i.e. power of orbs or demon) is given as input.

For the last cell i.e. [i-1][j-1], the finalHealth is 1.

For the last column, [i][j] cell’s finalHealth will be the same as [i+1][j] cell’s initialHealth as that’s the only way to move towards princess (from [i][j] to [i+1][j] along with the last column.) If the initialHealth is calculated non-positive i.e. less than 1, we will rather say or save initialHealth required for that cell is 1 in the dp array. As initialHealth can’t be negative or 0, otherwise our knight will die.

For the last row, [i][j] cell’s finalHealth will be the same as [i][j+1] cell’s initialHealth as that’s the only way to move towards princess (from [i][j] to [i][j+1] along wtih the last column.) If the initialHealth is calculated non-positive i.e. less than 1, we will rather say or save initialHealth required for that cell is 1 in the dp array. As initialHealth can’t be negative or 0, otherwise our knight will die.

For other cells, we will dynamic-programmically fill up the dp array values. That is, for each cell [i][j], dp[i][j] (which denotes required initalHealth for that cell) the finalHealth is the minimum of dp[i+1][j] and dp[i][j+1]. Because these two are valid moves towards the princess and we are choosing the way that will require minimum initialHealth. Then initialHealth will be calculated as finalHealth-cost where cost is grid[i][j]. And again, if the initialHealth is calculated non-positive i.e. less than 1, we will rather say or save initialHealth required for that cell is 1 in the dp array. As initialHealth can’t be negative or 0, otherwise our knight will die.

And finally we will return whatever our dp array holds in the [0][0] position as that’s what we are finding the answer of – the minimum initialHealth required to save the princess at the last cell starting from the first cell of the grid.

My code is given below:



```cpp
class Solution {
public:
    int calculateMinimumHP(vector<vector<int>>& grid) {
        int row = grid.size(), col = grid[0].size(); 
        long long int dp [row][col];
        //each dp state [i][j] holds the minimum initial health required to reach the bottom-right cell from [i][j] cell
        long long int rowFinal = 1, colFinal = 1;
 
        //bottom-up approach
        for (int i=row-1; i>=0; i--) //for each entry of the last column
        {
            dp [i][col-1] = max(rowFinal - grid[i][col-1] , (long long int)1); 
            rowFinal = dp[i][col-1]; 
        }
        for (int i=col-1; i>=0; i--) //for each entry of the last row
        {
            dp [row-1][i] = max(colFinal - grid[row-1][i], (long long int)1); 
            colFinal = dp[row-1][i];    
        }
 
        for (int i=row-2; i>=0; i--) //filling up other dp array values
        {
            for (int j=col-2; j>=0; j--)
            {
                dp[i][j] = max( min(dp[i+1][j], dp[i][j+1]) - grid [i][j], (long long int) 1); //max() for bounding so that negative values become 1 instead
            }
        }
 
        return dp [0][0] ; 
         
    }
};
```