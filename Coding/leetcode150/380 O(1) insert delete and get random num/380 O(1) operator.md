# 380. O(1) 时间插入、删除和获取随机元素
(mid)
感觉还是很重要的，因为可能会经常用到

变长数组 + hash 表 实现 O(1) 的operator

（**实际的operator在 hash，但实际存放元素的地方在 数组中**）

---


### 题目描述
实现RandomizedSet 类：

- RandomizedSet() 初始化 RandomizedSet 对象
- bool insert(int val) 当元素 val 不存在时，向集合中插入该项，并返回 true ；否则，返回 false 。
- bool remove(int val) 当元素 val 存在时，从集合中移除该项，并返回 true ；否则，返回 false 。
- int getRandom() 随机返回现有集合中的一项（测试用例保证调用此方法时集合中至少存在一个元素）。每个元素应该有 相同的概率 被返回。

你必须实现类的所有函数，并满足每个函数的 平均 时间复杂度为 O(1) 。


<br>
<br>



### 算法思想
**变长数组 + hash table**

当我们需要一个数据结构，能够快速插入、删除和获取随机元素时，常见的方案是结合使用变长数组和哈希表 （**删除，插入什么的实际上在hash中实现**）。

首先，我们知道**变长数组可以快速地获取随机元素**，**但是无法快速判断一个元素是否存在或在哪个位置。( ==也就说，导致查找是否存在最少需要O(n)量级== )** 相反，**哈希表可以很快地进行插入和删除操作，但是不能像数组那样直接通过下标来获取元素 (==因为 hash 是可以直接通过 key 找到 value 查找复杂度是 O(1)== )**。

为了解决这个问题，我们可以将元素存储在数组中，同时使用哈希表来记录每个元素在数组中的位置（即下标）。

当我们要插入一个新元素时，我们首先检查哈希表中是否已经存在这个元素，如果已经存在，则说明重复插入了，直接返回失败；如果不存在，则将元素添加到数组的末尾，并在哈希表中记录该元素的下标。这样，插入操作就完成了。

删除操作时，我们先检查哈希表中是否存在要删除的元素，如果不存在，则说明无法删除；**如果存在，则将数组中该元素的位置与数组末尾的元素交换，然后在数组中删除末尾元素，同时更新哈希表中的对应关系。** 这样，删除操作也完成了。（==**难点**==）

至于获取随机元素操作，由于数组中的元素下标是连续的，所以我们只需要随机选取一个下标，就可以直接在数组中获取到对应的元素了。

这样，通过结合变长数组和哈希表，我们就可以实现在 O(1) 的时间复杂度内完成插入、删除和获取随机元素的操作了。


<br>
<br>
<br>


### 代码
```cpp
class RandomizedSet {
public:
    RandomizedSet() {
        srand((unsigned)time(NULL));    // 设置随机种子
    }
    
    /**
        将元素存储在数组中，同时使用哈希表来记录每个元素在数组中的位置（即下标）
    **/
    bool insert(int val) {
        if(indices.count(val)){ //如果存在，则无法删除
            return false;
        }
        int index = nums.size();
        nums.emplace_back(val); //  尾部直接构造 新插入的 val
        indices[val] = index; // hash table 中存放 下标
        return true; //插入成功
    }
    
    bool remove(int val) {
        if(!indices.count(val)){    //元素不存在，无法删除
            return false;
        }
        // 需要删除的元素，在nums的下标是多少？
        int index = indices[val]; 	

        // 取最后的元素来作为中间变量，后续用来替换数组中需要删除的元素
        int temp = nums.back(); 	

        //之前不是拿到删除元素的index了吗，直接根据index找到，然后给他覆盖呗（理论上要删除的元素已经不在了）
        nums[index] = temp;	

        //覆盖后，要把temp的index更新一下吧？因为末尾不是还有一个吗，后尾那个肯定删掉的，那就直接用原来这个位置上的
        indices[temp] = index;  

        //这时候基本完事了，但是还得吧之前遗留的最后尾的删掉吧？这不是因为有两个，所以
        nums.pop_back();

        //实际存放的nums中，尾巴是去掉了，但是 hash table 也要管一下吧？把val这个键去掉
        indices.erase(val);

        return true;
    }
    
    int getRandom() {
        int randomIndex = rand()%nums.size();   // rand实现随机，取模其实也是为了限制 范围
        return nums[randomIndex];
    }

private:
    vector<int> nums;   //变长数组
    unordered_map<int, int> indices;    //hash表
};

```

<br>

**remove 函数稳重一点的逐行解释:**

这个 `remove` 函数用于删除指定值的元素。下面是它的逐行解释：

1. `if (!indices.count(val)) { return false; }`：首先，我们检查哈希表 `indices` 中是否存在要删除的值 `val`。如果哈希表中不存在该值，说明集合中没有要删除的元素，直接返回 false。

2. `int index = indices[val];`：如果哈希表中存在要删除的值，我们从哈希表中获取该值在数组 `nums` 中的索引，并将其保存在变量 `index` 中。

3. `int last = nums.back();`：然后，我们从数组 `nums` 中获取最后一个元素，并将其保存在变量 `last` 中。我们将用它来替换要删除的值。

4. `nums[index] = last;`：我们将要删除的值的位置（由 `index` 指定）替换为数组中的最后一个元素。

5. `indices[last] = index;`：同时，我们也要更新哈希表中最后一个元素的索引，将其指向原来要删除的值的位置。

6. `nums.pop_back();`：接着，我们从数组 `nums` 中移除最后一个元素，因为我们已经将它移到了要删除的值的位置。

7. `indices.erase(val);`：最后，我们从哈希表中移除要删除的值。

8. `return true;`：删除操作完成，返回 true 表示删除成功。

这样，通过将要删除的元素与数组中的最后一个元素交换，并同时更新哈希表中的索引，我们可以在 O(1) 的时间复杂度内完成删除操作。
