# 线索二叉树

## 概念

- 结点有左子树，LChild指向左孩子，否则LChild指向某种遍历序列中的直接前驱结点

- 结点有右子树，RChild指向右孩子，否则RChild指向某种遍历序列的直接后继节点

- 定义：

    |  LChild  |  LTag  |  Data  |  RTag  |  RChild  | 

## 二叉树线索化

- 遍历遇到左孩子指针域为空时填入当前节点的前驱结点（上一个访问的节点，故需要保存上一次访问的节点）
- 遍历遇到右孩子指针域为空时填入当前节点的后继结点（下一个访问的节点，访问到下一个结点后回填）

## 线索二叉树基本操作

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

