# 引言

One of TypeScript's core principles is that type-checking focuses on the *shape* that values have.
This is sometimes called "duck typing" or "structural subtyping".
In TypeScript, interfaces fill the role of naming these types, and are a powerful way of defining contracts within your code as well as contracts with code outside of your project.
TypeScript的核心准则之一便是其类型检查侧重于值所拥有的*形*。
这在有时候被称为“鸭子类型”或“结构子类型”。
在TypeScript中，接口充当了为这些类型命名的角色，也是为代码定义契约的有力手段，无论是项目内的还是项目外的代码。

# Our First Interface
# 我们的第一个接口

The easiest way to see how interfaces work is to start with a simple example:
了解接口是如何工作的最简单的方式是一个简单的例子开始：

```ts
function printLabel(labelledObj: { label: string }) {
    console.log(labelledObj.label);
}

let myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

The type-checker checks the call to `printLabel`.
The `printLabel` function has a single parameter that requires that the object passed in has a property called `label` of type string.
Notice that our object actually has more properties than this, but the compiler only checks that *at least* the ones required are present and match the types required.
There are some cases where TypeScript isn't as lenient, which we'll cover in a bit.
类型检查器检查了对`printLabel`的调用。
这里的`printLabel`函数存在一个变量，且传入该变量的对象应当包含一个名为`label`的字符串类型属性。
注意，我们实际传入的对象包含更多的属性，但是编译器仅仅检查*最小限度*必须的存在，并匹配所需的类型。
但是有些情况下TypeScript是不会如此宽松的，这里我们会稍微提到一些。

We can write the same example again, this time using an interface to describe the requirement of having the `label` property that is a string:
同一个例子我们可以再来一遍，这一次使用一个接口来描述对字符串类型的属性`label`的需求：

```ts
interface LabelledValue {
    label: string;
}

function printLabel(labelledObj: LabelledValue) {
    console.log(labelledObj.label);
}

let myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

The interface `LabelledValue` is a name we can now use to describe the requirement in the previous example.
It still represents having a single property called `label` that is of type string.
Notice we didn't have to explicitly say that the object we pass to `printLabel` implements this interface like we might have to in other languages.
Here, it's only the shape that matters. If the object we pass to the function meets the requirements listed, then it's allowed.
我们现在可以使用接口`LabelledValue`来作为描述之前例子中的需求的名字。
它依然意味着有一个叫做`label`的字符串类型的属性。
注意，我们并没有明确地说传入`printLabel`的对象继承了这个接口，而在其他语言中我们可能必须这样做。
在这里，重要的仅仅是形。如果我们传入函数的对象符合列出的需求，那它便是被允许的。

It's worth pointing out that the type-checker does not require that these properties come in any sort of order, only that the properties the interface requires are present and have the required type.
值得一提的是，类型检查器并不需要这些属性符合任何形式的排序，仅仅需要接口需要的属性存在并符合需要的类型即可。

# Optional Properties
# 可选属性

Not all properties of an interface may be required.
Some exist under certain conditions or may not be there at all.
These optional properties are popular when creating patterns like "option bags" where you pass an object to a function that only has a couple of properties filled in.
在一个接口中，并非所有属性都一定是必需的。
有一些属性存在于特定条件之下而有一些则不是。
这种可选参数在创建像“选项包”之类的模式时受到欢迎，此时你传递给函数的对象仅包含了部分属性。

Here's an example of this pattern:
这里有个这种模式的例子：

```ts
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
    let newSquare = {color: "white", area: 100};
    if (config.color) {
        newSquare.color = config.color;
    }
    if (config.width) {
        newSquare.area = config.width * config.width;
    }
    return newSquare;
}

let mySquare = createSquare({color: "black"});
```

Interfaces with optional properties are written similar to other interfaces, with each optional property denoted by a `?` at the end of the property name in the declaration.
带可选属性的接口的书写方式类似于其他接口，仅需要在声明可选属性时在属性名称后面跟上一个`?`。

The advantage of optional properties is that you can describe these possibly available properties while still also preventing use of properties that are not part of the interface.
For example, had we mistyped the name of the `color` property in `createSquare`, we would get an error message letting us know:
可选属性的优势在于你可以描述这些可能有用的属性的同时依然可以防止使用那些并非接口一部分的属性。
例如，当我们输错了`createSquare`中`color`属性的名字，我们将会得到一个错误信息来让我们注意到：

```ts
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
    let newSquare = {color: "white", area: 100};
    if (config.color) {
        // Error: Property 'collor' does not exist on type 'SquareConfig'
        newSquare.color = config.collor;
    }
    if (config.width) {
        newSquare.area = config.width * config.width;
    }
    return newSquare;
}

let mySquare = createSquare({color: "black"});
```

# Excess Property Checks
# 过剩属性检查

In our first example using interfaces, TypeScript let us pass `{ size: number; label: string; }` to something that only expected a `{ label: string; }`.
We also just learned about optional properties, and how they're useful when describing so-called "option bags".
在我们使用接口的第一个例子中，TypeScript允许我们向只需要一个`{ label: string; }`的某个对象传递`{ size: number; label: string; }`。
我们还学习了可选属性相关的内容，以及为何它在描述“选项包”时十分有用。

However, combining the two naively would let you to shoot yourself in the foot the same way you might in JavaScript.
For example, taking our last example using `createSquare`:
然而，如果像和你在JavaScript中相同的方式那样天真的将这两者结合起来将会让你搬起石头砸自己的脚。
让我们以之前的`createSquare`例子为例：

```ts
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
    // ...
}

let mySquare = createSquare({ colour: "red", width: 100 });
```

Notice the given argument to `createSquare` is spelled *`colour`* instead of `color`.
In plain JavaScript, this sort of thing fails silently.
注意到我们给`createSquare`的参数拼成了*`colour`*而不是`color`。
在JavaScript中，这种的事情将会默默的导致失败。

You could argue that this program is correctly typed, since the `width` properties are compatible, there's no `color` property present, and the extra `colour` property is insignificant.
你可能认为这段程序是被正确输入的，因为`width`属性是兼容的，而`color`属性并非强制需要的，并且额外的`colour`属性是无关紧要的。

However, TypeScript takes the stance that there's probably a bug in this code.
Object literals get special treatment and undergo *excess property checking* when assigning them to other variables, or passing them as arguments.
If an object literal has any properties that the "target type" doesn't have, you'll get an error.
然而，在这种情况下TypeScript会认为代码中可能存在一个bug。
对象文本会被特殊对待并在赋值给其他变量或作为参数传递时会经历*额外的属性检查*。
如果一个对象文本有一个“目标类型”不包含的属性，将会报出一个错误。

```ts
// error: 'colour' not expected in type 'SquareConfig'
let mySquare = createSquare({ colour: "red", width: 100 });
```

Getting around these checks is actually really simple.
The easiest method is to just use a type assertion:
处理这类检查的方法实际上十分简单。
最简单的方法是使用类型断言：

```ts
let mySquare = createSquare({ width: 100, opacity: 0.5 } as SquareConfig);
```

However, a better approach might be to add a string index signature if you're sure that the object can have some extra properties that are used in some special way.
If `SquareConfig`s can have `color` and `width` properties with the above types, but could *also* have any number of other properties, then we could define it like so:
然而，一个更好的途径是，如果你确定这个对象可以有一些额外的属性用于特殊途径，则添加一个字符串索引签名。
如果`SquareConfig`可以拥有如上类型的`color`和`width`属性，但是可能*同样*有任意数量的其他属性，那我们可以像这样定义它：

```ts
interface SquareConfig {
    color?: string;
    width?: number;
    [propName: string]: any;
}
```

We'll discuss index signatures in a bit, but here we're saying a `SquareConfig` can have any number of properties, and as long as they aren't `color` or `width`, their types don't matter.
我们将稍微讨论一下索引签名，然后在这里我们说一个`SquareConfig`可以拥有任意数量的属性，并且只要它们不是`color`或`width`，它们的类型便无关紧要。

One final way to get around these checks, which might be a bit surprising, is to assign the object to another variable:
Since `squareOptions` won't undergo excess property checks, the compiler won't give you an error.


```ts
let squareOptions = { colour: "red", width: 100 };
let mySquare = createSquare(squareOptions);
```

Keep in mind that for simple code like above, you probably shouldn't be trying to "get around" these checks.
For more complex object literals that have methods and hold state, you might need to keep these techniques in mind, but a majority of excess property errors are actually bugs.
That means if you're running into excess property checking problems for something like option bags, you might need to revise some of your type declarations.
In this instance, if it's okay to pass an object with both a `color` or `colour` property to `createSquare`, you should fix up the definition of `SquareConfig` to reflect that.

# Function Types

Interfaces are capable of describing the wide range of shapes that JavaScript objects can take.
In addition to describing an object with properties, interfaces are also capable of describing function types.

To describe a function type with an interface, we give the interface a call signature.
This is like a function declaration with only the parameter list and return type given. Each parameter in the parameter list requires both name and type.

```ts
interface SearchFunc {
    (source: string, subString: string): boolean;
}
```

Once defined, we can use this function type interface like we would other interfaces.
Here, we show how you can create a variable of a function type and assign it a function value of the same type.

```ts
let mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
    let result = source.search(subString);
    if (result == -1) {
        return false;
    }
    else {
        return true;
    }
}
```

For function types to correctly type-check, the names of the parameters do not need to match.
We could have, for example, written the above example like this:

```ts
let mySearch: SearchFunc;
mySearch = function(src: string, sub: string): boolean {
    let result = src.search(sub);
    if (result == -1) {
        return false;
    }
    else {
        return true;
    }
}
```

Function parameters are checked one at a time, with the type in each corresponding parameter position checked against each other.
If you do not want to specify types at all, Typescript's contextual typing can infer the argument types since the function value is assigned directly to a variable of type `SearchFunc`.
Here, also, the return type of our function expression is implied by the values it returns (here `false` and `true`).
Had the function expression returned numbers or strings, the type-checker would have warned us that return type doesn't match the return type described in the `SearchFunc` interface.

```ts
let mySearch: SearchFunc;
mySearch = function(src, sub) {
    let result = src.search(sub);
    if (result == -1) {
        return false;
    }
    else {
        return true;
    }
}
```

# Indexable Types

Similarly to how we can use interfaces to describe function types, we can also describe types that we can "index into" like `a[10]`, or `ageMap["daniel"]`.
Indexable types have an *index signature* that describes the types we can use to index into the object, along with the corresponding return types when indexing.
Let's take an example:

```ts
interface StringArray {
    [index: number]: string;
}

let myArray: StringArray;
myArray = ["Bob", "Fred"];

let myStr: string = myArray[0];
```

Above, we have a `StringArray` interface that has an index signature.
This index signature states that when a `StringArray` is indexed with a `number`, it will return a `string`.

There are two types of supported index signatures: string and number.
It is possible to support both types of indexers, but the type returned from a numeric indexer must be a subtype of the type returned from the string indexer.
This is because when indexing with a `number`, JavaScript will actually convert that to a `string` before indexing into an object.
That means that indexing with `100` (a `number`) is the same thing as indexing with `"100"` (a `string`), so the two need to be consistent.

```ts
class Animal {
    name: string;
}
class Dog extends Animal {
    breed: string;
}

// Error: indexing with a 'string' will sometimes get you a Dog!
interface NotOkay {
    [x: number]: Animal;
    [x: string]: Dog;
}
```

While string index signatures are a powerful way to describe the "dictionary" pattern, they also enforce that all properties match their return type.
This is because a string index declares that `obj.property` is also available as `obj["property"]`.
In the following example, `name`'s type does not match the string index's type, and the type-checker gives an error:

```ts
interface NumberDictionary {
    [index: string]: number;
    length: number;    // ok, length is a number
    name: string;      // error, the type of 'name' is not a subtype of the indexer
}
```

# Class Types

## Implementing an interface

One of the most common uses of interfaces in languages like C# and Java, that of explicitly enforcing that a class meets a particular contract, is also possible in TypeScript.

```ts
interface ClockInterface {
    currentTime: Date;
}

class Clock implements ClockInterface {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

You can also describe methods in an interface that are implemented in the class, as we do with `setTime` in the below example:

```ts
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date);
}

class Clock implements ClockInterface {
    currentTime: Date;
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
}
```

Interfaces describe the public side of the class, rather than both the public and private side.
This prohibits you from using them to check that a class also has particular types for the private side of the class instance.

## Difference between the static and instance sides of classes

When working with classes and interfaces, it helps to keep in mind that a class has *two* types: the type of the static side and the type of the instance side.
You may notice that if you create an interface with a construct signature and try to create a class that implements this interface you get an error:

```ts
interface ClockConstructor {
    new (hour: number, minute: number);
}

class Clock implements ClockConstructor {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

This is because when a class implements an interface, only the instance side of the class is checked.
Since the constructor sits in the static side, it is not included in this check.

Instead, you would need to work with the static side of the class directly.
In this example, we define two interfaces, `ClockConstructor` for the constructor and `ClockInterface` for the instance methods.
Then for convenience we define a constructor function `createClock` that creates instances of the type that is passed to it.

```ts
interface ClockConstructor {
    new (hour: number, minute: number): ClockInterface;
}
interface ClockInterface {
    tick();
}

function createClock(ctor: ClockConstructor, hour: number, minute: number): ClockInterface {
    return new ctor(hour, minute);
}

class DigitalClock implements ClockInterface {
    constructor(h: number, m: number) { }
    tick() {
        console.log("beep beep");
    }
}
class AnalogClock implements ClockInterface {
    constructor(h: number, m: number) { }
    tick() {
        console.log("tick tock");
    }
}

let digital = createClock(DigitalClock, 12, 17);
let analog = createClock(AnalogClock, 7, 32);
```

Because `createClock`'s first parameter is of type `ClockConstructor`, in `createClock(AnalogClock, 7, 32)`, it checks that `AnalogClock` has the correct constructor signature.

# Extending Interfaces

Like classes, interfaces can extend each other.
This allows you to copy the members of one interface into another, which gives you more flexibility in how you separate your interfaces into reusable components.

```ts
interface Shape {
    color: string;
}

interface Square extends Shape {
    sideLength: number;
}

let square = <Square>{};
square.color = "blue";
square.sideLength = 10;
```

An interface can extend multiple interfaces, creating a combination of all of the interfaces.

```ts
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

interface Square extends Shape, PenStroke {
    sideLength: number;
}

let square = <Square>{};
square.color = "blue";
square.sideLength = 10;
square.penWidth = 5.0;
```

# Hybrid Types

As we mentioned earlier, interfaces can describe the rich types present in real world JavaScript.
Because of JavaScript's dynamic and flexible nature, you may occasionally encounter an object that works as a combination of some of the types described above.

One such example is an object that acts as both a function and an object, with additional properties:

```ts
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

function getCounter(): Counter {
    let counter = <Counter>function (start: number) { };
    counter.interval = 123;
    counter.reset = function () { };
    return counter;
}

let c = getCounter();
c(10);
c.reset();
c.interval = 5.0;
```

When interacting with 3rd-party JavaScript, you may need to use patterns like the above to fully describe the shape of the type.

# Interfaces Extending Classes

When an interface type extends a class type it inherits the members of the class but not their implementations.
It is as if the interface had declared all of the members of the class without providing an implementation.
Interfaces inherit even the private and protected members of a base class.
This means that when you create an interface that extends a class with private or protected members, that interface type can only be implemented by that class or a subclass of it.

This is useful when you have a large inheritance hierarchy, but want to specify that your code works with only subclasses that have certain properties.
The subclasses don't have to be related besides inheriting from the base class.
For example:

```ts
class Control {
    private state: any;
}

interface SelectableControl extends Control {
    select(): void;
}

class Button extends Control {
    select() { }
}

class TextBox extends Control {
    select() { }
}

class Image extends Control {
}

class Location {
    select() { }
}
```

In the above example, `SelectableControl` contains all of the members of `Control`, including the private `state` property.
Since `state` is a private member it is only possible for descendants of `Control` to implement `SelectableControl`.
This is because only descendants of `Control` will have a `state` private member that originates in the same declaration, which is a requirement for private members to be compatible.

Within the `Control` class it is possible to access the `state` private member through an instance of `SelectableControl`.
Effectively, a `SelectableControl` acts like a `Control` that is known to have a `select` method.
The `Button` and `TextBox` classes are subtypes of `SelectableControl` (because they both inherit from `Control` and have a `select` method), but the `Image` and `Location` classes are not.
