# Queue

队列是一种先进现出的结构

## 结构定义

``` c
typedef struct Queue {
    int *data;
    int head, tail, size;
} Queue;
```

## 结构操作

### 1. init

```c
Queue *init(int n) {
    Queue *q = (Queue *)malloc(sizeof(Queue));
    q->data = (int *)malloc(sizeof(int) * n);
    q->head = 0, q->tail = 0;
    q->size = n;
    return q;
}
```

### 2. empty

```c
int empty(Queue *q) {
    return q->head == q->tail;
}
```

### 3. front

```c
int front(Queue *q) {
    return q->data[q->head];
}
```

### 4. expand

```c
int expand(Queue *q) {
    int extr_size = q->size;
    int *p;
    while (expand) {
        p = (int *)malloc(sizeof(int) * (q->size) + extr_size);
        if (p) break;
        extr_size /= 2;
    }
    if (p == NULL) return 0;
    for (int i = q->head, j = 0; j < (q->tail - q->head + q->size) % q->size; j++) {
        p[j] = q->data[(i + j) % q->size];
    }
    q->tail = (q->tail - q->head + q->size) % q->size;
    q->size += extr_size;
    q->head = 0;
    free(q->data);
    q->data = p;
    return 1;
}
```

### 5. push

```c
int push(Queue *q, int val) {
    if (q == NULL) return 0;
    if (q->head == ((q->tail + 1) % q->size)) {
        if (!expand(q)) return 0;
        printf("expand successful!\n");
    }
    q->data[q->tail++] = val;
    q->tail = q->tail % q->size;
    return 1;
}
```

### 6. pop

```c
int pop(Queue *q) {
    if (q == NULL) return 0;
    if (empty(q)) return 0;
    q->head += 1;
    q->head %= q->size;
    return 1;
}
```

### 7. output

```c
void output(Queue *q) {
    printf("Queue(%d) = [", (q->tail - q->head + q->size) % q->size);
    for (int i = q->head, j = 0; j < ((q->tail - q->head) + q->size) % q->size; j++) {
        j != 0 && printf(", ");
        printf("%d", q->data[(i + j) % q->size]);
    }
    printf("]\n");
    return ;
}
```

### 8. clear

```c
void clear(Queue *q) {
    if (q == NULL) return ;
    free(q->data);
    free(q);
    return ;
}
```