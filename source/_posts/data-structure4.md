---
title: 数据结构基本知识与常见的数据结构三：图
date: 2019-05-03 19:17:09
tags: [数据结构与算法]
categories: [数据结构与算法]
---

## 什么是图？

Graph，是一种数据结构，一种多对多的数据结构

先来回忆一下之前了解到的两种数据结构：

> 一对一：线性结构

![线性结构](/img/data/xianxingjiegou.jpg)

> 一对多：树形结构

![树形结构](/img/data/shuxingjiegou.jpg)

我们即将学习的图形结构长这样子：从图中可以看出，数据元素和数据元素之间的关系是多对多的

![图形结构](/img/data/tuxingjiegou.jpg)

## 图的概念

顶点：上图中数据元素叫做图的顶点

边：连接两个顶点的关系，叫做边

图是没有父顶点的子顶点之分，所有的顶点都是平等的

## 图的分类

> 无向图：边没有方向的图

![图形结构](/img/data/wuxiangtu.jpg)

> 有向图:边有方向的图，可能是单项的，也可能是双向的

![图形结构](/img/data/youxiangtu.jpg)

> 带权图

![图形结构](/img/data/daiquantu.jpg)

## 图的术语

1. 相邻顶点：当两个顶点通过一条鞭相连的时候

2. 图的度：某个顶点的图，是这个顶点边的个数

3. 子图：某个图中所有边的子集组成的图

4. 路径：图的边顺序连接的一系列顶点组成

5. 环：只有一条边，且终点和起点相同的路径

6. 连通图：如果图中任意一个顶点，都存在一条路径到达另外一个顶点

7. 连通子图：

## 图的存储结构

计算机可以使用数组去一次存储图的顶点，那么顶点和顶点之间的边，如何存储呢？

计算机存储图形结构边的时候，有两种存储方法：

1. 邻接矩阵

2. 邻接表

### 邻接矩阵

例如下面这个图，用邻接矩阵的方法如何表示呢

我们画一个N*N的矩阵，行和列分别表示图的各个顶点，行和列交叉的地方用数字表示两个顶点之间是否有关系

1表示两个顶点之间是有关系的，0表示顶点和顶点之间是没有关系的

![图形结构](/img/data/linjiejuzhen.jpg)

邻接矩阵对应到我们的代码里面，就是一个而为数组：

```javascript
[
  [0, 1, 0, 0, 0, 1],
  [1, 0, 1, 1, 0, 0],
  [0, 1, 0, 1, 0, 0],
  [0, 1, 1, 0, 1, 0],
  [0, 0, 0, 1, 0, 1],
  [1, 0, 0, 1, 1, 0]
]
```

> 邻接矩阵的优点： 表示非常明确，直观

> 缺点：浪费非常大的内存，去存储很多0，消耗的空间是图的顶点个数的平方，假设图有n个顶点，那么邻接矩阵占的空间就是n的平方

### 邻接表

邻接表是由图中的顶点和该顶点相邻的顶点列表组成

![图形结构](/img/data/linjiebiao.jpg)

## 用js代码实现图的存储

![图形结构](/img/data/wuxiangtu.jpg)

我们用js实现存储上面这个图的代码，我们使用邻接表的方法来实现

```javascript
class Graph {
  constructor() {
    this.vertexs = [] //存储顶点
    this.edgeList = {} //存储边
  }

  addVertex (v) {
    this.vertexs.push(v);
    this.edgeList[v] = [];
  }

  addEdge (a, b) {
    this.edgeList[a].push(b);
    this.edgeList[b].push(a);
  }
}

const graph = new Graph();
// 添加顶点
graph.addVertex(1)
graph.addVertex(2)
graph.addVertex(3)
graph.addVertex(4)
graph.addVertex(5)
graph.addVertex(6)
// 添加边
graph.addEdge(1, 2)
graph.addEdge(1, 6)
graph.addEdge(2, 3)
graph.addEdge(2, 4)
graph.addEdge(3, 4)
graph.addEdge(4, 5)
graph.addEdge(4, 6)
graph.addEdge(5, 6)

console.log(graph)
```

打印出来的结果

```javascript
Graph {
  vertexs: [ 1, 2, 3, 4, 5, 6 ],
  edgeList:
   { '1': [ 2, 6 ],
     '2': [ 1, 3, 4 ],
     '3': [ 2, 4 ],
     '4': [ 2, 3, 5, 6 ],
     '5': [ 4, 6 ],
     '6': [ 1, 4, 5 ] } 
    }
```

我们对比代码运行出来的结果和刚才在邻接表中的情况是一摸一样的

![图形结构](/img/data/linjiebiaojs.jpg)

## 图的遍历

从某个顶点出发，按照一定的路径，一次访问图中的每个顶点，且没有顶点只访问一次

图的遍历也有两种方式，和树的遍历一样：广度优先、深度优先

> 广度优先（BFS）

从图中的某个顶点出发，依次访问该顶点未被访问过的连接顶点，然后在分别从这些邻接的顶点开始一次访问他们的邻接顶点

其目的是图中所有的顶点都要被访问到

还是这个图，我们先用广度优先的思路来看一下遍历的结果

![图形结构](/img/data/tuguangduyouxian.jpg)

按照广度优先的规则，我们遍历出来的顺序应该是：1，2，6，3，4，5

> 深度优先（DFS）

从图的某个顶点出发，依次访问该顶点的未被访问过的邻接顶点，然后从每个邻接顶点出发，对每个邻接的顶点进行深度优先遍历，是一个递归的概念，直到图中所有和出发顶点有相通路径的顶点都被访问过

![图形结构](/img/data/tushenduyouxian.jpg)

上面的图，按照深度优先遍历，顺序应该是： 1，2，3，4，5，6

> 图的遍历思路

图中的每个顶点在遍历的时候有三种状态：

1. 未发现，没有发现这个顶点

2. 已经发现，发现这个顶点，但是没有遍历该顶点的全部连接顶点

3. 已探索，发现该顶点，并且已经遍历过该顶点的全部连接顶点

我们用三种颜色，分别表示图中顶点的访问状态：

1. 白色，未发现

2. 灰色，已发现

3. 已访问

### js实现广度优先

图的广度优先算法需要借助之前写的队列来完成，，需要置顶一个开始遍历的顶点，我们可以叫做开始顶点，

其思路是： 

1. 发现顶点的颜色是白色的时候，存入队列，等待遍历其邻接顶点，并将这些顶点标记为已发现

2. 从队列中拿出已发现的顶点，并开始访问其全部的邻接顶点，并且跳过已经访问的顶点

3. 遍历完这个顶点后，将其标记为黑色

4. 循环在队列中探索下一个顶点

```javascript
// 初始化颜色
  initColor () {
    const colors = {};
    for (let i = 0; i < this.vertexs.length; i++) {
      colors[this.vertexs[i]] = 'white';
    }
    return colors
  }

  bfs (v, callback) {
    const colors = this.initColor();

    const queue = new Queue();
    queue.enqueue(v);
    while(!queue.isEmpty()) {
      const qVertexs = queue.dequeue();
      const edge = this.edgeList[qVertexs]
      for(let i = 0; i < edge.length; i++) {
        const e = edge[i]
        if(colors[e] === 'white') {
          colors[e] = 'grey';
          queue.enqueue(e)
        }
      }
      colors[qVertexs] = 'black';
      if(callback) {
        callback(qVertexs)
      }
    }
  }
```

### js实现深度优先

深度优先需要使用的是递归，其思路是

1. 从某个顶点开始查找，并将其标记为灰色，即已发现状态

2. 从这个顶点开始访问其他的全部顶点，并需要跳过已访问过的顶点

3. 遍历完这个顶点之后，将这个顶点标记为黑色

4. 递归返回，继续访问下一个顶点

```javascript
dfsVisite (v, colors, callback) {
    colors[v] = 'grey';
    if(callback) {
      callback(v);
    }
    const edge = this.edgeList[v];
    for(let i = 0; i < edge.length; i++) {
      const e = edge[i]
      if(colors[e] === 'white') {
        this.dfsVisite(edge[i], colors, callback)
      }
    }
    colors[v] = 'black';
  }

  dfs (v, callback) {
    const colors = this.initColor();
    this.dfsVisite(v, colors, callback);
  }
```

## 最短路径

什么是路径：由边顺序连接的一些列的顶点所组成

最短路径：从图中的某个顶点开始（起点），到达另外一个顶点（终点），可能有多条，我们需要找到一条路径上的顶点权值总和最小

回溯点：离上一个顶点最近的顶点

回溯路径：所有的回溯点组成回溯路径

## 本节全部代码

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

class Graph {
  constructor() {
    this.vertexs = [] //存储顶点
    this.edgeList = {} //存储边
  }

  addVertex (v) {
    this.vertexs.push(v);
    this.edgeList[v] = [];
  }

  addEdge (a, b) {
    this.edgeList[a].push(b);
    this.edgeList[b].push(a);
  }

  // 初始化颜色
  initColor () {
    const colors = {};
    for (let i = 0; i < this.vertexs.length; i++) {
      colors[this.vertexs[i]] = 'white';
    }
    return colors
  }

  bfs (v, callback) {
    const colors = this.initColor();

    const queue = new Queue();
    queue.enqueue(v);

    let prev = {};
    for(let i = 0; i < this.vertexs.length; i++) {
      prev[this.vertexs[i]] = null
    }

    while(!queue.isEmpty()) {
      const qVertexs = queue.dequeue();
      const edge = this.edgeList[qVertexs]
      for(let i = 0; i < edge.length; i++) {
        const e = edge[i]
        if(colors[e] === 'white') {
          colors[e] = 'grey';
          queue.enqueue(e)
        }
      }
      colors[qVertexs] = 'black';
      if(callback) {
        callback(qVertexs)
      }
    }
  }

  dfsVisite (v, colors, callback) {
    colors[v] = 'grey';
    if(callback) {
      callback(v);
    }
    const edge = this.edgeList[v];
    for(let i = 0; i < edge.length; i++) {
      const e = edge[i]
      if(colors[e] === 'white') {
        this.dfsVisite(edge[i], colors, callback)
      }
    }
    colors[v] = 'black';
  }

  dfs (v, callback) {
    const colors = this.initColor();
    this.dfsVisite(v, colors, callback);
  }
}

const graph = new Graph();
// 添加顶点
graph.addVertex(1)
graph.addVertex(2)
graph.addVertex(3)
graph.addVertex(4)
graph.addVertex(5)
graph.addVertex(6)
// 添加边
graph.addEdge(1, 2)
graph.addEdge(1, 6)
graph.addEdge(2, 3)
graph.addEdge(2, 4)
graph.addEdge(3, 4)
graph.addEdge(4, 5)
graph.addEdge(4, 6)
graph.addEdge(5, 6)

console.log(graph)
console.log('广度优先')
graph.bfs(1, (v) => {
  console.log(v)
});
console.log('深度优先')
graph.dfs(1, (v) => {
  console.log(v)
})
```