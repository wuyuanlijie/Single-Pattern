# Single-Pattarn
🐶单例模式深入学习
# 单例模式

![](https://dn-mhke0kuv.qbox.me/4fe6204c1c4c507361da.jpg)
## 前言
这两天是不是被朋友圈里的恩爱狗们秀了一脸？别慌！学习使我们强大，躁起来！在这个5.20，5.21的神圣日子里来聊一个相对简单的**设计模式-单例模式**来入门这个设计模式，并对闭包、封装、命名空间、单一职责、惰性有一个应用场景的了解。
## 应用场景
这种模式，常用于线程池、全局缓存、浏览器中的window对象。在javascript中也有大显神威的地方。比如有个点击事件触发动态创建的浮窗，而这个浮窗是唯一的，在没有遮罩的情况下我可以一直点击创建，这样一来就会创建很多节点，可是只能创建一次，这时候就需要用到。

### 举个例子
之前玩的小demo，利用百度地图的api进行地址解析之后，找到这个地点，显示地点并new一个mark时，如果思维中没有单例的概念将会这样。有很多红色的标记重合也一直不断跳跃。

![](https://dn-mhke0kuv.qbox.me/f9fe825a0845e26e8cc8.png)

而要的效果是这样的： 一个地点对应的mark只能new一次。

![](https://dn-mhke0kuv.qbox.me/bb55c345d9f1b928281a.png)

待会我们来通过一个doger找对象的栗子来具体看看单例模式。

## 介绍
单例模式顾名思义，单个实例模式，老梗不是说了吗！程序员的对象哪来的？new出来的！把它看作是**找对象**来理解再好不过，做人要忠诚，**只能找（new，实例化）一个对象**。

#### 定义：保证一个类仅有一个实例，并提供一个访问全局访问点

![](https://dn-mhke0kuv.qbox.me/f25e641d4fade4974ab5.jpg)

我们想法无非是在**new之前做一个判断，用一个变量保存构造对象是否被new的状态**。
#### 最重要的就是处理一个instance的状态！
## 命名空间
**假设我们管理状态的变量叫instance，问题来了？这个变量放哪？放全局吗？**

在开发中，我们其实经常会将为**全局变量**当作单例模式来使用，在对javascript的 创造者Brendan Eich访谈中，他也提到了全局变量是设计的失误。因为很容易造成命名空间的**污染**，在中大型项目中不加以管理变量，很容易产生覆盖。

**对于变量污染问题，通常有两种解决方案**
1. 对象字面量的命名空间。
2. 闭包。

## 对象字面量实现单例模式
这种方法简单粗暴，但如果这个对象体积也很庞大，跟全局污染没有差别。
```
var Singleton = {
// 全局该放的都放这里

    instance: null,
    Create: function () {},
    ...
}
```

## 闭包实现单例模式
首先我们需要单例模式的实现是靠**闭包**实现的，作为今后面试很可能遇上的题目，如果问到了闭包，顺带提一下单例模式，相信自己底气会更足些。
## 谈一下闭包
* Nicolas C.Zakas：闭包是指有权访问另一个函数作用域中的变量的函数
* KYLE SIMPSON：当函数可以记住并访问所在的词法作用域时，就产生了闭包，这个函数持有对该词法作用域的引用，这个引用就叫做闭包
### 1. 作用域和作用域链（变量的访问）
**JavaScript的作用域就是词法作用域而不是动态作用域**
词法作用域最重要的特征是它的定义过程发生在代码的**书写编译阶段**。动态作用域的作用域链是基于调用栈的 词法作用域的作用域链是基于代码中的作用域嵌套。
只有在函数调用的时候，才会创建执行环境和作用域链，同时每个环境都只能逐级向上搜索作用域链，来查询变量和函数名等标识符。

![](https://dn-mhke0kuv.qbox.me/700a0d0c82999fd1a03a.png)

待会我们看到的对变量instance的访问
```
{
 var instance = null;
 return function (name) {
   if (!instance) {
     instance = new Singleton(name);
   }
   return instance;
 }
}
```
匿名函数function (name)在编译到(!instance)发生了一个**LHS查询（查找）**，先询问自己的作用域里有没有instance，很遗憾！并没有！然后再问上一级，也就是最外层两个括号包的全部区域（就是这段代码）。**因此匿名函数function (name）**是可以访问到instance的状态改变的。
### 2. 立即执行函数IIFE
```
function(){
  // 里面的内容立即执行
}()
```
等同于这个 a
```
function() {
    var a = function() {}
    a()
}
```
将它理解为创建了一块作用域，发生闭包的一种常见方式就好。

### 3. 重要的！垃圾回收机制和闭包
我们要用到的就是对状态instance的保留，而不被垃圾回收。不理解闭包的对这段代码经常判断失误：
```
Singleton.getInstance = (function () {
      var instance = null;
      return function (name) {
        if (!instance) {
          instance = new Singleton(name);
        }
        return instance;
      }
    })();
```
认为每次执行到这，会执行var instance = null，其实并不是这样的，instance = null不会每次被执行，而是作为状态保留在*Singleton*中。
**这源于javascript的垃圾回收机制**
1. Singleton的变量对象因为闭包的存在没有被释放，注意闭包保存的是整个变量对象，而不是只保存只被引用的变量。
2. JavaScript最常用的垃圾收集方式是标记清除，垃圾收集器会给存储在内存中的所有变量都加上标记，然后给环境中的变量解除标记，以及被环境中的变量引用的变量的标记**（包括instance）**，说明这些变量还有作用，暂时不能被删除，然后在此之后被加上标记的变量就是要删除的变量了，等待垃圾收集器对他们完成清除工作。
对函数来说，函数执行完毕后，会自动释放掉里面的变量，可是如果函数内部存在闭包，它们就不会被删除，因为这个函数还在被内部的函数所引用，所以他不会被加上标记，不会被清除，而是会一直存在内存中得不到释放！除非使用闭包的那个内部函数被销毁，外部函数才能得到释放。
### 闭包实现
那就来看*AlloyTeam《JavaSript设计模式与开发实践》*上的例子：
```
var Singleton = function (name) {
      this.name = name;
    }
    Singleton.prototype.getName = function () {
      alert(this.name);
    }
    var a = new Singleton('a');
    var b = new Singleton('b');

    Singleton.getInstance = (function () {
      var instance = null;
      return function (name) {
        if (!instance) {
          instance = new Singleton(name);
        }
        return instance;
      }
    })();

    var a = Singleton.getInstance('sven1');
    console.log(Singleton.instance);
    var b = Singleton.getInstance('sven2');
    a.getName(); //sven1
    b.getName(); //sven1
```
#### 虽然a、b都进行了实例化，但他们是同一个实例的不同引用变量指向同一个东西
#### 实现了我们初步的单例模式！！

因为我们知道JavaCript是没有像java一样的类的，*Singleton*是我们定义的构造函数（模拟的类），可以通过 *new Singleton(name)*
来实例，这个方法完全能实现单例模式。但是要注意从设计模式的角度出发考虑，有这几个问题：
 1.   ** 透明度不够**： 每次要用，*Singleton.getInstance('sven1')* 这种方式去创建实例，作为非开发这个程序的程序员更喜欢new！
    
 2.    ** 单一职责**： *Singleton.getInstance*这个函数过于肮脏，做了判断，实例，应该拆分开进行封装，以提高代码复用性。
 
了解到这，对单例模式就开始我们的故事

# 实现一个一夫一妻的柴犬找单例对象

![](https://dn-mhke0kuv.qbox.me/62641294cbc163799e66.png)
## 非单例惰性
首先看看一只不忠诚的柴犬，用最不可靠最肮脏的方法找对象
```
let CreateWife1 = function() {
    let div = document.createElement('div')
    div.innerText= "我是可爱的柴犬妹妹"
    div.style.display = "none"
    document.body.appendChild(div)
    return div
  }
  document.getElementById('findWife1').addEventListener('click', function() {
    let Wife1 = CreateWife1()
    Wife1.style.display = "block"
  })
```
点击第一个按钮，**不断点击不断实例化**！差评！
![](https://dn-mhke0kuv.qbox.me/203056b58aead0a1f1a6.png)
通过不断点击！哇！好多对象啊！花心注定没有真爱！这个方法不可取！

## 单例非惰性模式
这时候很多人选择先把实例造好，控制display就好，可以达到单例效果
```
let Wife2 = (function() {
    let div = document.createElement('div')
    div.innerText= "我是可爱的柴犬妹妹"
    div.style.display = "none"
    document.body.appendChild(div)
    return div
  })()
  document.getElementById('findWife2').addEventListener('click', function() {
    Wife2.style.display = "block"
  })
```

![](https://dn-mhke0kuv.qbox.me/4b8d7610317084bb867a.png)

果然实现了！只有不断点击只有一个柴妹妹出现！但是！
**DOM节点一直存在，虽然靠变化display改变，但如果我不找对象（并不触发按钮）浪费节点的非惰性**！我们需要**动态动态动态动态动态动态**创建DOM！要找就找新鲜的！放久的不要！

## 惰性模式
这个栗子解释了什么是**惰性**！其实又把**第一个例子**拿过来了，为了突出惰性的概念。理解了直接看第四个！
```
let CreateWife2 = function() {
    let div = document.createElement('div')
    div.innerText= "我是可爱的柴犬妹妹"
    div.style.display = "none"
    document.body.appendChild(div)
    return div
  }
  document.getElementById('findWife3').addEventListener('click', function() {
    let Wife3 = CreateWife1()
    Wife3.style.display = "block"
  })
```

![](https://dn-mhke0kuv.qbox.me/964c4541c57a586410fc.png)
实现了动态创建的效果，可是实例化了很多次。
## 惰性单例模式
满足动态创建和单例两个条件。
```
let CreateSingleWife = (function() {
    let div;
    return function() {
      if(!div) {
        div = document.createElement('div')
        div.innerText= "我是可爱的柴犬妹妹"
        console.log(div)
        div.style.display = "none"
        document.body.appendChild(div)
      }
      return div
    }
  })()
  document.getElementById('findWife4').addEventListener('click', function() {
    let SingleWife =  CreateSingleWife()
    SingleWife.style.display = "block"
  })
  ```
  
![](https://dn-mhke0kuv.qbox.me/d5e57cfec9c87770da48.png)
很好！我们通过前面介绍中的方法用闭包帮助我们实现了一个动态创建的单例模式。
这个例子通过**IIFE**创建了一个单例对象，函数里返回的对象字面量是这个单例模式的**公共接口**。通过闭包实现模块模式，可以做到很多强大的事情，模块模式能成功实现，最关键的是返回的API还能继续引用定义时所在的作用域，从而进行一些操作！

#### 但是！！！我们做的这个给柴犬生成对象实例的程序这时候需要做成一个给柴犬仅生成参考照片的功能相似的单例模式程序，这时候来看看 *单一职责复用性* 的问题
```
let CreateSingleWife = (function() {
    let img;
    return function() {
      if(!img) {
        img = document.createElement('img')
        document.body.appendChild(img)
        img.src = ''
      }
      return img
    }
  })()
  document.getElementById('findWife4').addEventListener('click', function() {
    let SingleWife =  CreateSingleWife()
    SingleWife.style.display = "block"
  })
  ```
  这样改写后是实现了没问题，但*CreateSingleWife*完成了太多任务，我们应该把任务拆分给不同的函数完成。
  
## 职责分离的单例模式

![](https://dn-mhke0kuv.qbox.me/7cd49d64857001880dd6.png)
### 代理器：只负责判断是否有实例，调用不同的构造器
```
let getSingle = function(fn) {
    let result;
    return function() {
      if(!result) {
        return result = fn.apply(this, arguments)
      }
      return result 
    }
  }
```
### div构造器：
```
let createDiv = function() {
    let div = document.createElement('div')
    div.innerText= "我是可爱的柴犬妹妹"
    div.style.display = "none"
    document.body.appendChild(div)
    return div
  }
  let createSingle = getSingle(createDiv)
```
### 图片构造器：
```
let createImg = function() {
    let oImg = document.createElement('img')
    document.body.appendChild(oImg)
    return oImg
  }
```

### 不同的场景：
```
  let createSingle = getSingle(createImg)
  let createSingle = getSingle(createDiv)
```

### 开始running：
```
document.getElementById('findWife5').addEventListener('click', function() {
    // 方案一
    let Single =  createSingle()
    Single.style.display = "block"

    // 方案二
    let SingleImg = createSingle()
    SingleImg.src = './dog4.png'
  })
```

![](https://dn-mhke0kuv.qbox.me/cc41213e044ea06f4a2e.png)

这次我们实例的是一张照片img，而不是div。src路径也是构建后动态加上去的，完成了对原来代理器和构造器的解耦。
# 结尾
##书籍参考：

 *AlloyTeam《JavaSript设计模式与开发实践》*
**良心推荐这本设计模式！! 比较通俗的方式讲的很深很透 ！！**

## Github
https://github.com/renjie1996/Single-Pattern
#### 嘤嘤嘤！求star！！！💗
#### 本汪2018届毕业生求实习！！qq：2578370399，微信：xrj1996kk
