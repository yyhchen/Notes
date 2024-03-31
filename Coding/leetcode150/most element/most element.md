# 169.多数元素
**（easy）**

---
给定一个大小为 n 的数组 `nums` ，返回其中的多数元素。多数元素是指在数组中出现次数 **大于** `⌊ n/2 ⌋` 的元素。

你可以假设数组是非空的，并且给定的数组总是存在多数元素。



**示例 1：**

    输入：nums = [3,2,3]
    输出：3


**示例 2：**

    输入：nums = [2,2,1,1,1,2,2]
    输出：2


<br>
<br>
<br>


**解题方法：**
我的想法，但是有bug

1. 双指针🪡，先给数组排序
2. `head` 和 `slow` 算长度，遍历数组，直到 `nums[head] != nums[slow]`, 找到了其中一个元素。
3. 此时，元素值 `v = nums[tail]`; 记录长度 `l = head - tail`; 并更新下一个元素起点 `tail = head`
4. 只要不符合 **2.** 则 `head++` （注意⚠️， **这不是else**）
5. 最后返回 v (bug在这，因为我下面代码中，只找到了最大的长度`l`，却没有记录对应的`v`)


```cpp{.line-numbers}
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        int head = 0, tail = 0;
        int v = 0, l = 0;
        int len = nums.size();
        sort(nums.begin(), nums.end());
        if(nums.empty()){
            return 0;
        }

        while(head < len){
            if(nums[head] != nums[tail]){
                v = nums[tail]; //bug在这，只会找到最后一次；并不是对应最大长度的值
                l = max(head-tail, l);
                tail = head;
            }
            head++;
        }
        return v;
    }
};

```

<br>
<br>

**修正后：**
```cpp{.line-numbers}
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        int head = 0, tail = 0;
        int v = 0, l = 0;
        int len = nums.size();
        sort(nums.begin(), nums.end());
        if(nums.empty()){
            return 0;
        }

        while(head < len){
            if(nums[head] != nums[tail]){
                int curr_len = head - tail;
                if(curr_len >= l){
                    v = nums[tail];
                    l = curr_len;
                }
                //l = max(head-tail, l);
                tail = head;
            }
            head++;
        }
        // 处理末尾的情况
        int current_length = head - tail;
        if (current_length > l) {
            l = current_length;
            v = nums[tail];
        }
        return v;
    }
};
```

1. 加入一个变量 `curr_len` 判断与当前 `l` 作比较，记录最长 `l`, 此时才给对应的 `v` 赋值; （**修复了之前找不到 最长 `l` 对应的 `v`** ）
2. 👮‍♀️⚠️因为遍历中查找最长的条件是: `nums[head] != nums[tail]`, 所以可能会忽略末尾最长的情况。因为**末尾**如果**全部元素相同**，**是无法找到** `nums[head] != nums[tail]` 
3. 故仍然需要给末尾做出判断，等所有的 `head++` ,再加多一次 `curr_len` 与 `l` 的比较即可。


---

### 官解：

**哈希表：**
```cpp{.line-numbers}
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        unordered_map<int, int> counts;
        int majority = 0, cnt = 0;
        for (int num: nums) {
            ++counts[num]; //检查 健num是否存在，不存在创建+1， 存在直接+1
            if (counts[num] > cnt) {
                majority = num;
                cnt = counts[num];
            }
        }
        return majority;
    }
};
```

**算法思想：**

1. 创建了一个哈希表 `counts`，用于记录每个元素出现的次数。
2. 定义了两个整数变量 `majority` 和 `cnt`，其中 `majority` 用于记录当前出现次数最多的元素，`cnt` 用于记录当前最大的出现次数。
3. 遍历输入的数组 `nums`，对于数组中的每个元素 `num`，更新哈希表 `counts` 中对应元素的出现次数，并判断当前元素的出现次数是否超过了 `cnt`，如果是，则更新 `majority` 和 `cnt` 的值为当前元素和对应的出现次数。
4. 最后返回 `majority`，即数组中的主要元素。

**复杂度分析：**
这段代码的核心思想是使用哈希表来统计每个元素出现的次数，并找到出现次数最多的元素。由于哈希表的查找操作的平均时间复杂度为 O(1)，因此整个算法的时间复杂度为 O(n)，其中 n 是数组的长度。因此，这是一种时间复杂度较低的方法来找到数组中的主要元素。

<br>
<br>

**需要注意的是：**
选择 `unorderd_map` 是因为时间复杂度为 O(1), 而 `map` 时间复杂度为 O(logn); 因为一个是无序，一个是有序(基于红黑树查询)。

