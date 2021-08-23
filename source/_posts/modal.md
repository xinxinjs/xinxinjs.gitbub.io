---
title: 模态框的最佳实践
date: 2020-10-27 11:31:09
tags: 用户体验
categories: "产品"
---

原文：[模态框的最佳实践](http://www.jerrychane.com/user-experience/3.html)

## 引言

为什么选择这篇文章呢？

1. 前端工程师的定位不仅仅在于讨论如何解决架构层面的问题，更应该重视交互体验

2. 前端工程师从未停止过对用户体验的追求，如何思考模态框在产品中的存在和使用，存在许多争议；

## 知识普及

### 定义

维基百科上对模态框的定义如下：模态框是一个定位于应用视窗顶层的元素。它创造了一种模式让自身保持在一个最外层的子视窗下显示，并让主视窗失效。用户必须通过在它上面做交互动作，才可以回到主视窗。

### 用途

- 抓住用户的吸引力；

- 需要用户输入内容；

- 在上下文下显示额外的信息；

- 不在上下文下显示额外的信息；

### 组成

- 退出的方式，可以是模态框上的一个按钮，键盘上的一个按键，也可以是模态框外的区域；

- 描述性标题，标题是给用户标识在哪个位置进行操作的上下文信息；

- 按钮的内容，它一定是用户可以理解，并促使用户进行下一步行动的内容，不会产生迷惑；

- 大小与位置，模态框不要太大，也不要太小，位置建议在视窗中间偏上(太低在移动端显示不全)；

- 焦点的切换，模态框的出现会吸引用户的注意力，建议键盘焦点切换到框内；

- 需用户发起，通过用户的动作，如点击按钮促使模态框出现，不然会惊吓到用户；

### 移动端

模态框由于太大，占用太多控件，在移动端很难适配。通常建议增加设备按键或内置滚动条来操作，用户可以滑动或放大缩小来操作模态框。

### 无障碍访问

- 快捷键 - 考虑在打开、移动、管理焦点和关闭时增加对模态框的快捷键；

- ARIA - 在前端代码层面加上 aria 的标识，如 Role = “dailog”,aria-hidden,aria-label;

## 思考与实践

### 定位思考

Modal 与 Toast、Notification、Message 以及 Popover 都会在某个时间点被触发并弹出一个浮层。但从定义上来看上述组件都不属于模态框，因为模态框始终会阻塞原来主视窗下的操作，只能在框内做后续操作。模态框的出现从界面上彻底打断了用户的心流。如果是一般的消息提醒，可以用信息条、小红点等交互形式，至少不会阻塞用户的操作。

那么模态框有什么优点吗？要知道模态框的体验要比页面跳转更加轻量，更让用户感觉舒适。例如，用户在电商网站看中一款产品，想登陆购买，此时弹出模态框的体验就远远好于跳转到登录页面，因为用户在模态框中登录后，就可以直接购买了。

也就是说，当我们设计好模态框出现的时机，流畅的弹出体验，必要的上下文信息，以及友好的退出反馈，足以提升体验。模态框的目的在于吸引用户的注意，而且一定要提供能够在本页面即可完成额外的流程操作或是信息告知，可以是一个重要的操作，也可以是一份重要的协议。

### 合理使用

-内容是否相关。模态框作为当前页面的一种衍生或补充，若其内容与当前页面无关联，可以使用其他操作代替模态框，例如页面跳转。

-避免过多操作。模态框应该给用户留下一种看完即走，潇洒自如的感觉，而非繁杂的交互留住或牵制用户。

-避免多模态框。出现多个模态框，会加深产品垂直深度，提高视觉复杂度，而且让用户感到烦躁。

-用户主动触发。不要突然打开或自动打开模态框，会惊吓到用户。

-大小尺寸适中。模态框大小要严格限制，避免内容过多出现滚动条的情况，除非需展示明细内容。

### 代码实现

错误示例