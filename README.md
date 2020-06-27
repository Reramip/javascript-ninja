# JavaScript 忍者秘籍（第2版）

## Web应用生命周期

1. 用户输入URL
2. 浏览器生成请求并发送至服务器
3. 服务器执行动作，将响应返回客户端
4. 浏览器解析HTML、CSS、JS，构建页面
5. 浏览器监控事件队列
6. 用户与页面元素交互
7. 用户关闭页面

从用户角度看，应用生命周期包括页面构建(4)、事件处理(5,6)。

### 页面构建

页面构建目标是建立Web应用的UI，主要包括交替执行的两个步骤：

1. 处理HTML节点过程中：解析HTML，构建DOM
2. 处理script节点过程中：停止构建DOM，逐行执行JavaScript代码。要重点注意windows对象存在于整个页面的生存期间，前面的script元素中的全局变量也可以被后面的script元素访问到。但script里的代码不能操作尚未构建的DOM节点，所以要放在页面底部以期所有待操作的节点都已经解析完成，存在于DOM树中。浏览器采用单线程执行模型，同一时刻仅执行一个代码片段。

### 事件处理

事件处理是Web应用的核心。浏览器采用事件队列跟踪待处理的事件。所有已生成的事件按照其被浏览器检测到的顺序加入事件队列。事件的产生与加入事件队列独立于页面构建和事件处理之外，在用户与页面元素交互的过程中发生。浏览器不断检查队首，调用处理器函数处理对应的事件。

## 函数

函数是一等公民。函数是对象，对象能做的，函数都能做。

### arguments

ES6引入剩余参数。剩余参数是一个**数组**。

```javascript
// 对不定数量的参数求和，形参paras是一个数组
function sum(...paras){
  console.log(paras);
  if(paras.length<1) return null;
  let result=0;
  for(let i=0;i<paras.length;++i){
    result+=paras[i];
  }
  return result;
}
sum(1,2,3,4,5);
```

输出如下，可以看到paras是一个数组。

```javascript
Array(5) [ 1, 2, 3, 4, 5 ]
```

而在ES6之前，使用隐式的函数参数arguments达到这个效果。arguments是一个**对象**，而不是数组。

MDN推荐使用ES6的剩余参数

>Note: If you're writing ES6 compatible code, then rest parameters should be preferred.

```javascript
// 不定数量参数求和，隐式参数arguments是一个对象
function sum(){
  console.log(arguments);
  if(arguments.length<1) return null;
  let result=0;
  for(let i=0;i<arguments.length;++i){
    result+=arguments[i];
  }
  return result;
}
sum(1,2,3,4,5);
```

输出如下，arguments是一个对象，它的前几个key是调用函数时传递的实参

```javascript
Arguments { 0: 1, 1: 2, 2: 3, 3: 4, 4: 5, … }
// 将其展开得到
Arguments
0: 1
1: 2
2: 3
3: 4
4: 5
callee: function sum()
length: 5
Symbol(Symbol.iterator): function values()
<prototype>: Object { … }
```

#### arguments对象作为函数的别名

```javascript
function tellYourWorld(name){
  console.log(`name: ${name}`);
  console.log(`arguments[0]: ${arguments[0]}`);
  arguments[0]="miku";
  console.log(`name: ${name}`);
  console.log(`arguments[0]: ${arguments[0]}`);
}
tellYourWorld("world");
```

输出

```javascript
name: world
arguments[0]: world
name: miku
arguments[0]: miku
```

这种方式可读性差，不推荐使用。在严格模式下不能使用。

```javascript
// 严格模式输出
name: world
arguments[0]: world
name: world
arguments[0]: miku
```

### this简介

调用函数时，另一个隐式的参数this用以指代函数的上下文（context），代表函数调用相关联的对象。

函数的调用主要有四种：

- 直接调用func()
- 方法式调用obj.func()
- 构造函数new Foo()
- apply/call func.call(obj)

这不同调用方式下this的指向有所不同。this是在函数被调用的时候才发生绑定。

```javascript
function meow(){
  return this;
}
meow();

let cat={
  meow:function(){return this;}
};
cat.meow();

function Cat(){
  this.meow=function(){
    return this;
  }
}

meow.call(cat);
```

控制台输入结果如下：

```javascript
Window about:home // 直接调用 严格模式下为undefined

Object { meow: meow() } // 作为cat对象的方法调用

Object { meow: meow() } // 在构造的新对象中调用

Object { meow: meow() } // apply/call
```

直接调用时，this指向全局上下文；作为对象的方法时，this指向紧跟着调用它的对象；构造函数中指向创建的新对象；apply、call可以显式地指定this的指向。

由于javascript中this在调用的时候才绑定，在一个对象中调用另一个对象的this方法会得到意料之外的结果：

```javascript
function Cat(){
  this.meow=function(){
    return this;
  }
}
const cat=new Cat();
const body=document.getElementsByTagName("body")[0];
body.meow=cat.meow;
console.log(body.meow()); // <body class=....>
console.log(cat.meow());  // Object { meow: meow() }
```

body的meow方法并没有指向cat的函数上下文，而是指向了body的上下文，这正是因为在函数真正执行之前，它的上下文并未绑定到实体。meow方法仅仅是在cat对象上声明了，它在body的上下文中执行，那么this要指向body的函数上下文。

使用箭头函数（ES6）或bind方法（ES5）

箭头函数的this与声明所在的上下文相同，调用箭头函数时不会隐式传入this参数：

```javascript
function Cat(){
  this.meow=()=>{
    return this;
  }
}
const cat=new Cat();
const body=document.getElementsByTagName("body")[0];
body.meow=cat.meow;
console.log(body.meow()); // Object { meow: meow() }
console.log(cat.meow());  // Object { meow: meow() }
```

cat.meow在cat中声明，它的箭头函数meow指向cat对象。

可以使用bind方法绑定上下文

```javascript
function Cat(){
  this.meow=function(){
    return this;
  }
}
const cat=new Cat();
cat.meow=cat.meow.bind(cat);
const body=document.getElementsByTagName("body")[0];
body.meow=cat.meow;
console.log(body.meow()); // Object { meow: meow() }
console.log(cat.meow());  // Object { meow: meow() }
``` 

要注意的是，bind与apply/call不同

- apply/call将函数**临时地**与传递的上下文绑定，并执行函数，在执行过程中使用传递的对象的上下文作为上下文，最终返回函数执行的返回值
- bind创建一个**永久地**与传递的上下文相绑定的**新函数**，不执行，返回这个新函数。

#### 构造函数的执行过程

1. 创建一个新的空对象
2. 将这个新的对象作为this参数传递给构造函数，成为构造函数的函数上下文
3. 新构造的对象将作为new运算符的返回值

特殊的是，构造函数有返回值时，若返回值是对象，`new`运算符将返回这个对象，否则返回新构造的对象。 

