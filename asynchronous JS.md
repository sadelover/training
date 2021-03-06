###### 同步异步
Javascript语言的执行环境是"单线程"（single thread.
所谓"单线程"，就是指一次只能完成一件任务。如果有多个任务，就必须排队，前面一个任务完成，再执行后面一个任务，以此类推。

这种模式的好处是实现起来比较简单，执行环境相对单纯；坏处是只要有一个任务耗时很长，后面的任务都必须排队等着，会拖延整个程序的执行。常见的浏览器无响应（假死），往往就是因为某一段Javascript代码长时间运行（比如死循环），导致整个页面卡在这个地方，其他任务无法执行。
为了解决这个问题，Javascript语言将任务的执行模式分成两种：同步（Synchronous）和异步（Asynchronous）。

* "同步模式"就是上一段的模式，后一个任务等待前一个任务结束，然后再执行，程序的执行顺序与任务的排列顺序是一致的、同步的
* "异步模式"则完全不同，每一个任务有一个或多个回调函数（callback），前一个任务结束后，不是执行后一个任务，而是执行回调函数，后一个任务则是不等前一个任务结束就执行，所以程序的执行顺序与任务的排列顺序是不一致的、异步的

"异步模式"非常重要。在浏览器端，耗时很长的操作都应该异步执行，避免浏览器失去响应，最好的例子就是Ajax操作。在服务器端，"异步模式"甚至是唯一的模式，因为执行环境是单线程的，如果允许同步执行所有http请求，服务器性能会急剧下降，很快就会失去响应。

######异步模式 编程的4种方法


1.  回调函数
    ```javascript
    //假定有两个函数f1和f2，后者等待前者的执行结果。
    f1();
    f2();
    //如果f1是一个很耗时的任务，可以考虑改写f1，把f2写成f1的回调函数。
    function f1(callback){
    　　setTimeout(function () {
    　　　　// f1的任务代码
    　　　　callback();
    　　}, 1000);
    }
    //执行代码就变成下面这样
    f1(f2);
    ```
    采用这种方式，我们把同步操作变成了异步操作，f1不会堵塞程序运行，相当于先执行程序的主要逻辑，将耗时的操作推迟执行。
    
    优点:简单、容易理解和部署

    缺点:不利于代码的阅读和维护，各个部分之间高度耦合（Coupling），流程会很混乱，而且每个任务只能指定一个回调函数。

2.  事件监听
    ```javascript
    f1.on('done', f2);
    function f1(){
    　　setTimeout(function () {
    　　　　// f1的任务代码
    　　　　f1.trigger('done');
    　　}, 1000);
    }
```
    优点:比较容易理解，可以绑定多个事件，每个事件可以指定多个回调函数，而且可以"去耦合"（Decoupling），有利于实现模块化。

    缺点:整个程序都要变成事件驱动型，运行流程会变得很不清晰。
3. 发布/订阅
    ```javascript
    //首先，f2向"信号中心"jQuery订阅"done"信号。
    jQuery.subscribe("done", f2);
    function f1(){
    　　setTimeout(function () {
    　　　　// f1的任务代码
    　　　　jQuery.publish("done");
    　　}, 1000);
    }
    //jQuery.publish("done")的意思是，f1执行完成后，向"信号中心"jQuery发布"done"信号，从而引发f2的执行。
    //取消订阅（unsubscribe）
    jQuery.unsubscribe("done", f2);
    ```
    这种方法的性质与"事件监听"类似，但是明显优于后者。因为我们可以通过查看"消息中心"，了解存在多少信号、每个信号有多少订阅者，从而监控程序的运行。

4. Promises对象
    思想:每一个异步任务返回一个Promise对象，该对象有一个then方法，允许指定回调函数

    promise对象是CommonJS工作组提供的一种规范，用于异步编程的统一接口。
    
    promise对象通常实现一种then的方法，用来在注册状态发生改变时作为对应的回调函数。
    
    promise模式在任何时刻都处于以下三种状态之一：未完成（unfulfilled）、已完成（resolved）和拒绝（rejected）

    ```javascript
    f1().then(f2);
    function f1(){
    　　var dfd = $.Deferred();
    　　setTimeout(function () {
    　　　　// f1的任务代码
    　　　　dfd.resolve();
    　　}, 500);
    　　return dfd.promise;
    }
    ```
    优点:回调函数变成了链式写法，程序的流程可以看得很清楚，而且有一整套的配套方法，可以实现许多强大的功能.如果一个任务已经完成，再添加回调函数，该回调函数会立即执行。所以，你不用担心是否错过了某个事件或信号
    
    缺点:编写和理解，都相对比较难
        
    ```javascript
    //定多个回调函数
    f1().then(f2).then(f3);
    //指定发生错误时的回调函数
    f1().then(f2).fail(f3);
    ```

######引用
[Javascript异步编程的4种方法](http://www.ruanyifeng.com/blog/2012/12/asynchronous%EF%BC%BFjavascript.html)

[谈谈使用promise时候的一些反模式](http://www.tuicool.com/articles/FvyQ3a)

[JavaScript异步编程的Promise模式](http://www.infoq.com/cn/news/2011/09/js-promise/)

[Asynchronous JS: Callbacks, Listeners, Control Flow Libs and Promises](http://sporto.github.io/blog/2012/12/09/callbacks-listeners-promises/)