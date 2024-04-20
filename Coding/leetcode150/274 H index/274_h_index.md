# 274. H 指数
(mid)

---

### 题目
给你一个整数数组 `citations` ，其中 `citations[i]` 表示研究者的第 `i` 篇论文被引用的次数。计算并返回该研究者的 `h` 指数。

根据维基百科上 h 指数的定义：h 代表“高引用次数” ，一名科研人员的 h 指数 是指他（她）**至少**发表了 h 篇论文，并且 **至少** 有 h 篇论文被引用次数大于等于 h 。如果 h 有多种可能的值，h 指数 是其中最大的那个。

<br>

**示例 1：**

    输入：citations = [3,0,6,1,5]
    输出：3 
    解释：给定数组表示研究者总共有 5 篇论文，每篇论文相应的被引用了 3, 0, 6, 1, 5 次。
        由于研究者有 3 篇论文每篇 至少 被引用了 3 次，其余两篇论文每篇被引用 不多于 3 次，所以她的 h 指数是 3。
**示例 2：**

    输入：citations = [1,3,1]
    输出：1


<br>
<br>
<br>


### 算法思想

- **排序**
先给数组排序（降序），然后核心思想就是 数组的index就相当于 是 h-index，只要当前 `citations[i] >= i+1` 就更新 `h-index` 为 `i+1` , 此时是至少 `h-index` 是 `i+1`, 并且设置一个早停 `break`, 一旦当前的 `citations[i]` 小于 数组索引(其实就是 `h-index`)， 那么此时已经找到最大的那个(因为是降序排序的)。

```cpp
class Solution {
public:
    int hIndex(vector<int>& citations) {
        sort(citations.begin(), citations.end(), [](int a, int b){
            return a > b;
        });
        int h = 0;
        for(int i = 0; i < citations.size(); i++){
            if(citations[i] >= i+1){
                h = i+1;    // h指数至少是 i+1
            }
            else{
                break;  //早停
            }
        }
        return h;
    }
};
```

<br>

- 计数排序

- 二分搜索