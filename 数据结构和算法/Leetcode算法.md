# 数组

## 两数之和(#1)(简单)

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

## 三数之和(#15)

给你一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？请你找出所有和为 0 且不重复的三元组。

注意：答案中不可以包含重复的三元组。

```
示例 1：
输入：nums = [-1,0,1,2,-1,-4]
输出：[[-1,-1,2],[-1,0,1]]

示例 2：
输入：nums = []
输出：[]


示例 3：
输入：nums = [0]
输出：[]
```

```java
public class ThreeSum {

    // 双指针法
    public List<List<Integer>> threeSum(int[] nums){
        int n = nums.length;

        List<List<Integer>> result = new ArrayList<>();

        // 0. 先对数组排序
        Arrays.sort(nums);

        // 1. 遍历每一个元素，作为当前三元组中最小的那个（最矮个做核心）
        for (int i = 0; i < n; i++){
            // 1.1 如果当前数已经大于0，直接退出循环
            if (nums[i] > 0 ) break;

            // 1.2 如果当前数据已经出现过，直接跳过
            if ( i > 0 && nums[i] == nums[i-1] ) continue;

            // 1.3 常规情况，以当前数做最小数，定义左右指针
            int lp = i + 1;
            int rp = n - 1;
            // 只要左右指针不重叠，就继续移动指针
            while (lp < rp){
                int sum = nums[i] + nums[lp] + nums[rp];
                // 判断sum，与0做大小对比
                if (sum == 0){
                    // 1.3.1 等于0，就是找到了一组解
                    result.add(Arrays.asList(nums[i], nums[lp], nums[rp]));
                    lp ++;
                    rp --;
                    // 如果移动之后的元素相同，直接跳过
                    while (lp < rp && nums[lp] == nums[lp - 1]) lp++;
                    while (lp < rp && nums[rp] == nums[rp + 1]) rp--;
                }
                // 1.3.2 小于0，较小的数增大，左指针右移
                else if(sum < 0)
                    lp ++;
                // 1.3.3 大于0，较大的数减小，右指针左移
                else
                    rp --;
            }
        }
        return result;
    }

    public static void main(String[] args) {
        int[] input = {-1, 0, 1, 2, -1, -4};

        ThreeSum threeSum = new ThreeSum();

        System.out.println(threeSum.threeSum(input));

    }
}

```

## 下一个排列(#31)

实现获取 下一个排列 的函数，算法需要将给定数字序列重新排列成字典序中下一个更大的排列（即，组合出下一个更大的整数）。

如果不存在下一个更大的排列，则将数字重新排列成最小的排列（即升序排列）。

必须 原地 修改，只允许使用额外常数空间。

```
示例 1：
输入：nums = [1,2,3]
输出：[1,3,2]

示例 2：
输入：nums = [3,2,1]
输出：[1,2,3]

示例 3：
输入：nums = [1,1,5]
输出：[1,5,1]

示例 4：
输入：nums = [1]
输出：[1]

```

```java
public class NextPermutation {

    // 思路：从后向前找到升序子序列，然后确定调整后子序列的最高位，剩余部分升序排列
    public void nextPermutation1(int[] nums){
        int n = nums.length;

        // 1. 从后向前找到升序子序列，找到第一次下降的数，位置记为k
        int k = n - 2;
        while ( k >= 0 && nums[k] >= nums[k+1] )
            k --;

        // 找到k，就是需要调整部分的最高位

        // 2. 如果k = -1，说明所有数降序排列，改成升序排列
        if ( k == -1 ){
            Arrays.sort(nums);
            return;
        }

        // 3. 一般情况，k >= 0
        // 3.1 依次遍历剩余降序排列的部分，找到要替换最高位的那个数
        int i = k + 2;
        while ( i < n && nums[i] > nums[k] )
            i ++;

        // 当前的i，就是后面部分第一个比nums[k]小的数，i-1就是要替换的那个数

        // 3.2 交换i-1和k位置上的数
        int temp = nums[k];
        nums[k] = nums[i-1];
        nums[i-1] = temp;

        // 3.3 k之后的剩余部分变成升序排列，直接前后调换
        int start = k + 1;
        int end = n - 1;
        while ( start < end ){
            int tmp = nums[start];
            nums[start] = nums[end];
            nums[end] = tmp;
            start ++;
            end --;
        }
    }

    // 方法改进：将降序数组翻转的操作提取出来
    public void nextPermutation(int[] nums){
        int n = nums.length;

        // 1. 从后向前找到升序子序列，找到第一次下降的数，位置记为k
        int k = n - 2;
        while ( k >= 0 && nums[k] >= nums[k+1] )
            k --;

        // 找到k，就是需要调整部分的最高位

        // 2. 如果k = -1，说明所有数降序排列，改成升序排列
        if ( k == -1 ){
            reverse(nums, 0, n - 1);
            return;
        }

        // 3. 一般情况，k >= 0
        // 3.1 依次遍历剩余降序排列的部分，找到要替换最高位的那个数
        int i = k + 2;
        while ( i < n && nums[i] > nums[k] )
            i ++;

        // 当前的i，就是后面部分第一个比nums[k]小的数，i-1就是要替换的那个数

        // 3.2 交换i-1和k位置上的数
        swap(nums, k, i - 1);

        // 3.3 k之后的剩余部分变成升序排列，直接前后调换
        reverse(nums, k + 1, n - 1);
    }

    // 定义一个方法，交换数组中两个元素
    private void swap( int[] nums, int i, int j ){
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
    // 定义一个翻转数组的方法
    private void reverse( int[] nums, int start, int end ){
        while ( start < end ){
            swap(nums, start, end);
            start ++;
            end --;
        }
    }

    public static void main(String[] args) {
        int[] nums = {2,4,4,3,3,2,1};

        NextPermutation permutation = new NextPermutation();

        permutation.nextPermutation(nums);

        for ( int num: nums )
            System.out.print(num + "\t");
    }
}

```

## 旋转图像(#48)

给定一个 n × n 的二维矩阵 matrix 表示一个图像。请你将图像顺时针旋转 90 度。

你必须在 原地 旋转图像，这意味着你需要直接修改输入的二维矩阵。请不要 使用另一个矩阵来旋转图像。

```
输入：matrix = 
    [
        [1,2,3],
        [4,5,6],
        [7,8,9]
    ]
输出：
	[
		[7,4,1],
		[8,5,2],
		[9,6,3]
	]

输入：matrix = 
    [
        [5,1,9,11],
        [2,4,8,10],
        [13,3,6,7],
        [15,14,12,16]
    ]
输出：
    [
        [15,13,2,5],
        [14,3,4,1],
        [12,6,8,9],
        [16,7,10,11]
    ]

```

```java
public class RotateImage {

    // 方法一：数学方法（矩阵转置，再翻转每一行）
    public void rotate1(int[][] matrix){
        int n = matrix.length;

        // 1. 转置矩阵
        for (int i = 0; i < n; i++){
            for (int j = i; j < n; j++){
                int temp = matrix[i][j];
                matrix[i][j] = matrix[j][i];
                matrix[j][i] = temp;
            }
        }
        // 2. 翻转每一行
        for (int i = 0; i < n; i++){
            for (int j = 0; j < n/2; j++){
                int temp = matrix[i][j];
                matrix[i][j] = matrix[i][n-j-1];
                matrix[i][n-j-1] = temp;
            }
        }
    }

    // 方法二：分治思想，分为四个子矩阵分别考虑
    public void rotate2(int[][] matrix){
        int n = matrix.length;

        // 遍历四分之一矩阵，左上角
        for (int i = 0; i < n/2 + n%2; i++){
            for (int j = 0; j < n/2; j++){
                // 对于matrix[i][j], 需要找到不同的四个矩阵中对应的另外三个位置和元素
                // 定义一个临时数组，保存对应的四个元素
                int[] temp = new int[4];
                int row = i;
                int col = j;
                // 行列转换的规律：row + newCol  = n - 1, col = newRow
                for ( int k = 0; k < 4; k++ ){
                    temp[k] = matrix[row][col];
                    int x = row;
                    row = col;
                    col = n - 1 - x;
                }

                // 再次遍历要处理的四个位置，将旋转之后的数据填入
                for (int k = 0; k < 4; k++){
                    // 用上一个值替换当前的位置
                    matrix[row][col] = temp[(k+3)%4];
                    int x = row;
                    row = col;
                    col = n - 1 - x;
                }
            }
        }
    }

    // 方法三：改进
    public void rotate(int[][] matrix){
        int n = matrix.length;

        // 遍历四分之一矩阵
        for (int i = 0; i < (n+1)/2; i++){
            for (int j = 0; j < n/2; j++){
                
                int temp = matrix[i][j];

                matrix[i][j] = matrix[n-j-1][i];    // 将上一个位置的元素填入

                matrix[n-j-1][i] = matrix[n-i-1][n-j-1];

                matrix[n-i-1][n-j-1] = matrix[j][n-i-1];

                matrix[j][n-i-1] = temp;
            }
        }
    }

    public static void main(String[] args) {
        int[][] image1 = {
                {1,2,3},
                {4,5,6},
                {7,8,9}
        };
        int[][] image2 = {
                { 5, 1, 9,11},
                { 2, 4, 8,10},
                {13, 3, 6, 7},
                {15,14,12,16}
        };

        RotateImage rotateImage = new RotateImage();
        rotateImage.rotate(image1);

        rotateImage.printImage(image1);

        rotateImage.rotate(image2);
        rotateImage.printImage(image2);

    }

    private void printImage(int[][] image){
        System.out.println("image: ");
        for ( int[] line: image ){
            for ( int point: line ){
                System.out.print(point + "\t");
            }
            System.out.println();
        }
    }
}

```

## 搜索二维矩阵(#74)

编写一个高效的算法来判断 m x n 矩阵中，是否存在一个目标值。该矩阵具有如下特性：

每行中的整数从左到右按升序排列。
每行的第一个整数大于前一行的最后一个整数。

```
示例 1
输入：matrix = [
	[1,3,5,7],
	[10,11,16,20],
	[23,30,34,60]
	], target = 3
输出：true

示例 2
输入：matrix = [
	[1,3,5,7],
	[10,11,16,20],
	[23,30,34,60]
	], target = 13
输出：false
```

```java
class SearchMatrix {
    public boolean searchMatrix( int[][] matrix, int target ){
        // 先定义m和n
        int m = matrix.length;
        if (m == 0) return false;
        int n = matrix[0].length;

        // 二分查找，定义左右指针
        int left = 0;
        int right = m * n - 1;

        while ( left <= right ){
            // 计算中间位置
            int mid = (left + right) / 2;
            // 计算二维矩阵中对应的行列号，取出对应元素
            int midElement = matrix[mid/n][mid%n];

            // 判断中间元素与target的大小关系
            if (midElement < target)
                left = mid + 1;
            else if (midElement > target)
                right = mid - 1;
            else
                return true;
        }
        return false;
    }

    public static void main(String[] args) {
        int[][] matrix = {{1,3,5,7}, {10,11,16,20}, {23,30,34,50}};
        int target = 13;
        SearchMatrix searchMatrix = new SearchMatrix();

        System.out.println(searchMatrix.searchMatrix(matrix, target));
    }
}
```

## 寻找重复数(#287)

给定一个包含 n + 1 个整数的数组 nums ，其数字都在 1 到 n 之间（包括 1 和 n），可知至少存在一个重复的整数。

假设 nums 只有 一个重复的整数 ，找出 这个重复的数 。

你设计的解决方案必须不修改数组 nums 且只用常量级 O(1) 的额外空间。

```
示例 1：

输入：nums = [1,3,4,2,2]
输出：2

示例 2：
输入：nums = [3,1,3,4,2]

输出：3

示例 3：
输入：nums = [1,1]
输出：1

示例 4：
输入：nums = [1,1,2]
输出：1
```

```java
class FindDuplicatedNumber {
	// 方法五：快慢指针
    public int findDuplicate(int[] nums){
        // 定义快慢指针
        int fast = 0, slow = 0;
        // 1. 寻找环内的相遇点
        do {
            // 快指针一次走两步，慢指针一次走一步
            slow = nums[slow];
            fast = nums[nums[fast]];
        }while (fast != slow);

        // 循环结束，slow和fast相等，都是相遇点
        // 2. 寻找环的入口点
        // 另外定义两个指针，固定间距
        int before = 0, after = slow;
        while (before != after){
            before = nums[before];
            after = nums[after];
        }

        // 循环结束，相遇点就是环的入口点，也就是重复元素
        return before;
    }

    public static void main(String[] args) {
        int[] nums1 = {1,3,4,2,2};
        int[] nums2 = {1,1};

        FindDuplicatedNumber findDuplicatedNumber = new FindDuplicatedNumber();
        System.out.println(findDuplicatedNumber.findDuplicate(nums1));
        System.out.println(findDuplicatedNumber.findDuplicate(nums2));
    }
    
}
```

# 字符串

## 字符串相加(#415)(简单)

给定两个字符串形式的非负整数 num1 和num2 ，计算它们的和并同样以字符串形式返回。

你不能使用任何內建的用于处理大整数的库（比如 BigInteger）， 也不能直接将输入的字符串转换为整数形式。

```
示例 1：

输入：num1 = "11", num2 = "123"
输出："134"

示例 2：

输入：num1 = "456", num2 = "77"
输出："533"

示例 3：

输入：num1 = "0", num2 = "0"
输出："0"
```

```java
public class AddStrings {
    public String addStrings(String num1, String num2){
        // 定义一个StringBuffer，保存最终的结果
        StringBuffer result = new StringBuffer();

        // 定义遍历两个字符串的初始位置
        int i = num1.length() - 1;
        int j = num2.length() - 1;
        int carry = 0;    // 用一个变量保存当前的进位

        // 从个位开始依次遍历所有数位，只要还有数没有计算，就继续；其他数位补0
        while ( i >= 0 || j >= 0 || carry != 0 ){
            // 取两数当前的对应数位
            int n1 = i >= 0 ? num1.charAt(i) - '0' : 0;     // 字符要将ascii码转换为数字
            int n2 = j >= 0 ? num2.charAt(j) - '0' : 0;

            // 对当前数位求和
            int sum = n1 + n2 + carry;

            // 把sum的个位保存到结果中，十位作为进位保存下来
            result.append(sum % 10);
            carry = sum / 10;

            // 移动指针，继续遍历下一位
            i --;
            j --;
        }

        return result.reverse().toString();
    }

    public static void main(String[] args) {
        String num1 = "745346";
        String num2 = "8564";

        AddStrings addStrings = new AddStrings();
        System.out.println(addStrings.addStrings(num1, num2));

//        char c = '0';
//        int i = c;
//        System.out.println(c);
//        System.out.println(i);
    }
}
```

## 字符串相乘(#43)

给定两个以字符串形式表示的非负整数 num1 和 num2，返回 num1 和 num2 的乘积，它们的乘积也表示为字符串形式。

```
示例 1:

输入: num1 = "2", num2 = "3"
输出: "6"

示例 2:

输入: num1 = "123", num2 = "456"
输出: "56088"

说明：

num1 和 num2 的长度小于110。
num1 和 num2 只包含数字 0-9。
num1 和 num2 均不以零开头，除非是数字 0 本身。
不能使用任何标准库的大数类型（比如 BigInteger）或直接将输入转换为整数来处理。
```

```java
/**
 * @title: MultiplyStrings
 * @Author Wen
 * @Date: 21/12/2021 下午5:28
 * @Version 1.0
 */
public class MultiplyStrings {

    public String multiply(String num1, String num2) {
        // 处理特殊情况，有一个乘数为0，结果就为0
        if (num1.equals("0") || num2.equals("0")) return "0";

        // 定义一个数组，保存计算结果的每一位
        int[] resultArray = new int[num1.length() + num2.length()];

        // 遍历num1和num2的每个数位，做乘积，然后找到对应数位，填入结果数组
        for (int i = num1.length() - 1; i >= 0; i--){
            int n1 = num1.charAt(i) - '0';
            for (int j = num2.length() - 1; j >= 0; j--){
                int n2 = num2.charAt(j) - '0';
                // 计算乘积
                int product = n1 * n2;

                // 保存到结果数组
                int sum = product + resultArray[i+j+1];
                resultArray[i+j+1] = sum % 10;    // 叠加结果的个位保存到i+j+1位置
                resultArray[i+j] += sum / 10;
            }
        }

        // 将结果数组转成String输出
        StringBuffer result = new StringBuffer();
        int start = resultArray[0] == 0 ? 1 : 0;    // 如果最高位为0，从1开始遍历
        for (int i = start; i < resultArray.length; i++)
            result.append(resultArray[i]);

        return result.toString();
    }

    public static void main(String[] args) {
        String num1 = "123";
        String num2 = "456";

        MultiplyStrings multiplyStrings = new MultiplyStrings();
        System.out.println(multiplyStrings.multiply(num1, num2));

    }
}
```

## 去除重复字母(#316)

给你一个字符串 s ，请你去除字符串中重复的字母，使得每个字母只出现一次。需保证 返回结果的字典序最小（要求不能打乱其他字符的相对位置）。

注意：该题与 1081 https://leetcode-cn.com/problems/smallest-subsequence-of-distinct-characters 相同

```
示例 1：

输入：s = "bcabc"
输出："abc"

示例 2：

输入：s = "cbacdcbc"
输出："acdb"
```

```java
public class RemoveDuplicateLetters {

    // 方法三：使用栈进行优化
    public String removeDuplicateLetters(String s){
        // 定义一个字符栈，保存去重之后的结果
        Stack<Character> stack = new Stack<>();

        // 为了快速判断一个字符是否在栈中出现过，用一个Set来保存元素是否出现
        HashSet<Character> seenSet = new HashSet<>();

        // 为了快速判断一个字符是否在某个位置之后重复出现，用一个HashMap来保存字母出现在字符串的最后位置
        HashMap<Character, Integer> lastOccur = new HashMap<>();

        // 遍历字符串，将最后一次出现的位置保存进map
        for (int i = 0; i < s.length(); i++){
            lastOccur.put(s.charAt(i), i);
        }

        // 遍历字符串，判断每个字符是否需要入栈
        for (int i = 0; i < s.length(); i++){
            char c = s.charAt(i);
            // 只有在c没有出现过的情况下，才判断是否入栈
            if (!seenSet.contains(c)){
                // c入栈之前，要判断之前栈顶元素，是否在后面会重复出现；如果重复出现就可以删除
                while ( !stack.isEmpty() && c < stack.peek() && lastOccur.get(stack.peek()) > i ){
                    seenSet.remove(stack.pop());
                }
                stack.push(c);
                seenSet.add(c);
            }
        }

        // 将栈中的元素保存成字符串输出
        StringBuilder stringBuilder = new StringBuilder();
        for (Character c: stack)
            stringBuilder.append(c.charValue());

        return stringBuilder.toString();
    }

    public static void main(String[] args) {
        String str1 = "bcabc";
        String str2 = "cbacdcbc";
        RemoveDuplicateLetters removeDuplicateLetters = new RemoveDuplicateLetters();

        System.out.println(removeDuplicateLetters.removeDuplicateLetters(str1));
        System.out.println(removeDuplicateLetters.removeDuplicateLetters(str2));
    }
}
```

# 滑动窗口

## 滑动窗口最大值(#239)(困难)

```
给你一个整数数组 nums，有一个大小为 k 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 k 个数字。滑动窗口每次只向右移动一位。

返回滑动窗口中的最大值。

 
示例 1：

输入：nums = [1,3,-1,-3,5,3,6,7], k = 3
输出：[3,3,5,5,6,7]
解释：
滑动窗口的位置                最大值
---------------               -----
[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7
示例 2：

输入：nums = [1], k = 1
输出：[1]
示例 3：

输入：nums = [1,-1], k = 1
输出：[1,-1]
示例 4：

输入：nums = [9,11], k = 2
输出：[11]
示例 5：

输入：nums = [4,-2], k = 2
输出：[4]

```

```java
public class SlidingWindowMaximum {

    // 方法二：使用大顶堆
    public int[] maxSlidingWindow(int[] nums, int k){
        // 定义一个结果数组
        int[] result = new int[nums.length - k + 1];

        // 用优先队列实现一个大顶堆
        PriorityQueue<Integer> maxHeap = new PriorityQueue<>(k, new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                return o2 - o1;
            }
        });

        // 准备工作：构建大顶堆，将第一个窗口元素（前k个）放入堆中
        for (int i = 0; i < k; i++)
            maxHeap.add(nums[i]);

        // 当前大顶堆的堆顶元素就是第一个窗口的最大值
        result[0] = maxHeap.peek();

        // 遍历所有窗口
        for (int i = 1; i <= nums.length - k; i++){
            maxHeap.remove(nums[i-1]);    // 删除堆中上一个窗口的第一个元素
            maxHeap.add(nums[i+k-1]);    // 添加当前窗口的最后一个元素进堆
            result[i] = maxHeap.peek();
        }

        return result;
    }


    public static void main(String[] args) {
        int[] input = {1,3,-1,-3,5,3,6,7};
        int k = 3;

        SlidingWindowMaximum slidingWindowMaximum = new SlidingWindowMaximum();

        int[] output = slidingWindowMaximum.maxSlidingWindow(input, k);

        for (int max: output)
            System.out.print(max + "\t");
    }
}
```

## 最小覆盖子串(#76)(困难)

给你一个字符串 `s` 、一个字符串 `t` 。返回 `s` 中涵盖 `t` 所有字符的最小子串。如果 `s` 中不存在涵盖 `t` 所有字符的子串，则返回空字符串 `"`

```
注意：

对于 t 中重复字符，我们寻找的子字符串中该字符数量必须不少于 t 中该字符数量。
如果 s 中存在这样的子串，我们保证它是唯一的答案。
 

示例 1：

输入：s = "ADOBECODEBANC", t = "ABC"
输出："BANC"
示例 2：

输入：s = "a", t = "a"
输出："a"
示例 3:

输入: s = "a", t = "aa"
输出: ""

解释: t 中两个字符 'a' 均应包含在 s 的子串中，
因此没有符合条件的子字符串，返回空字符串。
```

```java
public class MinWindowSubstring {
    
    // 方法四：进一步优化
    public String minWindow(String s, String t) {
        // 定义最小子串，保存结果，初始为空字符串
        String minSubString = "";

        // 定义一个HashMap，保存t中字符出现的频次
        HashMap<Character, Integer> tCharFrequency = new HashMap<>();

        // 统计t中字符频次
        for (int i = 0; i < t.length(); i++) {
            char c = t.charAt(i);
            int count = tCharFrequency.getOrDefault(c, 0);
            tCharFrequency.put(c, count + 1);
        }

        // 定义左右指针，指向滑动窗口的起始和结束位置
        int start = 0, end = 1;

        // 定义一个HashMap，保存s子串中字符出现的频次
        HashMap<Character, Integer> subStrCharFrequency = new HashMap<>();

        // 定义一个“子串贡献值”变量，统计t中的字符在子串中出现了多少
        int charCount = 0;

        while (end <= s.length()) {

            // end增加之后，新增的字符
            char newChar = s.charAt(end - 1);

            // 新增字符频次加1
            if (tCharFrequency.containsKey(newChar)) {
                subStrCharFrequency.put(newChar, subStrCharFrequency.getOrDefault(newChar, 0) + 1);
                // 如果子串中频次小于t中频次，当前字符就有贡献
                if (subStrCharFrequency.get(newChar) <= tCharFrequency.get(newChar))
                    charCount ++;
            }

            // 如果当前子串符合覆盖子串的要求，并且比之前的最小子串要短，就替换
            while ( charCount == t.length() && start < end) {
                if (minSubString.equals("") || end - start < minSubString.length()) {
                    minSubString = s.substring(start, end);
                }

                // 对要删除的字符，频次减1
                char removedChar = s.charAt(start);

                if (tCharFrequency.containsKey(removedChar)) {
                    subStrCharFrequency.put(removedChar, subStrCharFrequency.getOrDefault(removedChar, 0) - 1);
                    // 如果子串中的频次如果不够t中的频次，贡献值减少
                    if (subStrCharFrequency.get(removedChar) < tCharFrequency.get(removedChar))
                        charCount --;
                }

                // 只要是覆盖子串，就移动初始位置，缩小窗口，寻找当前的局部最优解
                start ++;
            }
            // 如果不是覆盖子串，需要扩大窗口，继续寻找新的子串
            end ++;
        }
        return minSubString;
    }
    
    
    public static void main(String[] args) {
        String s = "ADOBECODEBANC";
        String t = "ABC";

        MinWindowSubstring minWindowSubstring = new MinWindowSubstring();
        System.out.println(minWindowSubstring.minWindow(s, t));
    }

}
```

## 找到字符串中所有字母异位词(#438

给定两个字符串 s 和 p，找到 s 中所有 p 的 异位词 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。

异位词 指由相同字母重排列形成的字符串（包括相同的字符串）。

```
示例 1:

输入: s = "cbaebabacd", p = "abc"
输出: [0,6]
解释:
起始索引等于 0 的子串是 "cba", 它是 "abc" 的异位词。
起始索引等于 6 的子串是 "bac", 它是 "abc" 的异位词。


示例 2:

输入: s = "abab", p = "ab"
输出: [0,1,2]
解释:
起始索引等于 0 的子串是 "ab", 它是 "ab" 的异位词。
起始索引等于 1 的子串是 "ba", 它是 "ab" 的异位词。
起始索引等于 2 的子串是 "ab", 它是 "ab" 的异位词。
```

```java
public class SlidingWindowMaximum {

    // 方法四：左右扫描
    public int[] maxSlidingWindow(int[] nums, int k){
        int n = nums.length;
        // 定义一个结果数组
        int[] result = new int[n - k + 1];

        // 定义存放块内最大值的left和right数组
        int[] left = new int[n];
        int[] right = new int[n];

        // 遍历数组，左右扫描
        for (int i = 0; i < n; i++){
            // 1. 从左到右
            if (i % k == 0){
                // 如果能整除k，就是块的起始位置
                left[i] = nums[i];
            } else {
                // 如果不是起始位置，就直接跟left[i-1]取最大值即可
                left[i] = Math.max(left[i-1], nums[i]);
            }
            // 2. 从右到左
            int j = n - 1 - i;    // j就是倒数的i
            if (j % k == k - 1 || j == n - 1){
                right[j] = nums[j];
            } else {
                right[j] = Math.max(right[j+1], nums[j]);
            }
        }

        // 对每个窗口计算最大值
        for (int i = 0; i < n - k + 1; i++){
            result[i] = Math.max(right[i], left[i + k - 1]);
        }
        return result;
    }

    public static void main(String[] args) {
        int[] input = {1,3,-1,-3,5,3,6,7};
        int k = 3;

        SlidingWindowMaximum slidingWindowMaximum = new SlidingWindowMaximum();

        int[] output = slidingWindowMaximum.maxSlidingWindow(input, k);

        for (int max: output)
            System.out.print(max + "\t");
    }
}
```

