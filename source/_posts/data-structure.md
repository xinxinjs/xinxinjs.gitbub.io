---
title: 数据结构基本知识与常见的数据结构:数组、栈、队列、链表
date: 2019-04-20 19:17:09
tags: [数据结构与算法]
categories: [数据结构与算法]
---

数据结构用来描述数据元素之间的关系

数据元素之间存在特定的关系，这种关系可以分为两种：逻辑结构和物理结构

## 数据关系

### 逻辑结构

逻辑结构描述了数据对象中数据元素之间的关系，按照逻辑结构可以分为四种：集合、线性结构、树形结构、图形结构

#### 集合：

数据元素同属于一个集合，但是数据元素之间没有其他的关系，如图：

![集合](/img/data/jihe.jpeg)

#### 线性结构：

是一组有序数据的集合，除了第一个元素和最后一个元素，其他的元素都是首位相接。

js中数组就是典型的线性结构，除此之外还有队列、栈和链表也是线性结构

![线性结构](/img/data/xianxing.jpeg)

#### 树形结构：

数据元素是一对多的关系

前端最常接触的树形结构是DOM树

![树形](/img/data/shuxing.jpeg)

#### 图形结构：

数据元素是多对多的关系

比如地图导航就是使用的图形结构来存储数据

![图形](/img/data/tuxing.jpeg)

### 物理结构

物理结构描述数据在内存中是如何存储的，数据的存储结构应该正确的反应数据之间的逻辑关系

分为：顺序存储（连续内存）和链式存储（非连续内存）

#### 顺序存储

数据在内存中是连续内存地址来存储的，数据元素之间的逻辑关系和物理关系是一致的

给数据分配内存的时候会在内存中开辟一段连续的内存地址

![顺序存储](/img/data/shunxu.jpeg)

#### 链式存储

链式存储会把数据元素放在任意的内存地址，可以是连续的地址，也可以是不连续的，链式存储是不能反映出数据元素之间的逻辑关系，所以链式存储需要一个叫做指针的东西，

指针可以反映出链式存储的数据元素之间的逻辑关系

![链式存储](/img/data/lianshi.jpeg)

## 常见的数据结构

![常见的数据结构](/img/data/changjian.jpeg)

下面是常见的数据结构的基本知识，和优缺点

### 数组

数组是比较常用的数据结构。作为一个前端，主要需要了解js的数组。

首先先了解一下传统意义上的数组的概念是：相同类型元素的有序集合，计算机会在内存中开辟一段连续的内存空间来存储数组

但是js中的数组我们有时候会写成这样子：`[1, 'a', 'b', 2]`，这显然不符合传统意义对于数组的定义，人家要求是相同类型的数据，所以说js中的数组不是传统意义上的数组。

在js中定义一个数组：

```javascript
var arr = [];
var arr1 = new Array();
```

数组的每一个元素都有一个索引，从0开始，连续排列：

![数组](/img/data/shuzu.jpeg)

那么在js中数组作为比较常见的数据结构他有什么优点呢：

1. 从上图看出，可以按照索引查询数组元素，速度会很快，我们在js语言中可以通过`array[index]`的方式来快速的查询数组元素

2. 数组可以存储大量的数据

3. 可以通过索引去遍历数组

数组的缺点：在js的数组中插入和删除的时候不太方便，从上面的图片可以看出索引和数组元素一一对应、连续排列，当我们对数组进行插入和删除操作的时候，索引和数组元素的对应关系在内存会重新整理，比如插入一个元素，在插入元素的位置后面的元素都要依次往后移动相应的位置，删除数组元素的时候，从删除元素位置后面的元素都要依次往前移动相对应的位置。这些移动的操作都会消耗一定的内存

![数组](/img/data/caozuoshuzu.jpeg)

所以要尽量避免从数组的头部插入或者删除元素，因为这样子数组里的每一个元素都要移动。尽可能的从数组的尾部插入或者删除元素

### 栈

在内存中有一个叫做栈内存，内存中的栈是真实存在的物理区，而我们这里讨论的是数据结构中的栈，是从栈内存抽象而来的数据结构存储结构

我们先来看一下栈的一些概念：

1. 是一种受限制的线性表，遵循后进先出的规则：（LIFO）last in first out

2. 受限制：仅允许在栈的一端进行插入和删除操作，这一端称之为栈顶，相对的另外一端称之为栈底

3. 向一个栈插入新元素的操作称作：进栈、入栈、或者压栈

4. 从一个栈删除元素又称之为退栈、出栈

下面这张图可以形象的描述栈的概念和受限制的操作

![栈](/img/data/zhan.jpeg)

在js语言中可以通过数组的push和pop方法实现栈的数据结构方法

```javascript
class Stack {
  constructor() {
    this.items = [];
  }

  // 往栈内添加一个元素
  push(ele) {
    this.items.push(ele);
  }

  // 原属出栈
  pop() {
    return this.items.pop();
  }

  // 返回栈顶元素
  peek() {
    return this.items[this.items.length-1];
  }

  // 判断是否为空栈
  isEmpty() {
    return this.items.length === 0;
  }

  // 获取栈中元素的个数
  size() {
    return this.items.length;
  }

  // 清空栈
  clear() {
    this.items.length = 0;
  }
}

const stack = new Stack();
stack.push(1);
stack.push(2);
stack.push(3);
stack.push(4);
console.log(stack) // Stack {items: [1,2,3,4]}

console.log(stack.pop()) // 4
console.log(stack.isEmpty()) // false
console.log(stack.size()) // 3
stack.clear();
console.log(stack.isEmpty()) // true
```

#### 栈的实际运用

在js语言中实现栈的数据结构操作还是比较简单的，那么栈在实际运用中有什么作用呢？学习栈这个数据结构对我们学习和使用js有什么帮助呢

##### 十进制转二进制

例如我们要把十进制数字100转换成二进制，需要使用除2取余，逆序排列

![十进制转二进制](/img/data/10-2.jpeg)

由此得出100转二进制的结果是1100100

那么如何用js的栈区实现这个算法呢？

```javascript
const binary = number => {
  let stack = new Stack();
  let remainder = 0 // 余数
  let top = '';

  while(number > 0) {
    remainder = number % 2;
    stack.push(remainder);
    number = Math.floor(number / 2);
  }

  while(!stack.isEmpty()) {
    top += stack.pop();
  }

  return top;
}

console.log(binary(100)); // 1100100
```

##### 理解js的调用栈

js是单线程的，代码自上而下的执行，依次进栈，执行完成之后代码出栈

如下图所示，首先是全局执行上下文入栈，然后outer函数入栈，最后inner函数入栈，然后inner函数出栈，outer函数出栈，全局环境出栈

![调用栈](/img/data/diaoyongzhan.jpeg)

##### 递归的原理

从上面可以看出函数执行的时候就是函数的进栈和出栈，执行的时候函数先入栈，执行完了出栈，但是在函数调用的时候有一个特殊的情况，就是递归。

执行递归的时候会进入递归栈：函数先进栈，达到跳出条件后在出栈，如果没有出栈就会导致栈溢出

从一个数字n的阶乘来理解递归

```javascript
function factor(n) {
  if (n === 1) return 1;
  return n * factor(n-1);
}
console.log(factor(5)) // 120
```

这个5的阶乘的函数调用栈如图所示:

![调用栈](/img/data/jiecheng.jpeg)

### 队列

队列是一种运算受限制的线性表，它遵循一种先进先出的规则

1. 只能在队列的前端进行删除操作

2. 在队列的尾端进行插入

![队列](/img/data/duilie.jpeg

用js实现一个队列：

```javascript
class Queue {
  constructor() {
    this.items = [];
  }
  // 入队
  enqueue(ele) {
    this.items.push(ele);
  }
  // 出队
  dequeue() {
    return this.items.shift();
  }
  // 查看队头
  front() {
    return this.items[0]
  }
  // 查看队尾
  rear() {
    return this.items[this.items.length-1]
  }
  // 是否为空
  isEmpty() {
    return this.items.length === 0;
  }
  // 获取长度
  size() {
    return this.items.length;
  }
  // 清空队列
  clear() {
    this.items.length = 0;
  }
}
```

队列这种数据结构，一般用于排队去一个一个的解决问题

那么队列这种数据结构和我们前端有什么关系呢？

#### 理解js的任务队列

js在浏览器中运行是单线程的，在js程序运行的时候会，所有的代码都是依次执行，不同的是：有的代码会立即执行，而有的代码则需要等待一定的时间才可以执行，比较常见的就是请求一个接口，等待接口有返回内容之后再去执行相应的代码

立即执行的代码叫做同步任务，需要等待执行的代码叫做异步任务，同步任务立即执行，同步任务是立即执行的但是也需要在主线程排队依次执行，这个排队的地方叫做调用栈，或者执行栈,

为了避免异步任务阻塞后面的同步任务的执行，js有一个异步任务队列，在执行到异步任务的时候，会把异步任务先放进异步任务队列，等待执行

等同步代码的调用栈里面的代码全部都执行完了以后，会依次把异步任务队列里面的代码进入调用栈执行，等到调用栈再次清空之后，再去执行异步的任务队列，这个往复循环的过程叫做js的事件循环机制

主线程读取任务队列并执行任务遵循先进先出的机制顺序

js是单线程语言，浏览器只会分配一个主线程给js，用来执行任务

异步任务队列分为：宏任务队列和微任务队列，同样都是异步任务，微任务队列是优先于宏任务队列执行的

最后js的任务执行优先顺序：同步任务->微任务->宏任务

宏任务：I/O 、定时器、事件绑定、ajax

微任务：promise.then catch finally process.nextTick async await

### 链表

前面说过数组，数组是最常见的数据结构，数组的优点就是查询比较快速，但是当我们对数组进行插入和删除的时候就比较麻烦。

那么为了解决数组的插入和删除比较麻烦的这个事情，链表就可以很好的解决数组的这个痛点，链表可以很方便快速的进行插入和删除

#### 链表的定义

1. 链表在物理内存上是链式存储，相邻元素不一定是连续的物理空间

2. 链表的每一个元素包含元素本身和一个指向下一个元素的引用，或者早有些语言里面叫做指针

#### 链表和数组的比较

1. 插入、删除操作，链表速度快性能好

2. 查询、修改操作，数组的性能好

3. 链表没有大小的限制，支持动态的扩容，js的数组是支持动态改变长度的，但是在很多其他的语言中，数组的大小是不能动态修改的

4. 因为链表需要存储指向下一个元素的指针，在内存上的消耗也比数组来的大，就是翻倍的内存

#### 用js语言实现一个链表

```javascript
class Node {
        constructor(element) {
          this.element = element;
          this.next = null
        }
      }
      class LinkedList {
        constructor() {
          // 链表的长度
          this.head = null;
          // 链表的长度
          this.length = 0;
        }
        // 在链表的尾部追加元素
        append(element) {
          let node = new Node(element);
          if(this.length === 0) {
            this.head = node
          }else {
            let current = this.head;
            while(current.next) {
              current = current.next;
            }
            current.next = node;
          }
          this.length += 1;
        }
        // 获取链表的头
        getHead() {
          return this.head;
        }
        toString() {
          let current = this.head;
          let linkString = '';
          while(current){
            linkString += ',' + current.element;
            current = current.next
          }
          return linkString.slice(1);
        }
        // 插入元素
        insert(element, position) {
          if(position < 0 || position > this.length) {
            return false;
          }
          let index = 0;
          let current = this.head;
          // 上一个元素
          let prev = null;
          let node = new Node(element);
          if(position === 0) {
            node.next = this.head;
            this.head = node;
          }else {
            while(index < position) {
              prev = current;
              current = current.next;
              index++;
            }
            node.next = current;
            prev.next = node;
          }
          this.length += 1;
          return true;
        }
        getPosition(position) {
          if(position < 0 || position > this.length) {
            return null
          }
          let current = this.head;
          let index = 0;
          while(index < position) {
            current = current.next;
            index++;
          }
          return current.element;
        }
        removeAt(position) {
          if(position < 0 || position >= this.length) return null;
          let current = this.head;
          let index = 0;
          let prev = null;
          if(position === 0) {
            this.head === this.head.next;
          }else {
            while(index < position) {
              prev = current
              current = current.next;
              index++;
            }
            prev.next = current.next;
          }
          this.length--;
          return current.element;
        }
      }

      const linkedList = new LinkedList();
      linkedList.append(1)
      linkedList.append(2)
      linkedList.append(3)
      console.log(linkedList);
      console.log(linkedList.removeAt(2))
```

我们在控制台打印出来长这样子：

![十进制转二进制](/img/data/lianbiao.jpeg)

由图可见，每个元素里面都有一个指向下一个元素的引用

#### 链表分类

1. 单项链表

2. 双向链表

#### 从链表的角度理解js的原型链

众所周知，javascript这门语言是基于对象的开发的，在面向对象的语言里面实现继承有两种方式：接口继承和实现继承

js语言无法实现接口继承，只支持实现继承，而js的实现继承主要是依靠的js的原型链机制

首先要记住两个：

1. 所有的函数都有一个`prototype`属性，该属性是一个普通的对象，一个隐式的对象

2. 所有的引用类型，都有一个`__proto__`属性是一个普通的对象，显示对象

例如我们写一个构造函数，构造函数也是函数，所有构造函数会有一个prototype属性，是一个隐式的对象,我们通常叫做原型对象，包含由特定类型的实例共享的属性和方法

```javascript
function SuperType() {
  this.name = 'name';
}
SuperType.prototype.getName = function () {
  return this.name;
}
console.log(SuperType)
/**
 * 因为prototype属性是隐式的对象，所以打印出来的是函数体
 SuperType() {
  this.name = 'name';
 }
 */
//  我们可以手动的打印一下prototype属性
console.log(SuperType.prototype)
```

第二个console，我们手动的打印了prototype属性，打印出来的是构造函数的方法和一个原型对象，我们可以看到原型对象，如下图:

![图片](/img/data/yuanxing.jpeg)

从图中可以看出来构造函数的prototype是一个对象，所以包含一个`__proto__`属性和一个`constructor`属性，这个`constructor`指向这个原型对象的构造函数，也就是SuperType

还包含原型对象上的方法

然后我们实例化SuperType，返回一个对象

```javascript
console.log(new SuperType())
```

![图片](/img/data/shilihua.jpeg)

从上图可以看出，通过new关键字实例化返回的是一个对象，这个对象包含构造函数上的方法和属性，同时和其他普通对象一样，包含一个`__proto__`属性，这个`__proto__`属性包含构造函数的原型上的属性和方法。还有一个`constructor`属性，指向这个实例的构造函数

画个图表示一下

![图片](/img/data/gouzaohanshu.svg)

而我们是如何通过原型链实现j的继承呢？简单的说一下就是让第二个原型对象等于另一个对象的实例，

先从代码的角度看一下

```javascript
// 父级函数
function SuperType() {
  this.name = 'name';
}
SuperType.prototype.getName = function () {
  return this.name;
}
// 子级函数
function SubType() {
  this.age = 100;
}
// 子级的原型对象等于父级的实例
SubType.prototype = new SuperType();
SubType.prototype.getAge = function () {
  return this.age;
}
console.log(SubType.prototype);
```

我们先打印一下SubType.prototype，发现和我们刚才打印`console.log(new SuperType())`打印出来的是一样的，因为SubType的prototype就是Supertype的实例，但是也不是一摸一样，因为这还包含了SubType自己的原型上的方法

然后我们实例化一个这个新的构造函数，在打印一下

```javascript
var a = new SubType();
console.log(a);
```

![图片](/img/data/jicheng.svg)

从控制台可看出来，实例化返回一个新的对象，这个对象和其他实例化的对象一样，首先包含它构造函数上的属性和方法，然后包含一个`__proto__`属性，这个`__proto__`属性包含它的构造函数的原型上的属性和方法。而Subtype的原型又是SuperType的实例，所以同时包含了SuperType原型上的方法和属性，

再有第三个构造函数要实现继承也是如此实现，语言太啰嗦不清晰，画图表示一下

![图片](/img/data/jichenglianbiao.svg)

从图中可以看出来，原型链的继承的数据结构是链表的结构实现的，而所有的对象最终的继承都是Object
