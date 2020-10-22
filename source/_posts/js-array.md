---
title: js数组操作
date: 2020-08-04 10:56:34
tags: [javascript, 数组]
---

## 数组随机排序

### 随机数打乱数组的排序

遍历数组，每次循环都随机一个在数组长度范围内的数，并交换本次循环的位置和随机数位置上的元素

通过随机数打乱数组原有的排序

```javascript
function randomSort1(arr) {
  for (let i = 0, l = arr.length; i < l; i++) {
    // 生成一个在数组长度范围内的随机数
    let rc = parseInt(Math.random() * l)
    // 让当前循环的数组元素和随机出来的数组元素交换位置
    const current = arr[i]
    arr[i] = arr[rc]
    arr[rc] = current
  }
  return arr
}
```

### 生成新的随机排序数组

通过随机数从数组中随机取出某一项，重新排序组成新的数组

- 循环给定的数组，

- 每次循环根据数组的长度获取随机整数

- 以获取的随机数为索引，从数组中取出该随机数所对应的那一项，push到新的数组中去

- 然后用splice方法从原数组中删除已经放进新数组的那一项

- 循环执行以上操作

```javascript
function randomSort2(arr) {
  var mixedArr = []
  while (arr.length > 0) {
    let rc = parseInt(Math.random() * arr.length)
    mixedArr.push(arr[rc])
    arr.splice(rc, 1)
  }
  return mixedArr
}
```

### array.sort方法排序

sort方法会在愿数组上进行排序，接受一个排序的方法，通过生成随机数的方式，决定要不要调换当前两个元素的顺序

```javascript
function randomSort3(arr) {
  arr.sort(function (a, b) {
    return Math.random() - 0.5
  })
  return arr
}
```

## 对象数组排序

### 根据对象单个属性排序

```javascript
function compare(property) {
  return function (a, b) {
    let value1 = a[property]
    let value2 = b[property]
    return value1 - value2
  }
}

let arr = [
  { name: 'zopp', age: 10 },
  { name: 'gpp', age: 18 },
  { name: 'yjj', age: 8 },
]

arr.sort(compare('age'))
```

### 多个属性排序

## 数组扁平化

### ES6 中的 flat 方法

array.flat接受一个数字，表示要扁平化的深度

### 利用递归

```javascript
let result = []
let flatten = function (arr) {
  for (let i = 0; i < arr.length; i++) {
    let item = arr[i]
    if (Array.isArray(arr[i])) {
      flatten(item)
    } else {
      result.push(item)
    }
  }
  return result
}
```

### 利用 reduce 函数迭代

```javascript
function flatten2(arr) {
  return arr.reduce((pre, cur) => {
    return pre.concat(Array.isArray(cur) ? flatten2(cur) : cur)
  }, [])
}
```

### 扩展运算符

```javascript
function flatten3(arr) {
  while (arr.some((item) => Array.isArray(item))) {
    arr = [].concat(...arr)
  }
  return arr
}
```

## 数组去重

### 利用indexOf

```javascript
function unique(arr) {
  var newArr = []
  for (var i = 0; i < arr.length; i++) {
    if (newArr.indexOf(arr[i]) === -1) {
      newArr.push(arr[i])
    }
  }
  return newArr
}
```

### 先将原数组排序，在与相邻的进行比较，如果不同则存入新数组

```javascript
function unique2(arr) {
  var formArr = arr.sort()
  var newArr = [formArr[0]]
  for (let i = 1; i < formArr.length; i++) {
    if (formArr[i] !== formArr[i - 1]) {
      newArr.push(formArr[i])
    }
  }
  return newArr
}
```

### 利用对象属性存在的特性，如果没有该属性则存入新数组

```javascript
function unique3(arr) {
  var obj = {}
  var newArr = []
  for (let i = 0; i < arr.length; i++) {
    if (!obj[arr[i]]) {
      obj[arr[i]] = 1
      newArr.push(arr[i])
    }
  }
  return newArr
}
```

### 利用数组原型对象上的 includes 方法

```javascript
function unique4(arr) {
  var newArr = []
  for (var i = 0; i < arr.length; i++) {
    if (!newArr.includes(arr[i])) {
      newArr.push(arr[i])
    }
  }
  return newArr
}
```

### 利用数组原型对象上的 filter 和 includes 方法

```javascript
function unique5(arr) {
  var newArr = []
  newArr = arr.filter(function (item) {
    return newArr.includes(item) ? '' : newArr.push(item)
  })
  return newArr
}
```

### 利用 ES6 的 set 方法

```javascript
function unique(arr) {
  return Array.from(new Set(arr)) // 利用Array.from将Set结构转换成数组
}
```

## 根据属性去重

```javascript
function unique6(arr) {
  const res = new Map()
  return arr.filter((item) => !res.has(item.id) && res.set(item.id, 1))
}

console.log(unique6([{id: 1, name: 'aa'}, {id: 1, name: 'bb'}]))

function unique7(arr) {
  let result = []
  let obj = {}
  for (var i = 0; i < arr.length; i++) {
    if (!obj[arr[i].id]) {
      result.push(arr[i])
      obj[arr[i].id] = true
    }
  }
  return result;
}
```

## 取数组的交集/并集/差集

### includes 方法结合 filter 方法

```javascript
let a = [1, 2, 3]
let b = [2, 4, 5]

// 并集
let union = a.concat(b.filter((v) => !a.includes(v)))
// [1,2,3,4,5]

// 交集
let intersection = a.filter((v) => b.includes(v))
// [2]

// 差集
let difference = a.concat(b).filter((v) => !a.includes(v) || !b.includes(v))
// [1,3,4,5]
```

### ES6 的 Set 数据结构

```javascript
let a = new Set([1, 2, 3])
let b = new Set([2, 4, 5])

// 并集
let union = new Set([...a, ...b])
// Set {1, 2, 3, 4,5}

// 交集
let intersect = new Set([...a].filter((x) => b.has(x)))
// set {2}

// 差集
let difference = new Set([...a, ...b].filter((x) => !b.has(x) || !a.has(x)))
// Set {1, 3, 4 , 5}
```

## 数组求和

### for 循环

```javascript
function sum(arr) {
  var s = 0
  for (var i = arr.length - 1; i >= 0; i--) {
    s += arr[i]
  }
  return s
}
```

### 递归方法

```javascript
function sum2(arr) {
  var len = arr.length
  if (len == 0) {
    return 0
  } else if (len == 1) {
    return arr[0]
  } else {
    return arr[0] + sum(arr.slice(1))
  }
}
```

### ES6 的 reduce 方法

```javascript
function sum(arr) {
  return arr.reduce(function (prev, curr) {
    return prev + curr
  }, 0)
}
```

## 类数组转化

```javascript
let arr = Array.prototype.slice.call(arguments)

let arr = Array.from(arguments)

let arr = [...arguments]
```