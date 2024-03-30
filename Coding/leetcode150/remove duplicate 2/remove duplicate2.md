# 80. 删除有序数组中的重复项 II
**(mid)**

---

给你一个有序数组 `nums` ，请你 **原地** 删除重复出现的元素，使得出现次数超过两次的元素**只出现两次** ，返回删除后数组的新长度。

不要使用额外的数组空间，你必须在 **原地** 修改输入数组 并在使用 **O(1)** 额外空间的条件下完成。


<br>
<br>
<br>


**示例 1：**

    输入：nums = [1,1,1,2,2,3]
    输出：5, nums = [1,1,2,2,3]
    解释：函数应返回新长度 length = 5, 并且原数组的前五个元素被修改为 1, 1, 2, 2, 3。 不需要考虑数组中超出新长度后面的元素。

**示例 2：**

    输入：nums = [0,0,1,1,1,1,2,3,3]
    输出：7, nums = [0,0,1,1,2,3,3]
    解释：函数应返回新长度 length = 7, 并且原数组的前七个元素被修改为 0, 0, 1, 1, 2, 3, 3。不需要考虑数组中超出新长度后面的元素。


<br>
<br>
<br>


### 解题方法

1. 🙆同样用双指针
2. 指针从 第三位开始， 因为重复两次，所以最短可能是 0， 1， 2这样（不需要做算法的，直接返回）
3. 其他情况，因为从第三位开始比, `slow-2` 的元素与 当前 `fast` 比，如果一样，说明重复至少有三个以上; 如果不一样，则 让 nums[slow] = nums[fast]，这样在一开始也适用。
4. 此时，`fast` 应该向前移动一位，继续向前，一直找到尽头位置，即 找到不一样的元素 或 到数组末尾。
5. `fast` 移动后，所指的位置值 不等于 `slow-2`，即 `nums[slow-2] != nums[fast]` ，则继续 3. ，并且 slow++
6. 跳出当前if，`fast++` , 继续当前 `while` 循环， 继续重复，最后跳出循环，返回 `slow` 即为有效位置。
   

#### 需要注意的是，`fast++` 不能在else中，否则当前重复太多，会存在下一种不同元素开始 无法去重复

比如下下面这个反面案例:

```cpp{.line-numbers}
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {

        int len = nums.size();
        int fast = 2, slow = 2;

        if(len <= 1){
            return len;
        }

        while(fast < len){
            if(nums[slow-2] != nums[fast]){
                nums[slow] = nums[fast];
                slow++;
            }
            else{
                fast++; // this is a bug for duplicate that aren't remove 
            }
            

        }
        return slow;

    }
};
```


**正确解法:**

```cpp{.line-numbers}
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {

        int len = nums.size();
        int fast = 2, slow = 2;

        if(len <= 1){
            return len;
        }

        while(fast < len){
            if(nums[slow-2] != nums[fast]){
                nums[slow] = nums[fast];
                slow++;
            }
            
            fast++; //  not in else, because whatever it should advance

        }
        return slow;

    }
};

```