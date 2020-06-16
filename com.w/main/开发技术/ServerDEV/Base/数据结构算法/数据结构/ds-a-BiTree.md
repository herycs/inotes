# BiTree

## 二叉树

- 无其他要求，保证每个结点只有左右两个子结点

## 平衡二叉树

> 一种二叉排序树,每个结点的左子树和右子树的高度差至多等于1

- 二叉树上有一个结点的平衡因子的绝对值大于1，则不平衡

- 对二叉树进行了约束，为了防止出现单边特别长的时候影响查询效率

策略：

增加平衡因子（二叉树上结点的左子树深度减去右子树深度的值称为**平衡因子**BF）

> 结点平衡因子：nodeBF = heightL - heightR

### 建立

距离插入结点最近的，且平衡因子的绝对值大于1的结点为根的子树，称为**最小不平衡子树**。

- 在插入值的时候同时评价平衡因子
    - 若平衡因子大于1，左旋
    - 若平衡因子小于-1，右旋

> 对于树的旋转，为了方便调整，结点应包含：数据，指向父结点指针，指向左右子结点指针，深度

### 删除

删除叶子子结点直接删除，检查调整

删除非叶子结点，需要在删除时进行调整，确保树的关系

## 遍历

### 递归

伪代码

```java
void f(Node root){
	if(root != null){
		out(root.data);//1
		f(root.leftNode);//2
		f(root.rightNode);//3
	}
}
//上述三句交换顺序即就是先中后序遍历
//先：123;中：213;后：231
```

### 非递归

策略：使用额外存储空间辅助遍历，例如：列表，栈。

伪代码

```java
void f(Node root){
    //预备存储空间
    Node p = root;//用于在树结点间切换
    ArrayList list = new ArrayList<Node>();//结果集合
    Stack stack = new Stack();//辅助遍历，
    
    //遍历
    while ( !stack.empty() && p != null){
        //[stack.empty() != true]左子树及其结点未遍历完，[p != null]未到根结点
        while(p != null){
            list.add(p.data);//[1]
            {//[2]
                stack.push(p);//将结点保存到栈中，用于遍历其右子树
	            p = p.left;//递归
            }
        }
        while(!stack.empty()){
            //栈非空，则存在右子树或结点
            Node tempNode = stack.pop();//出去栈顶元素
            //[3]
            p = tempNode.right;//将右结点交给p继续遍历
        }
    }
}
//先中续遍历
//先：123
//中：将1放到3的位置处，代码变为list.add(tempNode.data);顺序213
```

策略：

- 先将所有结点遍历到stack中，并将stack2中的标记为左子树
- 若辅助栈元素标记为右子树，则输出结点数据
- 数据栈stack非空，修改标记栈标记为右子树标记，取出数据栈元素对其右子树进行判断，继续继续判断

```java
//后续遍历伪代码
void f(Node root){
    //数据栈
    Stack<BinaryNode> stack1 = new Stack<>();
    //辅助栈
    Stack<Integer> stack2 = new Stack<>();
    int i = 1;//表示为右结点
    while(Node != null || !stack1.empty())
    {
        while (Node != null)
        {
            stack1.push(Node);
            stack2.push(0);
            Node = Node.left;
        }
        //右子树回到根结点，则直接输出
        while(!stack1.empty() && stack2.peek() == i)
        {
            stack2.pop();
            System.out.print(stack1.pop().element + "   ");
        }
        //数据栈非空
        if(!stack1.empty())
        {
            //修改标记
            stack2.pop();
            stack2.push(1);
            //进入当前结点为根的右子树
            Node = stack1.peek();
            Node = Node.right;
        }
    }
}
```

### 层次遍历

- 一层一层的遍历

策略：队列，先是跟结点入队列，然后是根结点出队列，同时此结点的左右结点入队列

```java
如：树123层分别为
		1
	    / \
	   2   3
	  / \  / \
	 4  5  6  7

访问过程队列变化：
1
2 3 //1 out 2,3 in
3 4 5 // 2 out, 4,5 in
4 5 6 7 // 3 out, 6,7 in
```

```java
void f(Node root){
	Queue queue = new Queue();
    if (root == null)return;
    //队列非空，遍历未完成
    while(!queue.isEmpty()){
        	//出队列，
			TreeNode temp  = queue.poll();
			System.out.print(temp.val + " ");
        	//左非空入队列
			if(temp.left != null)
				queue.offer(temp.left);
        	//右非空入队列
			if(temp.right != null)
				queue.offer(temp.right);
		}
}
```

