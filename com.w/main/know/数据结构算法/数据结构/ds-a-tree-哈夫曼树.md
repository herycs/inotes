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