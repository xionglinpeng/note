# Algorithm



## Parity Exchange Sort

假设我们有一个有序数组，如果进行奇交换，一定不会发生交换；如果进行偶交换，也一定不会发生交换。所以可以得出结论：

```
一个数组奇交换是有序的，并且以偶交换也是有序的，那么这个数组一定是有序的。

奇交换有序 && 偶交换有序 = 数组有序
```

假设从偶交换开始，当某一次偶交换之后奇交换没有发生任何交换时，那么数组就是有序的了（奇偶必须成对出现），因为在偶交换的时候无论是否发生了交换，交换完成之后，其数组以偶交换的方式一定是有序的。如果当前偶交换之后的奇交换没有发生任何交换，那么数组就有序了，就可以停止进行排序，所以这就是奇偶排序算法的结束条件。

总结：


```java
public static void parityExchangeSort(int[] arr){
    //声明以奇交换开始还是以偶交换开始；0表示偶交换开始，1表示奇交换开始
    int start = 0;
    //用于标记是否发生了交换
    boolean flag;
    /*while循环，不停的切换它们的交换方式。*/
    while (true) {
        //重置标记
        flag = false;
        //因为是奇偶交换，所有步进数是2
        for (int i = start; i < arr.length - 1; i+=2) {
            //比较前一个数是否大于后一个数，大则交换它们的位置
            if (arr[i] > arr[i+1]) {
                int temp = arr[i];
                arr[i] = arr[i+1];
                arr[i+1] = temp;
                //标记当前交换方式发生了交换
                flag = true;
            }
        }
        //当前发生了交换，并且但当前是奇交换（偶交换开始），则结束排序
        if (!flag && start == 1)
            break;
        //切换交换方式
        start = start == 0 ? 1 : 0;
    };
}
```



## Parallel Parity Exchange Sort



```java
import java.util.Arrays;
import java.util.concurrent.*;

public class ParallelParityExchangeSort {
    /*注意：排序任务是CPU密集型任务，所以这里核心数为2，最大为6*/
    private static ThreadPoolExecutor threadPool = new ThreadPoolExecutor(
            2,6,60, TimeUnit.SECONDS,new LinkedBlockingQueue<>());

    /**
     * 并行奇偶交换排序的是否排序的标记声明为一个对象，
     * 因为其需要子线程中使用，以及为了线程安全。
     */
    public static class SortFlag {
        private boolean flag;
        public boolean isFlag() {
            return flag;
        }
        public void setFlag(boolean flag) {
            this.flag = flag;
        }
    }

    /**
     * 实现奇偶排序的交换逻辑，子线程中并行排序
     */
    private static class doSortTask implements Runnable {

        private int[] arr;
        private CountDownLatch latch;
        private int i;
        private SortFlag sortFlag;

        public doSortTask(int[] arr, CountDownLatch latch, int i,SortFlag sortFlag) {
            this.arr = arr;
            this.latch = latch;
            this.i = i;
            this.sortFlag = sortFlag;
        }

        @Override
        public void run() {
            if (arr[i] > arr[i+1]){
                int temp = arr[i];
                arr[i] = arr[i+1];
                arr[i+1] = temp;
                sortFlag.setFlag(true);
            }
            //当前排序完成，CountDownLatch标记+1
            latch.countDown();
        }
    }

    public static void parallelParityExchangeSort(int[] arr) 
        								throws InterruptedException {
        int start = 0;
        //涉及子线程和线程安全，所以是否排序标记变更为了对象
        SortFlag sortFlag = new SortFlag();
        while (true) {
            //重置标记
            sortFlag.setFlag(false);
            //使用CountDownLatch等待所有并行排序完成
            CountDownLatch latch = new CountDownLatch(
                arr.length/2-((arr.length&1) == 0?start:0));
            //for循环跟普通的奇偶交换排序是一样的，只是具体的交换逻辑交由子线程去实现了
            for (int i = start; i < arr.length - 1; i+=2) {
                //执行排序任务
                threadPool.execute(new doSortTask(arr,latch,i,sortFlag));
            }
            //等待并行交换排序完成
            latch.await();
            if (!sortFlag.isFlag() && start == 1)
                break;
            start = start == 0 ? 1 : 0;
        }
    }
}
```



## Insert Sort

一个未排序的数组（当然也可以是链表）可以分为两个部分，前半部分是已经排序的，后半部分是未排序的。在进行排序时，只需要在未排序的部分中选择一个元素，将其插入到前面有序的数组中即可。最终，未排序部分只会越来越少，直到为0，那么排序就完成了。初始时，可以假设已排序的部分就是第一个元素。





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

