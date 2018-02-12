---
title: NodeList的非数组性和有时实时性
date: 2017-05-25 15:54:23
tags:
- DOM
- NodeList
---
最近在重新复习以前学过的课程，大部分是基础的东西。

《JavaScript DOM 编程艺术》是一本经典的JavaScript入门书。去年年初怀着无线的憧憬刷过一遍，奈何后来买新房装修，耽误了大半年的时间，旧的知识不知不觉就忘了。

今年重整旗鼓，本来想直接看新书的，但做一些小项目经常出现基础不牢的情况。于是重走长征路，看过的书再刷一遍，做过的项目，用新技术重构一遍。

过去的一年node.js发展迅猛，去年哈尔滨应用这个技术的公司还相对较少，大部分都是PHP。今年以来，发现有了一些改变。加油吧，年轻人们。

闲言够了，接下来写写最近重新复习《JavaScript DOM 编程艺术》时重新归纳的一个小知识点吧，真的是基础中的基础。

<!--more-->
---

### 一、非数组特性
以下内容引用[MDN](https://developer.mozilla.org/zh-CN/)，这也是关于原生JavaScript最好的官方文档，没有之一。

> 为什么 NodeList 不是数组？
> 
> NodeList 对象在某些方面和数组非常相似，看上去可以直接使用从 Array.prototype 上继承的方法。然而，NodeList 没有这些类似数组的方法。
> 
> JavaScript 的继承机制是基于原型的。数组元素之所以有一些数组方法（比如 forEach 和 map），是因为它的原型链上有这些方法，如下:
> 
> myArray --> Array.prototype --> Object.prototype --> null (想要获取一个对象的原型链，可以连续的调用 Object.getPrototypeOf，直到原型链尽头).
> 
> forEach, map这些方式其实是 Array.prototype 这个对象的方法。
> 
> 和数组不一样，NodeList的原型链是这样的：
> 
> myNodeList --> NodeList.prototype --> Object.prototype --> null
> 
> NodeList.prototype 只有一个 item 方法，没有 Array.prototype 上的那些方法，所以 NodeList 对象用不了它们。

NodeList不是数组，这出乎我的意料。所以一定要记住，forEach、filter、map和reduce等方法就都不能用了，实际写代码要记住这个区别。

当然可以通过原型链的方式扩展forEach，不过官方不推荐。

1. 扩展DOM对象的原型

```javascript
var arrayMethods = Object.getOwnPropertyNames( Array.prototype );

arrayMethods.forEach( attachArrayMethodsToNodeList );

function attachArrayMethodsToNodeList(methodName)
{
	if(methodName !== "length") {
	NodeList.prototype[methodName] = Array.prototype[methodName];
	}
};

var divs = document.getElementsByTagName( 'div' );
var firstDiv = divs[ 0 ];

firstDiv.childNodes.forEach(function( divChild ){
	divChild.parentNode.style.color = '#0F0';
});
```
2. 不扩展DOM对象的原型

```javascript
var forEach = Array.prototype.forEach;

var divs = document.getElementsByTagName( 'div' );
var firstDiv = divs[ 0 ];

forEach.call(firstDiv.childNodes, function( divChild ){
	divChild.parentNode.style.color = '#0F0';
});
```

### 二、有时实时

大多数情况下，NodeList 对象都是个实时集合。意思是说，如果文档中的节点树发生变化，则已经存在的 NodeList 对象也可能会变化。例如，Node.childNodes 是实时的：

```javascript
var parent = document.getElementById('parent');
var child_nodes = parent.childNodes;
console.log(child_nodes.length); // 如果假设结果是“2”
parent.appendChild(document.createElement('div'));
console.log(child_nodes.length); // 此时的输出是“3”
```

在另一些情况下，NodeList 是一个静态集合，也就意味着随后对文档对象模型的任何改动都不会影响集合的内容。document.querySelectorAll 返回一个静态的 NodeList。

特别是当你选择如何遍历 NodeList 中所有项，或缓存列表长度的时候，最好牢记这种区分。

[原文链接](https://developer.mozilla.org/zh-CN/docs/Web/API/NodeList)
