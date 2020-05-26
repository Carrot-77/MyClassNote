# Heap

堆是一种特殊的完全二叉树，我们可以用数组来存储。

```c
typedef struct priority_queue {
    int *data;
    int cnt, size;
} priority_queue;
```

## 结构操作

堆的特殊操作是：pop, top, push

### 1. init

```c
priority_queue *init(int n) {
    priority_queue *q = (priority_queue *)malloc(sizeof(priority_queue));
    q->data = (int *)malloc(sizeof(int) * (n + 1));
    q->cnt = 0;
    q->size = n;
    return q;
}
```

### 2. empty （堆判空操作）

``` c
int empty(priority_queue *q) {
    return q->cnt == 0;
}
```

### 3. top (返回栈顶元素)

> 栈顶元素为最大或最小值

```c
int top(priority_queue *q) {
    return q->data[1];
}
```

### 5. output

```c
void output(Stack *s) {
    printf("Stack(%d) = [", s->top + 1);
    for (int i = 0; i <= s->top; i++) {
        i && printf(", ");
        printf("%d", s->data[i]);
    }
    printf("]\n");
}
```

### 6. push

**论什么元素，先把元素插入到数组的末尾，在和父节点进行比较，判断是否和父节点进行交换，否则不进行调整。调整以后，在比较。**

```c
int push(priority_queue *q, int val) {
    if (q == NULL) return 0;
    if (q->cnt == q->size) return 0;
    q->cnt += 1;
    q->data[q->cnt] = val;
    int ind = q->cnt;
    while (ind >> 1 && q->data[ind] > q->data[ind >> 1]) {
        swap(q->data[ind], q->data[ind >> 1]);
        ind >>= 1;
    }
    return 1;
}
```

### 7.pop

**算法思想：先使用数组最末尾的元素替换第一个元素(根元素),然后判断当前根元素的左右孩子是否比根元素大，如果是，进行调整。一直这样进行递归，直到当前根元素比左右孩子都大，这样，就得到了堆。**

```c
int pop(priority_queue *q) {
    if (q == NULL) return 0;
    if (empty(q)) return 0;
    q->data[1] = q->data[q->cnt--];
    int ind = 1;
    while ((ind << 1) <= q->cnt) {	//从根节点向下递归重新构建堆
        int temp = ind, l = ind << 1, r = ind << 1 | 1;
        if (q->data[l] > q->data[temp]) temp = l;
        if (r <= q->cnt && q->data[r] > q->data[temp]) temp = r;
        if (temp == ind) break;
        swap(q->data[ind], q->data[temp]);
        ind = temp;
    }
    return 1;
}
```

### 8. clear

```c
void clear(priority_queue *q) {
    if (q == NULL) return ;
    free(q->data);
    free(q);
    return ;
}
```

## 堆排序（时间复杂度nlgn)





![](C:\Users\黄昏眺望黎明\Pictures\Screenshots\屏幕截图(6).png)



**downUpdata函数：从上往下重新构造堆的过程**

**heap_sort函数：将待排序序列构造成一个大顶堆，此时，整个序列的最大值就是堆顶的根节点。将其与末尾元素进行交换，此时末尾就为最大值。然后将剩余n-1个元素重新构造成一个堆，这样会得到n个元素的次小值。如此反复执行，便能得到一个有序序列了**

```c
void downUpdate(int *arr, int n, int ind) {
    while ((ind << 1) <= n) {
        int temp = ind, l = ind << 1, r = ind << 1 | 1;
        if (arr[l] > arr[temp]) temp = l;
        if (r <= n && arr[r] > arr[temp]) temp = r;
        if (temp == ind) break;
        swap(arr[temp], arr[ind]);
        ind = temp;
    }
    return ;
}

void heap_sort(int *arr, int n) {
    arr -= 1;
    for (int i = n >> 1; i >= 1; i--) {
        downUpdate(arr, n, i);
    }
    for (int i = n; i > 1; i--) {
        swap(arr[i], arr[1]);
        downUpdate(arr, i - 1, 1);
    }
    return ;
}
```