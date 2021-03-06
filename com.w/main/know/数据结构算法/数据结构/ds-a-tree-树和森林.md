# 树和森林

## 树的存储

### 双亲表示法

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

### 孩子表示法

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

### 孩子兄弟表示法

定义

```c
//孩子兄弟表示法
typedef struct CSNode{
	DataType data;
	Struct CSNode * FirstChild;
	Struct CSNode * NextSibling;
}CSNode, * CSTree;
```



## 树---森林---二叉树

相关理解

- 对于以孩子兄弟二叉链表存储的树按二叉树的二叉链表来解释即成为一颗二叉树
- 对于二叉链表存储的二叉树桉树的孩子兄弟二叉链表来解释即成为一棵树

### 树转二叉树

操作

- 加线：所有兄弟之间加线
- 删线：每个接地单只保留与第一个孩子结点之间的连线
- 旋转调整：以树的根节点为轴心，旋转一定角度，使得其左右子树分明

### 森林转二叉树

操作

- 转换：森林中的每一棵树均转换成相应的二叉树
- 加线：相邻各子树根节点之间加线
- 旋转调整：以第一棵树的根节点为轴心，将整颗树顺时针旋转一定角度，使其结构清晰左右分明，依次把后一棵二叉树根节点调整座位前一棵二叉树根节点的右孩子位置

### 二叉树转森林

操作

- 加线：某结点是双亲的左孩子，则把该结点的右孩子，右孩子的右孩子...与该结点双亲结点连上线
- 删线：删掉原二叉树中所有双亲结点与右孩子间的连线
- 旋转调整：旋转，整理树使其结构清晰

## 数和森林的遍历

### 树的遍历

> 树非空

先根遍历

1. 访问根节点
2. 左到右，依次先根遍历根节点的每一课子树

后根遍历

1. 左到右，依次后根遍历根节点的每一课子树
2. 访问根节点

### 森林的遍历

先序遍历

1. 访问森林中的第一棵树的根结点
2. 先序遍历第一棵树中根节点的子树森林
3. 先序遍历出去第一棵树之后剩余的树构成的森林

中序遍历

1. 中序遍历森林中第一棵树的根结点
2. 访问第一棵树的根节点
3. 中序遍历出去第一棵树之后剩余的树构成的森林
