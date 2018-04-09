# 树

#### 内存摘要

二叉树链式映像，二叉树便利，排序

---

#### 链式映像

> * 深度优先遍历 DFS
>   * 前序遍历：根结点 => 左子树 => 右子树
>   * 中序遍历：左子树 => 根结点 => 右子树
>   * 后序遍历：左子树 => 右子树 => 根结点
> * 广度优先遍历 BFS
>   * 层次遍历：每一层都从左向右遍历

```Java
// 根据先序，中序，构造一颗二叉树
    public void preAndMidBuild(char[] pre, char[] mid) {
        this.pre = pre;
        this.mid = mid;
        root = preAndMidBuild(pre, 0, pre.length - 1, mid, 0, mid.length - 1);
    }

    // 递归先序中序构建一颗二叉树
    private TreeNode preAndMidBuild(char[] pre, int lPre, int rPre, char[] mid, int lMid, int rMid) {
        // 根节点
        char root = pre[lPre];
        // 找到中序序列中当前根节点下标
        int midRootIndex = findMid(mid, root, lMid, rMid);
        if (midRootIndex == -1)
            throw new IllegalArgumentException("Illeagl argument.");
        // 左子树节点个数
        int lNum = midRootIndex - lMid;
        // 右子树节点个数
        int rNum = rMid - midRootIndex;
        // 左右子树
        TreeNode left, right;
        // 若左子树节点个数 为 0
        if (lNum == 0)
            left = null;
        else // 若节点个数 不为 0 lNum = rPre - (lPre + 1) + 1 ==> rPre = lPre + lNum
            left = preAndMidBuild(pre, lPre + 1, lPre + lNum, mid, lMid, midRootIndex - 1);
        // 若右子树节点个数 为 0
        if (rNum == 0)
            right = null;
        else // 若节点个数 不为 0
            right = preAndMidBuild(pre, rPre - rNum + 1, rPre, mid, midRootIndex + 1, rMid);
        return new TreeNode(root, left, right);
    }

// 根据后序，中序，构造一颗二叉树
    public void postAndMidBuild(char[] post, char[] mid) {
        this.post = post;
        this.mid = mid;
        root = postAndMidBuild(post, 0, post.length - 1, mid, 0, mid.length - 1);
    }

    // 递归先序中序构建一颗二叉树
    private TreeNode postAndMidBuild(char[] post, int lPost, int rPost, char[] mid, int lMid, int rMid) {
        // 根节点
        char root = post[rPost];
        // 找到中序序列中当前根节点下标
        int midRootIndex = findMid(mid, root, lMid, rMid);
        if (midRootIndex == -1)
            throw new IllegalArgumentException("Illeagl argument.");
        // 左子树节点个数
        int lNum = midRootIndex - lMid;
        // 右子树节点个数
        int rNum = rMid - midRootIndex;
        // 左右子树
        TreeNode left, right;
        // 若左子树节点个数 为 0
        if (lNum == 0)
            left = null;
        else // 若节点个数 不为 0
            left = postAndMidBuild(post, lPost, lPost + lNum - 1, mid, lMid, midRootIndex - 1);
        // 若右子树节点个数 为 0
        if (rNum == 0)
            right = null;
        else // 若节点个数 不为 0
            right = postAndMidBuild(post, rPost - rNum, rPost - 1, mid, midRootIndex + 1, rMid);
        return new TreeNode(root, left, right);
    }
```

#### 扩展探索

* 线索二叉树
* 二叉搜索树
* 平衡二叉树：平衡因子（-1，0，1）
* 红黑树： linux 或 jdk 中的 TreeMap 实现就是红黑树，一种近似平衡二叉树的树

# 排序

#### 冒泡排序 *Bubble Sort*

* 算法思想：重复地走访过要排序的数列，一次比较两个元素，如果他们的顺序错误就交换。直到没有再需要交换，该数列已经排序完成。越小的元素会经由交换慢慢“浮”到数列的顶端

* 平均时间复杂度O(n^2^)，空间复杂度O(1)

* 相邻元素大小不符合要求时才调换位置，并不改变相同元素之间的相对顺序，是一种稳定排序

  ```java
  /**
   * Bubble Sort
   *
   * 1.比较相邻元素，如果顺序错误就交换
   * 2.对每一对相邻元素进行比较，最后一个元素会是最大的数
   * 3.对余下元素重复1～2步骤，直到没有任何数字需要比较
   * 
   * @param array  待排序数组
   */
      public void bubbleSort() {
          System.out.print("Bubble sort : ");
          // 外层倒序遍历，提高效率
          for (int i = arr.length - 1; i >= 0; i--) {
              // 内层，比较两个元素，较大元素后移
              for (int j = 0; j < i; j++) {
                  if (arr[j] > arr[j + 1])
                      swap(j, j + 1);
              }
          }
      }
  ---------------------------------------------------------------------------
  	// 某一趟冒泡过程中，并未发生交换操作
      //（说明此时整个带排序列表中的元素已经有序）
      // 此时不在进行接下来的比较过程，需要优化
      // 解决办法：设置一个 flag 在每趟比较中初始化为 false，若发生比较 flag = true
      // 若 flag 不是 false 的时候，后面序列无需再比较，直接退出循环
      public void bubbleSortPlus() {
          System.out.println("Bubble sort plus : ");
          for (int i = arr.length - 1; i >= 0; i--) {
              boolean flag = false;
              for (int j = 0; j < i; j++) {
                  if(arr[j] > arr[j+1]) {
                      flag = true;
                      swap(j, j + 1);
                  }
              }
              if(!flag) break;
          }
      }
  ```



#### 选择排序 *Selection Sort*

* 算法思想：在未排序序列中找到最小（大）元素，存放到未排序序列的起始位置

* 平均时间复杂度O(n^2^)，空间复杂度O(1)

* 不稳定

  ```java
  /**
  * Selection Sort
  *
  * 1.在未排序序列中找到最小元素
  * 2.如果最小元素不是待排序序列中第一个元素，则交换
  * 3.在余下的未排序元素中，重复1～2步骤，直到结束
  *
  * @param array  待排序数组
  */
  	public void selectionSort() {
          System.out.print("Selection sort : ");
          // 外层已排序序列
          for (int i = 0; i < arr.length - 1; i++) {
              int minIndex = i;
              // 内层未排序序列，当还剩最后一个未排序时，即为最大值，不需要比较，故 len - 2
              for (int j = i; j < arr.length - 2; j++) {
                  if (arr[j] < arr[minIndex])
                      minIndex = j;
              }
            	// 若最小元素不是待排序序列中第一个元素，交换
              if (minIndex != i)
                  swap(i, minIndex);
          }
      }
  ```

#### 插入排序 *Insertion Sort*

* 算法思想：将数组中的所有元素依次跟前面已经排好的元素相比较，如果选择的元素比已排序的元素小，则交换，直到全部元素都比较过为止

* 平均时间复杂度O(n^2^)，空间复杂度O(1)

* 每次只移动一个元素的位置，且不会改变值相同的元素之间的排序，是一种稳定排序

  ```java
  /**
  * Insertion Sort
  *
  * 1.从第一个元素开始，可以认为第一个元素已排序
  * 2.取出下一个元素i，在已排序的元素序列中从后向前扫描
  * 3.如果该已排序元素j大于i，则将该已排序元素j移到下一位置
  * 4.重复步骤3，直到找到已排序元素j小于或者等于i的位置
  * 5.将i插入到已排序元素j后
  * 6.重复步骤2～5
  *
  * @param array 待排序数组
  */
  	public void insertionSort() {
          System.out.print("Insertion sort : ");
          // 外层从第二个元素开始，假设第一个元素为已排序序列，遍历全部元素
          for (int i = 1; i < arr.length; i++) {
              // 内层在已排序序列中，从后向前扫描，若找到合适位置则插入（交换），无需比较则 break
              for (int j = i; j > 0; j--) {
                  if (arr[j] < arr[j - 1])
                      swap(j, j - 1);
                  else break;
              }

          }
      }
  ```

#### 归并排序 *Merge Sort*

* 算法思想：将两个**有序表**组成一个新的有序表，递归将两个元素看作有序表，再不断递归
* 归并**有序表**的时间复杂度：O(n + m)
* 归并**无序表**的时间复杂度：O(n * m)



















