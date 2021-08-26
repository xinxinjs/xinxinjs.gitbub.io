---
title: window.print打印设置水印和页头页底
date: 2020-05-08 16:39:46
tags: [技术方案]
---

需求：点击打印按钮，打印出两个不同的文件，第一个文件是普通的表格，无特殊要求，第二个是打印一份录用员工的offer，打印offer需要每一页都有水印、带有公司logo的页头和带有公司网站的页底

每一页的带有公司logo的页头页底，可以使用样式 `position: fixed;`设置为position: fixed;样式属性的元素，可以在打印的时候每一页纸上都显示在固定位置，样式如下：

```css
.header {
  position: fixed;
  top: 0;
  left: 0;
}

.footer {
  position: fixed;
  bottom: 0;
  left: 0;
}
```

正文部分设置适当的paddingTop值，使用 `pageBreakAfter: 'always'`在适当的位置强制分页，效果：

![效果](/img/print/test1.jpeg)

水印部分也要每个页面都有，同样思路使用 `position: fixed;`定位元素，正文部分要浮在水印的上面，使用`z-index`让正文部分的值高于水印部分的值：

```css
.text {
  padding-top: 60px;
  position: relative;
  z-index: 2;
}

.watermark {
  padding: 24px;
  margin-top: -24px;
  margin-left: -24px;
  position: fixed;
  z-index: -1;
  font-size: 30px;
  top: 50%;
  left: 50%;
  z-index: 1;
  transform: scale(10) rotate(-45deg);
  color: #fffffe;
  opacity: 0.3;
}
```

![效果](/img/print/test2.png)

碰到另外一个问题：点击一个按钮同时打印出来的除了offer还有另外一个审批的表格table，这个表格不需要页头页底和水印。

解决方案：设置一个足够高的白色蒙版，`absolute`相对于offer定位，遮住不需要的水印和页头页底

absolute定位的元素不会在每一页都显示，但是会根据相对的父级元素定位，可以用来遮住不想显示的内容

```jsx
<div
  style={{
    position: 'absolute',
    top: -1199,
    left: 0,
    width: '100%',
    height: 1200,
    zIndex: 1,
    background: '#fff',
  }}
/>
```
