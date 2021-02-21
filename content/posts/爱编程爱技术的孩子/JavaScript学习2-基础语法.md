---
title: JavaScript学习2-基础语法
date: 2020-05-22T17:07:00+08:00
categories: [爱编程爱技术的孩子]
slug: JavaScript learning 2 basic syntax
---

各种语言的基础语法部分都很相似，因此这里简单总结一下

## 1. 变量与常量

尽管以前使用 `var` 关键字，但现在更多使用 `let` 关键字声明变量

```js
let myName; 			// 声明
myName = 'shuzang'; 	// 初始化
let nyName = 'shuzang'; // 声明 + 初始化，这是最常使用的方式
myName = 'newName';     // 声明后更新变量值
let a=10,b=20,c=30;  	// 同时声明多个变量
```

注：关键字更换的原因参考 [var 与 let 的区别](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/First_steps/Variables#var_与_let_的区别)，更换只有好处没有坏处

常量则使用 `const` 关键字

```js
const button = document.querySelector('button');
```

变量与常量的命名规则与其它语言相同，由字母、数字、下划线、美元符组成，不能以数字开头。

## 2. 数据类型

JS 的数据类型有

- **值类型(基本类型)**：字符串（String）、数字(Number)、布尔(Boolean)、对空（Null）、未定义（Undefined）、Symbol。
- **引用数据类型**：对象(Object)、数组(Array)、函数(Function)。

注：Symbol 是 ES6 引入了一种新的原始数据类型，表示独一无二的值。

```js
let hello = 'hello js!'; 		// 字符串，单引号和双引号都可以，但应保持使用一种方式
let myAge = 17;			 		// 数字,整数和浮点数都是数字类型
let iAmAlive = true;     		// 布尔
let myNumberArray = [10,15,40]; // 数组
let dog = { name : 'Spot', breed : 'Dalmatian' }; //对象
```

JavaScript是一种「动态类型语言」，这意味着不需要指定变量将包含什么数据类型（例如number或string）

### 2.1 字符串

格式转换

```js
let myString = '123';
let myNum = Number(myString); // 字符串转换为数字
myString = myNum.toString();  // 数字转换回字符串
```

一些方法如下

```js
let browserType = 'mozilla';
browserType.length;  				// 获取字符串长度
browserType[0];						// 看作字符数组，获取第一个字符
browserType[browserType.length-1];  // 获取最后一个字符
browserType.indexOf('zilla');   	// 搜索子串，返回索引2
browserType.indexOf('vanilla'); 	// 返回-1
browserType.slice(0,3); 			// 提取索引 0-2 的子串，slice方法类似Go的切片
browserType.slice(2);   			// 提取索引2开始直到字符串结束的子串
browserType = browserType.replace('moz','van'); // 子串替换
```

大小写转换

```js
let radData = 'My NaMe Is MuD';
radData.toLowerCase();
radData.toUpperCase();
```

### 2.2 数组

JS 中数组元素可以是同一种类型，也可以混合

```js
let sequence = [1, 1, 2, 3, 5, 8, 13];
let random = ['tree', 795, [0, 1, 2]];
```

可以获取长度

```js
let sequence = [1, 1, 2, 3, 5, 8, 13];
sequence.length;   	 // 返回 7
sequence.push(11); 	 // 在末尾添加元素 11
var newLenght = sequence.push(21,22) // 一次可添加多个元素，返回值是新数组的长度, 可以不声明返回值
var removedItem = sequence.pop();    // 删除最后一个元素, 返回值是删除的元素
sequence.unshift(34);  // 在开头添加元素 34
sequence.shift();      // 删除开头的元素

```

尽管字符串可以看作字符数组，但也可以指定分隔符

```js
let myData = 'Manchester,London,Liverpool';
let myArray = myData.split(','); // myArray = ['Manchester','London','Liverpool']
myArray.lenght; // 返回 3
myArray[0];     // 返回 'Manchester'
let myNewString = myArray.join(','); // 相反的操作，数组转换成字符串
myArray.toString(); // 另外一种数组转换为字符串的方法，更简单，但无法指定分隔符
```

## 3. 运算符

- 算术运算符：+，-，*，/，%，**（幂）
- 递增递减运算符：++，--
- 赋值运算符：=，+=，-=，*=，/=
- 比较运算符：`===`，`!==`，<，>，<=，>=

`==` 和 `!=` 同样有效，但它们只测试值是否相等，数据类型可能不同，`===` 和 `!==` 严格测试值和数据类型是否相同，严格的版本出现的错误更少，因此推荐使用这种方式。

## 4. 控制结构

**条件结构**与其它语言相同

- `if...else` 或者 `if...else if...else`
- `switch...case...default`，需要主动执行 `break` 跳出

- 支持三元运算符：`( condition ) ? run this code : run this code instead`

**循环结构**包含了我们接触过的所有写法

- `for`
- `do...while`
- `while`
- `for...in`，返回索引
- `for...of`，返回值

最后，可以使用 `label` 来随意跳转，但没有想到任何这样做的好处

## 5. 函数

使用 `function` 关键字定义，格式如下

```js
func 函数名(形参列表){
    ...
}
```

这里注意到的是 JS 中无需声明返回值列表，直接在函数体中使用 `return` 返回相应的值即可，参数也无需声明类型

JS 支持匿名函数（也称为函数表达式）

```js
function() {
  alert('hello');
}
```

但匿名函数主要用来相应事件触发

```js
myButton.onclick = function() {
  alert('hello');
  // I can put as much code
  // inside here as I want
}
```

具名函数和匿名函数都可以赋值给变量

```js
var square = function(number) { return number * number; };
var x = square(4); // x gets the value 16

var factorial = function fac(n) {return n<2 ? 1 : n*fac(n-1)};
console.log(factorial(3));
```

