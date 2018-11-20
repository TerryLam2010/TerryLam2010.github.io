# 你搞明白yield是个啥没？

ES6真是颠覆JavaScript的东西，随便翻一个新特性出来，就让自以为是的老古董们傻眼跳楼。在之前接触ember.js的时候，接触到了yield，嫩是半天没明白，yield到底是什么，想要实现什么目的。后来在看ES6的东西的时候，总算好像知道了一点，迫不及待的写出来。

在MDN上，对yield的第一句解释就是：

```
The yield keyword is used to pause and resume a generator function.
// yield这个关键字是用来暂停和恢复一个遍历器函数（的运行）的。
```

这也就是yield的所有解释了，可谓大道至简，然并卵，深层的意思不去挖掘，根本还是没法用它，还是老老实实做老古董。

# 关键字yield

没错，yield是个关键字，不是函数。关键字用来干啥？它的作用是“命令”。和var不同，不是用来声明，但是和return一样，用来告知程序某种状态，return告诉程序要返回什么值（也意味着结束，结束的时候才会返回值嘛），而yield告诉程序当前的状态值，而且你运行到这里给我暂停一下。

因为yield是命令型的关键字，所以它的用法是：

```
[rv] = yield [expression];
```

rv是可选的，这里不是说它返回一个数组。yield后面的表达式也是可选的。yield的返回值是一个状态值。如果从返回值的角度讲，yield还可以当做是一种运算符，但是由于它的作用是暂停和恢复，所以严格意义上说，不能叫运算符，运算符是用来运算的，而yield是用来“命令”的。

# 遍历器函数（Generator）

这是ES6里面新增的Generator，为了把它和我们已知的东西联系起来，我把它翻译为遍历器函数，但是实际上现在还没有统一的叫法，坐等大神们开宗立派。

我个人的理解，Generator函数的最大用处就是用来生成一个遍历器。

不过它是一个函数，所以和普通的函数有点区别，因此在声明函数的时候，要在function和函数名之间加一个*号：

```
function *foo() {}
```

而yield也必须在Generator函数中才有意义，脱离了Generator就没意义了啊，这就像说“鲨鱼是用来在海里面消灭傻瓜鱼的”现在你吧鲨鱼丢到陆地上，请问它有啥意义？给人做包包吗？

Generator函数的一个重要特点就是需要执行next()方法才能运行，声明好它之后，根本不会马上运行。举个栗子：

```
var a = 0;
function *foo() {
  a += 1;
  yield '';
  return;
}
var f = foo();
alert(a); // 这个时候是啥值？
```

上面的例子如此简单，你不会看不懂吧？现在告诉你，alert的结果是0！

我前面都说了，要执行netx()方法才会运行。所以在上面的代码末尾添加：

```
f.next();
alert(a); // 这个时候就会alert(1)了
```

没怎明白？继续往下读

# .next()方法

就用上面的代码来说好了。Generator函数的特点就是“它是一个遍历器”，你不让它动，它绝对不会动。

不要用“动”这么猥琐的词好吗？

“执行、运行”，当调用f.next()的时候，程序从f这个遍历器函数的开始运行，当遇到yield命令时，表示“暂停”，并且返回yield [expression]的状态。比如程序在往下运行的时候，遇到yield 1 + 2，那么next()返回的结果就是这个时候的状态。

而f.next()就是让它往下一个元素遍历的动作，它的返回值其实表示一个状态，是一个object：{value: xxx, done: false}。value表示yield后面的表达式的结果值，done表示是否已经遍历完。把上面的f.next()那段代码改成下面的：

```
var s1 = f.next(); 
console.log(s1); // {value: '',done: false}
var s2 = f.next();
console.log(s2); // {value: undefined,done: true}
```

第一个console的地方，因为第一次执行f.next()，遇到yield就暂停了，得到s1这个状态，这个状态的value是yield后面紧跟的表达式的值，done表示遍历没有结束，还可以继续执行next()方法。第二次再执行f.next()的时候，遇到了return，但后面没有表达式，所以返回值是undefined，一旦遇到return就表示遍历可以结束了，所以done为true。

当运行到`yield '';`的时候，程序暂停了，不会往下继续执行，所以下面的各种加减乘除都不会运行，这也就是为什么我们上面的代码在运行f.next()之前，虽然执行了foo()这个函数，但是a的值是0，就是因为还没有执行f.next()，所以yield前面的a += 1还没有运行。

也正是因为有.next()方法，所以它叫遍历器。for...of，...，Array.from都是利用遍历器接口进行运算的，所以如果没有Generator，这几个方法就用不了。而在Babel里面，babel-core本身默认不支持Generator，所以这几个运算其实也需要另外的模块( babel-polyfill )来支持。

一不小心讲到了babel，罪过罪过。

# .next()的参数

造孽的地方终于要来了。一言不合，就上代码：

```
var foo = function *() { // 没错，尼玛还可以这样写
  var x = 1;
  var y =  yield (x + 1);
  var z = yield (x + y);
  return z;
}() // 你必须先执行一下Generator函数，才能把遍历器返回给某个变量
var a = foo.next(); // 第一次执行next()不可以传参
var b = foo.next(3);
var c = foo.next(4);
```

求abc的value各是多少？

你要没接触过，这个时候只会冒出来“吊雷老某”……a.value你应该知道，就2（x = 1, x + 1 = 2）。b.value呢？傻戳戳了吧。

.next()的参数的意思是将传入的参数用作上一次的yield。啥子意思？就是第二次执行foo.next(3)的时候，yield x + 1这一大坨就是3，所以y = 3！“xx老谋”。所以，b.value的结果就是4 （x = 1,y = 3, x + y = 4）.

这样推下去咯。最后返回的是z，但是传入的是4，yield (x + y) 这一大坨就用4来代替，z.value = yield (x + y) = 4。

两个问题：1. 为什么第一次执行next()不能传参？2.为什么yield后面要加括号？

第一个问题，因为第一次执行next()的时候，你传入的参数没法去代替某一个yield啊！

第二个问题，其实可以不用加括号，但是如果不加括号的话，那么后面的+运算就没了，比如说上面的代码改为：

```
var foo = function *() {
  var x = 1;
  var y =  yield (x + 1);
  var z = yield x + y;
  return z;
}()
var a = foo.next();
var b = foo.next(3);
var c = foo.next(4);
```

这差距就大了，yield x是一坨，不会等到你x+y它就会返回，所以b还是1.

# 把yield当变量看

上面这一个小节你有没有发现一个神奇的规律？就是你在给next()传参的时候，总是对应某一个yield，把这个yield以及后面紧跟着的表达式用传入的参数代替。于是乎，下面这个代码你就懂了：

```
var foo = function *() {
  var x = 1;
  var y =  yield (x + 1);
  var z = yield;
  return z;
}()
var a = foo.next();
var b = foo.next(3);
var c = foo.next(4);
```

上面这个代码里面，竟然把yield后面的表达式给去掉了。那会有啥影响？b.value将等于undefined，仅此而已。因为b这个位置就是得到yield [空]的结果，所以就是undefined咯。但是当foo.next(4)执行的时候，不管你上面是不是undefined，我现在就是要用4来覆盖你，所以z.value还是4.

也正因为这样，在字符串里面，可以这样使用：

```
var log = function *() {
  console.log(`you input: ${yeild}`)
}().next(); // 这里会提示错误: yeild undefined
log.next('hello world!');
```

上面这个例子把yield当做一个变量来使用，如果你感兴趣，可以想办法吧第一次的时候错误去除掉。

这个用法看上去很挫，但是很厉害的是，可以通过这个思路，实现字符串模板的次第渲染。可以控制渲染到哪个位置。

# 总结一下

写了这么多，总结一下yield，实际上：

- 只能在Generator函数内部使用
- 运行.next()，遇到一个yield命令，就暂停
- .next()的返回值表示一个状态{value,done}
- 再运行.next()，从之前遇到的那个yield [表达式]处（后）继续恢复运行
- 当.next()传参的时候，yield [表达式]整个被替换为传入的参数。

最后，说一下for...of是怎么运行的。

```
function *foo() {
  yield 1;
  yield 2;
  yield 3;
  return;
}
for(let v of foo()) {
  console.log(v);
}
```

for...of在遍历foo()返回的结果时，每遇到一个yield，就把yield后面表达式的结果作为of前面的v的值进行赋值（next()返回值的value字段）。没错，就这么不要脸的解释完了。