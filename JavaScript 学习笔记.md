# JavaScript 学习笔记

## 下面函数输出什么结果？

```js
['1', '2', '3'].map(parseInt);
===> [1, NaN, NaN]
```

#### 知识点一 Array.map() 原型

```js
Array.map(callback, thisArg);
- callback
  - item: 执行 callback 函数正在处理的元素
  - index: 执行 callback 函数正在处理的元素索引
  - arr: map方法调用的数组自身
- thisArg: 执行 callback 函数时的 this 指向
```

#### 知识点二 parseInt 原型

```js
parseInt(string, radix);

- string: 要被解析的值，如果不是 string 会调用 toString 函数转换成 string

- radix: 从 2 ～ 36 表示进位系统的数字
```

`parseInt` 的返回值只有两种结果：转换成功的数字或转换失败的 `NaN`。

#### 知识点三 如何才能实现想要的效果

- 基础版

```js
['1', '2', '3'].map((item) => parseInt(item, 10));
```

> 缺点，命令式编程，不能复用，每一次这种类似的都要重新书写一次，不符合函数式编程特点。

- 进阶版

```js
function unary = fn => fn.length === 1 ? fn : (arg) => fn(arg);
['1', '2', '3'].map(unary(parseInt));
```

> 封装 `uary` 函数，此函数的作用是传入一个函数作为参数，并强制将此函数转化为接受一个参数的函数。

##### 相关知识点

参数是函数 fn，可以通过 fn.length 获取到此函数接受的参数格数（也就是内部的 `arguments.length`）

## 实现一个 Once 函数

```js
Function.prototype.once = function(fn) {
    let done = false;
    return function() {
      return done ? undefined : (done = true, fn.apply(this, arguments))
    }
}

const doAddNum = once((a, b) => console.log(a + b));
doAddNum(1, 2); // 3
doAddNum(1, 2); // undefined，只会被执行一次
```

## 实现一个 Memorized 函数

### 背景

给定数字计算一个递归斐波那契数列

```js
var factorial = (n) => {
	if (n === 0) return 1;
 	 return n * factorial(n - 1);
} 
```

> 【存在问题】： 每一个数字对应的结果其实是固定的，但是因为是存函数，所以即使你连续输入10次相同数字，也是会计算十次。这样其实是浪费性能的（也算是纯函数的一个小弊端）。

### 解决方案 — 高阶函数

```js
/* 单一参数 */
const memorized = (fn) => {
  const memoDatabase = {};
  return (arg) => memoDatabase[arg] || (memoDatabase[arg] = fn(arg));
}
// 使用
const fastFactorial = memorized(factorial);
fastFactorial(6); // 第一次计算，存入对象
fastFactorial(6); // 第二次直接从对象里取 o(1)

/* 多参数 */
 const memorized = (fn) => {
    const memoDatabase = {};
    // 此处形式是关键 
    return function() {
      console.log(memoDatabase);
      const key = Array.from(arguments).join('/');
      return memoDatabase[key] || (memoDatabase[key] = fn.apply(this, arguments))
    };
  }
```

> 【关键点】：写的时候发现，原来箭头函数并没有 arguments 这个内置属性，所以多参数的内部函数要通过原始定义的形式进行返回。

### 实现一个 filter 函数

> 考查重点，filter 函数返回一个满足一定条件的新数组。

```js
const myFilter = (arr, fn) => {
  let result = [];
  for (const val of arr) {
    fn(val) ? result.push(val) : undefined;
  }
  return result;
}
```



## 数组相关

### Array.from

> ```js
> Array.from(arrayLike[, mapFn[, thisArg]])
> ```

- `arrayLike`

  想要转换成数组的伪数组对象或可迭代对象。

- `mapFn` 可选

  如果指定了该参数，新数组中的每个元素会执行该回调函数。

- `thisArg` 可选

  可选参数，执行回调函数 `mapFn` 时 `this` 对象。

```js
// 应用一：伪数组转换成数组
const set = new Set(['foo', 'bar', 'baz', 'foo']);
Array.from(set);
// [ "foo", "bar", "baz" ]

// 应用二：改造数组，返回一个新的但是是浅拷贝的数组
Array.from([1, 2, 3], x => x + x);
// [2, 4, 6]

// 应用三：可以用来构造数组，Mock简单数据的时候十分好用
Array.from({length: 5}, (v, i) => i);
// [0, 1, 2, 3, 4]

// 模拟的数据
const getItems = count =>
  Array.from({ length: count }, (v, i) => i)
	// 这里返回的是长度为 count 的数组 [0, 1, 2 ... count - 1]
		.map(k => ({
      id: `item-${k}`,
      content: `item ${k}`,
    }));
```

#### 伪数组定义

- 第一，可迭代的（可以通过下标索引到的）
- 第二，必须有 `length` 属性

### 数组求和

> 考查知识点： Array.reduce()

#### 基本实现

```js
Function.prototype.sumTotal = function (arr) {
  let sum = 0;
  for (let val of arr) {
    sum += val;
  }
  return sum;
}

const sumTotal = (arr) => {
  let sum = 0;
  arr.forEach(val => sum += val);
  return sum;
}

```



#### reduce 函数

> 【原理】：Array.reduce(callback(accumulator, currentValue, currentIndex, arr), initialValue)
>
> - callback
>   	- accumulator: 累计器累计回调的返回值; 它是上一次调用回调时返回的累积值，或`initialValue`，如果不提供 `initialValue` 则 accumulator 初识值就是 arr[0]。
>   	- currentValue: 当前值，数组中正在处理的元素。
>   	- currentIndex(可选): 当前索引，如果提供了`initialValue`，则起始索引号为0，否则从索引1起始。
>   	- arr(可选)：源数组。
> - initialValue: 初始值，作为第一次调用 `callback` 函数时的第一个参数的值。 如果没有提供初始值，则将使用数组中的第一个元素，**在没有初始值的空数组上调用 reduce 将报错**。

##### 最简单的用法，数组求和

```js
// 普通的用法，无法封装
let sum = 0;
arr.forEach(item => {
  sum += item;
});
console.log(sum);

// reduce 实现数组求和

const total = arr.reduce((accumulator, curVal) => accumulator + curVal);
```



##### 升级用法一：数组乘积

```js
// 数组乘积，初始值设置为1
const total = arr.reduce((acc, curVal) => acc * curVal, 1);
```

##### 升级用法二：合并二维数组

```js
const arr = [[1, 2], [3, 4, 5], [6, 7, 8, 9]];
arr.reduce((acc, curVal) => {
  return acc.concat(curVal);
}, []);
// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

##### 升级用法三：统计数组中出现单词的个数

```js
const arr = ["apple","orange","apple","orange","pear","orange"];
arr.reduce((acc, curVal) => {
  // 初始化空 obj，如果存在 + 1，不存在赋值为 1
  acc[curVal] = (acc[curVal] + 1) || 1;
  return acc;
}, {});
// {apple: 2, orange: 3, pear: 1}
```

##### 实现 reduce

```js
// 简易版实现数组累加的 reduce，不带初始值
const simpleReduce = (arr, fn) => {
  let accumulator = 0;
  for (const val of arr) {
    accumulator = fn(accumulator, val);
  }
  return accumulator;
}
const arr = [1, 2, 3, 4];
simpleReduce(arr, (acc, val) => acc + val);
// 上面简易版没办法计算数组乘积，因为初始化 accumulator 参数是0，所以永远是0

// 完整版

const fullReduce = (arr, fn, initialValue) => {
  // 初始化 accumulator，如果初始值存在，则为初始值，否则赋值为数组第一个元素
  let accumulator = initialValue !== undefined
  	? initialValue : arr[0];
  if (initialValue === undefined) {
    // 如果初始值不存在，索引应该从第二个元素开始，因为第一个是初始值了
    for (let i = 1; i < arr.length; i++) {
      accumulator = fn(accumulator, arr[i]);
    }
  } else {
    // 如果初始值存在，索引从第一个元素开始
    for (const val of arr) {
      accumulator = fn(accumulator, val);
    }
  }
  return accumulator;
}

```



## 函数柯里化

#### 定义

函数柯里化就是指把一个**多参数函数**转换为一个**嵌套的一元函数**的过程。

```js
// 一个最简单的柯里化的例子
const add = (x, y) => x + y;

// 柯里化以后
const curryAdd = x => y => x + y;

add(4, 4); // 8
curryAdd(4)(4); // 4
```

> 其实柯里化也是利用了闭包的特性，内部函数可以访问（保存）外部函数的参数

#### 

#### 实现柯里化

#### 简易版实现 —— 探寻原理

```js
// 	以两个参数为例，探究一下函数柯里化的原理
funtion curry(binaryFn) {
  return function(firstArg) {
    return function(secondArg) {
      return binaryFn(firstArg, secondArg);
    }
  }
}
```

> 从上面可以看出来，其实函数柯里化原理就是将传入的参数不断的以闭包的形式传递到内部返回函数，然后最后再拼接到原来的函数上去执行。上面这个例子是最简单的二次函数，当复杂的时候，参数可以是变化的。比如：`add(1)(2)(3)`和`add(1, 2)(3)`都是可以的，但是实现原理是一样的，只不过变参个数可以通过 es6 扩展运算符来处理。

#### 完整实现

~~~js
// 第一步 - 函数定义， curry 函数必须接受的是一个函数
const curryFn = (fn) => {
  if (typeof fn !== 'function') {
    throw Error('No function provided');
  }
}

// 第二步 - curry 函数接收一个函数，返回一个函数，返回函数接收的参数和原函数相同
const curryFn = (fn) => {
  if (typeof fn !== 'function') {
    throw Error('No function provided');
  }
  // 把接收参数传给传入参数执行一次，这里使用 apply
  return function(...args) {
    return fn.apply(null, args);
  }
}

// 测试1 - 三个数的乘积
const threeMulti = (x, y, z) => x * y * z;
curryFn(threeMulti)(1, 2, 3); // 6

// 第三步 - 参数柯里化，将多参数变成1个参数
const curryFn = (fn) => {
  if (typeof fn !== 'function') {
    throw Error('No function provided');
  }
  // 这里返回的函数不能是匿名函数，因为函数内部还会有判断要进行参数柯里化操作，所以要给个名字
  return function curryInnerFn(...args) {
    // 如果当前接收到的参数小于原函数参数，那么进行参数柯里化操作
    if (args.length < fn.length) {
  		return function() {
        // 这里又返回一个匿名函数，必须是 ES5 形式，因为下面合并参数要使用 arguments
        return curryInnerFn.apply(null, args.concat(
        	// 这里是最重要的，如果参数小于原函数 fn 的参数，返回柯里化函数之后说明后续还会继续接受参数，因此，递归继续返回函数直到参数不小于原函数的参数长度再执行，所以，参数需要一直的连接过来，内部函数调用时（会形成闭包）接收的参数 args （闭包所在的外部函数传递进来的）连接即将接收的参数 arguments（内部函数自身调用时传入的）
          [].slice.call(arguments)
        ));
      }
    }
    // 如果柯里化函数接收的参数个数和原函数接收参数个数一致（或大于原函数），直接执行原函数即可
  	return fn.apply(null, args);
  }
}

const add = (x, y, z) => {
    return x + y + z
  }
const curryAdd = curryFn(add);
console.log(curryAdd(1)(2)(3)); // 6
console.log(curryAdd(1, 2)(3)); // 6
~~~



#### 柯里化实战1 - 字符串数组中查找包含数字的内容

```js
const arr = ['number1', 'nofdsf', '32kfjskk'];
// 一般写法，命令式，无法复用
arr.filter(item => /\d+/.test(item));
// => ['number1', '32kfjskk']

// 稍微高级点的用法 - 一次封装
const filterHasNumberArr = (arr) => {
  return arr.filter(item => /\d+/.test(item));
}

// 上面封装的太简单了，并且只能做一种filter，其实还可以更高级一些,把正则表达式封装到参数里，那么想过滤什么就过滤什么就行了
const filterExprArr = (expr, arr) => {
  return arr.filter(item => expr.test(item));
}

// 最高级的用法 —— 柯里化封装
/**
 * 解析需求，其实这个需求就是两个函数的叠加
 * 第一函数 - 正则匹配字符串
 * 第二函数 - 数组filter函数
 * 为了能封装的彻底，达到高复用，可以进行如下设计
 **/

// 第一函数 - 正则匹配字符串
function match(expr, str) {
  // 这里选用的是 str.match 而不是 reg.test
  return str.match(expr);
}
// 第二函数 - 数组过滤 filter
function filter(fn, array) {
  return array.filter(fn);
}

// 柯里化第一函数
const curryMatch = curry(match); 
// 柯里化第二函数
const curryFilter = curry(filter);

// 柯里化的意思上面已经介绍过了，就是可以变参传值，当参数个数小于原函数个数的时候，返回函数继续等待接收参数

// 利用柯里化的match返回一个判断字符串是否包含数字的函数，此函数接收一个字符串作为参数。这条语句的意思就是，利用柯里化函数，接收一个正则表达式作为参数，返回一个接收一个字符串作为参数的函数，函数执行返回正则表达式与字符串match后的结果
const strHasNumber = curryMatch(/\d+/);

// 利用柯里化的filter函数返回一个接收参数是数组的函数,柯里化函数接收一个函数作为参数，传入上面的strHasNumber函数，然后返回一个接收数组的函数
const filterNumberArr = curryFilter(strHasNumber);

/**
 * 执行过程：
 * 第一步：filterNumberArr(arr) 传入数组 arr，此时传入了两个参数，满足了柯里化函    
 * 数 curryFilter 的原函数 filter 的两个参数，开始执行，执行过程是
 * arr.filter(fn) => fn 是上面传递过来的 strHasNumber
 * 第二步：strHasNumber是一个函数，它接收的是一个字符串，执行的是 
 * str.match(expr)，因为是在filter里调用的，因此执行的就是arr.filter(item => 
 * item.match(expr)) => 前面柯里化的时候 expr = /\d+/
 **/
const resArr = filterNumberArr(arr);

// 好处，拆分成了两个柯里化函数，可以随意搭配正则进行各种验证的封装，并且使用灵活
```

> 总结：函数柯里化的过程看起来挺复杂，其实总结下来就是一句话，不论调用了几次，只要参数个数小于封装的原函数参数个数，那么就一直返回函数，返回的函数接收剩余参数，当参数个数等于大于原函数参数个数的时候，将原来接收到的参数进行拼接并执行返回结果，执行过程与原函数没有区别。这也是利用了闭包的特性，可以不断的保存外层函数的参数。

#### 相关面试题

#####  设计一个函数，依次传入姓名，先传入姓氏再传入名称，当传入完姓氏名称后再输出出来。

```js
// 第三步 - 参数柯里化，将多参数变成1个参数
const curry = (fn) => {
  if (typeof fn !== 'function') {
    throw Error('No function provided');
  }
  // 这里返回的函数不能是匿名函数，因为函数内部还会有判断要进行参数柯里化操作，所以要给个名字
  return function curryInnerFn(...args) {
    // 如果当前接收到的参数小于原函数参数，那么进行参数柯里化操作
    if (args.length < fn.length) {
  		return function() {
        // 这里又返回一个匿名函数，必须是 ES5 形式，因为下面合并参数要使用 arguments
        return curryInnerFn.apply(null, args.concat(
        	// 这里是最重要的，如果参数小于原函数 fn 的参数，返回柯里化函数之后说明后续还会继续接受参数，因此，递归继续返回函数直到参数不小于原函数的参数长度再执行，所以，参数需要一直的连接过来，内部函数调用时（会形成闭包）接收的参数 args （闭包所在的外部函数传递进来的）连接即将接收的参数 arguments（内部函数自身调用时传入的）
          [].slice.call(arguments)
        ));
      }
    }
    // 如果柯里化函数接收的参数个数和原函数接收参数个数一致（或大于原函数），直接执行原函数即可
  	return fn.apply(null, args);
  }
}

function sayFullName(firstName, secondName) {
  console.log(firstName, secondName);
}
function _sayName = curry(sayFullName);
_sayName('luffy');
_sayName('zhou');
```



### 偏函数 — Partial

函数柯里化 curry 可以封装函数参数，顺序是从左至右；而偏函数正好相反，可以帮助我们对于那些固定参数在后面（结尾）的函数进行封装的函数。

#### 示例一：封装 setTimeout 函数

都知道，setTimeout 函数的第二个参数是时间，如果有这么一组操作，所有的操作都固定时间间隔 1s，那么就是下面这种代码：

```js
const timer1 = setTimeout(() => console.log('task 1'), 1000);
const timer2 = setTimeout(() => console.log('task 2'), 1000);
const timer3 = setTimeout(() => console.log('task 3'), 1000);
```

从上面可以看到，其实每一个任务都是相同的第二个参数 1000，从代码角度来说其实是可以进行封装的，但是柯里化封装是参数从左至右，而时间间隔在第二个，没办法使用柯里化，其实也是能用柯里化的。

##### 柯里化封装此场景

```js
// 其实就是额外封装一下，然后把参数掉一个位置
function setTimoutWrapper(time, fn) {
  setTimeout(fn, time);
}

const delay1s = curry(setTimeoutWrapper)(10);
// 应用
delay1s(() => console.log('task 1'));
delay1s(() => console.log('task 2'));
delay1s(() => console.log('task 3'));
```

##### 偏函数

上面的柯里化封装并没有什么问题，但是就额外多了一个函数，增加了开销，并且函数复杂了以后就会麻烦许多。

```js
// 偏函数
function partial(fn, ...partialArgs) {
  // 将偏函数除了第一个参数 fn 赋值给 args
  let args = partialArgs;
  // 返回函数，该函数也接受一串参数
  return function(...fullArgs) {
    let arg = 0;
     // 遍历两个参数列表，分别是外部封装函数的参数列表以及内部调用函数的参数列表，外部封装函数的参数列表存在 undefined，undefined 的意义表示，遇到 undefined 了就要进行参数的替换，undefined 就是封装固定参数的占位符，替换的内容就是内部调用函数的参数。
    for (let i = 0; i < args.length && arg < fullArgs.length; i++) {
      if (args[i] === undefined) {
        args[i] = fullArgs[arg++];
      }
    }
    return fn.apply(null, args);
  }
}

const delay1s = partial(setTimeout, undefined, 10);
delay1s(() => console.log('task 1'));
delay1s(() => console.log('task 2'));
delay1s(() => console.log('task 3'));
```

这么实现看起来好像很复杂，还是需要一个实例来一步一步看看：

```js
// 第一步：调用函数
partial(setTimeout, undefined, 10);
// 注意，偏函数与柯里化函数返回的都是一个函数，柯里化函数接收一个参数 fn，偏函数接收的是多个参数，但是第一个参数是fn，后面的通过扩展运算符传递。

// 第二步：args = [undefined, 10]; 第一个是fn， args 是 fn 后面的参数列表

// 第三步：返回函数 delay1s(...fullArgs)
delay1s(() => console.log('task 1'));
// 此时，fullArgs = [() => console.log('task 1')]，也就是函数是第一个参数

// 第四步：循环判断
// args[0] === undefined，此时，把 fullArgs 值依次赋值给 args。因为是从 0 开始，所以 args 变成了 args[0] = fullArgs[0] = () => console.log('task 1')，因此 args = [() => console.log('task 1'), 10]

// 第五步：执行代码
fn.apply(null, args); => 此时，fn 也就是 setTimeout，调用参数列表就变成了。
setTimeout(innerFn, 10); => 也就是
setTimeout(() => console.log('task 1'), 10);
```

此时，就实现了偏函数封装后位参数的场景，封装好的函数调用时只需要传前面的函数参数即可。

##### 应用一： 封装 parseInt

众所周知，parseInt接收两个参数，第一个是要转换的字符串，第二个是转换系数，一般来说我们都会选择默认的十进制。（第二个参数必须传，因为如果不传在某些场景下会出现bug），所以就可以封装一个默认十进制安全的parseInt。

```js
window.tenParseInt = partial(parseInt, undefined, 10);
// 或者
window.tenParseInt = function (str) {
  return partial(parseInt, undefined, 10)(str);
}
```

##### 应用二：封装多固定参数

`JSON.stringify(str, replacer, tab)`其实可以接三个参数，第一个是要转换的字符串，第二个可以是一个函数或者数组，第三个是转换后添加格式（空格或者换行符）。

```js
const obj = { name: 'luffyzh', email: 'luffhzh@163.com' };
JSON.stringify(obj); // => "{"name":"luffyzh","email":"luffhzh@163.com"}"
//使用 4 个空格缩进转换后的字符串
JSON.stringify(obj, null, 4);
// 转换后的字符串长下面这个样子
"{
    "name": "luffyzh",
    "email": "luffhzh@163.com"
}"
```

那么，也就是说，其实后面两个参数都是固定的，因此可以使用偏函数封装一个方便使用的简易版4个空格缩进的函数。

```js
const tab4Stringify = partial(JSON.stringify, undefined, null, 4);
// 发现问题
const obj = { name: 'luffyzh', email: 'luffyzh@163.com' };
tab4Stringify(obj);
"{
    "name": "luffyzh",
    "email": "luffhzh@163.com"
}"
const obj2 = { name: 'naruto', email: 'naruto@163.com' };
tab4Stringify(obj2);
"{
    "name": "luffyzh",
    "email": "luffhzh@163.com"
}"
```

从上面发现了问题，两次输入的都是相同的内容，这是为什么呢？仔细看一下代码其实不难发现， `partial`函数是一个闭包函数，内部函数可以访问外部函数的参数和变量，那么外部函数的第一行代码是`let args = partialArgs;`，这个 args 是数组也就是引用类型。

```js
const obj = { name: 'luffyzh', email: 'luffyzh@163.com' };
// 第一次调用
tab4Stringify(obj);
const obj2 = { name: 'naruto', email: 'naruto@163.com' };
// 第二次调用
tab4Stringify(obj2);
```

第一次调用时，`args = [undefined, null, 4]`，因为 args 是引用类型，并且代码里并没有销毁，所以第一次调用执行完成后，args 变成了 `[obj, null, 4]`，没有看错，因为调用时我们将调用参数赋值给了 `args[0]`，第二次调用的时候，因为没有占位符也就是`undefined`了，所以 args 依旧等于 `[obj, null, 4]`，所以打印的仍然是第一次调用的结果。

##### 第一次改进

从上面分析得知，第一次占位符也就是`undefined`可以正常被调用，调用成功之后此占位符就被赋值为第一次所传的参数。之后此占位符也就没有了用处，所以，解决办法就是每次调用内部函数的时候把此占位符释放出来。所以改进的代码如下：

```js
// 第一次改进
function partial(fn, ...partialArgs) {
  // 将偏函数除了第一个参数 fn 赋值给 args
  let args = partialArgs;
  // 返回函数，该函数也接受一串参数
  return function(...fullArgs) {
    // 重制占位符，因为占位符位置是第一个，所以重置 args[0] 即可
    args[0] = undefined;
    let arg = 0;
    // 遍历两个参数列表，如果外部函数参数列表遇到了 undefined，此时将内部函数参数列表逐次赋值给外部函数参数列表。undefined 的意义表示，从 undefined 开始要把内部函数的参数也赋值给 args 参数列表
    for (let i = 0; i < args.length && arg < fullArgs.length; i++) {
      if (args[i] === undefined) {
        args[i] = fullArgs[arg++];
      }
    }
    return fn.apply(null, args);
  }
}

const tab4Stringify = partial(JSON.stringify, undefined, null, 4);
const obj = { name: 'luffyzh', email: 'luffyzh@163.com' };
// 第一次调用
tab4Stringify(obj);
"{
    "name": "luffyzh",
    "email": "luffyzh@163.com"
}"
const obj2 = { name: 'naruto', email: 'naruto@163.com' };
// 第二次调用
tab4Stringify(obj2);
"{
    "name": "naruto",
    "email": "naruto@163.com"
}"
```

改进完了，看起来好像是没什么问题，但是这个改进有一个致命的问题，不要忘了，`partial`是一个功能函数，应该适用的是一切场景。偏函数的本质是封装固定参数，使用占位符来替换，那么再考虑一下下面的场景。

```js
// 返回四个参数相加的结果
function add4param(a, b, c, d) {
  return a + b + c + d;
}
// 现在有一个场景，我想固定 a 和 c 的参数为 1 和 3，然后 b d 通过传入的内容相加，这个场景也适合使用偏函数来进行封装
const _add4param = partial(add4param, 1, undefined, 3, undefined);
// 很明显，应该像上面这样封装，但是封装完成后我们来看看效果如何
_add4param(2, 4); // NaN
```

从上面结果可以看出来， `NaN`显然封装错了，那么为啥，其实结果很明显。第一次封装的`JSON.stringify`是接收三个参数，占位符是一个在第一个位置，所以代码修改的是 `args[0] = undefined;`但是这里的场景是接收四个参数，第二个第四个是占位，所以如果修改应该修改成`args[1] = undefined, args[3] = undefined;`前面说了，`partial`是一个通用函数，所以每一个场景都修改源代码，很明显是有问题的，也就迎来了第三次修改。

##### 第三次修改

第三次修改就很清晰了，虽然使用方式不同，但是场景都是接收多个（一个也可以）参数，然后替换占位符，那么我们只需要在外部函数内部记住此场景替换占位符的位置即可，然后内部函数每次调用之前都将这些位置重置，就解决问题了。说这可能很复杂，代码写完看着就清晰了。

```js
function partial(fn, ...particalArgs) {
  // 转换成另一个数组
  let args = particalArgs;
  // 记录占位符的位置
  const keyPositions = args.reduce((acc, cur, index) => {
    cur === undefined ? acc.push(index) : null;
    return acc;
  }, []);
  // 返回函数，该函数也接受一串参数
  return function(...fullArgs) {
    let arg = 0;
    // 每次调用之前重置占位符为 undefined
    for (let i = 0; i < keyPositions.length; i++) {
      args[keyPositions[i]] = undefined;
    }
    // 遍历两个参数列表，如果外部函数参数列表遇到了 undefined，此时将内部函数参数列表逐次赋值给外部函数参数列表。undefined 的意义表示，从 undefined 开始要把内部函数的参数也赋值给 args 参数列表
    for (let i = 0; i < args.length && arg < fullArgs.length; i++) {
      if (args[i] === undefined) {
        args[i] = fullArgs[arg++];
      }
    }
    return fn.apply(null, args);
  }
}
```

调用一下我们已经封装好的函数：

```js
const tab4Stringify = partial(JSON.stringify, undefined, null, 4);

const obj = { name: 'luffyzh', email: 'luffyzh@163.com' };
console.log(tab4Stringify(obj));
"{
    "name": "luffyzh",
    "email": "luffyzh@163.com"
}"
const obj2 = { name: 'naruto', email: 'naruto@163.com' };
console.log(tab4Stringify(obj2));
"{
    "name": "naruto",
    "email": "naruto@163.com"
}"
function add4param(a,b,c,d) {
  return a + b + c + d;
}
const _add4param = partial(add4param, 1, undefined, 3, undefined);
console.log(_add4param(2, 4)); // => 1 + 2 + 3 + 4 = 10
console.log(_add4param(1, 3)); // => 1 + 1 + 3 + 3 = 8
```

从上面可以看到，使用起来已经没什么问题了。接下来如果还想优化的话，那就是可以像`lodash`一样，我们的占位符是`undfined`，而可以通过特定字符串来代替`undefined`，比如`__`，使用`__`而不是`_`因为可以避免大家使用`lodash`的时候造成的冲突。

```js
<script>
  // 提前设置好 __
  window.__ = '__';
  // 偏函数
  function partial(fn, ...particalArgs) {
    var __ = window.__ || '__';
    let args = particalArgs.slice(0);
    const keyPositions = args.reduce((acc, cur, index) => {
      cur === __ ? acc.push(index) : null;
      return acc;
    }, []);
    // 返回函数，该函数也接受一串参数
    return function(...fullArgs) {
      let arg = 0;
      // 记住占位符的位置，也就是要替换的内容的位置
      for (let i = 0; i < keyPositions.length; i++) {
        args[keyPositions[i]] = __;
      }
      // 遍历两个参数列表，如果外部函数参数列表遇到了 undefined，此时将内部函数参数列表逐次赋值给外部函数参数列表。undefined 的意义表示，从 undefined 开始要把内部函数的参数也赋值给 args 参数列表
      for (let i = 0; i < args.length && arg < fullArgs.length; i++) {
        if (args[i] === __) {
          args[i] = fullArgs[arg++];
        }
      }
      return fn.apply(null, args);
    }
  }
  
  const tab4Stringify = partial(JSON.stringify, __, null, 4);
  
  const obj = { name: 'luffyzh', email: 'luffyzh@163.com' };
  console.log(tab4Stringify(obj));
  const obj2 = { name: 'naruto', email: 'naruto@163.com' };
  console.log(tab4Stringify(obj2));

  function add4param(a,b,c,d) {
    console.log(a, b, c, d);
    return a + b + c + d;
  }
  const _add4param = partial(add4param, 1, __, 3, __);
  console.log(_add4param(2, 4));
  console.log(_add4param(1, 3));
</script>
```

> 其实最好的方法就是直接使用 lodash 的`_.partial`方法。

> 事实上，我们在设计函数的时候（或者使用已经封装好的函数），一些固定参数有可能是从左至右，也可能从右至左，这两种都是合理的设计方式，因此才会出现柯里化与偏函数这两个概念。

**【区别】：柯里化和偏函数是在日常设计函数式编程时非常有效的手段，二者是必须的但是并不是同时需要的。某一场景只需要使用某一个手段即可。他们的区别其实也很明显，前面的分析可知，二者在应用的时候接收的第一个参数都是一个函数 fn，后边可能会接收其他参数，柯里化接收的第一个参数 fn 也就是操作函数是固定的，而调用fn函数的变量也就是后续参数是不固定的，依次传入的，比如寻找函数数组中包含数字的例子，函数fn是固定验证的，而调用此函数的数组是后续传入的，可以是任何数组变量；偏函数正好相反，是后面的参数一个或者多个是固定占位的，而前面的fn则是不固定的，可能是任何函数。比如setTimout那个场景，固定后续第二个参数为 1000，前面 fn 既可以是 `() => console.log(something)`也可以是`() => alert(something)`只不过表示的都是1s间隔做某些事 。所以也可以看出，二者基本上是不能同时使用的，如果我们调用函数固定，调用参数不定，那就是柯里化；如果我们参数固定，调用函数不定，那就是偏应用（偏函数）**



### 组合和管道

组合的概念经常遇到，只是我们并不知道概念而已，一般来说函数式组合的概念简易而言：一个函数的输出可以是另一个函数的输入，将此类函数组合起来就是函数式组合。

> 数组函数的设计概念/Jquery的设计概念，链式运算就是组合，调用结束的输出结果又是另一个函数的输入。

#### compose函数

##### 最简单的 compose 函数

```js
const compose = (a, b) => c => a(b(c));
```

上面的 compose 函数接收两个参数，分别是 a 和 b，返回一个接收参数 c 的函数，执行的时候，函数的调用方向是从右至左，先调用b再将返回的结果传入a之后调用。这就是最简单的compose组合函数。

**因为有了函数组合，因此，这也鼓励我们在设计函数的时候要设计粒度最小化的函数，每个函数只做一件事，而复杂功能可以通过函数组合来实现。**

- 组合应用一：输入字符串，输出四舍五入之后的整数

```js
const numberRound = compose(Math.round, parseFloat);
numberRound("3.56"); // 4
numberRound("4.23"); // 4
```

上面的简单组合应用到两个函数，`parseFloat` 和 `Math.round`。先将指定字符串转化成 Number 类型，再把所得结果通过 `Math.round`函数四舍五入。所以执行顺序是 `parseFloat -> Math.round`，调用顺序从右至左开始而函数的参数顺序就是`Math.round -> parseFloat`。

- 组合应用二：输入字符串，输出字符串内包含的单词个数

```js
// 函数一：分离字符串，得到数组
const splitStr = str => str.split(' ');
// 函数二：计算数组长度
const countArrayLength = arr => arr.length;
// 组合函数计算字符串里单词个数
const countWords = compose(countArrayLength, splitStr);
```

前面的 compose 函数功能上没什么问题，并且实现起来也很简单，但是正因为如此，所以存在一个很大的问题。再来回顾一下定义：函数的输出可以作为另一个函数的输入。这也就意味着函数可以接收的参数个数必须是一个，因为输出是唯一的。因此这个函数在使用上是有限制的，而事实上我们要使用的场景肯定是复杂的多的，因此要把这些场景统一起来，就需要用到前面学到过的`curry`和`partial`函数了。

#### compose 进阶 — 组合多个函数

前面的 compose 函数只能组合两个函数，那么既然是组合，能不能三个四个或者更多呢？答案肯定是能的，但是上面的代码是不可以的，下面来看看思路。

> 【思路】：组合多个函数和两个函数本质上都是相同的，就是从右至左依次调用并且将调用的结果也就是输出作为下次的输入再次执行，依次类推知道最后执行完成。听起来是不是感觉很熟悉，再确定化一下，组合的函数是作为参数传递的，也就是 arguments => 类数组，如果使用 ES6 语法可以是 ...fns，也就是数组，数组元素是函数，依次调用数组里面的函数然后返回最后的结果，很明显 ===> reduce

```js
// reduce 实现 compose
const compose = (...fns)
	=> val => fns.reverse().reduce((acc, curFn) => curFn(acc), val);
```

上面有一点需要注意，先调用了 `fns.reverse()` 将数组翻转一下，因为我们的参数函数调用顺序是从右至左。其实 ES6+ 已经有了新的方法，`reduceRight`，也就是可以下面这么实现：

```js
// reduceRight 实现 compose
const compose = (...fns) 
	=> val => fns.reduceRight((acc, curFn) => curFn(acc), val);
```



### 管道

`compose` 函数机制是从右至左，最右侧函数先执行所得结果传递给左侧函数，知道最后最左侧函数执行完毕；那么就出现了另外一种机制，就是从左至右，这种场景被称之为管道 — pi peline。

```js
// 实现 pipeline
const pipeline = (...fns)
	=> val => fns.reduce((acc, curFn) => curFn(acc), val);
```

可以看到，其实管道就是 `compose` `函数的变型`，只是改变了函数的执行顺序而已，也就是不用进行函数参数的翻转操作。同时使用的时候我们也要注意，调用顺序也不一样了，是从左到右的顺序传递函数参数。

#### 组合的优势

组合 compose 与管道 pipeline 本质都是相同的，根据习惯不同使用哪种都可以，不过事实上看到的大部分开发者或者库更推荐使用的是组合 compose 的形式。而组合的优势在开发过程中主要体现在 — **组合满足结合律**。

```js
// 组合结合律
compose(f1, compose(f2, f3)) === compose(compose(f1, f2), f3)
```

### Generator



#### 应用一：通过 yield 传递数据

下面代码的执行结果是什么？

```js
function* sayFullName() {
  const firstName = yield;
  const secondName = yield;
  console.log(firstName + secondName);
}
const fullName = sayFullName();
fullName.next();
fullName.next('luffy');
fullName.next('zhou');
// luffyzhou
```

##### 面试题：下面代码的输出是什么？

```js
function* sayFullName() {
  const firstName = yield;
  const secondName = yield;
  console.log(firstName + secondName);
}
const fullName = sayFullName();
console.log(fullName.next());
console.log(fullName.next('firstName'));
console.log(fullName.next('secondName'));
======================
{value: undefined, done: false}
{value: undefined, done: false}
firstName secondName
{value: undefined, done: true}
```