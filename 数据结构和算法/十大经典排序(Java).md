# 冒泡排序

```java
    @Test
    public void testBubbleSort() {
        int[] arr = {24, 69, 80, 57, 13};
        int temp;

        for (int i = 1; i < arr.length; i++) {
            for (int j = 0; j < arr.length - i; j++) {
                if (arr[j] > arr[j + 1]) {
                    temp = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = temp;
                }
            }
        }

        for (int i = 0; i < arr.length; i++) {
            System.out.println(arr[i]);
        }

    }
```

------

# 选择排序

```java
    @Test
    public void testSelectOrder() {
        int[] arr = {24, 69, 80, 57, 13, 1, 99, 45};

        for (int j = 0; j < arr.length - 1; j++) {
            int max = arr[j];
            int maxIndex = j;
            int temp;
            for (int i = j; i < arr.length - 1; i++) {
                if (max < arr[i + 1]) {
                    max = arr[i + 1];
                    maxIndex = i + 1;
                }
            }
            if (maxIndex != j) {
                temp = arr[j];
                arr[j] = max;
                arr[maxIndex] = temp;
            }
        }

        for (int i = 0; i < arr.length; i++) {
            System.out.println(arr[i]);
        }
    }
```

------

# 插入排序

```java
    @Test
    public void testInsertSort() {

        int[] arr = {23, 1, 5, 7, 34, 55, 77, 45};

        for (int i = 1; i < arr.length; i++) {

            int max = arr[i];
            int insertIndex = i - 1;

            while (insertIndex >= 0 && max > arr[insertIndex]) {
                arr[insertIndex + 1] = arr[insertIndex];
                insertIndex--;
            }

            if (arr[insertIndex + 1] != i) {
                arr[insertIndex + 1] = max;
            }
        }

        for (int i = 0; i < arr.length; i++) {
            System.out.println(arr[i]);
        }
    }
```

------

# 快速排序（重点中的重点）

```java
    @Test
    public void testQuickSort() {

        int[] arr = {13, 32, 54, 12, 1, 4, 3, 3};

        quickSort(arr, 0, arr.length - 1);

        for (int i = 0; i < arr.length; i++) {
            System.out.println(arr[i]);
        }

    }

  private void quickSort(int[] nums, int start, int end) {

    if (start >= end) {
      return;
    }
    int mid = partition(nums, start, end);
    quickSort(nums, start, mid - 1);
    quickSort(nums, mid + 1, end);
  }

  private int partition(int nums[], int start, int end) {

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
```

# 二分查找

```java
public class BinarySearch {

    public int binarySearch(int[] nums, int target) {

        int start = 0;
        int end = nums.length - 1;

        while (end > start) {

            int mid = (end + start) / 2;
            if (nums[mid] > target) {

                end = nums[mid - 1];

            } else if (nums[mid] < target) {
                start = nums[mid + 1];
            } else {
                return mid;
            }

        }

        return -1;
    }


    public static void main(String[] args) {

        BinarySearch search = new BinarySearch();
        int[] nums = {1, 2, 3, 4, 5, 6, 7, 8, 9, 0};

        int index = search.binarySearch(nums, 7);
        System.out.println(index);		//6

    }
}
```

