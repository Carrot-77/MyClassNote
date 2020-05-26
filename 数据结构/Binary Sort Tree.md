# Binary Sort Tree

## 二叉排序树：

或者是一棵空树，或者是具有下列性质的二叉树：

1. 若它的左子树不空，则左子树上所有结点的值均小于它的根结点的值；

2. 若它的右子树不空，则右子树上所有结点的值均大于它的根结点的值；

3. 它的左、右子树也分别为二叉排序树。

### 定义

```c
typedef struct Node {
	int key;
	struct Node *lchild, *rchild;
}Node;
```



### 创建新节点

```c
Node *getNewNode(int key) {
	Node *p = (Node *)malloc(sizeof(Node));
	p->key = key;
	p->lchild = p->rchild = NULL;
	return p;
}
```



### 寻找前驱

```c
Node *predeccessor(Node *root) {
	Node *temp = root->lchild;
	while (temp->rchild) temp = temp->rchild;
	return temp;
}
```



### 插入

```c
Node *insert(Node *root, int key) {
	if (root == NULL) return getNewNode(key);
	if (root->key == key) return root;
	if (root->key > key) root->lchild = insert(root->lchild, key);
	else root->rchild = insert(root->rchild, key);
	return root;
}
```



### 删除

```c
Node *erase(Node *root, int key) {
	if (root == NULL) return root;
	if (root->key > key) {
		root->lchild = erase(root->lchild, key);
	} else if (root->key < key){
		root->rchild = erase(root->rchild, key);
	} else {
		if (root->lchild == NULL || root->rchild == NULL) {
			Node *temp = root->lchild ? root->lchild : root->rchild;
			free(root);
			return temp;
		} else {
			Node *temp = predeccessor(root);
			root->key = temp->key;
			root->lchild = erase(root->lchild, temp->key);
		}
	}
	return root;
}
```



### 清空

```c
void clear(Node *root) {
	if (root == NULL) return;
	clear(root->lchild);
	clear(root->rchild);
	free(root);
	return;
}
```



### 输出

```c
void output(Node *root) {
	if (root == NULL) return;
	output(root->lchild);
	printf("%d ", root->key);
	output(root->rchild);
	return;
}
```



## 平衡二叉排序树

平衡二叉树定义
平衡二叉树（Balanced Binary Tree）

它是一棵空树，或者是具有以下性质：

* 它的左右两个子树的深度差的绝对值不超过1；

* 它的左右两个子树也分别是平衡二叉树。

  将二叉树节点的左子树的深度减去它的右子树的深度称为平衡因子BF,则平衡二叉树上所有节点的平衡因子只可能是-1、0和1

旋转：

* LL:**插入位置为左子树的左子树，以左子树为轴心，进行单次向右旋转**

* RR:**插入位置为右子树的右子树，右子树为轴心，进行单次向左旋转**
* LR:**插入位置为左子树的右子树，要进行两次旋转，先向小左旋，再向大右旋**
* RL：**插入位置为右子树的左子树，要进行两次旋转，先向小右旋，再向大左旋**



### 左旋

```c
Node *temp = root->rchild;
root->rchild = temp->lchild;
temp->lchild = root;
```



### 右旋

```c
Node *temp = root->lchild;
root->lchild = temp->rchild;
temp->rchild = root;
```



```cpp
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define H(root) (root)->h
#define K(root) (root)->key
#define L(root) (root)->lchild
#define R(root) (root)->rchild

typedef struct Node {
	int key, h;
	struct Node *lchild, *rchild;
} Node;

Node __NIL;
#define NIL (&__NIL)
__attribute__((constructor))
void init_NIL() {
	NIL->key = NIL->h = 0;
	NIL->lchild = NIL->rchild = NIL;
	return;
}

Node *getNewNode(int key) {
	Node *p = (Node *)malloc(sizeof(Node));
	p->key = key;
	p->h = 1;
	p->lchild = p->rchild = NIL;
	return p;
}

void update_height(Node *root) {
	root->h = (H(L(root)) > H(R(root))) ? H(L(root)) : H(R(root)) + 1;
	return;
}

Node *left_rotate(Node *root) {
	Node *temp = root->rchild;
	root->rchild = temp->lchild;
	temp->lchild = root;
	update_height(root);
	update_height(temp);
	return temp;
}

Node *right_rotate(Node *root) {
	Node *temp = root->lchild;
	root->lchild = temp->rchild;
	temp->rchild = root;
	update_height(root);
	update_height(temp);
	return temp;

}

Node *maintain(Node *root) {
	if (abs(H(L(root)) - H(R(root))) <= 1) return root;
	if (H(L(root)) > H(R(root))) {
		if (H(R(L(root))) > H(L(L(root)))) {
			root->lchild = left_rotate(root->lchild);
		}
		root = right_rotate(root);
	} else {
		if (H(L(R(root))) > H(R(R(root)))) {
			root->rchild = left_rotate(root->rchild);
		}
		root = left_rotate(root);
	}
	return root;
}

Node *insert(Node *root, int key) {
	if (root == NIL) return getNewNode(key);
	if (root->key == key) return root;
	if (root->key > key) root->lchild = insert(root->lchild, key);
	else root->rchild = insert(root->rchild, key);
	update_height(root);
	return maintain(root);
}

Node *predecessor(Node *root) {
	Node *temp = root->lchild;
	while (temp->rchild != NIL) temp = temp->rchild;
	return temp;
}

Node *erase(Node *root, int key) {
	if (root == NIL) return root;
	if (root->key > key) {
		root->lchild = erase(root->lchild, key);
	} else if (root->key < key) {
		root->rchild = erase(root->rchild, key);
	} else {
		if (root->lchild == NIL || root->rchild == NIL) {
			Node *temp = root->lchild == NIL ? root->rchild : root->lchild;
			free(root);
			return temp;
		} else {
			Node *temp = predecessor(root);
			root->key = temp->key;
			root->lchild = erase(root->lchild, temp->key);
		}
	}
	return maintain(root);
}

void clear(Node *root) {
	if (root == NIL) return;
	clear(root->lchild);
	clear(root->rchild);
	free(root);
	return;
}

void output(Node *root) {
	if (root == NIL) return;
	printf("(%d,%d,%d)\n", K(root), K(L(root)), K(R(root)));
	output(root->lchild);
	output(root->rchild);
	return;
}

int main() {
	int maxop = 30;
	Node *root = NIL;
	for (int i = 0; i < maxop; i++) {
		int val = rand() % 100;
		root = insert(root, val);
	}
	output(root);
    int val;
    while (~scanf("%d", &val)) {
        root = erase(root, val);
        printf("erase %d from AVL tree\n", val);
        output(root);
    }
	return 0;
}
```

