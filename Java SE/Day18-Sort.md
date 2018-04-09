# 排序

#### 内容摘要

冒泡排序，选择排序，插入排序，归并排序，快速排序，堆排序

---

#### 冒泡排序 *Bubble Sort*

* 算法思想：重复地走访过要排序的数列，一次比较两个元素，如果他们的顺序错误就交换。直到没有再需要交换，该数列已经排序完成。越小的元素会经由交换慢慢“浮”到数列的顶端
* 平均时间复杂度O(n^2^)，空间复杂度O(1)

#### 选择排序 *Selection Sort*

* 算法思想：在未排序序列中找到最小（大）元素，存放到未排序序列的起始位置
* 平均时间复杂度O(n^2^)，空间复杂度O(1)

#### 插入排序 *Insertion Sort*

* 算法思想：将数组中的所有元素依次跟前面已经排好的元素相比较，如果选择的元素比已排序的元素小，则交换，直到全部元素都比较过为止


* 平均时间复杂度O(n^2^)，空间复杂度O(1)

#### 归并排序 *Merge Sort*

* 算法思想：将两个**有序表**组成一个新的有序表，递归将两个元素看作有序表，再不断递归
* 归并**有序表**的时间复杂度：O(n + m)
* 归并**无序表**的时间复杂度：O(n * m)

```Java
	public void mergeSort() {
        System.out.print("Merge sort : ");
        mergeSort(arr, 0, arr.length - 1);
    }
    private void mergeSort(int[] arr, int startPos, int endPos) {
        if (startPos < endPos) {
            int mid = (startPos + endPos) / 2;
            // 让左表有序
            mergeSort(arr, startPos, mid);
            // 让右表有序
            mergeSort(arr, mid + 1, endPos);
            // 归并两个有序表
            merge(arr, startPos, mid, endPos);
        }
    }
    // Merge
    private void merge(int[] arr, int startPos, int mid, int endPos) {
        // 确定待归并的左表范围
        int lStart = startPos;
        int lEnd = mid;
        // 确定待归并的右表范围
        int rStart = mid + 1;
        int rEnd = endPos;
        // 辅助数组
        int[] temp = new int[endPos - startPos + 1];
        int tempStart = 0;
        // 当左右有序表都有元素时
        while (lStart <= lEnd && rStart <= rEnd) {
            if (arr[lStart] < arr[rStart])
                temp[tempStart++] = arr[lStart++];
            else
                temp[tempStart++] = arr[rStart++];
        }
        // 若左表还有元素
        while (lStart <= lEnd)
            temp[tempStart++] = arr[lStart++];
        // 若右表还有元素
        while (rStart <= rEnd)
            temp[tempStart++] = arr[rStart++];
        // 辅助数组的值给排序数组
        // 辅助数组下标从 0 开始
        // 排序数组下标从 0 + startPos 开始
        for (int i = 0; i < temp.length; i++) {
            arr[i + startPos] = temp[i];
        }
    }
```

#### 快速排序 *Quick Sort*

* 算法思想：通常取表头元素作为基准（pivot），将排序序列分为两部分，左边部分小于等于基准，右边部分大于等于基准，即基准的最终位置已确定，再对左部分和右部分递归重复，直到每个部分都只有1个元素为止，即每个元素最后位置已确定
* 算法时间复杂度：O(log n)

```Java
	public void quickSort() {
        System.out.print("Quick sort : ");
        quickSort(arr, 0, arr.length - 1);
    }
    // 递归找分别确定 pivot 的最终位置
    private void quickSort(int[] arr, int startPos, int endPos) {
        if(startPos < endPos) {
            int pivot = arr[startPos];
            int pivotPosition = findPivot(arr, startPos, endPos, pivot);
            quickSort(arr, startPos, pivotPosition - 1);
            quickSort(arr, pivotPosition + 1, endPos);
        }
    }
    // 双端比较，返回 pivot 最终位置下标
    // 注意，此处要先从后向前扫描，再从前向后扫描
    private int findPivot(int[] arr, int startPos, int endPos, int pivot) {
        while (startPos < endPos) {
            // 从后向前扫面，若小于 pivot 则与前端元素交换
            while (startPos < endPos && arr[endPos] >= pivot) endPos--;
            arr[startPos] = arr[endPos];
            // 从前向后扫描，若大于 pivot 则与后端元素交换
            while (startPos < endPos && arr[startPos] <= pivot) startPos++;
            arr[endPos] = arr[startPos];
        }
        // 最后即找到 pivot 最终位置，赋值并返回下标
        arr[startPos] = pivot;
        return startPos;
    }
```



#### 堆排序 *Heap Sort*

* 堆：
  * n 个元素的序列{k1,…, kn}，**满足 k~i~ <= k~2i~ ; k~i~ <= k~2i+1~**
  * 小顶堆：父节点的值是当前树中的最小值
  * 大顶堆：父节点的值时当前树中的最大值
  * 对应的一维数组，可以看作一颗完全二叉树顺序表示，所有父节点的值均不大于孩子节点的值
  * **下标从 1 开始，父节点下标 i ，其左孩子 2i ，右孩子 2i + 1**
* 构造堆
  * 从一个无序序列中构造一个原始堆
  * 若当前序列中有 n 个元素，则从其父节点 n ／ 2 开始处理，对其中每一个根节点执行 `fixDown`
  * 在删除堆顶元素之后，调整堆中元素，成为一个新堆
* 堆的基本操作
  * `fixDown`  删除元素后，自上向下调整堆中节点
  * `fixUp` 插入元素后，自底向上调整堆中节点

```Java
package day18.sort;

/*
        大顶堆，父节点值大于孩子节点值
        父节点 i 左孩子节点 2i 右孩子节点 2i + 1
        排序
 */
public class MyHeap {
    // 存放元素数组
    private int[] arr;
    public int size;
    // 堆元素下标从 1 开始
    public MyHeap(int[] arr) {
        this.arr = new int[arr.length + 10];
        for (int i = 0; i < arr.length; i++) {
            this.arr[i + 1] = arr[i];
        }
        this.size = arr.length;
    }

    // 构造堆
    private void heapify() {
        // 从最后一个元素开始,对每个节点进行构造堆操作
        for (int i = size; i > 0; i--) {
            fixDown(i);
        }
    }

    // 下沉操作
    private void fixDown(int i) {
        recursiveFixDown(i, size);
    }

    // 构造堆
    // 1. 找到当前序列中最后位置 n ，以及其父节点 n / 2，判断是否符合堆性质，不满足的执行下沉操作
    // 2. 处理下一个父节点 n / 2 - 1
    private void fixDown(int index, int size) {
        for (int i = index; i <= size; ) {
            // 左孩子节点下标
            int child = i * 2;
            // 如果其左孩子节点下标大于元素个数，则表示没有左孩子，也就没有右孩子
            if (child > size) break;
            // 如果右孩子存在，且右孩子值大于左孩子值，则孩子最大值下标为右孩子
            if (child + 1 <= size && arr[child + 1] > arr[child])
                child++;
            // 大顶堆，父节点值大于孩子节点值
            // 若孩子节点大于父节点，则交换，并从孩子节点向下扫描
            // 否则，退出此次循环，处理下一个父节点
            if (arr[child] > arr[i]) {
                swap(child, i);
                i = child;
            } else break;
        }
    }

    // 递归实现 fixDown
    private void recursiveFixDown(int index, int size) {
        int child = index * 2;
      	// 若孩子下标越界
        if (child > size)  return;
        if (child + 1 <= size && arr[child + 1] > arr[child]) {
            child++;
        }
        if(arr[child] > arr[index]) {
            swap(child, index);
            recursiveFixDown(child, size);
        }

    }

    // 插入元素，在最后一个位置插入元素，再进行节点调整
    public void insert(int e) {
        arr[++size] = e;
        fixUp();
    }

    // 插入元素后，自顶向下调整，扫描
    private void fixUp() {
        for (int i = size; i > 0; ) {
            int parent = i / 2;
            // 大顶堆，父节点值大于孩子节点值
            if (arr[parent] > arr[i]) {
                swap(parent, i);
                i = parent;
            } else break;

        }
    }

    // Heap sort
    public void heapSort() {
        // 先构建一个堆
        heapify();

        // 从后向前扫描堆元素
        for (int i = size; i > 0; ) {
            // 将堆顶元素与最后一个元素交换
            swap(1, i);

            // 最后一个元素出堆
            i--;

            // 删除后，调整堆顶节点
            fixDown(1, i);
        }
    }

    private void swap(int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }

    @Override
    public String toString() {
        if (size == 0)
            return "[]";
        StringBuilder builder = new StringBuilder("[");
        for (int i = 1; i <= size; i++) {
            builder.append(arr[i]);
            if (i == size) {
                builder.append(']');
                break;
            }
            builder.append(", ");
        }
        return "MyHeap : " + builder.toString();
    }
}

```

