## [15. 3Sum](https://leetcode-cn.com/problems/3sum/)

Given an array nums of n integers, are there elements a, b, c in nums such that a + b + c = 0? Find all unique triplets in the array which gives the sum of zero.

#### Example:

```
Given array nums = [-1, 0, 1, 2, -1, -4],

A solution set is:
[
  [-1, 0, 1],
  [-1, -1, 2]
]
```

Input: [1,3,2]
Output: "firstsecondthird"
Explanation: The input [1,3,2] means thread A calls first(), thread B calls third(), and thread C calls second(). "firstsecondthird" is the correct output.

#### Note:

 The solution set must not contain duplicate triplets. 

#### Solutions：

```Python
class Solution:
    def threeSum(self, nums: List[int]) -> List[List[int]]:
        n = len(nums)
        nums.sort()
        ans = list()
        
        # 枚举 a
        for first in range(n):
            # 需要和上一次枚举的数不相同
            if first > 0 and nums[first] == nums[first - 1]:
                continue
            # c 对应的指针初始指向数组的最右端
            third = n - 1
            target = -nums[first]
            # 枚举 b
            for second in range(first + 1, n):
                # 需要和上一次枚举的数不相同
                if second > first + 1 and nums[second] == nums[second - 1]:
                    continue
                # 需要保证 b 的指针在 c 的指针的左侧
                while second < third and nums[second] + nums[third] > target:
                    third -= 1
                # 如果指针重合，随着 b 后续的增加
                # 就不会有满足 a+b+c=0 并且 b<c 的 c 了，可以退出循环
                if second == third:
                    break
                if nums[second] + nums[third] == target:
                    ans.append([nums[first], nums[second], nums[third]])
        
        return ans
```

#### refer：

 https://leetcode-cn.com/problems/3sum/solution/san-shu-zhi-he-by-leetcode-solution/ 

- 暴力解法 

时间复杂度$O(n^3)$ 空间复杂度$O(1)$

- 双指针法

时间复杂度$O(n^2)$ (包含排序$O(nlogn)$和搜索解$O(n^2)$)  空间复杂度$O(1)$

关键字：不可以包含重复（先排序，并跳过重复值）

模式识别：利用排序避免重复答案

降低复杂度变成twoSum

利用双指针找到所有解