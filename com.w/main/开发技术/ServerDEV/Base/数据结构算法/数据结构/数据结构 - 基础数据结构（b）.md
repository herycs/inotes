#  Java  - 数据结构 - 基础数据结构（2）

# 树

## 基本概念

- n（n >= 0）个节点的有限集合，n=0 为空树

- n>0 时，集合满足以下条件

    - 1.有且仅有一个称为根的节点

        此节点无前驱，有0~多个后继

    - 2.除根节点外的n-1个结点可划分成m(m>=0)个互不相交的有限集（T1,T2,T3，...，Tm）

        每个节点有为一个树，称为根的子树，每颗子树的根节点有且仅有一个直接前驱，其前驱为根结点，同时可有0~多个直接后继节点。

## 树的表示法

- 图形表示法
- 嵌套集合表示法
- 广义表表示法
- 凹入表示法

## 树的基本术语

- 结点：包含一个数据元素及若干指向其子树的分支。
- 度：
    - 节点的度：节点拥有子树的个数
    - 树的度：树中所有节点的度的最大值。
- 叶子结点：
    - 度为0的节点
- 内部结点：度不为0，也称分支节点或非终结节点
- 孩子结点：结点的子树的跟称为该结点
- 双亲结点：结点是其子树的根称为该节点的孩子结点
- 双亲结点：结点是其子树的根的双亲，结点是其孩子的双亲
- 兄弟结点：同一双亲的孩子结点之间互称为兄弟结点
- 堂兄弟：双亲是兄弟或堂兄弟的节点间互称为堂兄弟节点
- 祖先节点：节点的祖先节点是指该结点的子树中的所有结点
- 结点的层次：根为第一层，根的孩子为第二层，...
- 树的层次：结点的层次的最大值，又称树的高度
- 前辈：层号比结点层号小的结点
- 后辈：层号比该结点层号大的结点
- 森林：m(m>=0)棵互不相交的树的集合
- 有序树：树中结点的各子树从左到右是有特定次序的称为有序树
- 无序树：树中结点的各子树从左到右是无特定次序的称为有序树

## 二叉树

### 概念

- 以树为基础
- 每个结点的度不大于2
- 结点没棵树的位置是明确区分左右的，不能随意改变。

### 性质

- 二叉树的第i层上至多有2^i-1个结点（i>=1)
- 深度为k的二叉树至多有2^k-1个结点（k>=1)
- 对任意一棵二叉树T，若终端结点为n0，度为2的结点为n2，n0 = n2 + 1
- 具有n个结点的完全儿叉树的深度为[log2n]+1
- 对于具有n个结点的完全二叉树，如果按照满二叉树方式编号（1开始），任意序号为i的结点有
    - 如果 i = 1，则结点 i 为根，其无双亲结点，i > 1，则结点 i 的双亲结点序号为 [i/2]
    - 如果 2*i <= n，则结点 i 的左孩子结点序号为 2*i，否则，结点 i 无左孩子
    - 如果 2\*i +1<=n，则结点 i 的右孩子结点序号为 2\*i +1，否则，结点 i 无右孩子

### 存储

#### 顺序存储结构

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

#### 链式存储结构

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

    

### 遍历

**策略**

- 先上（根）后下（子）层次遍历
- 先左（子树）后右（子树）深度遍历
- 先右（子树）后左（子树）深度遍历

**遍历顺序**

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

    


#### 递归实现

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

#### 非递归实现

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

    

#### 遍历应用

##### 统计结点个数

- C写法

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

- C写法

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

- C写法

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

- C写法

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

- C写法

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

- C写法

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

- C写法

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

##### 建立二叉链表存储的二叉树

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

## 线索二叉树

### 概念

- 结点有左子树，LChild指向左孩子，否则LChild指向某种遍历序列中的直接前驱结点

- 结点有右子树，RChild指向右孩子，否则RChild指向某种遍历序列的直接后继节点

- 定义：

    |  LChild  |  LTag  |  Data  |  RTag  |  RChild  | 

### 二叉树线索化

- 遍历遇到左孩子指针域为空时填入当前节点的前驱结点（上一个访问的节点，故需要保存上一次访问的节点）
- 遍历遇到右孩子指针域为空时填入当前节点的后继结点（下一个访问的节点，访问到下一个结点后回填）

### 线索二叉树基本操作

中序线索化

```c
//中序线索化
void Intread(BiTree root){
	if (root!=NULL){
		Inthread(root->LChild);
		if (root->LChild==NULL){
			root->LChild=pre;
			root->LTag=1;
		}
		if (pre!=NULL && pre->RChild==NULL){
			pre->RChild=root;
			pre->RTag=1;
		}
		pre=root;
		Inthread(root->RChild);
	}
} 
```

中序线索树找前驱

```c
//中序找结点前驱
BiThrTree InPre(BiThrTree p){
	if (p->Ltag=1)	pre=p->LChild;
	else{
		for (q=q->LChild; q->Rtag==0; q=q-RChild);
		pre=q;
	}
	return (pre);
}
```

中序找结点后继

```c
//中序找结点后继
BiThrTree InNext(BiTree p){
	if (p->Rtag == 1)	next=p->RChild;
	else{
		for (q=p->Rchild; q->Ltag==0; q=q->LChild);
		next=q;
	}
	return (next);
}
```

中序找遍历的第一个结点

```c
//中序线索树中求遍历的第一个结点 
BiThrTree InFirst(BiThrTree bt){
	BiThrTree p=bt;
	if (p!=NULL)	return (NULL);
	while (p->Ltag==0)	p=p->LChild;
	return p;
}
```

遍历中序二叉线索树

```c
//遍历中序线索树
void TinOrder(BiThrTree root){
	BiThrTree p;
	p=InFirst(root);
	while (p!=NULL){
	Visit(p->data);
	p=InNext(p); 
} 
```



## 树和森林

### 树的存储

#### 双亲表示法

定义

```c
#define MAX 100
typedef struct TNode{
	DataType data;
	int parent;
}TNode;

//树的定义 
typedef struct{
	TNode tree[MAX];
	int root;
	int num;
}PTree;
```

存储

| Data     | Parent   |
| -------- | -------- |
| 结点数据 | 双亲索引 |

- 每个节点只有一个双亲

缺点：查找某个孩子时需要在整个数组中寻找

#### 孩子表示法

定义

```c
typedef struct ChildNode{
	int Child;
	Struct ChildNode * next;
}ChildNode;
//顺序表结构定义  
typedef struct {
	DataType * FirstChild;
}DataNode;
//树定义
typedef struct{
	DataNode nodes[MAX];
	int root;
	int num;
}CTree;
```

结构

| Data | firstChild                                             |
| ---- | ------------------------------------------------------ |
| 数据 | 无孩子=>NULL，有孩子，此处指向孩子组成的链表（头结点） |

缺点

- 便于找到结点的孩子，但不利于找到结点的双亲

改进

- 在每个结点中设立Parent域，形成带双亲的孩子的表示法

- 结构

    | Data | Parent       | firstChild                                           |
    | ---- | ------------ | ---------------------------------------------------- |
    | 数据 | 双亲索引地址 | 无孩子=>NULL，有孩子，此处指向孩子组成的链表（头结点 |

#### 孩子兄弟表示法

定义

```c
//孩子兄弟表示法
typedef struct CSNode{
	DataType data;
	Struct CSNode * FirstChild;
	Struct CSNode * NextSibling;
}CSNode, * CSTree;
```



### 树---森林---二叉树

相关理解

- 对于以孩子兄弟二叉链表存储的树按二叉树的二叉链表来解释即成为一颗二叉树
- 对于二叉链表存储的二叉树桉树的孩子兄弟二叉链表来解释即成为一棵树

#### 树转二叉树

操作

- 加线：所有兄弟之间加线
- 删线：每个接地单只保留与第一个孩子结点之间的连线
- 旋转调整：以树的根节点为轴心，旋转一定角度，使得其左右子树分明



#### 森林转二叉树

操作

- 转换：森林中的每一棵树均转换成相应的二叉树
- 加线：相邻各子树根节点之间加线
- 旋转调整：以第一棵树的根节点为轴心，将整颗树顺时针旋转一定角度，使其结构清晰左右分明，依次把后一棵二叉树根节点调整座位前一棵二叉树根节点的右孩子位置

#### 二叉树转森林

操作

- 加线：某结点是双亲的左孩子，则把该结点的右孩子，右孩子的右孩子...与该结点双亲结点连上线
- 删线：删掉原二叉树中所有双亲结点与右孩子间的连线
- 旋转调整：旋转，整理树使其结构清晰

### 数和森林的遍历

#### 树的遍历

> 树非空

先根遍历

1. 访问根节点
2. 左到右，依次先根遍历根节点的每一课子树

后根遍历

1. 左到右，依次后根遍历根节点的每一课子树
2. 访问根节点

#### 森林的遍历

先序遍历

1. 访问森林中的第一棵树的根结点
2. 先序遍历第一棵树中根节点的子树森林
3. 先序遍历出去第一棵树之后剩余的树构成的森林

中序遍历

1. 中序遍历森林中第一棵树的根结点
2. 访问第一棵树的根节点
3. 中序遍历出去第一棵树之后剩余的树构成的森林

## 哈夫曼树

### 基本概念

- 路径：从树的一个结点到另一个结点之间的分支序列构成两个结点见的路径

- 路径长度：路径上分支的条数

- 树的路径长度：从树根到每个结点的路径长度之和

- 结点的权：节点所带有的数值

- 带权路径长度：结点到树根的路径长度与结点的权的乘积

- 树的带权路径长度：树中所有叶子结点的带权路径长度之和

    WPL = Wk x Lk（1<= k <=n）

- 最优二叉树：叶子个数确定，叶子权值确定，带权长度WPL值最小的二叉树

### 建立

1.初始化

2.最小树构造新树

3.删除与插入

### 算法实现

存储结构

| weight | Parent             | Lchild           | Rchild           |
| ------ | ------------------ | ---------------- | ---------------- |
| 权值   | 双亲结点数组中下标 | 左孩子数组中下标 | 右孩子数组中下标 |

定义

```c
#define N 30
#define M 2*N-1

typedef struct {
	int weight;
	int parent, Lchild, Rchild;
}HTNode, HuffmanTree[M+1];	//0号单元不用
```

算法

- C实现

    ```c
    //建立哈夫曼树
    void CrtHuffmanTree(HuffmanTree ht, int w[], int n){
    	m=2*n-1;
    	for (i=1; i<=n; i++){
    		ht[i] = {w[i], 0, 0, 0}
    	}
    	for (i=n+1; i<=m; i++){
    		ht[i] = {0, 0, 0, 0}
    	}
    	for (i=n+1; i<=m; i++){
    		select(ht, i-1, &s1, &s2);
    		ht[i].weight = ht[s1].weight + ht[s2].weight;
    		ht[i].Lchild = s1;
    		ht[i].Rchild = s2;
    		ht[i].parent = i;
    		ht[s2].parent = i;
    	}
    } 
    ```

    

### 应用

哈夫曼编码

概念

- 前缀编码：同一字符集中任何一个字符的编码都不是另一个字符编码的前缀（最左子串）
- 哈夫曼编码：最优二进制前缀编码

算法

1. 构造哈夫曼树
2. 哈夫曼树上求个叶子结点的编码
    1. 方向（叶子结点--->根节点），经过一个分支得到一个编码，左分支得到0，右分支得到1
    2. 上述步骤得到的是逆串，需要倒置
    3. 到达根节点（此结点编码完成）

# 图

## 概念

- 图：定点集（V）+弧集（R）
- 有向图：定点集和弧集构成的图
- 无向图：顶点集和边集构成的图
- 有向网和无向网：（有向图）弧或（无向图）边带权
- 子图：此图（子图）顶点集和弧集均属于另一个图的子集
- 完全图：n个结点，n*（n-1)/2条边的无向图
- 有向完全图：n个结点，n(n-1)条弧的有向图
- 稀疏图：n个结点，e条边，（e < nlogn)
- 邻接点：无向图中一条边的两点
- 顶点的度：无向图中与顶点v关联的边的数目为v的度，记为TD（v)
- 简单路径：顶点不重复
- 回路：首位相接的路径
- 简单回路：除了首位外其他点不重复
- 连通图：无向图中，任意两定点之间都有路径相通
- 强连通图：有向图中，任意顶点间都存在有向路径
- 生成树：连通图中全部顶点的极小连通子图

## 存储

### 邻接表

顶点结构

| vexdata | head |
| ------- | ---- |
|         |      |

图的边结构

| adjvex | next |
| ------ | ---- |
|        |      |

网的边结构

| adjvex | weight | next |
| ------ | ------ | ---- |
|        |        |      |

### 邻接矩阵

二维数组，行列均为结点

### 十字链表

边表结点结构

| tailvex        | headvex      | hnextarc               | tnextarc           | info         |
| -------------- | ------------ | ---------------------- | ------------------ | ------------ |
| 弧尾的顶点序号 | 弧头顶点序号 | 同一弧头顶点的下一条弧 | 同一弧尾的下一条弧 | 弧权值等信息 |

顶点表结点结构

| vexdata          | head                       | tail                       |
| ---------------- | -------------------------- | -------------------------- |
| 顶点相关数据信息 | 以该顶点为弧头的边表头指针 | 以该顶点为弧尾的边表头指针 |

### 多重链表

边表结点结构

| mark | jvex | inext | jvex | jnext | weight |
| ---- | ---- | ----- | ---- | ----- | ------ |
|      |      |       |      |       |        |

顶点表结点结构

| vexdata | head |
| ------- | ---- |
|         |      |

## 遍历

### 深度优先搜索

基本步骤

1. 从图中某个顶点v0出发，首先访问v0;  
2. 访问结点v0的第一个邻接点，以这个邻接点vt作为一个新节点，访问vt所有邻接点。直到以vt出发的所有节点都被访问到，回溯到v0的下一个未被访问过的邻接点，以这个邻结点为新节点，重复上述步骤。直到图中所有与v0相通的所有节点都被访问到。
3. 若此时图中仍有未被访问的结点，则另选图中的一个未被访问的顶点作为起始点。重复深度优先搜索过程，直到图中的所有节点均被访问过。

### 广度优先搜索

基本步骤

1. 从图中某个顶点v0出发，首先访问v0;
2. 依次访问v0的各个未被访问的邻接点；
3. 依次从上述邻接点出发，访问它们的各个未被访问的邻接点。
4. 若此时图中仍有未被访问的结点，则另选图中的一个未被访问的顶点作为起始点。重复广度优先搜索过程，直到图中的所有节点均被访问过。

- 一般可借助队列保存已经访问过的顶点

### 广搜深度搜索比较

- 本质：访问顺序不同
- 访问效率：时间复杂度相同
- 其它：
    - 对于连通图，只需要访问一次，对于非连通图，需访问多次（这个次数就是连通分量）

## 应用

### 最小生成树

#### 	Prim算法

思想：从连通网络的一点出发，选择与它关联的具有最小权值的边，将其顶点加到生成树的顶点集合中，之后选择顶点一边在U中，另一边不在U中的边中选取权值最小的边。

#### 	Kruskal算法

思想：首先我们把所有的边按照权值先从小到大排列，接着按照顺序选取每条边，如果这条边的两个端点不属于同一集合，那么就将它们合并，直到所有的点都属于同一个集合为止。（可使用并查集处理是否在同一个集合）

并查集(Union Find)是一种用于管理分组的数据结构。

- 具备两个操作：
    - 查询元素a和元素b是否为同一组 
    - 将元素a和b合并为同一组

> 其他
>
> ### 拓扑排序
>
> ### 关键路径
>
> - 具有最长路径长度的路径
>
> ###  最短路径
>
> #### 	单源最短路径
>
> #### 	每对顶点之间的最短路径