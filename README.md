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

调用函数时，另一个隐式的参数this用以指代函数的上下文（context），代表函数调用相关联的**对象**。

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

控制台输出结果如下：

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

body的meow方法并没有指向cat的函数上下文，而是指向了body的上下文，因为在函数真正执行之前，它的上下文并未绑定到实体。meow方法仅仅是在cat对象上声明了，它在body的上下文中执行，那么this要指向body的函数上下文。

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

简单的forEach迭代方法实现

```javascript
function forEach(list, callback){
  for(let i=0;i<list.length;++i){
    callback.call(list[i], i);
  }
}
// usage: forEach(arr, function(index){})
```

要注意的是，bind与apply/call不同

- apply/call将函数**临时地**与传递的上下文绑定，并执行函数，在执行过程中使用传递的对象的上下文作为上下文，最终返回函数执行的返回值
- bind创建一个**永久地**与传递的上下文相绑定的**新函数**，不执行，返回这个新函数。

### 构造函数的执行过程

1. 创建一个新的空对象
2. 将这个新的对象作为this参数传递给构造函数，成为构造函数的函数上下文
3. 新构造的对象将作为new运算符的返回值

特殊的是，构造函数有返回值时，若返回值是对象，`new`运算符将返回这个对象，否则返回新构造的对象。

### 执行上下文

上下文（函数上下文、全局上下文）代表的是与函数调用相关联的**对象**，不能与执行上下文混同。

执行上下文类似操作系统中的进程上下文。进程上下文保存了中断恢复所需的全部数据，是运行时的现场。对于单线程模型的Javascript而言，JS引擎利用执行上下文跟踪函数的执行。发生函数调用时，当前的执行上下文停止执行。要为新调用的函数创建一个新的执行上下文，并让它入调用栈，而调用栈的栈顶表示当前运行的函数的执行上下文或全局执行上下文。

### 词法环境/作用域

在作用域范围内，每次执行代码时，代码段都会获取与之相关的词法环境。词法环境是在函数**创建**的时候确定的。

词法环境有层次关系。如果内部的词法环境中找不到某个变量，就会查找外部环境直至全局环境。

```javascript
function foo(){
  var fooFoo="foofoofoo";
  function bar(){
    var barBar="barbarbar";
    console.log(fooFoo+barBar);
  }
  bar();
}
foo();
```

bar中可以访问到上级环境的元素fooFoo，输出

```javascript
foofoofoobarbarbar
```

在调用函数时，会创建一个新的执行上下文进入调用栈，还会为其创建一个相关联的词法环境。调用这个函数时的环境的引用储存在不可直接访问的内部属性`[[Environment]]`上，每个函数的`[[Environment]]`中都有指向创建这个函数的环境的引用。foo的`[[Environment]]`指向全局环境，而bar的`[[Environment]]`指向foo环境。一个函数在调用栈越靠顶，它自身的环境就越处于内部。全局环境在环境最外部，全局执行上下文也在调用栈的最底部。

### var, let, const

使用var声明（定义）一个变量，它将属于和它**距离最近的函数**或全局词法环境。

使用ES6的let/const声明（定义）变量，它属于和它距离最近的词法环境，这在var之上还包含了块作用域、循环

### 变量提升

JS中可以先访问函数，后声明函数，引擎会自动地将函数声明提前。变量类似。这被称作变量提升。

事实上，Javascript代码分两阶段执行：

1. 预解析阶段：访问并注册当前词法环境下声明的所有变量、函数
   - 当前词法环境是函数环境时，创建函数参数默认值
   - 当前词法环境是函数环境或全局环境时，扫描当前代码段，找到所有的**函数声明**（**忽略函数表达式和箭头函数**）。为每一个找到的函数声明创建函数，并绑定到当前环境下的同名标识符上。若标识符存在，则用新创建的函数**覆盖**之。
   - 查找所有var, let, const变量声明。若标识符不存在，注册标识符并初始化为undefined；若标识符存在，不进行覆盖，**保留原值**。
2. 执行阶段：运行代码

也就是说，在一个词法环境下，函数声明总是最优先的，然后是变量声明，声明完了才真正执行代码。所以它们会被“提升”。

```javascript
console.log(typeof declaredFunction);
console.log(typeof functionExpression);
console.log(typeof lambda);

function declaredFunction(){}
var functionExpression=function(){};
var lambda=()=>{};
```

运行结果如下：

```text
function
undefined
undefined
```

上例展示了预解析阶段仅对函数声明进行标识符注册，表达式和箭头函数被视作变量声明，初始化为undefined。

如果一个函数声明和变量声明重名会发生什么？

```javascript
console.log(typeof f);
var f="fff";
console.log(typeof f);
function f(){}
console.log(typeof f);
```

运行结果：

```text
function
string
string
```

究其原因，在同属于全局执行上下文和全局词法环境的这一代码段真正执行之前的预解析阶段，Javascript引擎先找到了函数声明`function f()`，将其绑定到标识符`f`上。而后来引擎为尝试为变量声明`f`注册标识符时，由于标识符`f`已经存在，将它忽略掉。然后进入执行阶段，第一行输出类型为function。为f赋值`"fff"`，标识符原来指向函数的引用被指向字符串的引用覆盖掉了，所以后面输出string。程序的执行过程中跳过了函数声明。

### 闭包

接下来用书中的例子解释闭包

```javascript
function Ninja(){
  var feints=0;
  this.getFeints=function(){
    return feints;
  }
  this.feint=function(){
    feints++;
  }
}

var ninja1=new Ninja();
console.log(ninja1.feints);
ninja1.feint();
console.log(ninja1.getFeints());

var ninja2=new Ninja();
console.log(ninja2.getFeints());
```

运行结果：

```javascript
undefined
1
0
```

在代码执行过程中，

1. 使用new关键字调用构造函数`Ninja`，创建新的函数执行上下文入调用栈；
2. 为函数创建词法环境。Ninja环境保存了对变量feints的引用，`[[Environment]]`指向全局环境；
3. 执行构造函数。为创建的两个表达式函数创建词法环境，这两个函数是在Ninja环境下创建的，它们的`[[Environment]]`都指向上一级词法环境Ninja环境；
4. 构造函数生成的新对象作为返回值赋给ninja1，调用栈顶退出，返回全局执行上下文；
5. 在全局词法环境下查找ninja1.feints，由于feints并不是ninja1的属性，找不到，且没有上级词法环境，返回undefined；
6. 调用ninja1.feint方法，创建新的函数执行上下文入栈；
7. ninja1.feint环境下没有feints，但它的`[[Environment]]`指向Ninja环境，其中有feints的引用，可以进行操作。执行操作后，栈顶退出，返回全局执行上下文；
8. 调用ninja1.getFeints方法，同理；
9. new调用Ninja构造函数赋给ninja2，略。要注意ninja1与ninja2分别有两个独立的Ninja环境，互不干涉。

```javascript
var imposter={};
imposter.getFeints=ninja1.getFeints;
console.log(imposter.getFeints()) // 1
```

结果是1，这验证了闭包并不是真正的`private`。这也说明了词法环境是在函数创建时就确定的。词法环境不像`this`上下文那样在运行时才确定，不绑定的话每次运行都可能不同。

只要存在能够通过闭包访问到“private”变量的引用，词法环境就会一直保留，不被回收。

### 生成器

可以**非阻塞挂起**，执行上下文类似闭包**不被回收**的特殊函数。它**基于每次请求**生成值。

以书中的作业题为例：

```javascript
function* Gen(val) {
  val = yield val*2;
  yield val;
}

let generator = Gen(2);
let a1 = generator.next(3).value; // a1 = 4
let a2 = generator.next(5).value; // a2 = 5

generator.next() // Object { value: undefined, done: true }
```

调用生成器函数Gen()将返回一个用来控制生成器执行的迭代器，赋给generator。

显然，generator迭代器要控制生成器的执行，它必须要获取Gen生成器函数的函数执行上下文才行。如若不然，Gen的上下文在离开调用栈顶时就消失了，生成器内部的数据没了，还怎么从之前的返回值里生成新值？

generator作为Gen生成器的迭代器，拥有指向Gen执行上下文的引用。这就和闭包一样，因为存在能从外界访问到它的引用，它就不会被javascript引擎回收，生成器的执行上下文就保留了下来。每次使用迭代器暴露的接口`next()`请求生成器生成新值时，都会直接把已保存下来的Gen函数执行上下文推入调用栈。挂起时再把它的执行上下文搁置到一边，而不销毁。

#### 为什么运行后的结果是a1=4，a2=5

上例的执行过程：

1. 传参数2进Gen，创建了生成器的执行上下文和词法环境，但是并不执行生成器内部的代码，而是返回一个迭代器；
2. 第一次调用迭代器的`next()`方法，传入实参3。此时生成器的状态称为“**挂起开始**”，其内部的代码还没有执行。**由于生成器并没有在`yield`字段挂起，这个传入的3会被忽略**，按val=2执行生成器代码。计算`val*2`，返回**含有中间结果4的新对象**给a1，然后在yield表达式`(yield val*2)`处挂起，称为“**挂起让渡**”；
3. 第二次调用迭代器的`next()`方法，传入实参5。此时生成器正**在yield表达式`(yield val*2)`处挂起**，**把5赋给这一整个yield表达式**，即是使`(yield val*2)`为`5`，执行赋值语句`val=5`。执行到下一行，返回含有中间结果5的新对象给a2，然后再在`(yield val)`处挂起；
4. 再调用迭代器的`next()`方法，可以看到返回对象的value未定义，而done属性为true表明这个生成器已用这个迭代器迭代完成，没有新值可以生成了。

生成器可以一定程度上让代码更优雅。以书上的深度优先的递归DOM树遍历为例(有改动)：

```javascript
function traverseDOM(element, callback){
  callback(element);
  element=element.firstElementChild;
  while(element){
    traverseDOM(element, callback);
    element=element.nextElementSibling;
  }
}
const tree=document.getElementById("tree");
traverseDOM(element, element=>{
  if(element) console.log(element.nodeName);
});
```

也可以用一个全局的数组用来保存每次遍历到的节点，再map。但这样的函数操作了外部的变量，不“纯正”。

使用生成器可以让函数更纯粹，只做一件事：

```javascript
function* traverseDOMGenerator(element){
  yield element;
  element=element.firstElementChild;
  while(element){
    yield* traverseDOMGenerator(element); // 转移到另一个traverseDOMGenerator实例上
    element=element.nextElementSibling;
  }
}
const tree=document.getElementById("tree");
for(let element of traverseDOMGenerator()){ // 遍历取值的let-of语法糖
  if(element) console.log(element.nodeName);
}
```

### Promise

promise（音标[ˈprɒmɪs]，重音在o不在i）对象是**异步**任务结果的占位符，是对未来结果的期望、保证。一个promise对象从等待（pending）状态开始，进入完成态（fulfilled/resolved）或拒绝态（rejected）。一旦promise对象进入完成态或拒绝态，它的状态就不可变了。

promise对象接受两个回调函数参数作为完成/拒绝时方法。

书中的promise案例：

```javascript
funtion getJSON(url){
  return new Promise((resolve, reject)=>{
    const request = new XMLHttpRequest();
    request.open("GET", url);
    request.onload = function(){
      try{
        if(this.status === 200){
          resolve(JSON.parse(this.response));
        }else{
          reject(`${this.status} ${this.statusText}`);
        }
      }catch(err){
        reject(err.message);
      }
    };
    request.onerror = function(){
      reject(`${this.status} ${this.statusText}`);
    };
    request.send();
  });
}
```

## 对象

### 原型

- 每个函数都有一个原型对象
- 这个原型对象含有constructor属性，指向创建对象的函数本身
- 将函数作为构造函数调用时（new），新创建的对象的原型为构造函数的原型

```javascript
function Ninja(){}
Ninja.prototype.swingSword = function(){
  return true;
}
const ninja = new Ninja();
```

- Ninja拥有原型对象Ninja.prototype
- Ninja.prototype.constructor === Ninja
- ninja的内部属性[[prototype]]指向Ninja.prototype

对象与原型的引用关系是在对象**创建**时确定的，如果改变Ninja的原型，已创建的ninja仍然会保持原来的原型不变。但是再调用Ninja构造函数得到的新对象将会以新的原型为原型。

通过原型可以实现继承关系：

```javascript
function Animal(){} // Animal函数拥有原型对象属性Animal.prototype
Animal.prototype.eat = function(){  // Animal.prototype.constructor === Animal
  return "eating";
}

function Cat(){}
// Cat.prototype = Animal.prototype 不建议使用，因为Cat的原型并不是Animal的原型。在之后如果还要对Cat.prototype进行修改，会全部应用到Animal.prototype上
Cat.prototype = new Animal(); // 让Cat.prototype指向一个新的Animal对象。新对象的内部[[prototype]]指向Animal.prototype
Object.defineProperty(Cat.prototype, "constructor", {
  enumerable: false,
  writable: true,
  value: Cat
}); // 或直接 Cat.prototype.constructor = Cat;

```

书上也是这样new了一个新的父类实例作为Ninja（这里是Cat）原型。这很奇怪，这样做是否表示每一个Cat都来自同一个特定的Animal？而且，如果Animal父类有自己的属性，也会被子类继承下去

为避免这样的问题，使用Object.create方法(ES5)从原型创建对象。

```javascript
Cat.prototype = Object.create(Animal.prototype);// 创建一个以Animal.prototype为[[prototype]]的新对象，赋值给Cat.prototype
Cat.prototype.constructor = Cat;
```

instanceof运算符表示右边的函数原型是否位于左边的对象的原型链上。

### class

以下两种代码等价：

```javascript
// ES6
class Warrior{
  constructor(weapon){
    this.weapon = weapon;
  }

  wield(){
    return "Wielding"+this.weapon;
  }

  static duel(warrior1, warrior2){
    return warrior1.wield()+' '+warrior2.wield();
  }
}

// Before ES6
function Warrior(weapon){
  this.weapon = weapon;
}
Warrior.prototype.wield = function(){
  return "Wielding"+this.weapon;
}
Warrior.duel = function(warrior1, warrior2){
  return warrior1.wield()+' '+warrior2.wield();
}
```


