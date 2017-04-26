---
layout: post
title:  二叉树遍历
date:   2016-10-18 1:29:08 +0800
tag: 数据结构
---

## 二叉树简介
二叉树：是每个结点最多有两个子树的有序树，在使用二叉树的时候，数据并不是随便插入到节点中的，一个节点的左子节点的关键值必须小于此节点，右子节点的关键值必须大于或者是等于此节点，所以又称二叉查找树、二叉排序树、二叉搜索树。

二叉树的每个结点至多只有二棵子树(不存在度大于2的结点)，二叉树的子树有左右之分，次序不能颠倒。二叉树的第i层至多有2^{i-1}个结点；深度为k的二叉树至多有2<sup>k-1</sup>个结点；对任何一棵二叉树T，如果其终端结点数为n_0，度为2的结点数为n_2，则n_0=n_2+1。

完全二叉树：若设二叉树的高度为h，除第 h 层外，其它各层 (1～h-1) 的结点数都达到最大个数，第h层有叶子结点，并且叶子结点都是从左到右依次排布，这就是完全二叉树。

满二叉树——除了叶结点外每一个结点都有左右子叶且叶子结点都处在最底层的二叉树。

深度——二叉树的层数，就是深度。

###二叉树特点：
1）树执行查找、删除、插入的时间复杂度都是O(logN)

2）遍历二叉树的方法包括前序、中序、后序

3）非平衡树指的是根的左右两边的子节点的数量不一致

4）在非空二叉树中，第i层的结点总数不超过2<sup>i-1</sup>, i>=1；

5）深度为h的二叉树最多有2<sup>h</sup>-1个结点(h>=1)，最少有h个结点；

6）对于任意一棵二叉树，如果其叶结点数为N0，而度数为2的结点总数为N2，则N0=N2+1；
### 遍历方法
``` bash
public class BTree{
private Node<Integer> root;

public BTree(){}
public BTree(int[] value){
	root = new Node<Integer>(value[0]);
	int length = value.length;
	if(length == 0)return;
	LinkList<Node<Integer>> quque = new LinkList<>();
	quque.addLast(root);
	Node<Integer> parent = null;
	Node<Integer> current = null;
	boolean isLeft = true;
	for(int i = 1; i < length; i++){
		current = new Node<Integer>(value[i]);
		quque.addLast(current);
		if(isLeft){
			parent = quque.getFirst();
		}else{
			parent = quque.removeFirst():
		}
		if(isLeft){
			parent.setLeftChild(current);
			isLeft = false;
		}else{
			parent.setRightChild(current);
			isLeft = true;
		}
	}
}

public void preOrder(){
	System.out.print("递归先序遍历：");
	preOrderTraverseRecursion(root);
	System.out.println();
}

public void preOrderTraverseRecursion(Node<Integer> value){
	System.out.print(value.getValue());
	if(null != value.getLeftChild()){
		preOrderTraverseRecursion(value.getLeftChild())
	}
	
	if(null != value.getRightChild()){
		preOrderTraverseRecursion(value.getRightChild());
	}
}

public void inorder(){
	System.out.print("递归中序遍历：");
	inorderTraverseRecursion(root);
	System.out.println();
}

public void inorderTraverseRecursion(Node<Integer> value){
	if(null != value.getLeftChild()){
		inorderTraverseRecursion(value.getLeftChild())
	}
	System.out.print(value.getValue());
	if(null != value.getRightChild()){
		inorderTraverseRecursion(value.getRightChild());
	}
}

public void postOrder(){
	System.out.print("递归后序遍历：");
	postOrderTraverseRecursion(root);
	System.out.println();
}

public void postOrderTraverseRecursion(Node<Integer> value){
	if(null != value.getLeftChild()){
		postOrderTraverseRecursion(value.getLeftChild())
	}
	
	if(null != value.getRightChild()){
		postOrderTraverseRecursion(value.getRightChild());
	}
	System.out.print(value.getValue());
}

class Node<V>{
	private V value;
	private Node<V> leftChild;
	private Node<V> rightChild;
	
	public Node(){
	}
	public Node(V value){
		this.value = value;
		leftChild = null;
		rightChild = null;
	}
	public void setLeftChild(Node<V> value){
		leftChild = value;
	}
	public void setRightChild(Node<V> value){
		rightChild = value;
	}
	public V getValue{
		return value;
	}
	public Node<V> getLeftChild(){
		return leftChild;
	}
	public Node<V> getRightChild(){
		return rightCHild;
	}
}
}
```
