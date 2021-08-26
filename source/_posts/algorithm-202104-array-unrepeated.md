---
title: 数组去重的三种思路
date: 2017-04-14 14:27:43
tags: [数据结构与算法]
categories: [数据结构与算法]
---

## 双层循环去重

基本思路就是：声明一个空数组作为去重之后的数组，第一层循环遍历要去重数组使用for或者forEach，第二层循环循环新数组，判断新数组里面没有第一次循环的当前项，就把当前项push进新的数组

### indexOf

```javascript
var a = [1,2,3,4,5,4,3,2,1,2,3,4,5];

      var b = []
      for(let i = 0; i < a.length-1; i++) {
        if(b.indexOf(a[i]) == -1) {
          b.push(a[i])
        }
      }
      console.log(b) // [1,2,3,4,5]
```

### includes

```javascript
var a = [1,2,3,4,5,4,3,2,1,2,3,4,5];

      var b = []
      for(let i = 0; i < a.length-1; i++) {
        if(!b.includes(a[i])) {
          b.push(a[i])
        }
      }
      console.log(b) // [1,2,3,4,5]
```

## es6方法 Array.from(new Set(arr))

```javascript
var a = [1,2,3,4,5,4,3,2,1,2,3,4,5];
Array.from(new Set(a)) // [1,2,3,4,5]
```

## 利用对象的属性不能重复

```javascript
var a = [1,2,3,4,5,4,3,2,1,2,3,4,5];
var c = {}
for(let i in a) {
  c[a[i]] = a[i]
}
console.log(Object.keys(c))
```

这个方法的缺点就是对象的key都是字符串，如果数组的值为number或者其他类型的时候，就会转换成字符串，在某些场景下不适用