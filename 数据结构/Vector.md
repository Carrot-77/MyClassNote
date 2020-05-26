# Vector

顺序表：一种类似与数组的存储结构

> 数据结构 = 结构定义 + 结构操作

## 结构定义

```c
typedef struct Vector {
    int *data; //数据域
    int size, length;
} Vector;
```

## 结构操作

### init

```c
Vector *init(int n) {
    Vector *vec = (Vector *)malloc(sizeof(Vector));
    vec->data = (int *)malloc(sizeof(int) * n);
    vec->size = n;
    vec->length = 0;
    return vec;
}
```

### expand

当顺序表的data数据域存满时需要进行扩充操作

> 注意： p = (int *)realloc(vec->data, sizeof(int) * (vec->size + extr_size));
>
> 若使用vec->data = = (int *)realloc(vec->data, sizeof(int) * (vec->size + extr_size)); 可能会造成数据的丢失（realloc申请不成功返回NULL

```c
int expand(Vector *vec) {
    int extr_size = vec->size;
    int *p;
    while (extr_size) {
        p = (int *)realloc(vec->data, sizeof(int) * (vec->size + extr_size));
        if (p) break;
        extr_size /= 2;
    }
    if (p == NULL) return 0;
    vec->data = p;
    vec->size += extr_size;
    return 1;
}
```

### insert

```c
int insert(Vector *vec, int ind, int val) {
    if (vec == NULL) return 0;
    if (ind < 0 || ind > vec->length) return 0;
    if (vec->length == vec->size) {
        if (!expand(vec)) return 0;
        printf("expand successfully ! size = %d\n", vec->size);
    }
    for (int i = vec->length - 1; i >= ind; i--) {
        vec->data[i + 1] = vec->data[i];
    }
    vec->data[ind] = val;
    vec->length++;
    return 1;
}
```

### erase

```c
int erase(Vector *vec, int ind) {
    if (vec == NULL) return 0;
    if (ind < 0 || ind >= vec->length) return 0;
    for (int i = ind + 1; i < vec->length; i++) {
        vec->data[i - 1] = vec->data[i];
    }
    vec->length--;
    return 1;
}
```

### output

```c
void output(Vector *vec) {
    printf("Vector(%d) = [", vec->length);
    for (int i = 0; i < vec->length; i++) {
        i && printf(", ");
        printf("%d", vec->data[i]);
    }
    printf("]\n");
    return ;
}
```

### clear

```c
void clear(Vector *vec) {
    if (vec == NULL)  return;
    free(vec->data);
    free(vec);
    return ;
}
```