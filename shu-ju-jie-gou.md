# 数据结构

## 二叉树

```java
class TreeNode {
    public int val;
    public TreeNode left;
    public TreeNode right;
    public TreeNode() {}
    public TreeNode(int value) {
        vla = value
    }
}
```

### 先序遍历

```java
public static int[] preorder(TreeNode root) {
    if (root == null) {
        return null;
    }
    TreeNode p = root;
    LinkedList<TreeNode> stack = new LinkedList<>();
    LinkedList<Integer> ressult = new LinkedList<>();
    while (p != null || !stack.isEmpty()) {
        if (p != null) {
            stack.push(p);
            result.add(p.val);  // 访问节点
            p = p.left;
        } else {
            p = stack.pop();
            p = p.right
        }
    }
    return result.stream().mapToInt(Integer::valueOf).toArray();
}
// 递归方式
public static void preoredr(TreeNode root, LinkedList<Integer> result) {
    if (root == null) return;
    result.add(root.val);
    preorder(root.left);
    preorder(root.right);
}
```

### 中序遍历

```java
public static int[] inorder(TreeNode root) {
    if (root == null) {
        return null;
    }
    TreeNode p = root;
    LinkedList<TreeNode> stack = new LinkedList<>();
    LinkedList<Integer> result = new LinkedList<>();
    while (p != null || !stack.isEmpty()) {
        if (p != null) {
            stack.push(p);
            p = p.left;
        } else {
            p = stack.pop();
            result.add(p.value);  // 访问节点
            p = p.right;
        }
    }
    return result.stream().mapToInt(Integer::valueOf).toArray();
}
```

### 后序遍历

```java
public static int[] postorder(TreeNode root) {
    if (root == null) {
        return null;
    }
    TreeNode p = root;
    TreeNode latestNode = null;  // 最近访问的节点
    Queue<TreeNode> stack = new LinkedList<>();
    LinkedList<Integer> result = new LinkedList<>();
    while (p != null || !stack.isEmpty()) {
        // 走到最左端
        while (p != null) {
            stack.push(p);
            p = p.left;
        }
        p = stack.peek();
        if (p.right != null && p.right != latestNode) {
            p = p.right;
            stack.push(p);
            p = p.left;
        } else {
            p = stack.pop();
            result.add(p.val);
            latestNode = p;
            // 访问之后需要重置p为null，使得不会再继续向最左走
            p = null;
        }
    }
    return result.stream().mapToInt(Integer::valueOf).toArray();
}
```

### 层次遍历

```java
public static int[] leaveOrder(TreeNode root) {
    if (root == null) return null;
    TreeNode p = root;
    LinkedList<TreeNode> queue = new LinkedList<>();
    LinkedList<Integer> result = new LinkedList<>();
    queue.add(p);
    while (!queue.isEmpty()) {
        p = queue.remove();
        result.add(p.val);
        if (p.left != null) {
            queue.add(p.left);
        }
        if (p.right != null) {
            queue.add(p.right);
        }
    }
    return result.stream().mapToInt(Integer::valueOf).toArray();
}
```

### 求树的高度

```java
public static int treeDepth(TreeNode root) {
    if (root == null) return 0;
    int left = treeDepth(root.left);
    int right = treeDepth(root.right);
    return left > right ? left+1 : right+1;
}
// 非递归
public static int treeFepth(TreeNode root) {
    if (root == null) return 0;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.add(root);
    int depth = 0; // 初始值为0
    while (!queue.isEmpty()) {
        for (int i = queue.size(); i > 0; i--) {
            TreeNode node = queue.remove();
            if (node.left != null) queue.add(node.left);
            if (node.right != null) queue.add(node.right);
        }
        depth++;
    }
    return depth;
}
```

### 求树的宽度

```java
public static int treeWidth(TreeNode root) {
    if (root == null) return 0;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.add(root);
    int width = 1;
    while (!queue.isEmpty()) {
        for (int i = queue.size(); i > 0; i--) {
            TreeNode node = queue.remove();
            if (p.left != null) {
                queue.add(p.left);
            }
            if (p.right != null) {
                queue.add(p.right);
            }
        }
        int temp = queue.size();
        if (temp > width) {
            width = temp;
        }
    }
    return width;
}
```

## 二叉搜索树

### 插入节点

```java
public static void insertNode(TreeNode root, int key) {
    TreeNode node = new TreeNode(key);
    if (root == null) {
        root = node;
    }
    TreeNode p = root;
    TreeNode preNode = null;

    while (p != null) {
        preNode = p;
        // val值为int型
        if (node.val <= p.val) {
            p = p.left;
        } else {
            p = p.right;
        }
    }
    if (node.val <= preNode.val) {
        preNode.left = node;
    } else {
        preNode.right = node;
    }
}
```

### 删除节点

* 没有左右子节点，可以直接删除
* 存在左节点或者右节点，删除后需要对子节点移动
* 同时存在左右子节点，不能简单的删除，但是可以通过和**后继**节点交换后转换为前两种情况

```java
TreeNode deleteNode(TreeNode root, int key) {
    if (root == null) return null;
    if (root.val == key) {
        // 这两个 if 把情况 1 和 2 都正确处理理了了
        if (root.left == null) return root.right;
        if (root.right == null) return root.left;
        // 处理理情况 3
        TreeNode minNode = getMin(root.right);
        root.val = minNode.val;
        root.right = deleteNode(root.right, minNode.val);
    } else if (root.val > key) {
        root.left = deleteNode(root.left, key);
    } else if (root.val < key) {
        root.right = deleteNode(root.right, key);
    }
    return root;
}
TreeNode getMin(TreeNode node) {
    // BST 最左边的就是最⼩小的
    while (node.left != null) node = node.left;
    return node;
}
```

```java
public static void deleteNode(TreeNode root, int key) {
    if (root == null) return;
    // 找到节点 node
    TreeNode p = root;
    while (p != null && p.val != key) {
        if (key < p.val) {
            p = p.left;
        }
        if (key > p.val) {
            p = p.right;
        }
    }
    if (p == null) {
        // 节点不存在
        return;
    }
    // 情况三，同时存在左右节点
    if (p.left != null && p.right != null) {
        TreeNode next = p.right;
        while (next.left != null) {
            next = next.left;
        }
        p.val = next.val; // 后置节点转移到当前节点
        p = next;  // 要删除的节点设置为后置节点
    }
    // 剩余两种情况，有子节点则获得子节点，没有子节点则为null
    TreeNode child = null;
    if (p.left != null) {
        child = p.left;
    } else {
        child = p.right;
    }
    if (child != null) {
         // 不等于null即存在一个节点
        child.parent = p.parent;
    }
    if (p.parent == null) {
        // 根节点，只有一个子节点或没有子节点，直接赋值child
        root = child;
    } else if (p == p.parent.left)
        p.parent.left = child;
    } else {
        p.parent.right = child;
    }
}
```

## 红黑树

二叉搜索树存在的问题：BST存在的主要问题是，数在插入的时候会导致树倾斜，不同的插入顺序会导致树的高度不一样，而树的高度直接的影响了树的查找效率。理想的高度是logN，最坏的情况是所有的节点都在一条斜线上，这样的树的高度为N。

基于BST存在的问题，一种新的树——平衡二叉查找树\(Balanced BST\)产生了。平衡树在插入和删除的时候，会通过旋转操作将高度保持在logN。其中两款具有代表性的平衡树分别为AVL树和红黑树**。AVL树由于实现比较复杂，而且插入和删除性能差，在实际环境下的应用不如红黑树**。

**红黑树从根节点到叶子节点的最长路径不会超过最短路径的两倍**

### 定义

红黑树首先是一个二叉查找树，其次它满足一下条件：

1. 任何一个节点都有颜色，黑色或者红色。
2. 根节点是黑色的。
3. 父子节点之间不能出现两个连续的红节点。（红节点的子节点都是黑的）
4. 从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点。
5. 空节点被认为是黑色的。

红黑树的高度在\[logN,logN+1\]（理论上，极端的情况下可以出现RBTree的高度达到2\*logN）。RBTree的删除和插入操作的时间复杂度也是O\(logN\)。

### 插入操作

Rotate分为left-rotate（左旋）和right-rotate（右旋），区分左旋和右旋的方法是：**待旋转的节点从左边上升到父节点就是右旋，待旋转的节点从右边上升到父节点就是左旋**。

设插入一个新节点X

1. 新插入的节点是红色的
2. 如果X是根节点，则标记为黑色，否则执行下一步
3. 如果X的父节点是黑色则直接插入，否则执行下一步
4. 如果X的父节点是红色，分为一下三种情况
   * 叔叔节点也为红色

     将父节点和叔叔节点与祖父节点的颜色互换，此时祖父节点为红色，相当于新节点，这个时候需要对祖父节点为起点进行调节（向上回溯）。

     ![](https://gitee.com/coldsun233/NotePic/raw/master/img/rbTree_add1.jpg)

   * 叔叔节点为空（黑色），且祖父节点、父节点和新节点处于一条斜线上。

     将父亲节点进行右旋（左旋）操作，并且父亲节点和祖父节点的颜色互换。

      左左情况 ![](https://gitee.com/coldsun233/NotePic/raw/master/img/rbTree_add2_1.jpg) 右右情况 ![](https://gitee.com/coldsun233/NotePic/raw/master/img/rbTree_add2_2.jpg)

   * 叔叔节点为空（黑色），且祖父节点、父节点和新节点不处于一条斜线上。

     将新节点进行左旋（右旋）操作，变为情况二的左左（右右）状态，然后在进行相应的处理。

     ![](https://gitee.com/coldsun233/NotePic/raw/master/img/rbTree_add3_1.jpg)

     ![](https://gitee.com/coldsun233/NotePic/raw/master/img/rbTree_add3_2.jpg)

### 删除操作

设删除一个节点X

1. 如果X是页节点，直接删除X

## 数组

## 排序

### 堆排序

#### 构建大顶堆

**假设数组从下标1开始存储数据**，代码如下：

```java
void buildMaxHeap(int[] A, int heapSize) {
    for (int i = heapSize/2; i > 0; i--) {
        heapAdjust(A, i, heapSize);
    }
}
// 对以元素 r 为根节点的子树进行调整
// 1.递归的堆调整
void heapAdjust(int[] A, int r, int heapSize) {
    int i = 2*r;
    if (i > heapsize) return;
    if (i < heapSize && A[i] < A[i+1]) i++;
    if (A[r] < A[i]) {
        swap(A, r, i);  // 交换 r 和 i
        heapAdjust(A, i, heapSize);  // 调整发生变化的子树
    }
}
// 2.非递归的堆调整
void heapAdjust(int[] A, int r, int heapSize) {
    A[0] = A[r]; // 使用A[0]暂存根节点
    for (int i = 2*r; i <= heapSize; i *= 2) {
        // i < heapSize 则 i + 1 <= heapSize 保证节点i存在右兄弟
        // 如果右兄弟更大，则 i 指向右兄弟，保证 i 指向最大的一个子节点
        if (i < heapSize && A[i] < A[i+1]) i++;
        // 如果最大子节点没有父节点大，则不需要继续调整，跳出循环
        if (A[0] >= A[i]) 
            break;
        else {
            // 最大子节点的值大于父节点的值则调整父节点的值
            // 使用A[0]记录了最开始的r节点的值，所以这里直接赋值，不需要交换
            A[r] = A[i]; 
            // 以 i 为根节点的子树发生了变化，需要进行调整
            // 所以修改 r 的值，继续向下调整
            r = i;
        }  
    }
    A[r] = A[0];  // 将被筛选节点的值放入最终位置
}
```

#### heapSort

```java
public static void maxHeapSort(int[] a, int size) {
    // 构建大顶堆，这里的a从下标1开始存储数据
    buildMaxHeap(a, size);
    for (int i = size; i > 1;) {
        swap(a, 1, i);
        i--;
        heapAdjust(a, 1, i);
    }
}
```

### 快排

#### partition

```java
/**
 * 在数组 nums 的子区间 [left, right] 执行 partition 操作，返回 nums[left] 排序以后应该在的位置
 * 变量 i 是小于 num[left] 和大于等于 num[left] 的分界
 * 变量 j 是循环变量
 * 在遍历过程中保持循环不变量的语义
 * 1、[left + 1, i] < nums[left]
 * 2、(i, j] >= nums[left]
 */
private int partition(int[] nums, int left, int right) {
    int pivot = nums[left];
    int i = left; // [left, i]的元素都是小于pivot的
    for (int j = left+1; j <= right; j++) {
        if (num[j] < pivot) {
            i++;
            if (i != j)
                swap(num, i, j);
        }
    }
    // 在之前遍历的过程中，满足 [left + 1, i] < pivot，并且 (i, right] >= pivot
    swap(num, left, i);
    // 交换之后[left, i-1] < pivot, [i+1, right] >= pivot
    return i;
}
```

#### quickSort

```java
public static void quickSort(int a[],int l,int r) {
    if (l >= r) return;
    int j = partition(a, l, r);
    quickSort(a, l, j-1);
    quickSort(a, j+1, r);
}
```

### 归并排序

```java
public static void merge_sort(int a[],int first,int last,int temp[]){
  if(first < last){
      int middle = (first + last)/2;
      merge_sort(a,first,middle,temp);//左半部分排好序
      merge_sort(a,middle+1,last,temp);//右半部分排好序
      mergeArray(a,first,middle,last,temp); //合并左右部分
  }
}
```

```java
public static void mergeArray(int a[],int first,int middle,int end,int temp[]){    
  int i = first;
  int m = middle;
  int j = middle+1;
  int n = end;
  int k = 0;
  while(i<=m && j<=n){
      if(a[i] <= a[j]){
          temp[k] = a[i];
          k++;
          i++;
      }else{
          temp[k] = a[j];
          k++;
          j++;
      }
  }    
  while(i<=m){
      temp[k] = a[i];
      k++;
      i++;
  }    
  while(j<=n){
      temp[k] = a[j];
      k++;
      j++;
  }

  for(int ii=0;ii<k;ii++){
      a[first + ii] = temp[ii];
  }
}
```

## 字符串

### KMP

求next数组，next\[i\] = k，表示模式串的子串`pattern[0:i]`的前缀和后缀最长公共元素的长度为k。

```java
public int[] getNext(String pattern) {
    char[] p = pattern.toCharArray();
    int len = p.length;
    int[] next = new int[len];
    /**
    已知初始值next[0] = 0
    求next[i]，需要根据next[i-1]来求，next[i-1] = k，代表子串pattern[0:i-1]的k前缀等于k后缀
    即pattern[0:k-1] == pattern[i-k:i-1]，此时需要比较pattern的第k+1个值即pattern[k]是否与
    pattern[i]相等，相等则next[i] = k+1，不相等，需要用到pattern的自匹配，即pattern[0:k-1]
    已经和pattern[i-k:i-1]匹配完毕，但是这两个串的下一个字符不相等，因为next[k-1]是已知的,所以我
    可以将K赋值为next[k-i]然后重新进行匹配。
    **/
    for (int i = 1; i < len; i++) {
        int k = next[i - 1];
        while (k > 0 && p[k] != p[i]) {
            k = next[k - 1];
        }
        if (p[k] == p[i]) {
            k++;
        }
        next[i] = k;
    }
}
```

```java
// 在文本 text 中寻找模式串 pattern，返回所有匹配的位置开头
public int[] KMP(String text, String pattern) {
    if (text.length() == 0 || pattern.length() == 0) {
        return new int[0];
    } 
    List<Integer> result = new ArrayList<>();
    int[] next = getNext(pattern);
    int j = 0;  // j表示pattern的下标
    for (int i = 0; i < text.length(); i++) {
        while (j > 0 && pattern.charAt(j) != text.charAt(i)) {
            j = next[j - 1];
        }
        if (pattern.charAt(j) == text.charAt(i)) {
            j++;
        }
        if (j == pattern.length) {
            result.add(i - pattern.length + 1); // 如果值求一个或判断是否存在，在这里就可以返回。
            j = next[j - 1];
        }
    }
    return result.stream().mapToInt(Integer::valueOf).toArray();
}
```

## 回溯

## 动态规划

动态规划一般是自顶向下，在这个过程中，可能会出现重复的子问题，这时可以使用备忘录的形式来减少不必要的计算，此外，可以根据递推公式自底向上，进行优化。5

## 股票交易

股票交易共有三个状态，天数，最大交易次数，是否持有股票\(0表示不持有，1表示持有\)，使用一个三维数组表示，共有$n\times k \times 2$种状态

```java
dp[i][k][0]  // 表示第i天，没有持有股票，最大交易次数为k，所获得的的利润
dp[i][k][1]  // 表示第i天，持有股票，最大交易次数为k，所获得的利润
dp[n-1][k][0]  // 所要求的最终结果
// 状态转移方程
dp[i][k][0] = max(dp[i-1][k][0], dp[i-1][k][1] + price[i-1])  
       // 第i-1天没有持有股票，也不买入    第i-1天持有股票，卖出
dp[i][k][1] = max(dp[i-1][k][1], dp[i-1][k-1][0] - price[i-1])
       // 第i-1天持有股票，不卖出       第i-1天不持有股票，买入
// 初始条件
// i = 0（还未开始）, k > 0
dp[0][k][0] = 0
dp[0][k][1] = Integer.MIN_VALUE
// i > 0（已经开始）, k = 0
dp[i][0][0] = 0
dp[i][0][1] = Integer.MIN_VALUE

// init basecase
int[][][] dp = new int[ni+1][nk+1][2];
```

