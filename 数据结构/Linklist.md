# Linklist

## 结构定义

链表的定义有节点的定义和链表的定义

### 1. 节点定义

```c
typedef struct ListNode {
    int data;
    struct ListNode *next;
} ListNode;
```

### 2. 链表定义

```c
typedef struct List {
    ListNode head;
    int length;
} List;
```

## 结构操作

### 1. init

+ getNewNode

  ```c
  ListNode *getNewNode(int val) {
      ListNode *p = (ListNode *)malloc(sizeof(ListNode));
      p->data = val;
      p->next = NULL;
      return p;
  }
  ```

+ getLinkList

  ```c
  List *getLinkList() {
      List *l = (List *)malloc(sizeof(List));
      l->head.next = NULL;
      l->length = 0;
      return l;
  }
  ```

### 2. insert

```c
int insert(List *l, int ind, int val) {
    if (l == NULL) return 0;
    if (ind < 0 || ind > l->length) return 0;
    ListNode *p = &(l->head), *node = getNewNode(val);
    while (ind--) p = p->next;
    node->next = p->next;
    p->next = node;
    l->length++;
    return 1;
}
```

### 3. erase

``` c
int earse(List *l, int ind) {
    if (l == NULL) return 0;
    if (ind < 0 || ind >= l->length) return 0;
    ListNode *p = &(l->head);
    while (ind--) p = p->next;
    ListNode *q = p->next;
    p->next = q->next;
    free(q);
    l->length--;
    return 1;
}
```

### 4. output

``` c
void output(List *l) {
    printf("List(%d) = [", l->length);
    for (ListNode *p = l->head.next; p; p = p->next) {
        printf("%d->", p->data);
    }
    printf("NULL]\n");
    return ;
}
```

### 5. clear

+ clear_LinkNode

  ```c
  void clear_ListNode(ListNode *node) {
      if (node == NULL) return ;
      free(node);
      return ;
  }
  ```

+ clear_List

  ```c
  void clear_List(List *l) {
      if (l == NULL) return ;
      ListNode *p = &(l->head), *q;
      while (p) {
          q = p->next;
          clear_ListNode(p);
          p = q;
      }
      free(l);
      return ;
  }
  ```

### 6. rev

``` c
void rev(List *l) {
    if (l == NULL) return ;
    ListNode *p = l->head.next, *q;
    l->head.next = NULL;
    while (p) {
        q = p->next;
        p->next = l->head.next;
        l->head.next = p;
        p = q;
    }
}
```