# Stack

栈是一种先进后出的数据结构

```c
typedef struct Stack {
    int *data;
    int top, size;
} Stack;
```

## 结构操作

栈的特殊操作是：pop, top, push

### 1. init

```c
typedef struct Stack {
    int *data;
    int top, size;
} Stack;
```

### 2. empty J（栈的判空操作）

``` c
int empty(Stack *s) {
    return s->top == -1;
}
```

### 3. top (返回栈顶元素)

```c
int empty(Stack *s) {
    return s->top == -1;
}
```

### 4. expand

```c
int expand(Stack *s) {
    int *old_data = s->data;
    s->data = (int *)realloc(s->data, sizeof(int) * s->size * 2);
    s->size = s->size * 2;
    if (s->data) {
        s->data = old_data;
        return 0;
    }
    free(old_data);
    return 1;
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

```c
int push(Stack *s, int val) {
    if (s == NULL) return 0;
    if (s->top + 1 == s->size) {
        if(!expand(s)) return 0;
        printf("expand successful! size = %d\n", s->size);
    }
    s->data[++(s->top)] = val;
    return 1;
}
```



### 7.pop

```c
int pop(Stack *s) {
    if (s == NULL) return 0;
    if (empty(s)) return 0;
    s->top--;
    return 1;
}
```



### 8. clear

```c
void clear(Stack *s) {
    if (s == NULL) return ;
    free(s->data);
    free(s);
    return ;
}
```