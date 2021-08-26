---
title: 数据结构基本知识与常见的数据结构三：树
date: 2019-04-27 19:17:09
tags: [数据结构与算法]
categories: [数据结构与算法]
---

前端常见的树结构的数据是DOM树：

```html
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
  </head>
  <body>
    <div class="wrap">
      <h1>title</h1>
      <div>
        <p>内容文字</p>
      </div>
  </body>
</html>
```

把上面的html代码想象成一棵树：

![dom树](/img/data/domTree.jpg)

从上面的DOM树我们可以看出来一些树的特点：

1. 每一棵树都有一个唯一的根节点，在DOM树结构里，这个根节点就是根元素：html节点，也叫页面的根节点，

2. HTML元素作为根节点，有两个子节点head和body，head和body节点又分别有自己的子节点，

3. head和body互为兄弟节点，兄弟节点就是都拥有相同的父节点的元素，

4. 每一个节点都可以拥有任意个数的子节点

## 什么是树，树的定义

那具体，如何来定义什么是树的概念呢？

一个简单的树结构，如下图：

![树](/img/data/shudegainian.jpg)

树（Tree）：n个节点构成的有限集合

n = 0 ,称之为空树

那么一颗非空的树（n > 0），应具备以下特点：

1. 树中有一个称之为根（root）的特殊节点，

2. 其余的节点可以分为m个互不相交的有限集合，注意是互不相交的有限集合

3. 每一个集合又是一棵树，称之为原来树的子树

4. 子树之间不可相交

5. 除了根节点，每一个节点都有且仅有一个父节点

6. 连接两个节点的叫边，n个节点的树，有n-1条边

7. 树形结构节点之间的关系是一对多

## 树结构的术语

1. 节点的度：节点所拥有的子树的个数

2. 树的度：树中节点度的最大值

3. 叶子：终端节点，度为0的节点

4. 分支节点：非终端节点，度不为0的节点

5. 子节点：节点的直接后继节点

6. 父节点：节点的直接前驱节点

7. 兄弟节点：拥有同一个父节点

8. 子孙节点，某个节点的所有后继节点

9. 祖先节点：某个结点的所有前驱节点

10. 路径：从某个节点咨商的到另一个节点，所经过的边

11. 节点的层：根节点的层是1，其余的节点是其父节点的层数加一

12. 树的深度：所有节点层数的最大值

13. 有序树、无序树

## 树的存储结构

一棵树在内存中是如何存储的呢？

计算机存储数据的时候，是链式存储或者顺序存储，树这种结构是不能被直接存储的，要将他转换为链式存储或者顺序存储的方式才可以

比如这一个树：

![树](/img/data/shudegainian.jpg)

计算机有两种存储的方式课存储树：

### 双亲表示法

采用数组存储普通的树，顺序存储数组，给每个节点附加一个记录当前节点父节点位置的变量，如下图表示

![双亲表示法](/img/data/shuangqinbiaoshi.jpg)

双亲表示法的优点：可以快速的去获取任意节点的父节点

双亲表示法的缺点：无法直接获取某个节点的子节点，需要遍历数组才可以获取某个节点的子结点

如上图表示，

a节点是根节点，没有父节点，索引没有父节点的索引，a节点存储的父节点的索引就是-1，b、c、d的父节点是a，a节点的索引是0，所以b、c、d节点存储的父节点的索引就是0

### 孩子表示法

建立多个指针域，指向子节点的地址，也就是说任何一个节点都掌握他子节点的信息

![孩子表示法](/img/data/haizibiaoshi.jpg)

其核心思想是：

1. 从根节点开始依次存储树的各个节点

2. 给每个节点配备一个链表，会给各个节点配备一个链表，用于存储各个节点的孩子节点位于数组中的位置。如果说节点没有子节点，该节点的链表为空

如上图表示：

a节点的子结点是b、c、d，这三个节点对应的索引是1，2，3所以a节点的指针里面存储的就是这三个子结点的索引值：1，2，3

### 孩子兄弟表示法

把普通的树转成二叉树，从树的根节点开始，依次用链表存储各个节点的孩子节点和兄弟节点

孩子表示法需要给每个节点存储子节点的指针域，那么孩子兄弟表示法，需要给每个节点在增加一个指针域，用来存储其兄弟节点索引的指针域

## 二叉树

![二叉树](/img/data/erchashu.jpg)

二叉树的概念：树中的所有节点，最多只能有两个子结点

二叉树的特点：

1. 每个节点最多有两个子结点

2. 左子树和右子树是有序的

3. 即使一个节点只有一颗子树，也要区分是左子树还是右子树

### 满二叉树

![二叉树](/img/data/manerchashu.jpg)

一颗二叉树，所有的分支节点都有左子树和右子树，并且所有的叶子节点都在同一层

满二叉树的特点：

1. 叶子节点都在同一层，且在最下层

2. 非叶子节点的度，一定是2

### 完全二叉树

![二叉树](/img/data/wanquanerchashu.jpg)

满二叉树一定是完全二叉树

一棵深度为k的有n个结点的二叉树，对树中的结点按从上至下、从左到右的顺序进行编号，如果编号为i（1≤i≤n）的结点与满二叉树中编号为i的结点在二叉树中的位置相同，则这棵二叉树称为完全二叉树

## 二叉搜索树

![二叉树](/img/data/erchasousuoshu1.jpg)

二叉搜索树，又称二叉查找树，二叉排序树

二叉搜索树或者是一棵空树，或者具有以下特点：

1. 若它的左子树不空，则左子树上所有结点的值均小于它的根结点的值

2. 若它的右子树不空，则右子树上所有结点的值均大于它的根结点的值

3. 它的左、右子树也分别为二叉排序树

### 自己实现一个二叉搜索树

```javascript
class BSTNode {
  constructor(value) {
    this.value = value;
    this.left = null;
    this.right = null
  }
}

class BST {
  constructor () {
    this.root = null
  }

  insertNode(node, newNode) {
    if(newNode.value > node.value) {
      if(node.right === null) {
        node.right = newNode
      }else {
        this.insertNode(node.right, newNode)
      }
    }else if(newNode.value < node.value) {
      if(node.left === null) {
        node.left = newNode
      }else {
        this.insertNode(node.left, newNode)
      }
    }
  }

  insert(value) {
    const newNode = new BSTNode(value);
    if(this.root === null) {
      this.root = newNode;
    }else {
      this.insertNode(this.root, newNode)
    }
  }
}

const bst = new BST();
bst.insert(10);
bst.insert(12);
bst.insert(9);
console.log(bst)
```

### 二叉搜索树的特点

小值在左边，大值在右边，查找元素的时候比较快

### 二叉搜索树的遍历

遍历二叉搜索树有三种遍历方式：先序遍历，中序遍历，后序遍历

下面这张图，我们分别使用三种遍历方式进行遍历

![二叉树](/img/data/erchasousuoshu2.jpg)

#### 先序遍历

先序遍历：首先遍历根节点，然后遍历左子树，遍历右子树

当前二叉搜索树的节点比较少，按照先序遍历的规则，我们可以用眼睛看一下就知道先序遍历结果应该是这样：10，5，3，6，15，12，16

我们来写一个先序遍历的算法，实现一下先序遍历，基于我们刚才写的二叉搜索树，已经具备插入节点的方法，接着给这个二叉搜索树增加先序遍历方法

```javascript
...

class BST {
  ...

  preOrder() {
    const arr = [];
    this.preOrderNode(this.root, arr);
    return arr;
  }

  preOrderNode(node, arr) {
    if(node === null) return;
    arr.push(node.value);
    const left = node.left;
    const right = node.right;
    if(left) this.preOrderNode(left, arr);
    if(right) this.preOrderNode(right, arr);
  }
}

const bst = new BST();
bst.insert(10);
bst.insert(5);
bst.insert(15);
bst.insert(3);
bst.insert(6);
bst.insert(12);
bst.insert(16);
const preOrderList = bst.preOrder()
console.log(preOrderList) // [10, 5, 3, 6, 15, 12, 16]
```

如上面的代码，我们新增来先序遍历的方法：preOrder，然后按照上面图中的节点依次插入节点，然后调用先序遍历的方法，并在控制台打印这个遍历的结果，和上面的用眼睛看出来的结果是一样的

#### 中序遍历

从根节点开始，先递归遍历其左子树，然后从最后一个节点存入数组，然后回溯遍历双亲节点，然后是右子树

我们同样使用js实现一个二叉搜索树的中序遍历

```javascript

class BST {
  inOrder() {
    const arr = [];
    this.inOrderNode(this.root, arr);
    return arr;
  }

  inOrderNode(node, arr) {
    if(node === null) return;
    const left = node.left;
    const right = node.right;
    if(left) this.inOrderNode(left, arr);
    arr.push(node.value);
    if(right) this.inOrderNode(right, arr);
  }
}

const bst = new BST();
bst.insert(10);
bst.insert(5);
bst.insert(15);
bst.insert(3);
bst.insert(6);
bst.insert(12);
bst.insert(16);
const inOrderList = bst.inOrder()
console.log(inOrderList) // [3, 5, 6, 10, 12, 15, 16]
```

#### 后续遍历

先遍历其左子树，在遍历其右子树，最后遍历根节点

同样，我们使用js代码来实现一些上面二叉搜索树的后续遍历

```javascript
...
class BST 
  ...
  postOrder() {
    const arr = [];
    this.postOrderNode(this.root, arr);
    return arr;
  }

  postOrderNode(node, arr){
    if(node === null) return;
    const left = node.left;
    const right = node.right;
    
    if(left) this.postOrderNode(left, arr);
    if(right) this.postOrderNode(right, arr);
    arr.push(node.value);
  }
}

const bst = new BST();
bst.insert(10);
bst.insert(5);
bst.insert(15);
bst.insert(3);
bst.insert(6);
bst.insert(12);
bst.insert(16);

const postOrderList = bst.postOrder();
console.log(postOrderList) // [3, 6, 5, 12, 16, 15, 10]
```

### 二叉搜索树的其他

我们已经实现了二叉搜索树的插入和三种遍历的方法，由于二叉搜索树的特殊性：小值在左边，大值在右边，我们可以利用这个特性，快速的找到树中的最大值和最小值。

让我们来写一段代码实现查找二叉搜索树中最大值和最小值的方法

```javascript
max() {
  let node = this.root;
  while(node.right !== null) {
    node = node.right;
  }
  return node.value
}
min() {
  let node = this.root;
  while(node.left !== null) {
    node = node.left;
  }
  return node.value
}

console.log(bst.max()) // 16
console.log(bst.min()) // 3
```

寻找特定的值

```javascript
search(val) {
  let node = this.root;
  while (node !== null) {
    if(node.value > val){
      node = node.left
    } else if (node.value < val) {
      node = node.right
    } else {
      return true;
    }
  }
  return false;
}

console.log(bst.search(25)) // false
console.log(bst.search(15)) // true
```

### 删除

删除二叉搜索树中的节点，可以分为三种情况：

1. 被删除的节点，没有子树，那么可以直接把这个节点删除掉

2. 被删除的节点，仅有一颗子树，删除掉该节点之后，直接把他的子节点放到被删除的节点的位置

3. 被删除的节点，有两颗子树，我们需要保证删除该节点之后他的中序遍历的顺序不变

### 二叉搜索树的优缺点

优点

1. 快速查找

2. 快速的插入和删除

缺点：同样的一组数据，插入的顺序不同，最后导致的二叉搜索树的结果可能也会不一样

比如，我要在二叉搜索树中插入：1、2、3、4、5、6、7这样子的一组数据，如果我先插入的是4，理想的状态下，最后二叉树的结构如下

![二叉树](/img/data/erchasousuoshu3.jpg)

我们理想的情况下，二叉搜索树的结构是左右分布均匀的，如上图那样，叫做平衡二叉搜索树，这种树结构的查询速度通常比较快

但是如果我先插入的是1这个最小的数据，那么在极端情况下，二叉搜索树的结构可能就会变成一颗完全的右斜树，如下图，看起来像链表

![二叉树](/img/data/erchasousuoshu4.jpg)

在这种情况下，会影响二叉搜索树的查询速度，也叫做非平衡的二叉搜索树

所以为了保证我们的查询速度，我们要尽量保证我们的二叉搜索树是左右平衡的，这种树叫做平衡二叉搜索树

### 平衡二叉搜索树

我们先来了解一个概念：平衡因子

在二叉树中每个节点的左子树的高度减去右子树高度，取绝对值，就是这个节点的平衡因子

在平衡树中，我们要求每个节点的平衡因子的值不超过1，也就是说平衡因子的值只能是0，或者1，由此可以保证我们的二叉搜索树是一颗平衡树

例如，下面这棵二叉搜索树，目前来看是平衡的二叉搜索树，因为他的每个节点的平衡因子都没有超过1

![二叉树](/img/data/pinghengshu.jpg)

当我们想要在这个树中插入一个节点，比如，插入一个节点的值是3，那么插入之后这个二叉树就会变成下面这个样子，变成了一颗非平衡树，因为节点7的平衡因子，变成了2

![二叉树](/img/data/feipinghengshu.jpg)

为了保证二叉搜索树的平衡，我们需要对节点进行调整，那么红黑树就是可以自平衡的二叉搜索树，我们对一个二叉搜索树执行插入的时候，可以自动对节点的平衡因子进行调整

## 红黑树

也叫R-B Tree,

红黑树的特点：

1. 在设计红黑树的时候需要给节点增加一个属性，也就是节点的颜色属性，红色或者是黑色

```javascript
class BSTNode {
  constructor(value) {
    this.value = value;
    this.left = null;
    this.right = null;
    this.color = 'black'; // 新增的节点颜色属性
  }
}
```

2. 根节点是黑色

3. 叶子节点都是黑色，且为null

4. 连接红色节点的两个子节点都是黑色，红色节点的父节点都是黑色

5. 从任意的节点出发，到其每个叶子节点的路径中，包含相同的数量的黑色节点

例如下面这个红黑树,就满足上面所说的特点

![红黑树](/img/data/rbtree.jpg)

新插入的节点都是红色，在我们往红黑树中插入节点的时候，必须遵守红黑树的这五个特点

在插入节点的时候，会先去遍历当前要插入的节点，应该在哪个位置，因为新插入的节点都是红色的，为了保证红黑树的性质，还需要看是否需要调整节点的颜色属性

## 代码

### 二叉搜索树的完整代码

```javascript
class BSTNode {
  constructor(value) {
    this.value = value;
    this.left = null;
    this.right = null
  }
}

class BST {
  constructor () {
    this.root = null
  }

  insertNode(node, newNode) {
    if(newNode.value > node.value) {
      if(node.right === null) {
        node.right = newNode
      }else {
        this.insertNode(node.right, newNode)
      }
    }else if(newNode.value < node.value) {
      if(node.left === null) {
        node.left = newNode
      }else {
        this.insertNode(node.left, newNode)
      }
    }
  }

  insert(value) {
    const newNode = new BSTNode(value);
    if(this.root === null) {
      this.root = newNode;
    }else {
      this.insertNode(this.root, newNode)
    }
  }

  preOrder() {
    const arr = [];
    this.preOrderNode(this.root, arr);
    return arr;
  }

  preOrderNode(node, arr) {
    if(node === null) return;
    arr.push(node.value);
    const left = node.left;
    const right = node.right;
    if(left) this.preOrderNode(left, arr);
    if(right) this.preOrderNode(right, arr);
  }

  inOrder() {
    const arr = [];
    this.inOrderNode(this.root, arr);
    return arr;
  }

  inOrderNode(node, arr) {
    if(node === null) return;
    const left = node.left;
    const right = node.right;
    if(left) this.inOrderNode(left, arr);
    arr.push(node.value);
    if(right) this.inOrderNode(right, arr);
  }

  postOrder() {
    const arr = [];
    this.postOrderNode(this.root, arr);
    return arr;
  }

  postOrderNode(node, arr){
    if(node === null) return;
    const left = node.left;
    const right = node.right;
    if(left) this.postOrderNode(left, arr);
    if(right) this.postOrderNode(right, arr);
    
    arr.push(node.value);
  }

  max() {
    let node = this.root;
    while(node.right !== null) {
      node = node.right;
    }
    return node.value
  }

  min() {
    let node = this.root;
    while(node.left !== null) {
      node = node.left;
    }
    return node.value
  }

  search(val) {
    let node = this.root;
    while (node !== null) {
      if(node.value > val){
        node = node.left
      } else if (node.value < val) {
        node = node.right
      } else {
        return true;
      }
    }
    return false;
  }
}

const bst = new BST();
bst.insert(10);
bst.insert(5);
bst.insert(15);
bst.insert(3);
bst.insert(6);
bst.insert(12);
bst.insert(16);
const inOrderList = bst.inOrder()
console.log(inOrderList) // [3, 5, 6, 10, 12, 15, 16]

const postOrderList = bst.postOrder();
console.log(postOrderList) // [3, 6, 5, 12, 16, 15, 10]

console.log(bst.max()) // 16
console.log(bst.min()) // 3

console.log(bst.search(25)) // false
console.log(bst.search(15)) // true
```