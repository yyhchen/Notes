# 189. 轮转数组
**（mid）**

---

给定一个整数数组 `nums`，将数组中的元素向右轮转 `k` 个位置，其中 `k` 是非负数.

<br>
<br>
<br>

**示例 1:**

    输入: nums = [1,2,3,4,5,6,7], k = 3
    输出: [5,6,7,1,2,3,4]
    解释:
    向右轮转 1 步: [7,1,2,3,4,5,6]
    向右轮转 2 步: [6,7,1,2,3,4,5]
    向右轮转 3 步: [5,6,7,1,2,3,4]
<br>

**示例 2:**

    输入：nums = [-1,-100,3,99], k = 2
    输出：[3,99,-1,-100]
    解释: 
    向右轮转 1 步: [99,-1,-100,3]
    向右轮转 2 步: [3,99,-1,-100]


<br>
<br>

### 算法思想：
1. 新建一个临时数据复制 `nums` 的值
2. 计算真正轮转的位置，通过 `mod` 计算得到，`k %= len` 即为轮转位置。比如：`k=10`, `len=7` ==实际上就是轮转了 3 步==。举例 `nums = [1, 2, 3, 4, 5, 6, 7]` 轮转后为 `[5, 6, 7, 1, 2, 3, 4]`
3. 通过 `for` 循环逐个将元素对应 `nums[i] = temp[(len - k + i) % len];` 放入即可。这里通过 `%len` 是为了保证 实际的位置，因为如果 `k` 太大会超过 `len`。

<br>
<br>


### 代码：
```cpp{.line-numbers}
class Solution {
public:
    void rotate(vector<int>& nums, int k) {
        int len = nums.size();
        vector<int> temp = nums; // 创建一个和 nums 大小相同的额外数组

        // 根据旋转步数 k 和数组长度 len，计算正确的旋转位置
        k %= len;

        // 将 temp 数组中的元素按正确顺序赋值给 nums 数组
        for (int i = 0; i < len; ++i) {
            nums[i] = temp[(len - k + i) % len];
        }
    }
};

```

