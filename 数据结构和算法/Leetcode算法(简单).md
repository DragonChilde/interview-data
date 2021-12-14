# 两数之和

给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 target  的那 两个 整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。

你可以按任意顺序返回答案。

```
输入：nums = [2,7,11,15], target = 9
输出：[0,1]
解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。
示例 2：

输入：nums = [3,2,4], target = 6
输出：[1,2]
示例 3：

输入：nums = [3,3], target = 6
输出：[0,1]
```

```java
package com.spring.annotation;

import java.util.HashMap;

/**
 * @title: TheSumOfTwoNumbers
 * @Author Wen
 * @Date: 14/12/2021 下午4:55
 * @Version 1.0
 */
public class TheSumOfTwoNumbers {

    public static void main(String[] args) {


        int[] ints = twoSum(new int[]{2, 7, 11, 15}, 9);

        //int[] ints =twoSum(new int[]{3,2,4},6);

        //int[] ints = twoSum(new int[]{3, 3}, 6);


        for (int num : ints) {
            System.out.println(num);
        }
        
    }

    /**
     * 暴力解法(不建议,下面哈希表为最优方法)
     *
     * @param nums
     * @param target
     * @return
     */
/*    public static int[] twoSum(int[] nums, int target) {

        for (int i = 0; i < nums.length - 1; i++) {
            for (int j = i + 1; j < nums.length; j++) {
                if (nums[i] + nums[j] == target) {
                    return new int[]{i, j};
                }

            }
        }


        throw new IllegalArgumentException("no solution");
    }*/

    /**
     * 哈希解法
     *
     * @param nums
     * @param target
     * @return
     */
    public static int[] twoSum(int[] nums, int target) {


        int[] index = new int[2];
        HashMap<Integer, Integer> map = new HashMap<>();

        for (int i = 0; i < nums.length; i++) {

            int num = nums[i];
            int cut = target - num;
            if (map.containsKey(cut)) {

                return new int[]{map.get(cut), i};

            }
            map.put(nums[i], i);
        }
        throw new IllegalArgumentException("no solution");
    }
}
```

