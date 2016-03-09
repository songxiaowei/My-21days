# 单例模式


## 实现单例模式

单例模式的`定义`是：`保证一个类仅有一个实例，并提供一个访问它的全局访问点`。
单例模式是一种常用的模式，有一些对象我们往往只需要一个，比如登录窗口，这个窗口是唯一的，无论我们点击多少次登录按钮，这个窗口只会被创建一次，那么这个窗口就适合用单例模式来创建。

要实现一个标准的单例模式并不复杂，无非是用`一个变量`来标志当前是否已经为某个类创建过对象，如果是，则在下一次获取该类的实例时，直接返回之前创建的对象。代码如下：
```javascript
var Singleton=function(name){
    this.name=name;
    this.instance=null;
};
Singleton.prototype.getName=function(){
    alert(this.name);
};
Singleton.getInstance=function(){
    if(!this.instance){
        this.instance=new Singleton(name);
    }
    return this.instance;
};
var a=Singleton.getInstance('sven1');
var b=Singleton.getInstance('sven2');

alert(a===b);//true
```
或者
```javascript
var Singleton=function(name){
    this.name=name;
};
Singleton.prototype.getName=function(){
    alert(this.name);
};
Singleton.getInstance=(function(){
    var instance=null;
    return function(name){
        if(!instance){
            instance=new Singleton(name);
        }
        return instance;
    }
})();
```
我们通过Singleton.getInstance来获取Singleton类的唯一对象，这种方式相对简单，但是`不透明`。跟以往通过new XXX的方式获取对象不同，这里偏要使用Singleton.getInstance来获取对象，所以这段代码的意义并不大。

## 透明的单例模式

我们现在的目标是实现一个`透明`的单例类。
```javascript
var CreateDiv = (function() {
	
	var instance;
	
	var CreateDiv = function(html) {
		if (instance) {
			return instance;
		}
		this.html = html;
		this.init();
		return instance = this;
	};
	CreateDiv.prototype.init = function() {
		var div = document.createElement('div');
		div.innerHtml = this.html;
		document.body.appendChild(div);
	};
	return CreateDiv;
})()

var a = new CreateDiv('sven1');
var b = new CreateDiv('sven2');
alert(a===b);//true
```
虽然现在完成了一个透明的单例类的编写，但它同样有一些缺点。为了把instance封装起来，我们使用了自执行的匿名函数和闭包，并且让这个匿名函数返回真正的Singleton构造方法，这增加了一些程序的复杂度，阅读起来也不是很舒服。
假设我们某天需要让这个单例类变成一个普通的类，即可以产生多个实例，那我们必须改写CreateDiv构造函数，这种修改会带来很多不必要的麻烦。

## 用代理实现单例模式

我们首先创建一个普通的CreateDiv类：
```javascript
var CreateDiv = function(html) {
  this.html = html;
  this.init();
};
CreateDiv.prototype.init = function() {
  var div = document.createElement('div');
  div.innerHtml = this.html;
  document.body.appendChild(div);
};
```
接下来引入代理类ProxySingletonCreate：
```javascript
var ProxySingletonCreate = (function() {
  var instance;
  return function(html) {
    if (!instance) {
      instance = new CreateDiv(html);
    }
    return instance;
  }
})();

var a = new ProxySingletonCreate('sven1');
var b = new ProxySingletonCreate('sven2');
console.log(a === b);
```
这样一来，`CreateDiv`和`ProxySingletonCreate`组合起来，实现了单例模式的效果。

## JavaScript中的单例模式

前面提到的几种单例模式的实现，更多的是接近`传统面向对象语言`中的实现，单例对象从`类`中创建而来。但`JavaScript`其实是一门`无类`的语言，所以生搬单例模式的概念并无意义。
单例模式的`核心`是：`确保只有一个实例，并提供全局访问`。
`全局变量`不是单例模式，但在实际应用中，我们经常会把全局变量当成单例来使用。
例如：
```javascript
var a = {};
```
全局变量可以满足上述的两个条件，但却存在许多问题:它很容易造成`命名空间污染`。作为普通的开发者，我们有必要减少全局变量的使用，一下几种方式可以`相对降低`全局变量带来的命名污染。

### 使用命名空间

最简单的方法依然是使用`对象字面量`方式：
```javascript
var namespace1={
    a:function(){
    },
    b:function(){
    }
};
```
我们还可以动态的创建命名空间：
```javascript
var myApp = {};
myApp.namespace = function(name) {
  var parts = name.split('.');
  var current = myApp;
  for (var i in parts) {
    if (!current[parts[i]]) {
      current[parts[i]] = {};
    }
    current = current[parts[i]];
  }
};

myApp.namespace('event');
myApp.namespace('dom.style');

console.log(myApp);
```
### 使用闭包封装私有变量

```javascript
var user=(function(){
    var _name='sven',_age=29;
    return {
        getUserInfo:function(){
            return _name+'_'+_age;
        }
    }
})();
```

## 惰性单例

`惰性单例`指的是**在需要的时候才创建对象实例**。`惰性单例`是单例模式的重点，这种技术在实际开发中`非常有用`。
我们先抽取出一个管理单例的逻辑对象：
```javascript
var getSingle=function(fn){
    var result;
    return function(){
        return result||(result=fn.apply(this.arguments));
    }
};
```
创建对象的方法fn被当做参数传入getSingle。
```javascript
var getSingle = function(fn) {
  var result;
  return function() {
    return result || (result = fn.apply(this.arguments));
  };
};

var createLoginLayer = function() {
  var div = document.createElement('div');
  div.innerHTML = '我是登录窗';
  div.style.width = '100px';
  div.style.height = '100px';
  div.style.background = 'red';
  document.body.appendChild(div);
  return div;
};

aa = getSingle(createLoginLayer);
aa();
```
`result`变量因为身在`闭包`中，永远不会被销毁。在将来的请求中，如果result已经被赋值，那么他将返回这个值。
以下是演示地址：

[惰性单例演示地址][1]
[其他演示地址][2]


  [1]: http://jsbin.com/kutosub/edit?html,js,console,output
  [2]: http://jsbin.com/wojobij/edit?js,console