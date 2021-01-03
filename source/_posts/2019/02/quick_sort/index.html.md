---
layout: post
title: 升级版冒泡排序：快速排序
date: 2019-02-28
tags: ["O(n)","日志","quickSort","优化","冒泡排序","快排","快速排序","排序方案","时间复杂度","直接插入排序","空间","空间复杂度","计算机基础知识"]
categories:
- 计算机
---

![快速排序](quick_sort.png "快速排序")

快速排序算法是由图灵奖获得者 [Tony Hoare](https://zh.wikipedia.org/wiki/%E6%9D%B1%E5%B0%BC%C2%B7%E9%9C%8D%E7%88%BE) 设计出来的，被列为 20 世纪十大算法之一；之所以说快速排序是我们一直认为最慢的冒泡排序的升级，是因为它们都属于交换排序类。也就是它们都是通过不断的比较和移动交换元素来实现排序的。

快速排序的基本思想是：**通过一轮排序把待排序的元素分割成独立的两部分，其中一部分元素均小于另一部分元素；然后用相同规则继续递归分割这两部分元素，直到整个序列有序为止。**根据指导思想便可以写出下列快速排序算法的主函数：

    void quickSort(int arr[], int low, int high)
    {
        if (low < high) { // 当 low >= high 时，递归停止
            int pivot;
            pivot = partition(arr, low, high); // 调用分区函数，将待排序序列一分为二，pivot 为中轴元素下标
            quickSort(arr, low, pivot - 1);     // 分区左边进行递归分区
            quickSort(arr, pivot + 1, high);   // 分区右边进行地柜分区
        }
    }

先从宏观上写出这个函数，其实不是很复杂；基本思路是用分区函数对待排序的序列进行一分为二的分区，拿到中轴元素的下标后，对中轴元素左边的待排序序列继续递归分区，中轴元素右边相同。关键的是实现分区函数：

    // 分区函数
    int partition(int arr[], int low, int high)
    {
        int temp;
        int pivotKey;
        pivotKey = arr[low]; // 开始把 low 对应的元素设置为分区的中轴

        while (low < high) {

            while (low < high && arr[high] >= pivotKey) {
                high--;
            }

            temp = arr[low];
            arr[low] = arr[high];
            arr[high] = temp;

            // 执行到这里，high 下标指向的元素正是 pivotKey (中轴元素)
            // 此时需要从低位开始分区

            while (low < high && arr[low] <= pivotKey) {
                low++;
            }

            temp = arr[low];
            arr[low] = arr[high];
            arr[high] = temp;

            // 执行完后，pivotKey (中轴元素)又回到 low 下标指向的元素
        }

        return low;
    }

仔细阅读分区函数代码，也不是很复杂；交换的关键，是用中轴元素与序列中的其他元素一一比较，进行换位操作；假设待排序的序列是 {50, 10, 20, 60, 40, 30, 70}，开始 pivotKey 设置为 low 下标的元素，即 50，如下图：

![快速排序](quickSort01.png)

接下来，开始执行分区代码中的第一部分交换代码:

    while (low < high && arr[high] >= pivotKey) {
        high--;
    }

    temp = arr[low];
    arr[low] = arr[high];
    arr[high] = temp;
    // 执行到这里，high 下标指向的元素正是 pivotKey (中轴元素)

执行后，high 下标指向 pivotKey，如下图：

![快速排序](quickSort02.png)

这时需要从 low 下标(低位)开始进行比较分区，即执行第二部分分区代码，

    while (low < high && arr[low] <= pivotKey) {
        low++;
    }

    temp = arr[low];
    arr[low] = arr[high];
    arr[high] = temp;
    // 执行完后，pivotKey (中轴元素)又回到 low 下标指向的元素

分区后 pivotKey 又回到 low 下标指向的元素，如图：

![快速排序](quickSort03.png)

如此，再次外层 while 循环，直到 low >= hight，整个分区排序完成。

## 时间、空间复杂度

最好情况下，也就是分区函数每次都选择一个中间值作为中轴，时间复杂度为 O(nlogn)；最坏情况，待排序的序列为正序或者逆序，时间复杂度为 O($n^2$)；综合分析，平均时间复杂度为： O(nlogn)。

> 常用时间复杂度从低阶到高阶：O(1) < O(logn) < O(n) < O(nlogn) < O($n^2$)

就空间复杂度来说，主要是递归调用栈的开销；平均情况的递归调用栈为 O(logn)；

## 快速排序优化

**1: 优化分区函数**

对于分区函数来说，如果我们每次选取的中轴数 pivotKey 都趋向于中间位置，那么复杂度就趋向于最优情况。那么问题的关键就是选取这个中轴数字，一般会采用 **三数取中(如果序列整体基数大，可以适当的扩大"三数")** 的办法，选序列中最左、最右、中间的三个数中的中间数字作为中轴数(或者随机选序列中的三个数，但是注意，随机生成数也有开销，所以一般选左、中、右三数)，这样至少能保证这个中轴数肯定不是整体序列中最小的或者最大的数字；以三数取中为例，代码如下：

    int partition(int arr[], int low, int high)
    {
        int pivotKey;
        int mid = low + (high - low) / 2;
        if (arr[low] > arr[high]) {
            swap(arr, low, high);
        }

        if (arr[mid] > arr[high]) {
            swap(arr, mid, high);
        }

        if (arr[mid] > arr[low]) {
            swap(arr, mid, low);
        }

        pivotKey = arr[low]; // 开始把 low 对应的元素设置为分区的中轴

        while (low < high) {

            while (low < high && arr[high] >= pivotKey) {
                high--;
            }
            swap(arr, low, high);

            // 执行到这里，high 下标指向的元素正是 pivotKey (中轴元素)
            // 此时需要从低位开始分区

            while (low < high && arr[low] <= pivotKey) {
                low++;
            }
            swap(arr, low, high);

            // 执行完后，pivotKey (中轴元素)又回到 low 下标指向的元素
        }

        return low;
    }

这里由于代码中多处用到了交换元素位置的功能功能，所以抽离了一个数据交换函数 swap ，实现也很简单，就不贴示例代码；上述示例中，待排序的序列基数是比较小的(共 7 个元素排序)，pivotKey 的取值采用了**三数取中**的办法；如果待排序序列有 70 个元素，那么为了更好的找到一个合适的 pivotKey ，我们也可以采用**九数取中**甚至更大的数字来确定 pivotKey，但是也不宜过大，因为即使是 9 数取中，我们也要先对这 9 个数进行排序，而这又涉及到另外一次排序的开销了。

**2：优化元素的不必要交换**

观察分区函数 partition 中的外层 while 循环代码:

    while (low < high) {
            while (low < high && arr[high] >= pivotKey) {
                high--;
            }
            swap(arr, low, high);
            // 执行到这里，high 下标指向的元素正是 pivotKey (中轴元素)
            // 此时需要从低位开始分区

            while (low < high && arr[low] <= pivotKey) {
                low++;
            }
            swap(arr, low, high);

            // 执行完后，pivotKey (中轴元素)又回到 low 下标指向的元素
    }

代码中，共执行了两次元素交换，但其实，我们大可不必交换元素，因为一次循环下来， 中轴元素又落到了 low 下标位置，我们只需要在循环结束后，重新给 low 下标元素赋值回 pivotKey 即可，中间交换值的代码直接改为赋值即可。那么循环体部分代码实现如下：

    while (low < high) {

            while (low < high && arr[high] >= pivotKey) {
                high--;
            }
            arr[low] = arr[high]; // 改为直接赋值

            while (low < high && arr[low] <= pivotKey) {
                low++;
            }
            arr[high] = arr[low]; // 改为直接赋值
    }

    arr[low] = pivotKey; // while 循环结束，low 和 high 会和，把中轴元素值赋值给 low 下标位置

因为代码少了多次数据交换，在性能上也得到了提高。

**3: 优化递归操作代码**

递归对性能是有一定影响的，每次递归调用都要耗费一定的栈空间，函数的参数越多。优化递归，就需要优化外层函数 quickSort，这里贴出 quickSort 优化前后的代码：

    // 优化前函数
    void quickSort(int arr[], int low, int high)
    {
        if (low < high) { // 当 low >= high 时，递归停止
            int pivot;
            pivot = partition(arr, low, high);
            quickSort(arr, low, pivot - 1);
            quickSort(arr, pivot + 1, high);
        }
    }

    // 优化后函数
    void quickSort1(int arr[], int low, int high)
    {
        while (low < high) { // 改为 while 循环
            int pivot;
            pivot = partition(arr, low, high);
            quickSort(arr, low, pivot - 1);
            low = pivot + 1; // 尾递归，把双重递归改为迭代
        }
    }

这样修改代码后，原来的 quickSort(arr, pivot + 1, high) 会在低位子表递归完成后执行，这样用迭代的方式来缩减堆栈深度，从而提高性能。

到这里，单独对快速排序的优化已经差不多了，贴出整体代码：

    void swap (int arr[], int i, int j)
    {
        if (i != j) {
            int temp = arr[i];
            arr[i] = arr[j];
            arr[j] = temp;
        }
    }

    int partition(int arr[], int low, int high)
    {
        int pivotKey;
        int mid = low + (high - low) / 2;
        if (arr[low] > arr[high]) {
            swap(arr, low, high);
        }

        if (arr[mid] > arr[high]) {
            swap(arr, mid, high);
        }

        if (arr[mid] > arr[low]) {
            swap(arr, mid, low);
        }

        pivotKey = arr[low]; // 开始把 low 对应的元素设置为分区的中轴

        while (low < high) {

            while (low < high && arr[high] >= pivotKey) {
                high--;
            }
            arr[low] = arr[high];

            while (low < high && arr[low] <= pivotKey) {
                low++;
            }
            arr[high] = arr[low];
        }

        arr[low] = pivotKey; // low 和 high 回合，把中轴元素值赋值给 low 下标位置

        return low;
    }

    void quickSort(int arr[], int low, int high)
    {
        while (low < high) { // 当 low >= high 时，递归停止
            int pivot;
            pivot = partition(arr, low, high);
            quickSort(arr, low, pivot - 1);
            low = pivot + 1; // 尾递归，把双重递归改为迭代
        }
    }

## 最后，多种排序方案混合使用，来提高排序元素较少情况下的效率

* * *

快速排序算法，对于一个基数很大的排序序列来说，效率很好；但是如果待排序序列基数很小，快速排序反而不如直接插入排序来的高效，所以我们可以设定某个基准值(元素个数)作为分割，来选择不同的排序算法，所以就需要改造 quickSort 函数，代码中包含部分伪代码：

    void quickSort2(int arr[], int low, int high)
    {
        if ((high - low) > MAX_Length_INSERT) { // MAX_Length_INSERT 为某个基准值
            while (low < high) { // 当 low >= high 时，递归停止
                int pivot;
                pivot = partition(arr, low, high);
                quickSort(arr, low, pivot - 1);
                low = pivot + 1; // 尾递归，把双重递归改为迭代
            }
        } else {
            insertSort(arr, high - low + 1); // 此时采用直接插入排序
        }
    }

MAX_Length_INSERT 作为分割的基准值(有资料显示 7 比较合适，也有说 50 的)，当元素个数大于这个数时候，采用快速排序，反之，采用直接插入排序。这样就能结合两种排序算法的共同优势来提高整体的排序效率和性能。

(完)