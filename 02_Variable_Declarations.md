# 变量声明

`let`和`const`是JavaScript中两种相对较新的变量声明方式。
正如我们之前所提到的，`let`和`var`在一些方面是相似的，但它能让使用者避免一些在JavaScript中常见的陷阱。
`const`是`let`的一个强化版，用于阻止重新对变量进行赋值。

TypeScript作为JavaScript的超集，其理所当然的支持`let`和`const`。
这里我们将更多地阐述这些新的声明方式，以及为何它们优于`var`的方式。

如果你以毫不在意的方式使用JavaScript，那下一节内容将会很好地刷新你的记忆。
如果你非常熟悉JavaScript中`var`声明方式的各种怪癖，你可能这些内容很容易被接受。

# `var` 声明方式

在JavaScript中声明变量一直使用传统的`var`关键字来完成。

```ts
var a = 10;
```

正如你所想到的，我们声明了一个命名为`a`且值为`10`的变量。

我们同样可以在函数中声明一个变量：

```ts
function f() {
    var message = "Hello, world!";

    return message;
}
```

并且在其他函数中我们照样可以访问该变量：

```ts
function f() {
    var a = 10;
    return function g() {
        var b = a + 1;
        return b;
    }
}

var g = f();
g(); // returns '11'
```

在上面的例子里，`g`捕获到了在`f`中声明的变量`a`。
无论`g`何时被调用，`a`的值将会和`f`中`a`的值绑定。
甚至在`f`完成运行后再去调用`g`，它仍将能够访问和修改`a`。

```ts
function f() {
    var a = 1;

    a = 2;
    var b = g();
    a = 3;

    return b;

    function g() {
        return a;
    }
}

f(); // returns '2'
```

## 作用域规则

`var`声明方式和其他语言相比有一些奇特的作用域规则。
请看下面的例子：

```ts
function f(shouldInitialize: boolean) {
    if (shouldInitialize) {
        var x = 10;
    }

    return x;
}

f(true);  // returns '10'
f(false); // returns 'undefined'
```

一些读者可能会在这个例子上楞了一下。
变量`x`是*在`if`块里*声明的，但是我们可以在块的外部访问它。
那是因为`var`声明方式是可以在包含它们的函数、模块、命名空间或者全局作用域里任意访问的——这些我们会在之后介绍，而不是包含它的块。
一些人将其称为*`var`作用域*或*函数作用域*。
函数参数也同样满足函数作用域。

这些作用域规则可能会引起一些错误。
这会加剧的一个问题是虽然不是错误但是重复的声明同一个变量。

```ts
function sumMatrix(matrix: number[][]) {
    var sum = 0;
    for (var i = 0; i < matrix.length; i++) {
        var currentRow = matrix[i];
        for (var i = 0; i < currentRow.length; i++) {
            sum += currentRow[i];
        }
    }

    return sum;
}
```

可能对于有些人来说这很容易察觉到，在内层的`for`循环会意外地覆盖变量`i`因为`i`指向同一个函数作用域的变量。
正如现在一些有经验的开发者所知道的，在代码审查过程中类似于这种的bug会进入一个无限循环的代码挫折。

## 代码捕获怪癖

来简单的猜测一下，如下代码片段的输出结果是是什么：

```ts
for (var i = 0; i < 10; i++) {
    setTimeout(function() { console.log(i); }, 100 * i);
}
```

为不熟悉的人介绍下，`setTimeout`会尝试在经过特定毫秒后运行一个函数（但是会先等待其他事务完成运行）。

准备好了吗？让我们来看看：

```text
10
10
10
10
10
10
10
10
10
10
```

许多JavaScript开发者已经暗自熟悉于这种行为，但是如果你感到惊奇，你并不是一个人。
许多人期待的输出应该是这样的：

```text
0
1
2
3
4
5
6
7
8
9
```

还记得关于变量捕获我们之前说了什么吗？
每一个我们传递给`setTimeout`的函数表达式，事实上指向了同一个作用域中的同一个`i`。

让我们花点时间来考虑这其中的意义。
`setTimeout`将会在一定毫秒数后运行一个函数，*但仅仅*发生在`for`循环停止运行后；
在`for`循环停止的时候，`i`的值将会变为`10`。
所以在每次调用给定函数的时候，它都将打印出`10`！

一个常见的解决办法是使用IIFE——立即调用函数表达式（Immediately Invoked Function Expression），用来在每次迭代中捕获`i`：

```ts
for (var i = 0; i < 10; i++) {
    // capture the current state of 'i'
    // by invoking a function with its current value
    (function(i) {
        setTimeout(function() { console.log(i); }, 100 * i);
    })(i);
}
```

这种看上去古怪的方式事实上是很常见的。
参数列表中的`i`实际上遮蔽了`for`循环中声明的变量`i`，我们将它们命名成一样的是因为这样就不用对循环体做太多的修改。

# `let`声明方式

你现在应该能知道使用`var`存在一些问题，而这些问题成为了引入`let`语句的原因。
除了使用的关键字不同，`let`语句和`var`语句使用同样的书写方式。

```ts
let hello = "Hello!";
```

两者的区别不在于语法上，而是在于语义上，我们接下去会对此进行深入。

## 块作用域

当使用`let`声明了一个变量，它的用法被一些人称为*词法作用域*或*块作用域*。
使用`var`声明的变量会将其作用域泄漏到所在的函数，不同与此，块作用域的变量在包含它最近的块或`for`循环外不可见。

```ts
function f(input: boolean) {
    let a = 100;

    if (input) {
        // Still okay to reference 'a'
        let b = a + 1;
        return b;
    }

    // Error: 'b' doesn't exist here
    return b;
}
```

在这里我们声明了两个局部变量`a`和`b`。
`a`的作用域被限定在`f`的范围内而`b`的作用域被限定在`if`语句块的范围内。

在`catch`语句范围内的变量声明也有相似的作用域规则。

```ts
try {
    throw "oh no!";
}
catch (e) {
    console.log("Oh well.");
}

// Error: 'e' doesn't exist here
console.log(e);
```

块作用域变量的另一个属性是它们不能在被实际声明前进行读写操作。
虽然这些变量“现在”遍布于它们的作用域，但是直到它们的声明部分位置都是他们的*暂时性死区*。
这仅仅是一种复杂的方式来说明你在`let`语句之前不能访问它们，然后幸运的是TypeScript会让你知道这点。

```ts
a++; // illegal to use 'a' before it's declared;
let a;
```

需要注意的是你依然可以在声明前*捕获*一个快作用域的变量。
唯一美中不足的是在声明前调用那个函数是非法的。
如果针对ES2015，一个现代化的运行时会抛出一个错误；无论如何，现在TypeScript现在是允许这样做的，且不会将其报告为错误。

```ts
function foo() {
    // okay to capture 'a'
    return a;
}

// illegal call 'foo' before 'a' is declared
// runtimes should throw an error here
foo();

let a;
```

获取更多关于暂时性死区的信息，请访问相关链接：[Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#Temporal_dead_zone_and_errors_with_let).

## 重声明和遮蔽

在使用`var`声明方式是，我们会注意到对于某个变量无论声明，你会得到只有一个变量。

```ts
function f(x) {
    var x;
    var x;

    if (true) {
        var x;
    }
}
```

在上面的例子里，所有对`x`的声明实际上指向了*同一个*`x`，而且它是完全有效的。
这往往会成为bug的一个来源。
值得庆幸的是，`let`声明方式并不会如此宽松。

```ts
let x = 10;
let x = 20; // error: can't re-declare 'x' in the same scope
```

变量在TypeScript中并不需要同时作为块作用域来告诉我们这里有个问题。

```ts
function f(x) {
    let x = 100; // error: interferes with parameter declaration
}

function g() {
    let x = 100;
    var x = 100; // error: can't have both declarations of 'x'
}
```

但这并不是说块作用域变量永远不能作为函数作用域变量被声明。
块作用域变量仅仅是需要在确实不同的块中被声明。

```ts
function f(condition, x) {
    if (condition) {
        let x = 100;
        return x;
    }

    return x;
}

f(false, 0); // returns '0'
f(true, 0);  // returns '100'
```

在多层嵌套作用域中引入一个新的的名称的行为被称为*遮蔽*。
这稍微有点双刃剑的效果因为它会在偶发的遮蔽事件中引入一些Bug，同样也会阻止某些Bug的发生。
例如，想象一下使用`let`来书写我们之前写过的`sumMatrix`函数。

```ts
function sumMatrix(matrix: number[][]) {
    let sum = 0;
    for (let i = 0; i < matrix.length; i++) {
        var currentRow = matrix[i];
        for (let i = 0; i < currentRow.length; i++) {
            sum += currentRow[i];
        }
    }

    return sum;
}
```

这个版本的循环将会正确的执行求和因为内循环的`i`遮蔽了外循环的`i`。

如果致力于书写更为清晰的代码，则应*尽量*避免使用遮蔽。
如果存在某些场景需要用到遮蔽的优点，你应当使用你的最佳判断。

## 捕获块作用域的变量

当我们通过`var`声明方式第一次接触变量捕获的时候，我们简要地了解了一下当变量被捕获后是如何运作的。
更直观的说法是，在每次运行一个作用域的时候，它都会创建一个变量的“环境”。
这个环境和它捕获的变量甚至在其所在的作用域全部结束运行后依然可以存在。

```ts
function theCityThatAlwaysSleeps() {
    let getCity;

    if (true) {
        let city = "Seattle";
        getCity = function() {
            return city;
        }
    }

    return getCity();
}
```

因为我们在`city`所在的环境中捕获了它，因此尽管`if`块已经结束运行，我们依然可以访问到它。

回顾我们之前的`setTimeout`例子，我们最终需要使用IIFE来捕获`for`循环每个嵌套中变量的状态。
事实上，我们所做的事情是为我们捕获的变量创建了一个新的变量环境。
这样子做会有点痛，但是幸运的是，你并不必在TypeScript中做同样的事情。

`let`声明方式在作为一个循环中的一部分使用的时候，其行为存在巨大的不同。
不同于仅仅为循环本身引入一个新的环境，这些声明方式为*每一次迭代*都创建了一个新的作用域。
因为这正是我们通过IIFE所做的事情，因此我们可以将老的`setTimeout`例子通过`let`声明方式进行改写。

```ts
for (let i = 0; i < 10 ; i++) {
    setTimeout(function() { console.log(i); }, 100 * i);
}
```

并且正如预期的，将打印出如下内容。

```text
0
1
2
3
4
5
6
7
8
9
```

# `const`声明方式

`const`声明方式是另一种声明变量的方式。

```ts
const numLivesForCat = 9;
```

它们和`let`声明相似，不同点在于变量值一但被绑定后将不再能够修改，正如其名字所暗示的。
换言之，它们和`let`有相同的作用域规则，但你不能对它们重新赋值。

这不应和它所指向的值是*永恒不变*的想法互相混淆。

```ts
const numLivesForCat = 9;
const kitty = {
    name: "Aurora",
    numLives: numLivesForCat,
}

// Error
kitty = {
    name: "Danielle",
    numLives: numLivesForCat
};

// all "okay"
kitty.name = "Rory";
kitty.name = "Kitty";
kitty.name = "Cat";
kitty.numLives--;
```

除非你采取具体手段来进行避免，一个`const`变量的内部状态依然是可修改的。
幸运的是，TypeScript允许你指定对象中的成员为readonly`的。
在[接口的章节](./Interfaces.md)中包含更多细节。

# `let` vs. `const`

现在我们有两种作用域语义相似的两种声明方式，那自然会产生哪个更好用的疑问。
和大多数广泛的问题一样，答案是：基于需求。

根据[最小特权原则](https://en.wikipedia.org/wiki/Principle_of_least_privilege)，所有的声明，除了你准备修改其值的以外，都应当使用`const`。
这个理论的基础是如果一个变量并不需要去写入，那其他基于相同代码作业的人不应该自动获得去写入这个对象的能力，并需要去考虑是否需要真的要对该变量赋值。
使用`const`还能在分析数据流的时候让代码更为可预测。

另一方面，使用`let`并不会比`var`写更多东西，这会让使用者感到便利。
基于这种兴趣，本手册的大部分内容使用了`let`声明方式。

使用你最佳的判断，然后如果可以的话，请和你团队的其它成员进行沟通。

# 解构

另一个TypeScript实现的ECMAScript 2015特性是解构。
要查看完整引用，请参见[Mozilla Developer Network上的文章](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)。
在这一节中，我们将做简短的概述。

## 数组解构

解构最简单的形式就是数组解构赋值：

```ts
let input = [1, 2];
let [first, second] = input;
console.log(first); // outputs 1
console.log(second); // outputs 2
```

这段代码创建了两个新的变量，分别叫作`first`和`second`。
这和使用索引的方式等价，但它更为便捷：

```ts
first = input[0];
second = input[1];
```

解构同样可以和已经声明的变量一同使用：

```ts
// swap variables
[first, second] = [second, first];
```

以及作为函数的参数：

```ts
function f([first, second]: [number, number]) {
    console.log(first);
    console.log(second);
}
f(input);
```

你可以使用`...name`语法来为列表中的剩余项目创建一个变量：

```ts
let [first, ...rest] = [1, 2, 3, 4];
console.log(first); // outputs 1
console.log(rest); // outputs [ 2, 3, 4 ]
```

当然，因为这是JavaScript，你可以忽略你不不关心的剩余元素：

```ts
let [first] = [1, 2, 3, 4];
console.log(first); // outputs 1
```

或者任意元素：

```ts
let [, second, , fourth] = [1, 2, 3, 4];
```

## 对象解构

你同样可是使用解构对象：

```ts
let o = {
    a: "foo",
    b: 12,
    c: "bar"
}
let {a, b} = o;
```

这段代码基于`o.a`和`o.b`创建了`a` and `b`两个变量。
要注意到如果你不需要`c`那你可以跳过它。

和数组解构类似，你可以不声明即进行赋值：

```ts
({a, b} = {a: "baz", b: 101});
```

注意到我们使用了小括号包围了表达式。
JavaScript通常将`{`解析为代码块的开始。

### 属性重命名

你同样可以给属性赋予不同的名称：

```ts
let {a: newName1, b: newName2} = o;
```

这里的代码可能会让人困惑。
你可以把`a: newName1`理解成“`a`作为`newName1`”。
这里的方向为从左到右，正如你会这样写：

```ts
let newName1 = o.a;
let newName2 = o.b;
```

令人困惑的，这里的冒号*并不是*指代类型。
类型，如果你希望进行制定，依然需要被写在整个解构的后面：

```ts
let {a, b}: {a: string, b: number} = o;
```

### 默认值

默认值能让你在属性值为undefined时指定一个默认值：

```ts
function keepWholeObject(wholeObject: {a: string, b?: number}) {
    let {a, b = 1001} = wholeObject;
}
```

`keepWholeObject`现在既有一个叫做`wholeObject`的变量，又有两个属性`a`和`b`，哪怕`b`是undefined状态。

## 函数声明

解构同样适用于函数声明。
对于简单的情况，这样是最直接的：

```ts
type C = {a: string, b?: number}
function f({a, b}: C): void {
    // ...
}
```

但是指定默认值更常见于参数，而且用解构来获得默认值是很微妙的。
首先，你需要记得将类型放在默认值之前。

```ts
function f({a, b} = {a: "", b: 0}): void {
    // ...
}
f(); // ok, default to {a: "", b: 0}
```

然后，你需要记得在解构属性里给可选参数一个默认值而不是在主初始化里。
记住`C`是使用可选的`b`定义的：

```ts
function f({a, b = 0} = {a: ""}): void {
    // ...
}
f({a: "yes"}) // ok, default b = 0
f() // ok, default to {a: ""}, which then defaults b = 0
f({}) // error, 'a' is required if you supply an argument
```

小心地使用解构。
正如之前的例子所展示的，除了最简单那种表达式，任何解构表达式都是充满了拐弯抹角的情况的。
这在深层嵌套解构中尤为明显，即使不堆叠重命名、默认值和类型注解，这依然会变得*十分*难以理解。
尝试让解构表达式尽可能小而简单。
你可以一直手动分配而让解构由你自己生成。
