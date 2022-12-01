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

## 找到字符串中所有字母异位词(#438)

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
public class FindAllAnagrams {

    // 方法二：滑动窗口法，分别移动起始和结束位置
    public List<Integer> findAnagrams(String s, String p){
        // 定义一个结果列表
        ArrayList<Integer> result = new ArrayList<>();

        // 1. 统计p中所有字符频次
        int[] pCharCounts = new int[26];

        for (int i = 0; i < p.length(); i++){
            pCharCounts[p.charAt(i) - 'a'] ++;
        }

        // 统计子串中所有字符出现频次的数组
        int[] subStrCharCounts = new int[26];
        // 定义双指针，指向窗口的起始和结束位置
        int start = 0, end = 1;

        // 2. 移动指针，总是截取字符出现频次全部小于等于p中字符频次的子串
        while (end <= s.length()){
            // 当前新增字符
            char newChar = s.charAt(end - 1);
            subStrCharCounts[newChar - 'a'] ++;

            // 3. 判断当前子串是否符合要求
            // 如果新增字符导致子串中频次超出了p中频次，那么移动start，消除新增字符的影响
            while ( subStrCharCounts[newChar - 'a'] > pCharCounts[newChar - 'a'] && start < end){
                char removedChar = s.charAt(start);
                subStrCharCounts[removedChar - 'a'] --;
                start ++;
            }

            // 如果当前子串长度等于p的长度，那么就是一个字母异位词
            if (end - start == p.length())
                result.add(start);

            end ++;
        }
        return result;
    }

    public static void main(String[] args) {
        String s = "cbaebabacd";
        String p = "abc";

        FindAllAnagrams findAllAnagrams = new FindAllAnagrams();

        List<Integer> result = findAllAnagrams.findAnagrams(s, p);

        for (int index: result){
            System.out.print(index + "\t");
        }
        System.out.println();
    }
}
```

# 链表

```java
public class ListNode {

    int val;    // 当前节点保存的数据值
    ListNode next;

    public ListNode() {
    }

    public ListNode(int val) {
        this.val = val;
    }

    public ListNode(int val, ListNode next) {
        this.val = val;
        this.next = next;
    }
}
```

```java
public class TestLinkedList {

    public static void main(String[] args) {
        // 构建一个链表，把所有节点创建出来，然后连接
        ListNode listNode1 = new ListNode(2);
        ListNode listNode2 = new ListNode(5);
        ListNode listNode3 = new ListNode(3);
        ListNode listNode4 = new ListNode(17);


        listNode1.next = listNode2;
        listNode2.next = listNode3;
        listNode3.next = listNode4;
        listNode4.next = null;

        printList(listNode1);
    }

    // 遍历打印链表元素
    public static void printList(ListNode head){
        ListNode curNode = head;
        while (curNode != null){
            System.out.print(curNode.val + " -> ");
            curNode = curNode.next;
        }
        System.out.print("null");
        System.out.println();
    }
}
```

## 反转链表(#206)(简单)

给你单链表的头节点 `head` ，请你反转链表，并返回反转后的链表。

```
输入：head = [1,2,3,4,5]
输出：[5,4,3,2,1]

输入：head = [1,2]
输出：[2,1]
```

```java
public class ReverseLinkedList {
    // 方法一：迭代
    public ListNode reverseList1(ListNode head){
        // 定义两个指针，指向当前访问的节点，以及上一个节点
        ListNode curr = head;
        ListNode prev = null;

        // 依次迭代链表中的节点，将next指针指向prev
        while (curr != null){
            // 临时保存当前节点的下一个节点
            ListNode tempNext = curr.next;
            curr.next = prev;

            // 更新指针，当前指针变为之前的next，上一个指针变为curr
            prev = curr;
            curr = tempNext;
        }
        //  prev指向的就是末尾的节点，也就是翻转之后的头节点
        return prev;
    }

    // 方法二：递归
    public ListNode reverseList(ListNode head){
        // 基准情况
        if (head == null || head.next == null) return head;

        // 递归调用，翻转剩余所有节点
        ListNode restHead = head.next;
        ListNode reversedRest = reverseList(restHead);

        // 把当前节点接在翻转之后的链表末尾
        restHead.next = head;
        // 当前节点就是链表末尾，直接指向null
        head.next = null;

        return reversedRest;
    }

    public static void main(String[] args) {
        ListNode listNode1 = new ListNode(1);
        ListNode listNode2 = new ListNode(2);
        ListNode listNode3 = new ListNode(3);
        ListNode listNode4 = new ListNode(4);
        ListNode listNode5 = new ListNode(5);


        listNode1.next = listNode2;
        listNode2.next = listNode3;
        listNode3.next = listNode4;
        listNode4.next = listNode5;
        listNode5.next = null;

        TestLinkedList.printList(listNode1);

        ReverseLinkedList reverseLinkedList = new ReverseLinkedList();
        TestLinkedList.printList(reverseLinkedList.reverseList(listNode1));
    }
}
```

## 合并两个有序链表(21)(简单)

将两个升序链表合并为一个新的 **升序** 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

```
示例:1
输入：l1 = [1,2,4], l2 = [1,3,4]
输出：[1,1,2,3,4,4]

示例 2：

输入：l1 = [], l2 = []
输出：[]

示例 3：

输入：l1 = [], l2 = [0]
输出：[0]
```

```java
public class MergeTwoSortedLists {
    // 方法一：迭代法
    public ListNode mergeTwoLists1(ListNode l1, ListNode l2){
        // 首先，定义一个哨兵节点，它的next指向最终结果的头节点
        ListNode sentinel = new ListNode(-1);

        // 保存当前结果链表里的最后一个节点，作为判断比较的“前一个节点”
        ListNode prev = sentinel;

        // 迭代遍历两个链表，直到至少有一个为null
        while ( l1 != null && l2 != null ){
            // 比较当前两个链表的头节点，较小的那个就接在结果链表末尾
            if (l1.val <= l2.val){
                prev.next = l1;    // 将l1头节点连接到prev后面
                prev = l1;    // 指针向前移动，以下一个节点作为链表头节点
                l1 = l1.next;
            } else {
                prev.next = l2;    // 将l2头节点连接到prev后面
                prev = l2;    // 指针向前移动，以下一个节点作为链表头节点
                l2 = l2.next;
            }
        }

        // 循环结束，最多还有一个链表没有遍历完成，因为已经排序，可以直接把剩余节点接到结果链表上
        prev.next = (l1 == null) ? l2 : l1;

        return sentinel.next;
    }

    // 方法二：递归
    public ListNode mergeTwoLists(ListNode l1, ListNode l2){
        // 基准情况
        if (l1 == null) return l2;
        if (l2 == null) return l1;

        // 比较头节点
        if (l1.val <= l2.val){
            // l1头节点较小，直接提取出来，剩下的l1和l2继续合并，接在l1后面
            l1.next = mergeTwoLists(l1.next, l2);
            return l1;
        } else {
            l2.next = mergeTwoLists(l1, l2.next);
            return l2;
        }
    }

    public static void main(String[] args) {
        ListNode listNode11 = new ListNode(1);
        ListNode listNode12 = new ListNode(2);
        ListNode listNode13 = new ListNode(4);
        ListNode listNode21 = new ListNode(1);
        ListNode listNode22 = new ListNode(3);
        ListNode listNode23 = new ListNode(4);

        listNode11.next = listNode12;
        listNode12.next = listNode13;
        listNode13.next = null;
        listNode21.next = listNode22;
        listNode22.next = listNode23;
        listNode23.next = null;

        TestLinkedList.printList(listNode11);
        TestLinkedList.printList(listNode21);

        MergeTwoSortedLists mergeTwoSortedLists = new MergeTwoSortedLists();
        TestLinkedList.printList(mergeTwoSortedLists.mergeTwoLists(listNode11, listNode21));
    }
}
```

## 删除链表的倒数第 N 个结点(#19)

给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。

```
示例 1：
输入：head = [1,2,3,4,5], n = 2
输出：[1,2,3,5]

示例 2：

输入：head = [1], n = 1
输出：[]

示例 3：

输入：head = [1,2], n = 1
输出：[1]

```

```java
public class RemoveNthNodeFromEnd {

    // 方法三：双指针
    public ListNode removeNthFromEnd(ListNode head, int n){
        // 定义一个哨兵节点，next指向头节点
        ListNode sentinel = new ListNode(-1, head);

        // 定义前后双指针
        ListNode first = sentinel, second = sentinel;

        // 1. first先走n+1步
        for (int i = 0; i < n + 1; i++)
            first = first.next;

        // 2. first、second同时前进，当first变为null，second就是倒数第n+1个节点
        while (first != null){
            first = first.next;
            second = second.next;
        }

        // 3. 删除倒数第n个节点
        second.next = second.next.next;

        return sentinel.next;
    }

    public static void main(String[] args) {
        ListNode listNode1 = new ListNode(1);
        ListNode listNode2 = new ListNode(2);
        ListNode listNode3 = new ListNode(3);
        ListNode listNode4 = new ListNode(4);
        ListNode listNode5 = new ListNode(5);


        listNode1.next = listNode2;
        listNode2.next = listNode3;
        listNode3.next = listNode4;
        listNode4.next = listNode5;
        listNode5.next = null;

        TestLinkedList.printList(listNode1);

        RemoveNthNodeFromEnd removeNthNodeFromEnd = new RemoveNthNodeFromEnd();

        TestLinkedList.printList(removeNthNodeFromEnd.removeNthFromEnd(listNode1, 2));
    }
}
```

# 哈希表

## 只出现一次的数字(#136)(简单)

给定一个非空整数数组，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。

说明：

你的算法应该具有线性时间复杂度。 你可以不使用额外空间来实现吗？

```
示例 1:

输入: [2,2,1]
输出: 1

示例 2:

输入: [4,1,2,1,2]
输出: 4
```

```java
public class SingleNumber {
    // 方法一：暴力法
    public int singleNumber1(int[] nums){
        // 定义一个列表，保存当前所有出现过一次的元素
        ArrayList<Integer> singleNumList = new ArrayList<>();

        // 遍历所有元素
        for (Integer num: nums){
            if (singleNumList.contains(num)){
                // 如果已经出现过，删除列表中的元素
                singleNumList.remove(num);
            } else {
                // 没有出现过，直接保存
                singleNumList.add(num);
            }
        }
        return singleNumList.get(0);
    }

    // 方法二：保存单独的元素到HashMap
    public int singleNumber2(int[] nums){
        HashMap<Integer, Integer> singleNumMap = new HashMap<>();

        for (Integer num: nums){
            if (singleNumMap.get(num) != null)
                singleNumMap.remove(num);
            else
                singleNumMap.put(num, 1);
        }

        return singleNumMap.keySet().iterator().next();
    }

    // 方法三：用set去重，a = 2 * (a+b+c) - (a+b+c+b+c)
    public int singleNumber3(int[] nums){
        // 定义一个HashSet进行去重
        HashSet<Integer> set = new HashSet<>();
        int arraySum = 0;
        int setSum = 0;

        // 1. 遍历数组元素，保存到set，并直接求和
        for (int num: nums){
            set.add(num);
            arraySum += num;
        }
        // 2. 集合所有元素求和
        for (int num: set)
            setSum += num;

        // 3. 计算结果
        return setSum * 2 - arraySum;
    }

    // 方法四：数学方法（做异或）
    public int singleNumber(int[] nums){
        int result = 0;
        // 遍历所有数据，按位做异或
        for (int num: nums)
            result ^= num;

        return result;
    }

    public static void main(String[] args) {
        int[] nums = {4,1,2,1,2};
        SingleNumber singleNumber = new SingleNumber();
        System.out.println(singleNumber.singleNumber(nums));
    }
}

```

## 最长连续序列(#128)

给定一个未排序的整数数组 nums ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。

请你设计并实现时间复杂度为 O(n) 的算法解决此问题。

```
示例 1：

输入：nums = [100,4,200,1,3,2]
输出：4
解释：最长数字连续序列是 [1, 2, 3, 4]。它的长度为 4。

示例 2：

输入：nums = [0,3,7,2,5,8,4,6,0,1]
输出：9
```

```java
public class LongestConsecutiveSequence {
    // 方法三：进一步改进
    public int longestConsecutiveSequence(int[] nums){
        // 定义一个变量，保存当前最长连续序列的长度
        int maxLength = 0;

        // 定义一个HashSet，保存所有出现的数值
        HashSet<Integer> hashSet = new HashSet<>();

        // 1. 遍历所有元素，保存到HashSet
        for (int num: nums){
            hashSet.add(num);
        }

        // 2. 遍历数组，以每个元素作为起始点，寻找连续序列
        for (int i = 0; i < nums.length; i++){
            // 保存当前元素作为起始点
            int currNum = nums[i];
            // 保存当前连续序列长度
            int currLength = 1;

            // 判断：只有当前元素的前驱不存在的情况下，才去进行寻找连续序列的操作
            if (!hashSet.contains(currNum - 1)) {
                // 寻找后续数字，组成连续序列
                while ( hashSet.contains(currNum + 1) ){
                    currLength ++;
                    currNum ++;
                }

                // 判断当前连续序列长度是否为最大
                maxLength = currLength > maxLength ? currLength : maxLength;
            }
        }

        return maxLength;
    }

    public static void main(String[] args) {
        int[] nums1 = {100,4,200,1,3,2};
        int[] nums2 = {0,3,7,2,5,8,4,6,0,1};

        LongestConsecutiveSequence longestConsecutiveSequence = new LongestConsecutiveSequence();
        System.out.println(longestConsecutiveSequence.longestConsecutiveSequence(nums1));
        System.out.println(longestConsecutiveSequence.longestConsecutiveSequence(nums2));
    }
}
```

## LRU 缓存(146)

请你设计并实现一个满足  LRU (最近最少使用) 缓存 约束的数据结构。
实现 `LRUCache` 类：

- `LRUCache(int capacity)` 以 正整数 作为容量 `capacity` 初始化 LRU 缓存
- `int get(int key)` 如果关键字 `key `存在于缓存中，则返回关键字的值，否则返回 -1 。
- `void put(int key, int value)` 如果关键字 `key `已经存在，则变更其数据值 `value `；如果不存在，则向缓存中插入该组 `key-value` 。如果插入操作导致关键字数量超过 `capacity `，则应该 逐出 最久未使用的关键字。

函数 `get `和 `put` 必须以 O(1) 的平均时间复杂度运行。

```
示例：

输入
["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"]
[[2], [1, 1], [2, 2], [1], [3, 3], [2], [4, 4], [1], [3], [4]]
输出
[null, null, null, 1, null, -1, null, -1, 3, 4]

解释
LRUCache lRUCache = new LRUCache(2);
lRUCache.put(1, 1); // 缓存是 {1=1}
lRUCache.put(2, 2); // 缓存是 {1=1, 2=2}
lRUCache.get(1);    // 返回 1
lRUCache.put(3, 3); // 该操作会使得关键字 2 作废，缓存是 {1=1, 3=3}
lRUCache.get(2);    // 返回 -1 (未找到)
lRUCache.put(4, 4); // 该操作会使得关键字 1 作废，缓存是 {4=4, 3=3}
lRUCache.get(1);    // 返回 -1 (未找到)
lRUCache.get(3);    // 返回 3
lRUCache.get(4);    // 返回 4

```

详细见[Redis缓存淘汰策略](https://github.com/DragonChilde/interview-data/blob/main/Redis/Redis%E7%BC%93%E5%AD%98%E6%B7%98%E6%B1%B0%E7%AD%96%E7%95%A5.md)

# 栈和队列

## 使用队列实现栈#(225)(简单)

https://leetcode.cn/problems/implement-stack-using-queues/solution/yong-dui-lie-shi-xian-zhan-by-leetcode-solution/

请你仅使用两个队列实现一个后入先出（LIFO）的栈，并支持普通栈的全部四种操作（`push`、`top`、`pop` 和 `empty`）。

```
实现 MyStack 类：

void push(int x) 将元素 x 压入栈顶。
int pop() 移除并返回栈顶元素。
int top() 返回栈顶元素。
boolean empty() 如果栈是空的，返回 true ；否则，返回 false 。
 

注意：

你只能使用队列的基本操作 —— 也就是 push to back、peek/pop from front、size 和 is empty 这些操作。
你所使用的语言也许不支持队列。 你可以使用 list （列表）或者 deque（双端队列）来模拟一个队列 , 只要是标准的队列操作即可。
 

示例：

输入：
["MyStack", "push", "push", "top", "pop", "empty"]
[[], [1], [2], [], [], []]
输出：
[null, null, null, 2, 2, false]

解释：
MyStack myStack = new MyStack();
myStack.push(1);
myStack.push(2);
myStack.top(); // 返回 2
myStack.pop(); // 返回 2
myStack.empty(); // 返回 False

```

使用一个队列解法

```java
public class MyStack {

  Queue<Integer> queue;

  public MyStack() {
    this.queue = new LinkedList<Integer>();
  }

  public void push(int x) {

    int size = queue.size();

    queue.offer(x);

    for (int i = 0; i < size; i++) {

      queue.offer(queue.poll());
    }
  }

  public int pop() {
    return queue.poll();
  }

  public int top() {
    return queue.peek();
  }

  public boolean empty() {
    return queue.isEmpty();
  }

  public static void main(String[] args) {
    MyStack myStack = new MyStack();
    myStack.push(1);
    myStack.push(2);
    System.out.println(myStack.top());
    System.out.println(myStack.pop());
    myStack.empty(); // 返回 False
  }
}
```

## 使用栈实现队列（#232）(简单)

https://leetcode.cn/problems/implement-queue-using-stacks/solution/yong-zhan-shi-xian-dui-lie-by-leetcode-s-xnb6/

请你仅使用两个栈实现先入先出队列。队列应当支持一般队列支持的所有操作（`push`、`pop`、`peek`、`empty`）：

```
实现 MyQueue 类：

void push(int x) 将元素 x 推到队列的末尾
int pop() 从队列的开头移除并返回元素
int peek() 返回队列开头的元素
boolean empty() 如果队列为空，返回 true ；否则，返回 false
 

说明：

你只能使用标准的栈操作 —— 也就是只有 push to top, peek/pop from top, size, 和 is empty 操作是合法的。
你所使用的语言也许不支持栈。你可以使用 list 或者 deque（双端队列）来模拟一个栈，只要是标准的栈操作即可。
 

进阶：

你能否实现每个操作均摊时间复杂度为 O(1) 的队列？换句话说，执行 n 个操作的总时间复杂度为 O(n) ，即使其中一个操作可能花费较长时间。
 

示例：

输入：
["MyQueue", "push", "push", "peek", "pop", "empty"]
[[], [1], [2], [], [], []]
输出：
[null, null, null, 1, 1, false]

解释：
MyQueue myQueue = new MyQueue();
myQueue.push(1); // queue is: [1]
myQueue.push(2); // queue is: [1, 2] (leftmost is front of the queue)
myQueue.peek(); // return 1
myQueue.pop(); // return 1, queue is [2]
myQueue.empty(); // return false
```

出队列时反转

```java
public class MyQueue {

  Deque<Integer> inStack;
  Deque<Integer> outStack;

  public MyQueue() {
    inStack = new ArrayDeque<>();
    outStack = new ArrayDeque<>();
  }

  public void push(int x) {

    inStack.push(x);
  }

  public int pop() {
    if (outStack.isEmpty()) {
      in2out();
    }
    return outStack.pop();
  }

  public int peek() {
    if (outStack.isEmpty()) {
      in2out();
    }

    return outStack.peek();
  }

  public boolean empty() {
    return inStack.isEmpty() && outStack.isEmpty();
  }

  public void in2out() {
    while (!inStack.isEmpty()) {
      outStack.push(inStack.pop());
    }
  }

  public static void main(String[] args) {
    MyQueue myQueue = new MyQueue();
    myQueue.push(1); // queue is: [1]
    myQueue.push(2); // queue is: [1, 2] (leftmost is front of the queue)
    System.out.println(myQueue.peek());
    System.out.println(myQueue.pop());
    myQueue.empty(); // return false
  }
}
```

------

## 有效的括号(#20)(简单)

```
给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串 s ，判断字符串是否有效。

有效字符串需满足：

左括号必须用相同类型的右括号闭合。
左括号必须以正确的顺序闭合。
每个右括号都有一个对应的相同类型的左括号。
 

示例 1：

输入：s = "()"
输出：true
示例 2：

输入：s = "()[]{}"
输出：true
示例 3：

输入：s = "(]"
输出：false
```

```java
/**
 * @title: 有效的括号 ValidParentheses @Author Wen @Date: 1/7/2022 下午4:57 @Version 1.0
 */
public class ValidParentheses {

  public static boolean isValid(String s) {

    int length = s.length();
    if (length % 2 == 1) {
      return false;
    }

    HashMap<Character, Character> hasMap = new HashMap<>();
    hasMap.put(')', '(');
    hasMap.put('}', '{');
    hasMap.put(']', '[');

    Deque<Character> stack = new LinkedList<>();

    for (int i = 0; i < length; i++) {
      char c = s.charAt(i);
      if (hasMap.containsKey(c)) {
        if (stack.isEmpty() || stack.peek() != hasMap.get(c)) {
          return false;
        }
        stack.pop();
      } else {
        stack.push(c);
      }
    }

    return stack.isEmpty();
  }

  public static void main(String[] args) {
    String s = "()";

    System.out.println(isValid(s));

    String s2 = "()[]{}";

    System.out.println(isValid(s2));

    String s3 = "(]";
    System.out.println(isValid(s3));

    String s4 = "([)]";
    System.out.println(isValid(s4));

    String s5 = "{[]}";
    System.out.println(isValid(s5));
  }
}
```



------

## 柱状图中最大的矩形(#84)(困难)

https://leetcode.cn/problems/largest-rectangle-in-histogram/solution/zhu-zhuang-tu-zhong-zui-da-de-ju-xing-by-leetcode-/

- 使用单调栈

  ```java
  public class Solution84 {
  
    public int largestRectangleArea(int[] heights) {
      int n = heights.length;
      int[] lefts = new int[n];
      int[] rights = new int[n];
      int largestArea = 0;
      // 定义一个栈，保存“候选列表”
      Deque<Integer> stack = new ArrayDeque<Integer>();
      // 遍历所有柱子，计算左右边界
      for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && heights[stack.peek()] >= heights[i]) {
          stack.pop();
        }
        lefts[i] = stack.isEmpty() ? -1 : stack.peek();
        stack.push(i);
      }
      stack.clear();
      for (int i = n - 1; i >= 0; i--) {
  
        while (!stack.isEmpty() && heights[stack.peek()] >= heights[i]) {
  
          stack.pop();
        }
  
        rights[i] = stack.isEmpty() ? n : stack.peek();
        stack.push(i);
      }
  
      for (int i = 0; i < n; i++) {
  
        int currArea = (rights[i] - lefts[i] - 1) * heights[i];
  
        largestArea = currArea > largestArea ? currArea : largestArea;
      }
      return largestArea;
    }
  
    public static void main(String[] args) {
      Solution84 soulution = new Solution84();
  
      int[] arr = {6, 7, 5, 2, 4, 5, 9, 3};
  
      int i = soulution.largestRectangleArea(arr);
  
      System.out.println(i);
    }
  }
  
  ```

- 单调栈优化

  ```java
  public class Solution84 {
  
    public int largestRectangleArea(int[] heights) {
  
      int n = heights.length;
      int[] lefts = new int[n];
      int[] rights = new int[n];
  
      for (int i = 0; i < n; i++) {
        rights[i] = n;
      }
  
      int largestArea = 0;
  
      Deque<Integer> stack = new ArrayDeque<Integer>();
  
      for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && heights[stack.peek()] >= heights[i]) {
          rights[stack.pop()] = i;
        }
        lefts[i] = stack.isEmpty() ? -1 : stack.peek();
        stack.push(i);
      }
      
      for (int i = 0; i < n; i++) {
        int currArea = (rights[i] - lefts[i] - 1) * heights[i];
        largestArea = currArea > largestArea ? currArea : largestArea;
      }
  
      return largestArea;
    }
  
    public static void main(String[] args) {
      Solution84 soulution = new Solution84();
  
      int[] arr = {6, 7, 5, 2, 4, 5, 9, 3};
  
      int i = soulution.largestRectangleArea(arr);
  
      System.out.println(i);
    }
  }
  
  ```

  > **为什么不用Stack?用Deque**https://mp.weixin.qq.com/s/Ba8jrULf8NJbENK6WGrVWg

------

# 排序关问题

## 数组中的第K个最大元素(#215)(中等)

```
给定整数数组 nums 和整数 k，请返回数组中第 k 个最大的元素。

请注意，你需要找的是数组排序后的第 k 个最大的元素，而不是第 k 个不同的元素。

你必须设计并实现时间复杂度为 O(n) 的算法解决此问题。

示例 1:
输入: [3,2,1,5,6,4], k = 2
输出: 5


示例 2:
输入: [3,2,3,1,2,4,5,5,6], k = 4
输出: 4
```

- 基于快速排序的选择

  ```java
  public class Solution215 {
  
    public int findKthLargest(int[] nums, int k) {
      return quickSelect(nums, 0, nums.length - 1, nums.length - k);
    }
  
    public int quickSelect(int[] nums, int start, int end, int index) {
  
      int q = randomPatition(nums, start, end);
      if (q == index) {
        return nums[q];
      } else {
        return q > index
            ? quickSelect(nums, start, q - 1, index)
            : quickSelect(nums, q + 1, end, index);
      }
    }
  
    public int randomPatition(int[] nums, int start, int end) {
      Random random = new Random();
      int randIndex = start + random.nextInt(end - start + 1);
      swap(nums, start, randIndex);
      return partition(nums, start, end);
    }
  
    public int partition(int[] nums, int start, int end) {
  
      int left = start;
      int right = end;
      int pivot = nums[start];
  
        while (left < right) {
  
            while (left < right && nums[right] >= pivot) {
                right--;
            }
  
            nums[left] = nums[right];
            while (left < right && nums[left] <= pivot) {
                left++;
            }
            nums[right] = nums[left];
        }
  
      nums[left] = pivot;
      return left;
    }
  
    public void swap(int[] nums, int i, int j) {
  
      int tmp = nums[i];
      nums[i] = nums[j];
      nums[j] = tmp;
    }
  
    public static void main(String[] args) {
      int[] num1 = {3, 2, 1, 5, 6, 4};
      int k1 = 2;
  
      int[] num2 = {3, 2, 3, 1, 2, 4, 5, 5, 6};
      int k2 = 4;
  
      Solution215 solution = new Solution215();
  
      int r1 = solution.findKthLargest(num1, k1);
      System.out.println(r1); //5
  
      int r2 = solution.findKthLargest(num2, k2);
      System.out.println(r2); //4
    }
  }
  ```
  

## 颜色分类(#75)(中等)

```
给定一个包含红色、白色和蓝色、共 n 个元素的数组 nums ，原地对它们进行排序，使得相同颜色的元素相邻，并按照红色、白色、蓝色顺序排列。

我们使用整数 0、 1 和 2 分别表示红色、白色和蓝色。

必须在不使用库内置的 sort 函数的情况下解决这个问题。

示例 1：

输入：nums = [2,0,2,1,1,0]
输出：[0,0,1,1,2,2]

示例 2：

输入：nums = [2,0,1]
输出：[0,1,2]
```

基于快速排序

```java
public void sortColors(int[] nums) {
    int left = 0, right = nums.length - 1;
    int i = left;
    while ( left < right && i <= right ){
        while ( i <= right && nums[i] == 2 )
            swap( nums, i, right-- );
        if ( nums[i] == 0 )
            swap( nums, i, left++ );
        i++;
    }
}

public void swap(int[] nums, int start, int end) {
    int tmp = nums[start];
    nums[start] = nums[end];
    nums[end] = tmp;
}
```

## 合并区间(#56)(中等)

```
以数组 intervals 表示若干个区间的集合，其中单个区间为 intervals[i] = [starti, endi] 。请你合并所有重叠的区间，并返回 一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间 。

示例 1：

输入：intervals = [[1,3],[2,6],[8,10],[15,18]]
输出：[[1,6],[8,10],[15,18]]
解释：区间 [1,3] 和 [2,6] 重叠, 将它们合并为 [1,6].

示例 2：

输入：intervals = [[1,4],[4,5]]
输出：[[1,5]]
解释：区间 [1,4] 和 [4,5] 可被视为重叠区间。
```

```java
public class MergeIntervals {
    public int[][] merge(int[][] intervals) {
        List<int[]> result = new ArrayList<int[]>();
        // 先对原数组按左边界排序
        Arrays.sort(intervals, new Comparator<int[]>() {
            @Override
            public int compare(int[] o1, int[] o2) {
                return o1[0] - o2[0]; 
            }
        });
        // 遍历排序后的数组，逐个判断合并
        for ( int[] interval: intervals ){
            int left = interval[0], right = interval[1];
            int length = result.size();
            if ( length == 0 || left > result.get(length - 1)[1] ){
                result.add(interval);
            } else {
                int mergedLeft = result.get(length - 1)[0];
                int mergedRight = Math.max( result.get(length - 1)[1], right );
                result.set( length - 1, new int[]{mergedLeft, mergedRight} );
            }
        }
        return result.toArray(new int[result.size()][]);
    }
}

```

------

# 二叉树及递归

## 数和二叉树结构

### 树

### 二叉树

### 递归

### 二叉树的遍历

### 平衡二叉树

### 红黑树

### B树

### B+树

## 翻转二叉树(#226)(简单)

```
给你一棵二叉树的根节点 root ，翻转这棵二叉树，并返回其根节点

输入：root = [4,2,7,1,3,6,9]
输出：[4,7,2,9,6,3,1]

输入：root = [2,1,3]
输出：[2,3,1]

输入：root = []
输出：[]
```

- 方法一:先序遍历

  ```java
  public TreeNode invertTree(TreeNode root) {
      if ( root == null ) return null;
      TreeNode temp = root.left;
      root.left = root.right;
      root.right = temp;
      invertTree( root.left );
      invertTree( root.right );
      return root;
  }
  ```

- 方法二:后序遍历

  ```java
  public TreeNode invertTree(TreeNode root) {
      if ( root == null ) return null;
      TreeNode left = invertTree(root.left);
      TreeNode right = invertTree(root.right);
      root.left = right;
      root.right = left;
      return root;
  }
  ```

## 验证二叉树(#98)(中等)

```
给你一个二叉树的根节点 root ，判断其是否是一个有效的二叉搜索树。

有效 二叉搜索树定义如下：

节点的左子树只包含 小于 当前节点的数。
节点的右子树只包含 大于 当前节点的数。
所有左子树和右子树自身必须也是二叉搜索树。

输入：root = [2,1,3]
输出：true

输入：root = [5,1,4,null,null,3,6]
输出：false
解释：根节点的值是 5 ，但是右子节点的值是 4 。
```

# 贪心算法

## 跳跃游戏(#55)(中等)

```
给定一个非负整数数组 nums ，你最初位于数组的 第一个下标 。

数组中的每个元素代表你在该位置可以跳跃的最大长度。

判断你是否能够到达最后一个下标。

示例 1：

输入：nums = [2,3,1,1,4]
输出：true
解释：可以先跳 1 步，从下标 0 到达下标 1, 然后再从下标 1 跳 3 步到达最后一个下标。

示例 2：

输入：nums = [3,2,1,0,4]
输出：false
解释：无论怎样，总会到达下标为 3 的位置。但该下标的最大跳跃长度是 0 ， 所以永远不可能到达最后一个下标。
```

```java
public class JumpGame {
    // 贪心策略：维护当前能到的最远位置
    public boolean canJump(int[] nums) {
        int farthest = 0;
        // 遍历数组，更新farthest
        for (int i = 0; i < nums.length; i++ ){
            if (i <= farthest){
                farthest = Math.max(farthest, i + nums[i]);
                if (farthest >= nums.length - 1)
                    return true;
            } else {
                return false;
            }
        }
        return false;
    }
}

```

## 跳跃游戏II(#45)

```
给你一个非负整数数组 nums ，你最初位于数组的第一个位置。

数组中的每个元素代表你在该位置可以跳跃的最大长度。

你的目标是使用最少的跳跃次数到达数组的最后一个位置。

假设你总是可以到达数组的最后一个位置。

示例 1:

输入: nums = [2,3,1,1,4]
输出: 2
解释: 跳到最后一个位置的最小跳跃数是 2。
     从下标为 0 跳到下标为 1 的位置，跳 1 步，然后跳 3 步到达数组的最后一个位置。
     
示例 2:

输入: nums = [2,3,0,1,4]
输出: 2
```

## 任务调度器(#621)(中等)

```
给你一个用字符数组 tasks 表示的 CPU 需要执行的任务列表。其中每个字母表示一种不同种类的任务。任务可以以任意顺序执行，并且每个任务都可以在 1 个单位时间内执行完。在任何一个单位时间，CPU 可以完成一个任务，或者处于待命状态。

然而，两个 相同种类 的任务之间必须有长度为整数 n 的冷却时间，因此至少有连续 n 个单位时间内 CPU 在执行不同的任务，或者在待命状态。

你需要计算完成所有任务所需要的 最短时间 。

示例 1：

输入：tasks = ["A","A","A","B","B","B"], n = 2
输出：8
解释：A -> B -> (待命) -> A -> B -> (待命) -> A -> B
     在本示例中，两个相同类型任务之间必须间隔长度为 n = 2 的冷却时间，而执行一个任务只需要一个单位时间，所以中间出现了（待命）状态。 
     
示例 2：

输入：tasks = ["A","A","A","B","B","B"], n = 0
输出：6
解释：在这种情况下，任何大小为 6 的排列都可以满足要求，因为 n = 0
["A","A","A","B","B","B"]
["A","B","A","B","A","B"]
["B","B","B","A","A","A"]
...
诸如此类

示例 3：

输入：tasks = ["A","A","A","A","A","A","B","C","D","E","F","G"], n = 2
输出：16
解释：一种可能的解决方案是：
     A -> B -> C -> A -> D -> E -> A -> F -> G -> A -> (待命) -> (待命) -> A -> (待命) -> (待命) -> A
```

------

# 动态规划

## 0-1背包问题

## 买卖股票的最佳时机(#121)(简单)

```
给定一个数组 prices ，它的第 i 个元素 prices[i] 表示一支给定股票第 i 天的价格。

你只能选择 某一天 买入这只股票，并选择在 未来的某一个不同的日子 卖出该股票。设计一个算法来计算你所能获取的最大利润。

返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 0 。

示例 1：

输入：[7,1,5,3,6,4]
输出：5
解释：在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格；同时，你不能在买入前卖出股票。
     
示例 2：

输入：prices = [7,6,4,3,1]
输出：0
解释：在这种情况下, 没有交易完成, 所以最大利润为 0。
```

```java
public int maxProfit(int[] prices){
    int minPrice = Integer.MAX_VALUE; 
    int maxProfit = 0;
    // 遍历数组，以每天的价格作为卖出点，进行比较
    for (int i = 0; i < prices.length; i++){
        minPrice = Math.min(minPrice, prices[i]);
        maxProfit = Math.max(maxProfit, prices[i] - minPrice);
    }
    return maxProfit;
}
```

## 爬楼梯(#70)(简单)

```
假设你正在爬楼梯。需要 n 阶你才能到达楼顶。

每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？

示例 1：

输入：n = 2
输出：2
解释：有两种方法可以爬到楼顶。
1. 1 阶 + 1 阶
2. 2 阶

示例 2：

输入：n = 3
输出：3
解释：有三种方法可以爬到楼顶。
1. 1 阶 + 1 阶 + 1 阶
2. 1 阶 + 2 阶
3. 2 阶 + 1 阶
```

```java
public class ClimbStairsSolution {

  public static int climbStairs(int n) {

    int p = 0, q = 0, r = 1;
    for (int i = 0; i < n; i++) {
      p = q;
      q = r;
      r = p + q;
    }

    return r;
  }

  public static void main(String[] args) {
    System.out.println(climbStairs(2));		//2

    System.out.println(climbStairs(3));		//3
	
    System.out.println(climbStairs(4));		//5

    System.out.println(climbStairs(5));		//8
  }
}
```

## 最长公共子序列（#1143）(中等)

```
给定两个字符串 text1 和 text2，返回这两个字符串的最长 公共子序列 的长度。如果不存在 公共子序列 ，返回 0 。

一个字符串的 子序列 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。

例如，"ace" 是 "abcde" 的子序列，但 "aec" 不是 "abcde" 的子序列。
两个字符串的 公共子序列 是这两个字符串所共同拥有的子序列。

示例 1：

输入：text1 = "abcde", text2 = "ace" 
输出：3  
解释：最长公共子序列是 "ace" ，它的长度为 3 。

示例 2：

输入：text1 = "abc", text2 = "abc"
输出：3
解释：最长公共子序列是 "abc" ，它的长度为 3 。

示例 3：

输入：text1 = "abc", text2 = "def"
输出：0
解释：两个字符串没有公共子序列，返回 0 。
```

```java
public class LCS {
    // 动态规划实现
    public int longestCommonSubsequence(String text1, String text2) {
        int length1 = text1.length();
        int length2 = text2.length();
        int[][] dp = new int[length1+1][length2+1]; 
        // 遍历所有状态位置
        for (int i = 1; i <= length1; i++){
            for (int j = 1; j <= length2; j++){
                // 状态转移
                if ( text1.charAt(i-1) == text2.charAt(j-1) ){
                    dp[i][j] = dp[i-1][j-1] + 1;
                } else {
                    dp[i][j] = Math.max( dp[i-1][j], dp[i][j-1] );
                }
            }
        }
        return dp[length1][length2];
    }
}

```

## 打家劫舍（#198）(中等)

```
你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警。

给定一个代表每个房屋存放金额的非负整数数组，计算你 不触动警报装置的情况下 ，一夜之内能够偷窃到的最高金额。

示例 1：

输入：[1,2,3,1]
输出：4
解释：偷窃 1 号房屋 (金额 = 1) ，然后偷窃 3 号房屋 (金额 = 3)。
     偷窃到的最高金额 = 1 + 3 = 4 。
     
示例 2：

输入：[2,7,9,3,1]
输出：12
解释：偷窃 1 号房屋 (金额 = 2), 偷窃 3 号房屋 (金额 = 9)，接着偷窃 5 号房屋 (金额 = 1)。
     偷窃到的最高金额 = 2 + 9 + 1 = 12 。
```

```java
public int rob(int[] nums) {
    int n = nums.length;
    if (nums == null || n == 0) return 0;
    int pre2 = 0;
    int pre1 = nums[0];
    // 遍历状态，依次转移
    for (int i = 1; i < n; i++){
        int curr = Math.max(pre1, pre2 + nums[i]);
        pre2 = pre1;
        pre1 = curr;
    }
    return pre1; 
}

```

## 零钱兑换（#322）(中等)

```
给你一个整数数组 coins ，表示不同面额的硬币；以及一个整数 amount ，表示总金额。

计算并返回可以凑成总金额所需的 最少的硬币个数 。如果没有任何一种硬币组合能组成总金额，返回 -1 。

你可以认为每种硬币的数量是无限的。

示例 1：

输入：coins = [1, 2, 5], amount = 11
输出：3 
解释：11 = 5 + 5 + 1

示例 2：

输入：coins = [2], amount = 3
输出：-1

示例 3：

输入：coins = [1], amount = 0
输出：0
```

```java
// 动态规划
public int coinChange(int[] coins, int amount) {
    int n = coins.length;
    int[] dp = new int[amount + 1];
    dp[0] = 0;

    for (int i = 1; i <= amount; i++){
        int minCoinNum = Integer.MAX_VALUE;
        // 遍历所有硬币面值，作为可能的“最后一步”
        for (int coin: coins){
            if (coin <= i && dp[i - coin] != -1){
                minCoinNum = Math.min(minCoinNum, dp[i - coin] + 1);
            }
        }
        dp[i] = minCoinNum == Integer.MAX_VALUE ? -1 : minCoinNum;
    }
    return dp[amount];
}
```

# 回溯算法

## 八皇后问题

```java
// 定义辅助集合
HashSet<Integer> cols = new HashSet<>();
HashSet<Integer> diags1 = new HashSet<>();
HashSet<Integer> diags2 = new HashSet<>();

// 方法二：回溯法
public List<int[]> eightQueens(){
    ArrayList<int[]> result = new ArrayList<>();
    int[] solution = new int[8];
    Arrays.fill(solution, -1);    // 初始填充-1 
    // 传入行号0，开始调用
    backtrack(result, solution, 0);
    return result;
}
// 定义一个回溯方法
private void backtrack(ArrayList<int[]> result, int[] solution, int row){
    if (row >= 8){
        result.add(Arrays.copyOf(solution, 8));
    } else {
        // 遍历每一列，考察可能的皇后位置
        for (int column = 0; column < 8; column ++){
            if (cols.contains(column))
                continue;
            int diag1 = row - column;
            if (diags1.contains(diag1))
                continue;
            int diag2 = row + column;
            if (diags2.contains(diag2))
                continue;
            solution[row] = column;    // 当前位置可以放置皇后
            cols.add(column);
            diags1.add(diag1);
            diags2.add(diag2);
            // 递归调用，找下一行的皇后
            backtrack(result, solution, row + 1);
            // 回溯状态
            solution[row] = -1;
            cols.remove(column);
            diags1.remove(diag1);
            diags2.remove(diag2);
        }
    }
}

```

## 全排列(#46)(中等)

```
给定一个不含重复数字的数组 nums ，返回其 所有可能的全排列 。你可以 按任意顺序 返回答案。

示例 1：

输入：nums = [1,2,3]
输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]

示例 2：

输入：nums = [0,1]
输出：[[0,1],[1,0]]

示例 3：

输入：nums = [1]
输出：[[1]]
```

```java
public List<List<Integer>> permute(int[] nums){
    ArrayList<List<Integer>> result = new ArrayList<>();
    // 定义一个可行的排列，并复制nums填入
    ArrayList<Integer> solution = new ArrayList<>();
    for (int num: nums) {
        solution.add(num);
    }
    // 从0位置开始填充
    backtrack(result, solution, 0);
    return result;
}
public void backtrack(List<List<Integer>> result, List<Integer> solution, int i){
    int n = solution.size();
    if (i >= n){
        result.add(new ArrayList<>(solution));
    } else {
        for (int j = i; j < n; j++){
            Collections.swap(solution, i, j);
            // 递归调用，继续填充后面的位置
            backtrack(result, solution, i + 1);
            // 回溯
            Collections.swap(solution, i, j);
        }
    }
}

```

## 电话号码的字母组合（#17）(中等)

```
给定一个仅包含数字 2-9 的字符串，返回所有它能表示的字母组合。答案可以按 任意顺序 返回。

给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。

1  2  3
4  5  6
7  8  9
*  0  #

示例 1：

输入：digits = "23"
输出：["ad","ae","af","bd","be","bf","cd","ce","cf"]

示例 2：

输入：digits = ""
输出：[]

示例 3：

输入：digits = "2"
输出：["a","b","c"]
```

```java
public class LetterCombinationsOfPhoneNumber {
    // 定义一个HashMap，保存数字对应的字母
    HashMap<Character, String> numberMap = new HashMap<Character, String>() {
        {
            put('2', "abc");
            put('3', "def");
            put('4', "ghi");
            put('5', "jkl");
            put('6', "mno");
            put('7', "pqrs");
            put('8', "tuv");
            put('9', "wxyz");
       }
    };
    public List<String> letterCombinations(String digits) {
        ArrayList<String> result = new ArrayList<>();
        if ("".equals(digits)) return result;
        StringBuffer combination = new StringBuffer();
        // 从第一个数字开始回溯处理
        backtrack(digits, result, combination, 0);
        return result;
    }
    // 定义一个回溯方法
    public void backtrack(String digits, List<String> result, StringBuffer combination, int i){
        int n = digits.length();
        if (i >= n){
            result.add(combination.toString());
        } else {
            char digit = digits.charAt(i);
            String letters = numberMap.get(digit);
            // 遍历所有可能的字母
            for (int j = 0; j < letters.length(); j++){
                combination.append(letters.charAt(j));
                // 递归调用，继续处理后续数字
                backtrack(digits, result, combination, i + 1);
                // 回溯
                combination.deleteCharAt(i); 
            }
        }
    }
}

```

------

# 深度优先搜索和广度优先搜索

## 二叉树的序列化与反序列化(#297)(困难)

## 课程表(#207)(中等)

# 位运算和数学方法

## 2的幂(#231)(简单)

```
给你一个整数 n，请你判断该整数是否是 2 的幂次方。如果是，返回 true ；否则，返回 false 。

如果存在一个整数 x 使得 n == 2x ，则认为 n 是 2 的幂次方。

示例 1：

输入：n = 1
输出：true
解释：20 = 1

示例 2：

输入：n = 16
输出：true
解释：24 = 16

示例 3：

输入：n = 3
输出：false

示例 4：

输入：n = 4
输出：true

示例 5：

输入：n = 5
输出：false
```

## 汉明距离(#461)(简单)

```
两个整数之间的 汉明距离 指的是这两个数字对应二进制位不同的位置的数目。

给你两个整数 x 和 y，计算并返回它们之间的汉明距离。

 

示例 1：

输入：x = 1, y = 4
输出：2
解释：
1   (0 0 0 1)
4   (0 1 0 0)
       ↑   ↑
上面的箭头指出了对应二进制位不同的位置。
示例 2：

输入：x = 3, y = 1
输出：1

```

## 可爱的小猪(#458)(困难)

```
有 buckets 桶液体，其中 正好有一桶 含有毒药，其余装的都是水。它们从外观看起来都一样。为了弄清楚哪只水桶含有毒药，你可以喂一些猪喝，通过观察猪是否会死进行判断。不幸的是，你只有 minutesToTest 分钟时间来确定哪桶液体是有毒的。

喂猪的规则如下：

选择若干活猪进行喂养
可以允许小猪同时饮用任意数量的桶中的水，并且该过程不需要时间。
小猪喝完水后，必须有 minutesToDie 分钟的冷却时间。在这段时间里，你只能观察，而不允许继续喂猪。
过了 minutesToDie 分钟后，所有喝到毒药的猪都会死去，其他所有猪都会活下来。
重复这一过程，直到时间用完。
给你桶的数目 buckets ，minutesToDie 和 minutesToTest ，返回 在规定时间内判断哪个桶有毒所需的 最小 猪数 。


示例 1：

输入：buckets = 1000, minutesToDie = 15, minutesToTest = 60
输出：5

示例 2：

输入：buckets = 4, minutesToDie = 15, minutesToTest = 15
输出：2

示例 3：

输入：buckets = 4, minutesToDie = 15, minutesToTest = 30
输出：2
```

