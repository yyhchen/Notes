# 55. 跳跃游戏

(mid), 一开始没理解

主要思想：贪心，找到最远可到达的位置，并且大于当前位置。

---

### 问题描述

给你一个非负整数数组 nums ，你最初位于数组的 第一个下标 。数组中的每个元素代表你在该位置可以跳跃的最大长度。

判断你是否能够到达最后一个下标，如果可以，返回 true ；否则，返回 false 。

 

**示例 1：**

    输入：nums = [2,3,1,1,4]
    输出：true
    解释：可以先跳 1 步，从下标 0 到达下标 1, 然后再从下标 1 跳 3 步到达最后一个下标。


**示例 2：**

    输入：nums = [3,2,1,0,4]
    输出：false
    解释：无论怎样，总会到达下标为 3 的位置。但该下标的最大跳跃长度是 0 ， 所以永远不可能到达最后一个下标。


<br>
<br>
<br>

### 算法思想

这个算法的思想是贪心算法。贪心算法是一种在每一步选择中都采取在当前状态下最好或最优（即最有利）的选择，从而希望导致结果是全局最好或最优的算法。在这个问题中，贪心算法的**核心思想是尽可能地跳远，以便能够覆盖更多的位置**。

具体来说，算法维护一个变量 `max_jump`，表示从起点出发，在不违反跳跃规则的情况下，最远能够到达的位置。在遍历数组 `nums` 的过程中，算法会不断地更新 `max_jump` 的值，试图达到最远的位置。**如果在某个位置 `i`，`max_jump` 小于 `i`，那么说明无法到达位置 `i`**，也就无法继续向前跳，因此返回 `false`。如果在遍历结束后，`max_jump` 大于等于数组的最后一个位置，那么说明可以从起点跳到数组的最后一个位置，返回 `true`。

这个算法的关键在于，它通过维护一个最远可达位置的变量，来简化了问题。每次遍历时，都在尝试更新这个最远可达位置，如果发现某个位置无法到达，则直接返回 `false`。这种方法的时间复杂度是 O(n)，其中 n 是数组 `nums` 的长度。


### 解题代码
```cpp
class Solution {
public:
    bool canJump(vector<int>& nums) {
        int max_jump = 0;
        for(int i = 0; i<nums.size(); i++){ //遍历位置（当前位置）
            if(max_jump<i){ //如果可走的最远位置 无法到达 当前位置
                return false;
            }
            max_jump = max(nums[i] + i, max_jump);  //贪心, 最远可以到达的位置
        }
        return (max_jump >= nums.size() -1);
    }
};
```



<br>
<br>
<br>


### 另一种解法（倒序贪心）
```cpp
class Solution {
public:
    bool canJump(vector<int>& nums) {
        int minNums = 1,size = nums.size();
        for(int i = size - 2;i  >= 0;i--) {
            if(nums[i] >= minNums) {
                minNums = 1;
            } else {
                minNums++;
            }
        }
        return minNums == 1;
    }
};
```

<br>

#### 算法思想

这种解法的思想是从后向前遍历数组，**检查从每个位置开始是否能够到达数组的最后一个位置**。在遍历过程中，维护一个变量 `minNums`，**表示从当前点到终点至少需要跳跃的最小次数**。如果在一个位置 `i` 上，`nums[i]` 的值大于或等于 `minNums`，那么从当前位置可以一步到达终点或者通过较少的跳跃到达终点，==**因此 `minNums` 可以重置为 1**==。如果 `nums[i]` 的值小于 `minNums`，那么从当前位置到达终点至少需要 `minNums + 1` 次跳跃，因此 `minNums` 需要增加 1。

遍历完成后，如果 `minNums` 的值为 1，这意味着从数组的起始位置只需要跳跃一次或者不需要跳跃就可以到达终点，因此返回 `true`。如果 `minNums` 的值大于 1，这意味着从数组的起始位置至少需要跳跃 `minNums` 次才能到达终点，因此返回 `false`。

这种解法的关键在于，它通过从后向前检查，逐步确定从每个位置到达终点所需的最小跳跃次数，从而避免了检查每个可能到达的路径，而是通过一个简单的计数器来跟踪所需的最小跳跃次数。这种方法的时间复杂度是 O(n)，其中 n 是数组 `nums` 的长度。