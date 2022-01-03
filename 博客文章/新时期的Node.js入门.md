# 新时期的Node.js入门

## Node的内部机制

- 在任务完成之前,CPU在任何情况下都不会暂停或者停止执行，CPU如何执行和同步或是异步、阻塞或者非阻塞都没有必然关系。
- 操作系统始终保持CPU处在运行状态,这是通过系统调度来实现的,具体一点就是通过在不同进程/线程间切换实现的。

### 何为回调

1. 回调的定义

回调就是将一个函数作为参数传递给另外一个函数，并且作为参数的函数可以被执行，其本质上是一个高阶函数。在数学和计算机科学中，高阶函数是至少满足下列一个条件的函数：

- 接受一个或多个函数作为输入。
- 输出一个函数。

回调方法和主线程处于同一层级，假设主线程发起了一个底层的系统调用，那么操作系统转而去执行这个系统调用，当调用结束后，又回到主线程上调用其后的方法，这也是为什么那被称为回调（call then back）。

关于回调函数在何时执行并没有具体的要求，回调函数的调用既可以是同步的（例如map方法）,也可以是异步的（例如setTimeout方法中的匿名函数）。

### 同步/异步和阻塞/非阻塞

1. 同步与异步

	同步和异步描述的是进程和线程的调用方式。
	同步调用指的是进程/线程发起调用后，一直等待调用返回才继续执行下一步操作，这并不代表CPU在这段时间内也会一直等待，操作系统多半会切换到另一个进程/线程上去，等到调用返回后再切换回原来的进程/线程。
	异步就相反，发起调用后，进程/线程继续向下执行，当调用返回后，通过某种手段来通知调用者。

	注意：同步和异步中的“调用返回”，是指内核进程将数据复制到调用进程（Linux环境下）。

 > 我们学说JavaScript是一门异步语言，但ECMAScript里并没有关于异步的规范，JavaScript的异步更多是依靠浏览器runtime内部其他线程来实现，并非JavaScript本身的功能，是浏览器提供的运行让JavaScript看起来像是一个异步的语言。
	
2. 阻塞与非阻塞

	阻塞与非阻塞的概念是针对I/O状态而言的，关注程序在等待I/O调用返回这段时间的状态。
	关于Node中的I/O，这里借用官网的说法：
	
	`Node.js uses event-driven, non-blocking I/O model that makes it lightweight and efficent`
	
	需要注意的是,Node也没有使用asynchronous（异步）之类的词汇，而是使用了non-blocking（非阻塞）这样的描述。
	
	阻塞/非阻塞和同步/异步完全是两组概念，他们之间没有任何的必然联系。很多人认为，阻塞=同步，非阻塞=异步，这种观点是不正确的。我们下面介绍I/O编程模型中，除了纯粹的IO之外，阻塞和非阻塞I/O都是同步的。
	
	在介绍I/O编程模型之前，先回答两个问题。
	
	（1）什么是I/O操作
	
	输入/输出（I/O）是内存和外部设备（如磁盘、终端和网络）之间复制数据的过程。
	在实践中I/O操作几乎无处不在，因为大多程序都要产生输出结果才有意义（往往是输出到磁盘或者屏幕），除非你只在内存中计算一个斐波那契数列而且不采取其他任何操作。
	
	在Node中，I/O特指Node程序在Libuv支持下与系统磁盘和网络交互的过程。
	
	（2）IO调用的结果怎么返回给调用的进程/线程
	通过内核进程复制给调用进程，在Linux下，用户进程没办法直接访问内核空间，通常是内核调用copy_to_user方法来传递数据的，大致的流程就是IO的数据会先被内核空间读取，然后内核将数据复制给用户进程。还有一种零复制技术，大致是内核进程和用户进程共享一块内存地址，这避免的内存的复制，想了解可以搜索更多相关内容。
	
3. IO编程模型

	编程模型是指操作系统在处理IO时所采用的方式，这通常是为了解决IO速度比较慢的问题而诞生的。
	
	一般来说，编程模型有以下几种：
	
	- blocking I/0
	- non-blocking I/0
	- I/0 multiplexing (select and poll)
	- signal driven I/0 (SIGIO)
	- asynchronous I/0 (the POSIX aio_functions)

	上面的5种模型中，signal driven I/O模型不常用，下面主要介绍其它4种，它们均指Linux下的IO模型。
	
	（1）阻塞IO（blocking I/O）
	
	对于IO来说，通常可以分为两个阶段，准备数据和返回结果，阻塞型IO在进程发出一个系统调用请求之后，进程就一直等待上述两个阶段完成，等待拿到返回结果之后再重新运行。
	
	（2）非阻塞IO（non-blocking I/O)

	和上面的过程相似，不同之处是当进程发起的一个调用后，如果数据还没有就绪，就会马上返回上一个结果告诉进程现在还没有就绪，和阻塞IO的区别是用户进程会不断查询内核状态。这个过程依旧是同步的。(个人理解是，这个进程下面的操作可以继续执行，但是不能执行其它进程）
	
	（3)IO multiplexing/Event Driven
	
	这种IO通常也被称为事件驱动IO，同样是以轮询的方式来查询内核的执行状态，和非阻塞IO的区别是一个进程可能会管理多个IO请求，当某个IO调用有了结果之后，就返回对应的结果。
	> 注意：select和poll都是IO复用的机制，另外Node使用epoll（改进后的poll）。
	
	（4） Asynchronous I/O
	异步IO和前面的模型相比，当进程发出调用后，内核会立刻返回结果，进程会继续做其他事情，直到操作系统返回数据，给用户进程发送一个信号。(个人理解是，可以继续执行这个进程或者其它进程的操作）
	
	> 注意：异步IO并没有涉及任何关于回调函数的概念，此外，这里的异步IO只存在于Linux系统下。
	
总结阻塞/非阻塞和同步/异步就是，同步调用会造成调用进程的IO阻塞，异步调用不会造成调用进程的IO阻塞。

### 单线程和多线程

Node并没有提供多线程的支持这代表用户编写的代码只能运行在当前线程中，用于运行代码的事件循环也是单线程运行的。开发者无法在一个独立进程中增加新的线程，但是可以派生出多个进程来达到并行完成工作的目的。

> 进程和线程的区别：

Node的底层实现并非是单线程的，libuv会通过类似线程池的实现来模拟不同操作系统下的异步调用，这对开发者来说是不可见的的。

#### Libuv中的多线程

开发者编写的代码运行在单线程环境中，这句话是没错的，但如果说整个Node都是依靠单线程运行的，那就不正确了，因为libuv中是有线程池的概念存在的。

Libuv是一个跨平台的异步IO库，它结合了UNIX下的libev和Windows下的IOCP的特性，最早由Node的作者开发，专门为Node提供多平台下的异步IO支持。libuv本身是由C/C++语言实现的，Node中的非阻塞IO以及事件循环的底层机制，都是由libuv来实现的。

在Windows环境下，libuv直接使用Windows的IOCP（I/O Completion Port）来实现异步IO。在非Windows环境下，libuv使用多线程来模拟异步IO。

![Libuv的架构](http://image.mamicode.com/info/201803/20180307105410133675.png)

> 注：图来自[http://www.mamicode.com/info-detail-2213450.html](http://www.mamicode.com/info-detail-2213450.html)

Node的异步调用是由libuv来支持的，以readFile为例，读取文件的系统调用是由libuv来完成的，Node只负责调用libuv的接口，等数据返回后再执行对应的回调方法。

> 资料参考：[http://www.mamicode.com/info-detail-2213450.html](http://www.mamicode.com/info-detail-2213450.html)

### 并行与并发

并行（Parallel）与并发（Concurrent）是两个很常见的概念，两者虽然中文译名相似，但实际上差别很大。

* 并行——同一个时间里，不同的等待处理队列会有对应的不同的处理者在处理。
* 并发——同一个时间里，不同的等待处理队列只有一个处理者在处理，但处理者会在不同的等待处理队列中来回切换处理。

![并发、并行的关系](https://images2015.cnblogs.com/blog/1096235/201704/1096235-20170414115344439-812878192.png)

> 注：图来自[https://www.cnblogs.com/xiaowangzi1987/p/6706416.html](https://www.cnblogs.com/xiaowangzi1987/p/6706416.html)

#### Node中的并发

单线程支持高并发，通常都是依靠异步+事件驱动（循环）来实现的，异步使得代码在面临多个请求时不会发生阻塞，事件循环提供了IO调用结束后调用回调函数的能力。

Java可以依靠多线程实现并发，Node本身不支持开发者书写多线程的代码，事件循环也是单线程运行的，但是通过异步和事件驱动都能够很好地实现并发。

### 事件循环（Event Loop）

通俗来讲，事件循环就是一个程序启动期间运行的死循环。Node代码虽然运行在单线程中，但是仍然支持高并发，就是依靠事件循环实现的。

#### 事件与循环

- 什么是事件
- 什么是循环

1. 事件
	在可交互的用户页面上，用户会产生一系列的事件，包括单击按钮、拖动元素等，这些事件会按照顺序被加载到一个队列中去。出来页面事件之外，还有一些如Ajax执行成功、文件读取完毕等事件。
	
2. 循环
	在GUI程序中，代码本身就处在一个循环的包裹中，例如作Java Swing开发桌面程序，就要启动一个JFrame，还要调用run 方法，而run方法内部就包括了一个循环，该循环位于主线程上。
	
	这个循环通常对开发者来说是不看见，只有当开发者点击了窗体的关闭按钮，该循环才会结束。当用户单击了页面上的按钮或者进行其他操作时，就会产生相应的事件，这些事件会被加入到一个队列中，然后主循环会逐个处理他们。
	
	JavaScript也是一样，用户在前台不断产生事件，背后的循环（由浏览器实现）会逐个地处理他们。
	
	而JavaScript是单线程的，为了避免一个过于耗时的操作阻塞了其他操作的执行，就要通过异步加回调的方式解决问题。
	
	以Ajax请求为例，当JavaScript执行到对应的代码时，就为这句代码注册了一个事件，在发出请求后该语句就执行完毕了，后续的操作会交给回调函数来处理。
	
	此时，浏览器背后的循环正在不断遍历事件队列，在Ajax操作完成之前，事件队列里还是空的（并不是发出请求这一动作被加入事件队列，而是请求完成这一事件才会加入队列）。
	
	如果Ajax操作完成了，这个队列就会增加一个事件，随后被循环遍历到，如果这个事件绑定了一个回调方法，那么循环就会去调用这个方法。
	
#### Node中的事件循环

Node中的事件循环比起浏览器中的JavaScript还是有一些区别的，各个浏览器在底层的实现上可能有些细微的出入；而Node只有一种实现，相对起来就少了一些理解上麻烦。

首先要明确的是，事件循环同样运行在单线程环境中，JavaScript的事件循环是依靠浏览器实现的，而Node作为另一种运行时，事件循环由底层的libuv实现。

下面是Node中事件循环的具体流程：

![Node_Event_Loop](https://user-gold-cdn.xitu.io/2018/9/11/165c858f28d852aa?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

事件循环一般分为6个不同的阶段(Phase)，其中每个阶段都维护着一个回调函数的队列，在不同的阶段，事件循环会处理不同类型的事件，其代表的含义分别为：

- Timer:用来处理setTimeOut()和setInterval()的回调
- I/O callbacks:大多数的回调方法在这个阶段执行，除了timers、close和setImmediate事件的回调。
- idle,prepare:仅仅在内部使用，不用了解。
- Poll: 轮询，不断检查有没有新的IO事件，事件循环可能会在这里阻塞。
- Check:处理setImmediate()事件的回调。
- close callback: 处理一些close相关的事件，例如socket.on('close', ...)。

> 参考资料：[https://blog.csdn.net/i10630226/article/details/81369841](https://blog.csdn.net/i10630226/article/details/81369841)

事件循环并没有什么特别之处，本质上就是不同方法的顺序调用，用代码描述一下大约就是：

```
while(true) {
	// ...
	uv__run_timers();
	
	uv__run_pending(loop);
	
	uv__run_idle();
	
	uv__io_poll();
	
	uv__run_check();
	
	uv__run_closing_handles();
	//...
}
```

上述每一个方法即代表一个“阶段”。假设事件循环现在进入了某个阶段（即开始执行上面其中一个方法），即使在这期间有其他队列中的事件就绪，也会先将当前阶段队列里的全部回调方法执行完毕再进入到下个阶段，结合代码这就比较容易理解了。

#### timers

这个阶段主要用来处理定时器相关的回调，当一个定时器超时后，一个事件就会加入到队列中，事件循环会跳转至这个阶段执行对应的回调函数。

定时器的回调会在触发后尽可能早（as early as they can）地被调用，这表示实际的延时器可能会比定时器规定的时间要长。

如果事件循环，此时正在执行一个比较耗时的callback，例如处理一个比较耗时的循环，那么定时器的回调只能等当前回调执行结束了才能被执行，即被阻塞。事实上，timers阶段的执行受到poll阶段控制。

#### IO callbacks阶段

官方文档对这个阶段的描述为除了timers、setImmediate，以及close操作之外的大多数的回调方法都位于这个阶段执行。事实上从源码来看，该阶段只是用执行pending callback，例如一个TCP socket执行出现了错误，在一些*nix系统下可能希望稍后再处理这里的错误，那么这个回调就会放在IO callback阶段来执行。

一些常见的回调，例如fs.readFile的回调是放在poll阶段来执行的。

#### poll阶段

poll阶段的主要任务是等待新的事件出现（该阶段用epoll来获取新的事件），如果没有，事件循环可能会在此阻塞（关于是否在poll阶段阻塞以及阻塞多长时间，libuv有一些复杂的判定方法，这里不作深究，如有兴趣，可以参考libuv源码文件src/unix/core.c下的uv_run方法，该方法是事件循环的核心方法）。

Poll阶段主要有两个步骤如下：

（1）如果有到期的定时器（readFile、IO读取之类函数的执行结束），那么就执行定时器的回调方法。

（2）处理poll阶段对应的事件队列（以下简称poll队列）里的事件。

当事件循环到达poll阶段时，如果这时没有要处理的定时器的回调方法，则会进行下面的判断：

（1）如果poll队列不为空，则事件循环会按照顺序遍历执行队列中的回调函数，这个过程是同步的。（类似readFile、IO读取之类的回调会在poll阶段执行）

（2）如果poll队列为空，会接着进行如下判断：

- 如果当前代码定义了setImmediate方法，事件循环会离开poll阶段，然后进入check阶段去执行setImmediate方法定义的回调方法。
- 如果当前代码没有定义setImmediate方法，那么事件循环可能会进入等待状态，并还会不断检查是否有相关的定时器(setTimout之类的定时器)超时，如果有，就会跳转到timers阶段，然后执行对应的回调。

#### check阶段

setImmediate是一个特殊的定时器方法，它占据了事件循环的一个阶段，整个check阶段就是为setImmediate方法而设置的。

一般情况下，当事件循环到达poll阶段后，就会检查当前代码是否调用 了setImmediate，但如果一个回调函数是被setImmediate方法调用的，事件循环就会跳出poll阶段进而进入check阶段。

#### close阶段

如果一个socket或者一个句柄被关闭，那么就会产生一个close事件，该事件会被加入到对应的队列中。close阶段执行完毕后，本轮事件循环结束，循环进入到下一轮。

看完上面的描述，我们明白了Node中的事件循环是分阶段处理的，对于每一阶段来说，处理事件队列中的事件就是执行对应的回调方法，每一阶段的事件循环都对应着不同的队列。

在Node中，事件队列不止一个，定时器相关的事件和磁盘IO产生的事件需要不同的处理方式，如果把所有的事件都放在一个队列里，那样要增加许多类似switch/case的代码；那样的话倒不如将不同类型的事件归类到不同事件队列里，然后一层层的遍历下来，如果当中出现新的事件，就进行相应的处理。

这个例子：

```
var fs = require('fs');
var timeoutScheduled = Date.now();

setTimeout(function(){
var delay = Date.now() - timeoutScheduled;
console.log(delay + "ms have passed since I was scheduled");
}, 100);

fs.readfile('/path/to/file', function(err, data) {
  var startCallback = Date.now();
  // 使用while是循环阻塞10ms
  while(Date.now() - startCallback < 10) {
    ;
  }
})
```

上面的代码开始运行后,事件循环也开始运作。

首先检查timers，然而timers对的事件队列目前为空（100ms后才会有事件产生），事件循环向后执行到了poll阶段，到目前为止还没有事件出现，由于代码中没有定义setImmediate操作，事件循环便在此一直等待新的事件出现。

直到95ms后（假设readFile耗费的时间为95ms，实际上可能比这个时间长或短一些），readFile读取文件完毕，产生的一个事件，加入到了poll这一队列中，此时事件循环将队列中的事件取出，准备执行之后的callback（此时err和data的值已经就绪），readFile方法什么都没有做，只是暂停了10ms。

事件循环本身也是被阻塞10ms，按照正常的思维，95ms+10ms=105ms>100ms，timers队列中的事件已经就绪，应该先执行对应的回调方法才是，然而由于事物也是单线程运行的，因此也会停止10ms，如果readFile的回调函数中包含了一个死循环，那么整个事件循环都会被阻塞，setTimeout的回调永远不会执行。

readFile的回调完成后，事件循环切换到timers阶段，接着取出timers队列中的事件执行对应的回调方法。

### process.nextTick

process.nextTick的意思是定义出一个异步动作，并且让这个动作在事件循环当前阶段(Phase)结束后执行。

例如下面的代码，将打印first的操作放在nextTick的回调中执行，最后先打印出next，再打印出first。

```
process.nextTick(function(){
	console.log('first');
}
console.log('next');
// next
// first
```

process.nextTick其实并不是事件循环的一部分，但它的回调方法也是由事件循环调用的，该方法定义的回调方法会被加入到名为nextTickQueue的队列中。在事件循环的任何阶段，如果nextTickQueue不为空，都会在当前阶段操作结束后优先执行nextTickQueue中的回调函数，当nextTickQueue中的回调方法被执行完毕后，事件循环才会继续向下执行。

Node限制了nextTickQueue的大小，如果递归调用了process.nextTick，那么当nextTickQueue达到最大限制后会抛出一个错误，我们可以写一段代码来证实这一点。

```
function recurse(i) {
	while(i<9999) {
		process.nextTick(recurse(i++);
	}
}
recurse(0);
```

运行上面代码，马上就会出现：
`RangeError: Maxinum call stack size exceeded`
的错误。

既然nextTickQueue也是一个队列，那么先被加入队列的回调会先执行，我们可以定义多个
process.nextTick，然后观察他们的执行顺序：
```
process.nextTick(function(){
	console.log('first');
});
process.nextTick(function(){
	console.log('second');
});
console.log('next');
// next
// first
// second
```

和其他回调函数一样，nextTick定义的回调也是由事件循环执行的，如果nextTick的回调方法中出现了阻塞操作，后面的要执行的回调同样会被阻塞。

```
process.nextTick(function(){
	console.log('first');
	// 由于死循环的存在，之后的事件被阻塞
	while(true){}
});
process.nextTick(function(){
	console.log('second');
});
console.log('next');
// 依次打印next first， 不会打印second
```

### nextTick与setImmediate

setImmediate方法不属于ECMAScript标准，而是Node提出的新方法，它同样将一个回调函数加入到事件队列中，不同于setTimeout和setInterval，setImmediate并不接受一个时间作为参数，setImmediate的事件会在当前事件循环的结尾触发，对应的回调方法会在当前事件循环末尾(check阶段)执行。

setImmediate方法和process.nextTick方法很相似，二者经常被拿来放在一起比较，由于process.nextTick会在当前操作完成后立刻执行，因此总会在setImmediate之前执行。

### setImmediate和setTimeout

通过上面的内容，我们已经知道了setImmediate方法会在poll阶段结束后执行，而setTimeout会在规定的时间到期后执行，由于无法预测执行代码时循环当前处于哪个阶段，因此当代码中同时存在这两个方法时，回调函数的执行顺序不是固定的。

```
setTimeout(function() {
	console.log('timeout');
})

setImmediate(function(){
	console.log('immediate');
});
```

但如果将二者放在一个IO操作的callback中，则永远是setImmediate先执行：

```
require('fs').readFile('foo.txt', function() {
  setTimeout(() => {
    console.log('timeout')
  }, 0);
  setImmediate(function(){
    console.log('immediate')
  })
})
```

这是因为readFile的回调方法执行时，事件循环位于poll阶段，因此事件循环会先进入check阶段执行setImmediate的回调，然后再进入times阶段执行setTimeout的回调。

> 更多例子：[简单-Node.js Event Loop 的理解 Timers，process.nextTick()](https://www.jianshu.com/p/d6a47421a3e2)


## 模块化

### JavaScript的模块规范

#### CommonJS

CommonJS将每个文件都看作一个模块，模块内部定义的变量都是私有的，无法被其它模块使用，除非使用预定义的方法将内部的变量暴露出来（通过exports和require关键字来实现），CommonJS最为出名的实现就是Node.js。

CommonJS一个显著特点就是模块的加载是同步的，就目前来说，受限于宽带速度，并不适用于浏览器中的JavaScript。

#### AMD

AMD是Asynchronous Module Definition的缩写，意思就是“异步模块定义”。它采用异步方式加载模块，模块的加载不影响它后面语句的运行。依赖这个模块的代码定义在一个回调函数中，等到加载完成后，这个回调函数才会运行。目前在前端流行的RequireJS就是AMD规范的一种实现。

#### ES6的模块化标准

ES6同样使用export关键字来导出变量，但和CommonJS略有不同。

```
export cosnt Name = "Lear";
export const Age = 10;
```

也可以将多个要导出的对象打包到一起：

```
const Name = "Lear";
const Age = 10;
export { Name, Age };
```

使用export 导出量变时，通常要使用花括号将其包裹起来，否则会出现错误：

```
export const Name = "Lear"; // right

const Name = "Lear";
export { Name }; // right

const Name = "Leasr";
export Name; // wrong
```

导出一个方法可以这样：

```
export function add(x, y) {
	return x + y;
}
```

导出方法时同样要使用花括号包裹起来：

```
function add(x, y) {
	return x + y;
}
export {add};
```

导入模块使用的关键字为improt,**import命令同样会执行目标模块的内部代码**：

```
import Name from module
import { Name, Age } from module
```

### Require的隐患

当调用require加载一个模块时，模块内部的代码都会被调用，有时候这可能会带来隐藏的bug。例如下面的例子：

```
// module.js
function test() {
	setInterval(function() {
		console.log('test');
	}, 1000);
}
test();

module.exports = test;

// run.js
var test = require('./module.js');
```

run.js除了加载一个模块之外没有进行任何操作，试着运行一下会发现会每隔一秒输出test字符串，同时run.js进程不会退出。

加载一个模块相当于执行模块内部的代码，在module.js中由于设置了一个不间断的定时器，导致run.js也会一直运行下去。

上面是一个极端的例子，设想一种情景，当你调用某个别人已经编写完成的模块时，明明所有的调用都已经结束，但调用者进程无论如何都不会退出，这很可能是被调用的模块内部有一个隐蔽的循环或者一个一直打开的数据库连接，这个问题在开发过程中可能不会被注意到或者不会被触发，如果真正到了生产环境，这种情况可能导致严重的内存泄露。

这一方面告诉我们要对引用未知的模块保持警惕，即使那个模块是你自己写的；另一方面也提示了测试的重要性。

### 模块化与作用域

模块化中的作用域重要关注点在this上，Node和JavaScript中的this指向有一些区别，其中Node控制台和脚本文件的策略也不一样。对于浏览器中的JavaScript来说，无论是在脚本或者是Chrome控制台中，其this的指向和行为都是一致的；而Node则不是这样。

1. 控制台中的this
	首先是全局的this，分别在Node Repl和Chrome控制台中运行：
	
	```
	console.log(this);
	// Chrome 输出 Window {stop: function, open: function, alert: function, confirm: function...}
	// Node 输出 global对象
	```
	可以看出，在Node Repl环境中，全局的this指向global对象。
	继续运行下面的代码：
	
	```
	var a = 10;
	cosnole.log(this.a);
	// Chrome 输出 10
	// Node Repl 输出10
	```
	
	在Node控制台中，全局变量会被挂载到global中。
	
2. 脚本中的this
	我们新建一个名为this.js的文件，在文件中添加如下 代码:
	
	```
	console.log(this);; // {}
	```
	
	运行node this.js，打印出的结果是一个空对象。
	然后是下面的代码：
	
	```
	var a = 10;
	console.log(this.a); // undefined
	console.log(global.a); // undefined
	```
	
	仍然全都是undefined，说明第一行的代码定义的变量a并没有挂载在全局的this或者global对象。
	
	然而如果声明变量时不使用var或者let关键字，例如下面的代码：
	
	```
	a = 10;
	console.log(global.a);
	```
	
	却可以正常打印出结果。那么在Node脚本文件中定义的全局this又指向了什么呢？答案是module.exports。
	
	```
	this.a = 10;
	console.log(module.exports); // 10
	```
	
	总结一下，在Node repl环境中控制台的全局this和global可以看作是同一个对象，而在脚本文件中，二者并不等价。


## ES6
### Iterator

在Java中，想遍历一个Map,会这样写：

```Java
Map<String,String> = map = new HashMap();
Iterator<String> itor = map.keySet().Iterator();
while(itor.hasNext()){
  String key = itor.next();
  String value = map.get(key);
  System.out.println(key + "---" + value);
}
```

#### ES6中的Iterator

ES6中的Iterator接口通过Symbol.iterator属性来实现，如果一个对象设置了Symbol.iterator属性，就表示该对象是可以被遍历的，我们就可以用next方法来遍历它。

```JavaScript
var Iter = {
  [Symbol.iterator]: function() {
    var i = 0;
    return {
      next: function() {
        return i++;
      }
    };
  }
}

var obj = new Iter[Symbol.iterator]();
obj.next(); // 1
obj.next(); // 2
```

#### Iterator的遍历

在ES6中，所有内部实现了Symbol.iterator接口的对象都可以使用for/of循环进行遍历。

```JavaScript
function myIter(array) {
  this.array = array;
}

myIter.prototype[Symbol.iterator] = function() {
  let index = 0;
  var next = () => {
    if (index < this.array.length) {
      return {
        value: this.array[index++],
        done: false
      }
    } else {
      return {
        value: undefined,
        done: true
      }
    }
  }
  return { next: next }
}

var myiter = new myIter(['a', 'b']);
for(var i of myiter) {
  console.log(i); // 依次打印 a b
}
```

### 3.9类的继承

#### ES5中的继承

首先假设有一个父类Person，并且在类的内部和原型链上各定义了一个方法；

```JavaScript
function Person(name, age) {
  this.name = name;
  this.age = age;
  this.greed = function() {
    console.log('Hello, I am ', this.name);
  }
}

Person.prototype.getInfo = function() {
  return this.name + ',' + this.age;
}
```

1. 修改原型链

这是最普遍的继承做法，通过将子类的prototype指向父类的实例来实现：

```JavaScript
function Student() {

}

Student.prototype = new Person();
Student.prototype.name = 'Lear';
Student.prototype.age = 10;
var stu = new Student('Lear', 10);
stu.greed();
stu.getInfo();
```

在这种继承中，stu对象既是子类的实例，也是父类的实例。然而也有缺点，在子类的构造函数中无法通过传递参数对父类继承的属性进行修改，只能通过修改prototye的方式进行修改。

2. 调用父类的构造函数

```JavaScript
function Student(name, sex, age) {
  Person.call(this);
  this.name = name;
  this.sex = sex;
  this.age = age;
}

var stu = new Student('Lear', 'male', 10);
stu.greed(); // Hello, I am Lear
stu.getInfo(); // Error
```

这种方式避免了原型链继承的缺点，直接在子类中调用父类的构造函数，在这种情况下，stu对象只是子类的实例，不是父类的实例，而且只能调用父类实例中定义的方法，不用调用父类原型链上定义的方法。

3. 组合继承

这各继承方式是前面两种继承方式的合体。

```JavaScript
function Student(name, age, sex) {
  this.name = name;
  this.age = age;
  this.sex = sex;
}

Student.prototype = new Person();
var stu = new Student('Lear', 10, 'male');
stu.greed();
stu.getInfo();
```

这种方式结合上面两种继承方式 的优点，也是Node源码中标准的继承方式。
唯一的问题是调用了父类的构造函数两次，分别是在设置子类的prototype和实例化子类新对象时调用的，造成了一定的内在浪费。

```JavaScript
function* Generation() {
  yield 'hello world';
  yield 'from lear';
  return 'end';
}

var gen = Generation();
for(let i of gen) {
  console.log(i);
}

Array.from(Generation());
```

## 使用Koa2构建站点

### Koa入门

#### context对象

Node提供了request(IncomingMessage) 和 response(ServerResponse)两个对象，Koa把两者封装到同一个对象中，即context，缩写ctx。

除了自行封装的属性外，ctx也提供了直接访问原生对象的手段，ctx.req和ctx.res即代表原生的request和response对象，例如ctx.req.url和ctx.url就是同一个对象。

除了上面列出的属性外，ctx对象还自行封装了一些对象，例如ctx.request和ctx.response，它们和原生对象之间的区别在于里面只有一部分常用的属性。但是二者的结构却有很大的区别，ctx.response只有最基本的几个属性，上面没有注册任何事件和方法，这表示下面的使用方法是错误的：

`fs.createReadStream('foo.txt').pipe(ctx.response)`

上面代码会抛出TypeError: dest.on is not a function的错误，原因也很简单，ctx.response只是一个简单的对象，没有定义任何事件，要使用pipe方法，则要改成：

`fs.createReadStream('foo.txt').pipe(ctx.res)`

#### ctx.state

state属性是官方推荐的命名空间，如有开发者从后端的消息想要传递到前端，可以将属性挂在ctx.state下面，这和react中的概念有些相似。例如我们从数据库中查找一个用户的id:

`ctx.state.user = await User.find(id)`

#### 处理http请求

Koa在ctx对象中封装了request以及response对象，那么在处理请求的时候，使用ctx就可以完成所有的处理了。例如：

```JavaScript
ctx.body = 'hello world';
```
相当于：

```JavaScript
res.statusCode = 200;
res.end('hello world');
```

ctx相当于ctx.request和ctx.response的别名，判断http请求类型可以通过ctx.method来进行判断，get请求的参数可以通过ctx.query获取。

Koa处理get请求可以直接通过ctx.req.< param >就能拿到get参数的值，post请求的处理要使用bodyParser这一个中间件，但也**仅限于普通表单**，获取格式为ctx.request.body.< param >（文件上传不一样）。

### middleware

#### 中间件的概念

Express 本身就是由路由和中间件构成的，从本质上来说，Express的运行就是在不断调用各种中间件。

中间件本质上是接收请求并且做出相应动作的函数，该函数通常接收req和res作为参数，以便对request和response对象进行操作，在web应用中，客户端发起的每一个请求，首先要经过中间件处理才能继续向下。

中间件的第三个参数是next，它表示下一个方法，即下一个中间件，如果没有调用next方法，则表示处理到此为止，下一个中间件不会执行。

#### 中间件的功能和执行

由于中间件是一个函数，所以它可以做到Node代码能做到的任何事，还包括修改request和respnose对象、终结请求-响应循环，以前调用下一个中间件等功能。

中间件的加载使用use方法来实现。

Expresss应用可以使用如下几种中间件：

- 应用级中间件
- 路由级中间件
- 错误处理中间件
- 内置中间件
- 第三方中间件

上面是官网的分类，实际上这几个概念有一些重合之处。

1. 应用级中间件

使用app.use方法或者app.METHOD(Method表示http方法，即get/post)绑定在app对象上的中间件。

```JavaScript
var app = require('express')();

//use 方法
app.use(function(req, res, next) {
  console.log('Time', Date.now());
  next();
});

// http 方法
app.use('/user/:id', function(req, res) {
  console.log('http method');
})
```

2. 路由级中间件

和Koa不同，路由处理是Express的一部分，通常通过router.use方法来绑定到router对象上：

```JavaScript
var app = express();
var router = express.Router();

router.use('/login', function() {
  console.log('Time:', Date.now());
  next();
});
```

3. 错误处理中间件

错误处理中间件有4个参数，即使不需要通过next方法来调用下一个中间件，也必须在参数列表中声明它，否则中间件会被识别为一个常规中间件，不能处理错误。

```JavaScript
app.use(funciton(err, req, res, next) {
  console.log(err.stack);
  res.status(500).send('Something broke');
});
```

4. 内置中间件

从4.x版本开始，Express已经不再依赖Connect了。除了负责管理的静态资源的static模块外，Express以前内置的中间件现在已经全部单独作为模块安装使用。

5. 第三方中间件

第三方中间件可以为Express应用增加更多功能，通常通过npm来安装，例如获取Cookie信息常用 的cookie-parser模块，或者解析表单用的bodyParser等。

#### 中间件的串行调用

在web开发中，我们通常希望一些操作能够串行执行，例如等待写入日志完成后再进行数据库操作，最后再进行路由处理。

在技术层面，上面的业务场景表现为串行调用某些异步中间件。
比较容易想到的一种做法是把next方法放到回调里面，但如果异步操作一多就会形成回调地狱。

下面的代码定义了两个Express中间件，和之前不同这处于第二个中间件中调用了process.nextTick()，表示这是一个异步操作。

```JavaScript
var app = require('express')();
app.use(function(req, res, next) {
  next();
  console.log("I am middleware1");
});

app.use(function(erq, res, next) {
  process.nextTick(function() {
    console.log('I am midlleware');
    next();
  });
});

app.listen(3000);
```

## 爬虫系统的开发

### 爬虫技术概述

### 技术栈