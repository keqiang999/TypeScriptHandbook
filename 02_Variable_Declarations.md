# 变量声明

`let`和`const`是JavaScript中两种相对较新的变量声明方式。
正如我们之前所提到的，`let`和`var`在一些方面是相似的，但它能让使用者避免一些在JavaScript中常见的陷阱。
`const`是`let`的一个强化版，用于阻止重新对变量进行赋值。

TypeScript作为JavaScript的超集，其理所当然的支持`let`和`const`。
这里我们将更多地阐述这些新的声明方式，以及为何它们优于`var`的方式。

如果你以毫不在意的方式使用JavaScript，那下一节内容将会很好地刷新你的记忆。
如果你非常熟悉JavaScript中`var`声明方式的各种怪癖，你可能这些内容很容易被接受。

# `var` 声明方式

Declaring a variable in JavaScript has always traditionally been done with the `var` keyword.
在JavaScript中声明变量一直使用传统的`var`关键字来完成。

```ts
var a = 10;
```

As you might've figured out, we just declared a variable named `a` with the value `10`.
正如你所想到的，我们声明了一个命名为`a`且值为`10`的变量。

We can also declare a variable inside of a function:
我们同样可以在函数中声明一个变量：

```ts
function f() {
    var message = "Hello, world!";

    return message;
}
```

and we can also access those same variables within other functions:
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
g(); // returns 11;
```

In this above example, `g` captured the variable `a` declared in `f`.
At any point that `g` gets called, the value of `a` will be tied to the value of `a` in `f`.
Even if `g` is called once `f` is done running, it will be able to access and modify `a`.
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

f(); // returns 2
```

## 作用域规则

`var` declarations have some odd scoping rules for those used to other languages.
Take the following example:
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

Some readers might do a double-take at this example.
The variable `x` was declared *within the `if` block*, and yet we were able to access it from outside that block.
That's because `var` declarations are accessible anywhere within their containing function, module, namespace, or global scope - all which we'll go over later on - regardless of the containing block.
Some people call this *`var`-scoping* or *function-scoping*.
Parameters are also function scoped.
一些读者可能会在这个例子上楞了一下。
变量`x`是*在`if`块里*声明的，但是我们可以在块的外部访问它。
那是因为`var`声明方式是可以在包含它们的函数、模块、命名空间或者全局作用域里任意访问的——这些我们会在之后介绍，而不是包含它的块。
一些人将其称为*`var`作用域*或*函数作用域*。
函数参数也同样满足函数作用域。

These scoping rules can cause several types of mistakes.
One problem they exacerbate is the fact that it is not an error to declare the same variable multiple times:
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

Maybe it was easy to spot out for some, but the inner `for`-loop will accidentally overwrite the variable `i` because `i` refers to the same function-scoped variable.
As experienced developers know by now, similar sorts of bugs slip through code reviews and can be an endless source of frustration.
可能对于有些人来说这很容易察觉到，在内层的`for`循环会意外地覆盖变量`i`因为`i`指向同一个函数作用域的变量。
正如现在一些有经验的开发者所知道的，在代码审查过程中类似于这种的bug会进入一个无限循环的代码挫折。

## 代码捕获怪癖

Take a quick second to guess what the output of the following snippet is:
来简单的猜测一下，如下代码片段的输出结果是是什么：

```ts
for (var i = 0; i < 10; i++) {
    setTimeout(function() {console.log(i); }, 100 * i);
}
```

For those unfamiliar, `setTimeout` will try to execute a function after a certain number of milliseconds (though waiting for anything else to stop running).
为不熟悉的人介绍下，`setTimeout`会尝试在经过特定毫秒后运行一个函数（但是会先等待其他事务完成运行）。

Ready? Take a look:
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

Many JavaScript developers are intimately familiar with this behavior, but if you're surprised, you're certainly not alone.
Most people expect the output to be
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

Remember what we mentioned earlier about variable capturing?
还记得关于变量捕获我们之前说了什么吗？

> At any point that `g` gets called, the value of `a` will be tied to the value of `a` in `f`.
> 无论`g`何时被调用，`a`的值将会和`f`中`a`的值绑定。

Let's take a minute to consider that in this context.
`setTimeout` will run a function after some number of milliseconds, and also after the `for` loop has stopped executing.
By the time the `for` loop has stopped executing, the value of `i` is `10`.
So each time the given function gets called, it will print out `10`!
让我们来考虑下现状。
`setTimeout`会在经过一定毫秒数后运行一个函数，而这是在`for`循环停止运行后发生的。
在`for`循环停止的时候，`i`的值将会变为`10`。
所以在每次调用给定函数的时候，它都将打印出`10`！

A common work around is to use an IIFE - an Immediately Invoked Function Expression - to capture `i` at each iteration:
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

This odd-looking pattern is actually a commonplace.
The `i` in the parameter actually shadows the `i` declared in the `for` loop, but since we named it the same, we didn't have to modify the loop body too much.
这种看上去古怪的模式实际上是十分常见的。
参数中的`i`实际上遮蔽了`for`循环中声明的变量`i`，我们将它们命名成一样的是因为这样就不用对循环体做太多的修改。

# `let`声明方式

By now you've figured out that `var` has some problems, which is precisely why `let` statements are a new way to declare variables.
Apart from the keyword used, `let` statements are written the same way `var` statements are.
你现在应该能知道使用`var`存在一些问题，而这些问题恰恰是`let`语句成为声明变量的一种新方式的原因。
除了使用的关键字不同，`let`语句和`var`语句使用同样的书写方式。

```ts
let hello = "Hello!";
```

The key difference is not in the syntax, but in the semantics, which we'll now dive into.
两者的区别不在于语法上，而是在于语义上，我们接下去会对此进行深入。

## 块作用域

When a variable is declared using `let`, it uses what some call *lexical-scoping* or *block-scoping*.
Unlike variables declared with `var` whose scopes leak out to their containing function, block-scoped variables are not visible outside of their nearest containing block or `for`-loop.
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

Here, we have two local variables `a` and `b`.
`a`'s scope is limited to the body of `f` while `b`'s scope is limited to the containing `if` statement's block.
在这里我们声明了两个局部变量`a`和`b`。
`a`的作用域被限定在`f`的范围内而`b`的作用域被限定在`if`语句块的范围内。

Variables declared in a `catch` clause also have similar scoping rules.
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

Another property of block-scoped variables is that they can't be read or written to before they're actually declared.
While these variables are "present" throughout their scope, all points up until their declaration are part of their *temporal dead zone*.
This is just a sophisticated way of saying you can't access them before the `let` statement, and luckily TypeScript will let you know that.
块作用域变量的另一个属性是它们不能在被实际声明前进行读写操作。
虽然这些变量“现在”遍布于它们的作用域，但是直到它们的声明部分位置都是他们的*暂时性死区*。
这仅仅是一种复杂的方式来说明你在`let`语句之前不能访问它们，然后幸运的是TypeScript会让你知道这点。

```ts
a++; // illegal to use 'a' before it's declared;
let a;
```

Something to note is that you can still *capture* a block-scoped variable before it's declared.
The only catch is that it's illegal to call that function before the declaration.
If targeting ES2015, a modern runtime will throw an error; however, right now TypeScript is permissive and won't report this as an error.
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

For more information on temporal dead zones, see relevant content on the [Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#Temporal_dead_zone_and_errors_with_let).
获取更多关于暂时性死区的信息，请访问相关链接：[Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#Temporal_dead_zone_and_errors_with_let).

## 重声明和遮蔽

With `var` declarations, we mentioned that it didn't matter how many times you declared your variables; you just got one.
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

In the above example, all declarations of `x` actually refer to the *same* `x`, and this is perfectly valid.
This often ends up being a source of bugs.
Thankfully, `let` declarations are not as forgiving.
在上面的例子里，所有对`x`的声明实际上指向了*同一个*`x`，而且它是完全有效的。
这往往会成为bug的一个来源。
值得庆幸的是，`let`声明方式并不会如此宽松。

```ts
let x = 10;
let x = 20; // error: can't re-declare 'x' in the same scope
```

The variables don't necessarily need to both be block-scoped for TypeScript to tell us that there's a problem.
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

That's not to say that block-scoped variable can never be declared with a function-scoped variable.
The block-scoped variable just needs to be declared within a distinctly different block.
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

f(false, 0); // returns 0
f(true, 0);  // returns 100
```

The act of introducing a new name in a more nested scope is called *shadowing*.
It is a bit of a double-edged sword in that it can introduce certain bugs on its own in the event of accidental shadowing, while also preventing certain bugs.
For instance, imagine we had written our earlier `sumMatrix` function using `let` variables.
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

This version of the loop will actually perform the summation correctly because the inner loop's `i` shadows `i` from the outer loop.
这个版本的循环将会正确的执行求和因为内循环的`i`遮蔽了外循环的`i`。

Shadowing should *usually* be avoided in the interest of write clearer code.
While there are some scenarios where it may be fitting to take advantage of it, you should use your best judgement.
如果致力于书写更为清晰的代码，则应*尽量*避免使用遮蔽。
如果存在某些场景需要用到遮蔽的优点，你应当使用你的最佳判断。

## Block-scoped variable capturing
## 捕获块作用域的变量

When we first touched on the idea of variable capturing with `var` declaration, we briefly went into how variables act once captured.
To give a better intuition of this, each time a scope is run, it creates an "environment" of variables.
That environment and its captured variables can exist even after everything within its scope has finished executing.
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

Because we've captured `city` from within its environment, we're still able to access it despite the fact that the `if` block finished executing.
因为我们在`city`所在的环境中捕获了它，因此尽管`if`块已经结束运行，我们依然可以访问到它。

Recall that with our earlier `setTimeout` example, we ended up needing to use an IIFE to capture the state of a variable for every iteration of the `for` loop.
In effect, what we were doing was creating a new variable environment for our captured variables.
That was a bit of a pain, but luckily, you'll never have to do that again in TypeScript.
回顾我们之前的`setTimeout`例子，我们最终需要使用IIFE来捕获`for`循环每个嵌套中变量的状态。
事实上，我们所做的事情是为我们捕获的变量创建了一个新的变量环境。
这样子做会有点痛，但是幸运的是，你并不必在TypeScript中做同样的事情。

`let` declarations have drastically different behavior when declared as part of a loop.
Rather than just introducing a new environment to the loop itself, these declarations sort of create a new scope *per iteration*.
Since this is what we were doing anyway with our IIFE, we can change our old `setTimeout` example to just use a `let` declaration.
`let`声明方式在作为一个循环中的一部分使用的时候，其行为存在巨大的不同。
不同于仅仅为循环本身引入一个新的环境，这些声明方式为*每一次迭代*都创建了一个新的作用域。
因为这正是我们通过IIFE所做的事情，因此我们可以将老的`setTimeout`例子通过`let`声明方式进行改写。

```ts
for (let i = 0; i < 10 ; i++) {
    setTimeout(function() {console.log(i); }, 100 * i);
}
```

and as expected, this will print out
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

# `const` declarations
# `const`声明方式

`const` declarations are another way of declaring variables.
`const`声明方式是另一种声明变量的方式。

```ts
const numLivesForCat = 9;
```

They are like `let` declarations but, as their name implies, their value cannot be changed once they are bound.
In other words, they have the same scoping rules as `let`, but you can't re-assign to them.
它们和`let`声明相似，不同点在于变量值一但被绑定后将不再能够修改，正如其名字所暗示的。
换言之，它们和`let`有相同的作用域规则，但你不能对它们重新赋值。

This should not be confused with the idea that the values they refer to are *immutable*.
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

Unless you take specific measures to avoid it, the internal state of a `const` variable is still modifiable.
除非你采取具体手段来进行避免，一个`const`变量的内部状态依然是可修改的。

# `let` vs. `const`

Given that we have two types of declarations with similar scoping semantics, it's natural to find ourselves asking which one to use.
Like most broad questions, the answer is: it depends.

Applying the [principle of least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege), all declarations other than those you plan to modify should use `const`.
The rationale is that if a variable didn't need to get written to, others working on the same codebase shouldn't automatically be able to write to the object, and will need to consider whether they really need to reassign to the variable.
Using `const` also makes code more predictable when reasoning about flow of data.

On the other hand, `let` is not any longer to write out than `var`, and many users will prefer its brevity.
The majority of this handbook uses `let` declarations in that interest.

Use your best judgement, and if applicable, consult the matter with the rest of your team.

# Destructuring

Another ECMAScript 2015 feature that TypeScript has is destructuring.
For a complete reference, see [the article on the Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment).
In this section, we'll give a short overview.

## Array destructuring

The simplest form of destructuring is array destructuring assignment:

```ts
let input = [1, 2];
let [first, second] = input;
console.log(first); // outputs 1
console.log(second); // outputs 2
```

This creates two new variables named `first` and `second`.
This is equivalent to using indexing, but is much more convenient:

```ts
first = input[0];
second = input[1];
```

Destructuring works with already-declared variables as well:

```ts
// swap variables
[first, second] = [second, first];
```

And with parameters to a function:

```ts
function f([first, second]: [number, number]) {
    console.log(first);
    console.log(second);
}
f(input);
```

You can create a variable for the remaining items in a list using the syntax `...name`:

```ts
let [first, ...rest] = [1, 2, 3, 4];
console.log(first); // outputs 1
console.log(rest); // outputs [ 2, 3, 4 ]
```

Of course, since this is JavaScript, you can just ignore trailing elements you don't care about:

```ts
let [first] = [1, 2, 3, 4];
console.log(first); // outputs 1
```

Or other elements:

```ts
let [, second, , fourth] = [1, 2, 3, 4];
```

## Object destructuring

You can also destructure objects:

```ts
let o = {
    a: "foo",
    b: 12,
    c: "bar"
}
let {a, b} = o;
```

This creates new variables `a` and `b` from `o.a` and `o.b`.
Notice that you can skip `c` if you don't need it.

Like array destructuring, you can have assignment without declaration:

```ts
({a, b} = {a: "baz", b: 101});
```

Notice that we had to surround this statement with parentheses.
JavaScript normally parses a `{` as the start of block.

### Property renaming

You can also give different names to properties:

```ts
let {a: newName1, b: newName2} = o;
```

Here the syntax starts to get confusing.
You can read `a: newName1` as "`a` as `newName1`".
The direction is left-to-right, as if you had written:

```ts
let newName1 = o.a;
let newName2 = o.b;
```

Confusingly, the colon here does *not* indicate the type.
The type, if you specify it, still needs to be written after the entire destructuring:

```ts
let {a, b}: {a: string, b: number} = o;
```

### Default values

Default values let you specify a default value in case a property is undefined:

```ts
function keepWholeObject(wholeObject: {a: string, b?: number}) {
    let {a, b = 1001} = wholeObject;
}
```

`keepWholeObject` now has a variable for `wholeObject` as well as the properties `a` and `b`, even if `b` is undefined.

## Function declarations

Destructuring also works in function declarations.
For simple cases this is straightforward:

```ts
type C = {a: string, b?: number}
function f({a, b}: C): void {
    // ...
}
```

But specifying defaults is more common for parameters, and getting defaults right with destructuring can be tricky.
First of all, you need to remember to put the type before the default value.

```ts
function f({a, b} = {a: "", b: 0}): void {
    // ...
}
f(); // ok, default to {a: "", b: 0}
```

Then, you need to remember to give a default for optional properties on the destructured property instead of the main initializer.
Remember that `C` was defined with `b` optional:

```ts
function f({a, b = 0} = {a: ""}): void {
    // ...
}
f({a: "yes"}) // ok, default b = 0
f() // ok, default to {a: ""}, which then defaults b = 0
f({}) // error, 'a' is required if you supply an argument
```

Use destructuring with care.
As the previous example demonstrates, anything but the simplest destructuring expressions have a lot of corner cases.
This is especially true with deeply nested destructuring, which gets *really* hard to understand even without piling on renaming, default values, and type annotations.
Try to keep destructuring expressions small and simple.
You can always write the assignments that destructuring would generate yourself.
