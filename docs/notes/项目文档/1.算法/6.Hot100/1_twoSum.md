---
title: 1、两数之和
icon: file
order: 1
author: bugcode
date: 2024-11-19T00:00:00.000Z
category:
  - ALGORITHM
  - 算法
tag:
  - algorithm
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2026/01/17 19:50:13
permalink: /project-docs/algorithm/hot100/两数之和/
---

# 两数之和

[两数之和](https://leetcode.cn/problems/two-sum/?envType=study-plan-v2&envId=top-100-liked)

## 题目分析

首先来分析以下题目中隐含的条件：

1. 已知不能两次使用相同的元素
2. 第二，题目中没有明确说明数组是否有序，因此算法需要考虑无序情况，因此不能使用二分法降低复杂度；

## 题解思路一

最容易想到的思路，双层遍历数组循环，如果num[i] + num[j] =  target，就返回i,j数组下标，但是很明显这样的算法复杂度很高，双层循环复杂度达到n2级别。

### go实现

```go
func twoSum2(nums []int, target int) []int {

	/*第一种方式遍历数组*/
	for i := 0; i < len(nums); i++ {
		for j := 0; j < len(nums); j++ {
			if nums[i]+nums[j] == target {
				return []int{i, j}
			}
		}
	}
	return nil
}

func twoSum3(nums []int, target int) []int {

	/*第二种方式遍历数组*/
	for k1, v1 := range nums {
		for k2, v2 := range nums {
			if v1+v2 == target {
				return []int{k1, k2}
			}
		}
	}

	return nil
}
```

## 题解思路二

思路二使用哈希表，num1+num2=target ,我们已知target和其中一个num,因此很容易想到使用哈希表查找target-num的另一个元素是否存在，存在返回下表即可，这种算法能将复杂度降低到o(n)；

### go实现

```go
func twoSum(nums []int, target int) []int {

	/*声明创建一个map key存储nums数组的值，value存储nums数组的下表*/
	idx := map[int]int{}

	/*循环遍历map k表示map的key v表示map的value*/
	for k, v := range nums {
		res := target - v
		/*go中判断map中是否包含某一个元素方法 i表示map中的value ok是true或者false*/
		if i, ok := idx[res]; ok {
			return []int{i, k}
		}
		idx[v] = k
	}
	return nil
}

```

### java实现

~~~java
class Solution {
    public int[] twoSum(int[] nums, int target) {

        // // 使用暴力法
        // // 首先判断数组是否是合法的
        // if(nums == null || nums.length == 0){
        //     return null;
        // }

        // for(int i=0;i< nums.length;i++){
        //     for(int j=i+1;j<nums.length;j++){
        //         if(nums[i]+nums[j] == target){
        //             arr[0]=i;
        //             arr[1]=j;
        //         }
        //     }
        // }
        // return arr;

        // 方法二：借助哈希表

       // HashMap
    // N is the size of nums
    // Time Complexity: O(N)
    // Space COmplexity: O(N)
        int[] result = new int[2];
        // hashmap:如果key相同的话，会做覆盖操作
        HashMap<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            map.put(nums[i], i);
        }
        for (int j = 0; j < nums.length; j++) {
            int diff = target - nums[j];
            if (map.containsKey(diff) && map.get(diff) != j) {
                result[0] = j;
                result[1] = map.get(diff);
                return result;
            }
        }
        return result;

    }
}
~~~

