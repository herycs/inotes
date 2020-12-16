# 二叉树

## 概念

- 以树为基础
- 每个结点的度不大于2
- 结点没棵树的位置是明确区分左右的，不能随意改变。

## 性质

- 二叉树的第i层上至多有2^i-1个结点（i>=1)
- 深度为k的二叉树至多有2^k-1个结点（k>=1)
- 对任意一棵二叉树T，若终端结点为n0，度为2的结点为n2，n0 = n2 + 1
- 具有n个结点的完全儿叉树的深度为[log2n]+1
- 对于具有n个结点的完全二叉树，如果按照满二叉树方式编号（1开始），任意序号为i的结点有
    - 如果 i = 1，则结点 i 为根，其无双亲结点，i > 1，则结点 i 的双亲结点序号为 [i/2]
    - 如果 2*i <= n，则结点 i 的左孩子结点序号为 2*i，否则，结点 i 无左孩子
    - 如果 2\*i +1<=n，则结点 i 的右孩子结点序号为 2\*i +1，否则，结点 i 无右孩子

## 存储

### 顺序存储结构

- 基本结构

    ```java
    public class SeqBiTree {
    
        private int MAX_SIZE;//最大可容纳节点数,用于申请空间
        private int DEFAULT_SIZE = 20;//默认可容纳节点数
        private Object[] seqBiTree;//存放所有节点，0号单元不用
        private int nodeMAX = 0;//最后一个节点下标
    
        public SeqBiTree() {
            this.MAX_SIZE = DEFAULT_SIZE;
            this.seqBiTree = new Object[this.MAX_SIZE+1];
        }
    
        public SeqBiTree(int MAX_SIZE) {
            this.MAX_SIZE = MAX_SIZE;
            this.seqBiTree = new Object[this.MAX_SIZE+1];
        }
    }
    ```

树的表现及对应到数据结构

- 树的表现

    ```java
    				1
    			  /    \
    			 2       3
    		    / \      / \
    		  4    5    6   7
    		 / \
    		8   9
    ```

对应到数据结构

> 为便于计算其父子关系等0号单元不使用

| 索引 | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 值   | /\   | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    |

应用

- 适用：完全二叉树 或 满二叉树，此种策略的存储密度高
- 对于非满二叉树可采用空节点将其补为满二叉树

### 链式存储结构

- 链式结构基础代码

    ```java
    public class LinkedBiTree {
    
        class BiNode{
            Object data;
            BiNode Lchild;
            BiNode Rchild;
    
            public BiNode(Object data) {
                this.data = data;
                this.Lchild = null;
                this.Rchild = null;
            }
    
            public Object getData() {
                return data;
            }
    
            public void setData(Object data) {
                this.data = data;
            }
    
            public BiNode getLchild() {
                return Lchild;
            }
    
            public void setLchild(BiNode lchild) {
                Lchild = lchild;
            }
    
            public BiNode getRchild() {
                return Rchild;
            }
    
            public void setRchild(BiNode rchild) {
                Rchild = rchild;
            }
        }
    
        //整个二叉树的根，后续的节点扩展以此为基础
        private BiNode root;
    
        public LinkedBiTree(Object data) {
            this.root = new BiNode(data);
            this.Lchild = null;
            this.Rchild = null;
        }
    }
    ```


## 遍历

**策略**

- 先上（根）后下（子）层次遍历
- 先左（子树）后右（子树）深度遍历
- 先右（子树）后左（子树）深度遍历

### **遍历顺序**

先序遍历

1. 访问根节点
2. 先序遍历左子树
3. 先序遍历右子树

中序遍历

1. 中序遍历左子树
2. 访问根节点
3. 中序遍历右子树

后续遍历

1. 1.后续遍历左子树
2. 2.后续遍历右子树
3. 3.访问根节点

示例：

- 树结构：

    ```XML
    			A
    		  /    \
    		B        C
    		 \     /   \
    		  D   E     F
    		 /         /
    		G         H
    ```

- 遍历结果：

    - 先序：A, B, D, G, C, E, F, H
    - 中序：B, G, D, A, E, C, H, F
    - 后序：G, D, B, F, H, F, C, A

    


### 递归实现

- 基本代码显示

    ```java
    public class LinkedBiTree {
    
        private BiNode root;
    
        public LinkedBiTree() {
            this.root.data = null;
            this.root.Lchild = null;
            this.root.Rchild = null;
        }
    
        public LinkedBiTree(Object data) {
            this.root = new BiNode();
            this.root.data = data;
            this.root.Rchild = null;
            this.root.Lchild = null;
        }
    	//先序遍历
        public void PreOrder(BiNode tRoot) {
            if (null != tRoot) {
                visit(tRoot.data);
                PreOrder(tRoot.Lchild);
                PreOrder(tRoot.Rchild);
            }
        }
    	//中序遍历
        public void InOrder(BiNode tRoot) {
            if (null != tRoot) {
                PreOrder(tRoot.Lchild);
                visit(tRoot.data);
                PreOrder(tRoot.Rchild);
            }
        }
    	//后序遍历
        public void PostOrder(BiNode tRoot) {
            if (null != tRoot) {
                PreOrder(tRoot.Lchild);
                PreOrder(tRoot.Rchild);
                visit(tRoot.data);
            }
        }
    
        public void visit(Object data) {
            System.out.print(" " + data);
        }
    
        public BiNode getRoot() {
            return root;
        }
        public void setRoot(BiNode root) {
            this.root = root;
        }
    	//基本结点
        class BiNode {
            Object data;
            BiNode Lchild;
            BiNode Rchild;
    
            public BiNode() {
            }
    
            public BiNode(Object data) {
                this.data = data;
                this.Lchild = null;
                this.Rchild = null;
            }
    }
    ```

### 非递归实现

- 代码示例C语言

    ```c
    //先序遍历
    void PreOrder(BiTree root){
        SeqStack *S;
        BiTree p;
        InitStack(S);
        p = root;
        while(p != NULL || !IsEmpty(s)){
            while (p != NULL){
                Visit(p->data);
                Push(S, p);
                p=p->LChild;
            }
            if (!IsEmpty(S)){
                Pop(S, &p);
                p=p->RChild;
            }
        }
    }
    //中序遍历
    void InOrder(BiTree root){
        SeqStack *S;
        BiTree p;
        InitStack(S);
        p=root;
        while (p != NULL || !IsEmpty(S)){
            while (p != NULL){
                Push(S, p);
                p=p->LChild;
            }
            if (!IsEmpty(S)){
            Pop(S, &p);
            Visit(p->data);
            p=p->RChild;
            }
        }
    }
    
    //中序遍历
    void InOrder2(BiTree root){
        SeqStack *s;
        InitStack(S);
        p=root;
        while (p != NULL || !IsEmpty(S)){
            if (p != NULL){
                Push(S, p);
                p=p->LChild;
            }else {
                Pop(S, &p);
                Visit(p->data);
                p=p->RChild;
            }
        }
    }
    
    //后序遍历
    void PostOrder(BiTree root){
        SeqStack *S;
        BiTree p, q;
        InitStack(S);
        p=root;
        q=NULL;
        while (p != NULL || !IsEmpty(S)){
            while (p != NULL){
                Push(S, p);
                p=p->LChild;
            }
            if (!IsEmpty(S)){
                Top(S, &p);
                if (p->RChild == NULL || p->RChild == q){
                    Pop(S, &p);
                    Visit(p->data);
                    q=p;
                    p=NULL;
                }else{
                    p=p->RChild;
                }
            }
        }
    }
    ```


### 遍历应用

##### 统计结点个数

```c
//统计结点个数
void PreOrder(BiTree root){
	if (root){
		Count++;
		PreOrder(root->LChild);
		PreOrder(root->RChild);
	}
}
```



##### 输出叶子结点

```c
//输出叶结点
void InOrder(BiTree root){
	if (root){
		inPreOrder(root->LChild);
		if (root->LChild==NULL && root->Rchild==NULL){
			print(root->data);
			InOrder(root->EChild);
		}
	}
}
```



##### 统计叶子结点数目

```c
//统计叶子结点数目
int leaf(Bitree root){
	int nl, nr;
	if (root==NULL)	return 0;
	if ((root->LChild==NULL) && (root->RChild==NULL)){
		nl=leaf(root->LChild);
		nr=leaf(root->RChild);
		return (nl+nr);
	}
}
```

##### 求树高

```c
//数高度，全局变量 
int depth;

//二叉树高度
void TreeDepth(BiTree root, int h){
	if (root){
		if (h>depth)	depth=h;
		TreeDepth(root->LChild, h+1);
		TreeDepth(root->RChild, h+1); 
	}
}

//二叉树高度
int PostTreeDepth(BiTree root){
	int hl, hr, h;
	if (root==NULL)	return 0;
	else {
		hl=PostTreeDepth(root->LChild);
		hr=PostTreeDepth(root->RChild);
		h=(hl>hr ? hl : hr) + 1;
		return h;
	}
} 
```

##### 求节点的双亲

```c
//结点双亲
BiTree parent(BiTree root, BiTree current){
	BiTree * p;
	if (root==NULL)	return NULL;
	if (root->LChild==current || root->RChild==current){
		return root;
	}
	p=parent(root->LChild, current);
	if (p!=NULL)	return p;
	else return(parent(root->RChild, current));
}
```



##### 二叉树的相似性的判定

```c
//二叉树相似判定
int like(BiTree t1, BiTree t2){
	int like1, link2;
	if (t1==NULL && t2==NULL)	return 1;
	else if(t1==NULL || t2 == NULL)	return 0;
	else {
		like1=like(t1->LChild, t2->LChild);
		like2=like(t1->RChild, t2->Rchild);
		return (like1 && like2);
	}
}
```



##### 树状打印二叉树

```c
//树状打印二叉树
void PrintTree(BiTree root, int h){
	if (root==NULL)	return;
	PrintTree(root->RChild, h+1);
	for (int i=0; i<h; i++){
		printf(" ");
	} 
	printf("%c\n", root->data);
	PrintTree(root->LChild, h+1);
} 
```

## 建立二叉链表存储的二叉树

#### 遍历序列=>二叉树

> 先序先访问根，再左子树，再右子树，故先序=>根（第一个结点）
>
> 中序先访问左子树，再根，再右子树，故中序=>左子树（根的左侧）
>
> 后序先访问左子树，再右子树，再根，故后续=>根（最后一个结点）

##### 先+中 => 二叉树

> 先序第一个结点=>根
>
> 由根拆分中序=>左子树+右子树
>
> 由左子树结点=>先序=根节点+左子树先序+右子树先序

##### 中+后=> 二叉树

> 后序最后一个结点=>根
>
> 由根=>分割中序=>中序=左子树中序+根+右子树中序
>
> 由左子树结点个数分割后序=>后序=左子树后序+右子树后序+根

