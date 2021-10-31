# Algorithm











## Insert Sort

一个未排序的数组（当然也可以是链表）可以分为两个部分，前半部分是已经排序的，后半部分是未排序的。在进行排序时，只需要在未排序的部分中选择一个元素，将其插入到前面有序的数组中即可。最终，未排序部分只会越来越少，直到为0，那么排序就完成了。初始时，可以假设已排序的部分就是第一个元素。

1+2+3+...+n

(n+1)*(n/2) = (n*n/2)+()

10+1 * 5 = 55

50 + 5 = 55

O(n^2)

```java
public static void insertSort(int[] arr){
    for (int i = 1; i < arr.length; i++) {
        int insertValue = arr[i];
        int j = i - 1;
        while (j >= 0 && arr[j] > insertValue) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = insertValue;
    }
}
```





```java
public static void shellSort(int[] arr) {
    int h = 1;
    while (h <= arr.length / 3) {
        h = h *3 + 1;
        System.out.println(h);
    }
    while (h > 0) {
        for (int i = h; i < arr.length; i++) {
            if (arr[i - h] > arr[i]) {
                int insertValue = arr[i];
                int j = i - h;
                while (j >= 0 && arr[j] > insertValue) {
                    arr[j + h] = arr[j];
                    j -= h;
                }
                arr[j + h] = insertValue;
            }
        }
        h = (h - 1) / 3;
        System.out.println("---"+h);
    }
}
```



```java

```

