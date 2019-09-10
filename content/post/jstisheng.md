---
title:      "JavaScript中的声明提升"
date: 2018-06-01T01:37:56+08:00
lastmod: 2018-06-01T01:37:56+08:00
author:     "Adrian"
tags: ["JavaScript"]
categories: ["Coding"]
draft: false
---

**提升的定义**：所有的声明（包括变量和函数声明）都会被“移动”到各自作用域的最顶端。

先看一个例子：

```javascript
a = 2;
var a;
console.log(a);
```

输出结果为2。其原因在于：

**重点1：声明会被提升（声明在编译阶段进行），赋值或其他运算操作会留在原地等待执行阶段**

因此，上述代码的实际处理顺序为：

```javascript
var a; // <--声明提升
a = 2;
console.log(a); // <--输出2
```

另一个例子：

```javascript
//原始代码
console.log(a);
var a = 2;

//实际处理顺序
var a; // <--声明提升
console.log(a); // <--输出undifined
a = 2;
```

**重点2：每个作用域都会进行提升操作**

```JavaScript
//原始代码
foo();
function foo(){
	console.log(a); // <--undifined
	var a = 2;
}

//实际处理顺序
function foo(){ // <--函数声明提升
	var a;        // <--函数内变量声明提升
	console.log(a); // <--undifined
	a = 2;
}
foo();
```

**重点3：函数声明会被提升，函数表达式不会被提升**

```js
//原始代码
foo(); // <--TypeError而非ReferenceError
var foo = function bar(){ ... }

//实际处理顺序
var foo; // <--声明提升，但由于是表达式而非函数声明，未赋值
foo(); // <--TypeError, foo is not a function
foo = function(){  // <--函数表达式赋值
	var bar = ...
} 
```

```js
//原始代码2
foo(); // TypeError
bar(); // ReferenceError
var foo = function bar(){... } // <--具名函数表达式，仍是函数表达式而非函数声明

//实际处理顺序2
var foo; // 声明提升
foo(); // TypeError
bar(); // ReferenceError
foo = function(){
	var bar = ...
}
```

**重点4：函数声明先被提升，变量声明后被提升**

```js
//原始代码
foo(); // <--输出1
var foo();
function foo(){
	console.log(1);
}
foo = function(){
	console.log(2);
}

//实际处理顺序
function foo(){  //函数声明提升在变量声明之前
	console.log(1);
}
// 原本在此处的变量声明var foo()由于是重复声明，被忽略
foo(); // <--输出1
foo = function(){
	console.log(2);
}
```



Reference: [You Don't Know JS](https://github.com/getify/You-Dont-Know-JS)