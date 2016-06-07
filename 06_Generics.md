# 引言

软件工程的主要部分不仅仅是构建定义清晰并且拥有统一API的组件，还要确保这些组件是可以重复使用的。确保组件在当前并且将来都可以用于处理数据会让你在构建大型软件系统时最大的灵活度。

在C#和Java这类语言中，可重复使用的最主要的组件之一就是“泛型”，也就是说，你可以创建一个在各种数据类型中都可以使用的组件。这样就可以让用户在他们自建的类型中使用这类组件。

# 泛型的初次实践

让我们开始做第一个泛型：身份函数。身份函数是一种返回任意输入的函数。你可以把它理解为‘echo’。

不使用泛型的情况下，我们要么给身份函数一个特定的类：

```ts
function identity(arg: number): number {
    return arg;
}
```

或者，我们可以把身份函数定义为`any`类型：

```ts
function identity(arg: any): any {
    return arg;
}
```

虽然使用`any`可以接受任意或者`arg`中的全部类型，但事实上，当函数返回时，我们不知道原始数据是什么类型的。如果我们传递一个数字类型给函数，函数只会返回一个any类型。

因此，我们需要一种既可以捕获数据类型又可以表示返回值的方法。在这里，我们使用的是*类型变量*一种特殊的，可以作用在类型上的变量。

```ts
function identity<T>(arg: T): T {
    return arg;
}
```

我们现在给身份函数添加了一个`T`变量。
这个`T`可以让我们捕获用户提供的变量类型（比如`number`），然后我们就在稍后使用这个获得的变量类型。
在这里，我们使用`T`作为返回类型。回顾一下，我们现在可以看到变量和返回类型用的是同样的类型。
这让我们可以监控数据从进入到走出函数时的类型信息。

让我们把这个版本的`identity`函数定义为一个泛型，因为它可以在多种类型中正常运作。
不同于使用`any`类型，这次的`identity`函数就和第一次的函数一样准确（也就是说，它不会丢失任何信息），因为它使用了数字类型作为变量和返回值的类型。

当我们完成了身份函数的泛型之后，我们可以通过两种方式来调用它。
第一种方式是传递所有的变量，包括类型变量给函数：

```ts
let output = identity<string>("myString");  //  type of output will be 'string'
```

在这里我们明确地设置了`T`在被函数调用时变量的类型需要是string，使用`<>`而不是`()`标记在变量的周围。

The second way is also perhaps the most common. Here we use *type argument inference*, that is, we want the compiler to set the value of `T` for us automatically based on the type of the argument we pass in:第二种方式可能也是最常用的方式。在这里我们使用*变量类型推断*，也就是说，我们想要编译器根据我们传入的变量类型自动为`T`赋值。

```ts
let output = identity("myString");  // type of output will be 'string'
```

请注意我们并不一定要明确的用尖括号(`<>`)来传递类型，编译器只会查找`"myString"`的值，然后给这个值设置一个`T`类型。
虽然变量类型推断是一个可以让代码变得尽可能短而且易读的实用工具，在编译器不能推断类型的时候，比如一些更复杂的例子中，你可能还是要像第一个例子那样明确地传递变量类型。

# 使用泛型类的变量

在你刚开始使用泛型的时候，你会发现当你创建了一个类似`identity`函数的泛型时，编译器会强制你在函数体中正确使用任意的一般类型参数。
也就是说，你需要考虑到这些参数可以是任意一种类型。

让我们重新回顾一下之前的`identity`函数：

```ts
function identity<T>(arg: T): T {
    return arg;
}
```

那么如果我们想要在每次调用的时候在控制器上记录变量`arg`的长度呢？
我们可能想这么写：

```ts
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T doesn't have .length
    return arg;
}
```

当我们这么做的时候，编译器会返回一个错误，我们使用的是`arg`的成员`.length`，但是我们没有申明过`arg`拥有这个成员。
请记住，我们之前说过这类变量可以是任何类型，所以想要实现这个功能只需要传递一个`number`，虽然它并没有`.length`成员。

假设我们成功地让这个功能在`T`数组而不是`T`函数上实现了。因为我们在和数组打交道，所以`.length`成员现在是可用的。
我们可以像创建一个其他类型的数组一样来描述它：

```ts
function loggingIdentity<T>(arg: T[]): T[] {
    console.log(arg.length);  // Array has a .length, so no more error
    return arg;
}
```

你可以看到`loggingIdentity`类是“泛型函数`loggingIdentity` 有一个类型变量`T`，还有一个作为数组`T`存在的变量`arg`，最后返回数组`T`”。
如果我们传入一组数字，我们会得到一组数字作为返回，因为`T`是绑定为`number`类型的。
这允许我们使用自己的泛型类变量`T`作为我们正在使用的类型中的一部分，而非整个类型，这么做会带来更好的灵活性。

我们也可以把示例用这种方法来实现：

```ts
function loggingIdentity<T>(arg: Array<T>): Array<T> {
    console.log(arg.length);  // Array has a .length, so no more error
    return arg;
}
```

你可能已经对其他语言中的这类类型形式相当熟悉。
在下一章节，我们会告诉你如何创建类似`Array<T>`这样的自己定义的泛型。


在之前的章节中，我们创建了可以在大量类型中使用的泛型身份函数。
在本章中，我们会探索类型函数并且知道如何创建泛型接口。

泛型函数的类型就和这些非泛型函数一样，把类型变量列在最前面，和函数定义相似：

```ts
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: <T>(arg: T) => T = identity;
```

我们也可以在类型中对泛型类变量进行不同的命名，只要类型变量的数目和最终的使用是一一对应的。

```ts
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: <U>(arg: U) => U = identity;
```

We can also write the generic type as a call signature of an object literal type:我们也可以把泛型类作为一个对象文本类型的调用签名：

```ts
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: {<T>(arg: T): T} = identity;
```

这样我们就可以写第一个泛型接口了。
让我们把上一个例子的对象文本放到一个接口中：

```ts
interface GenericIdentityFn {
    <T>(arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn = identity;
```

在一个类似的例子中，我们可能想要把泛型变量变成整个接口的变量。
这样我们就能看到我们泛型了什么类型（例如：`Dictionary<string>`而不是`Dictionary`）。
在这种接口中，类型变量对其他成员都是可见的。

```ts
interface GenericIdentityFn<T> {
    (arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn<number> = identity;
```

请注意我们的例子有了一些微小的变化。
Instead of describing a generic function, we now have a non-generic function signature that is a part of a generic type.与其描述整个泛型函数，我们现在拥有一个非泛型函数签名，而它是一个泛型类的一部分。
当我们使用`GenericIdentityFn`的时候，我们也需要确定对应变量的类型（在这里是`number`），有效地确定底层的调用签名会使用什么类型。
Understanding when to put the type parameter directly on the call signature and when to put it on the interface itself will be helpful in describing what aspects of a type are generic.正确理解何时该把调用签名直接放在类型变量上还是放在接口本身对于描述泛型部分的类型的非常有用的。

除了泛型接口，我们还可以创建泛型类。
请注意泛型枚举和泛型命名空间是不可创建的。

# Generic Classes

A generic class has a similar shape to a generic interface.
Generic classes have a generic type parameter list in angle brackets (`<>`) following the name of the class.

```ts
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

This is a pretty literal use of the `GenericNumber` class, but you may have noticed that nothing is restricting it to only use the `number` type.
We could have instead used `string` or even more complex objects.

```ts
let stringNumeric = new GenericNumber<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function(x, y) { return x + y; };

alert(stringNumeric.add(stringNumeric.zeroValue, "test"));
```

Just as with interface, putting the type parameter on the class itself lets us make sure all of the properties of the class are working with the same type.

As we covered in [our section on classes](./Classes.md), a class has two sides to its type: the static side and the instance side.
Generic classes are only generic over their instance side rather than their static side, so when working with classes, static members can not use the class's type parameter.

# Generic Constraints

If you remember from an earlier example, you may sometimes want to write a generic function that works on a set of types where you have some knowledge about what capabilities that set of types will have.
In our `loggingIdentity` example, we wanted to be able access the `.length` property of `arg`, but the compiler could not prove that every type had a `.length` property, so it warns us that we can't make this assumption.

```ts
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T doesn't have .length
    return arg;
}
```

Instead of working with any and all types, we'd like to constrain this function to work with any and all types that also have the `.length` property.
As long as the type has this member, we'll allow it, but it's required to have at least this member.
To do so, we must list our requirement as a constraint on what T can be.

To do so, we'll create an interface that describes our constraint.
Here, we'll create an interface that has a single `.length` property and then we'll use this interface and the `extends` keyword to denote our constraint:

```ts
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);  // Now we know it has a .length property, so no more error
    return arg;
}
```

Because the generic function is now constrained, it will no longer work over any and all types:

```ts
loggingIdentity(3);  // Error, number doesn't have a .length property
```

Instead, we need to pass in values whose type has all the required properties:

```ts
loggingIdentity({length: 10, value: 3});
```

## Using Type Parameters in Generic Constraints

You can declare a type parameter that is constrained by another type parameter.
For example, here we'd like to take two objects and copy properties from one to the other.
We'd like to ensure that we're not accidentally writing any extra properties from our `source`, so we'll place a constraint between the two types:

```ts
function copyFields<T extends U, U>(target: T, source: U): T {
    for (let id in source) {
        target[id] = source[id];
    }
    return target;
}

let x = { a: 1, b: 2, c: 3, d: 4 };

copyFields(x, { b: 10, d: 20 }); // okay
copyFields(x, { Q: 90 });  // error: property 'Q' isn't declared in 'x'.
```

## Using Class Types in Generics

When creating factories in TypeScript using generics, it is necessary to refer to class types by their constructor functions. For example,

```ts
function create<T>(c: {new(): T; }): T {
    return new c();
}
```

A more advanced example uses the prototype property to infer and constrain relationships between the constructor function and the instance side of class types.

```ts
class BeeKeeper {
    hasMask: boolean;
}

class ZooKeeper {
    nametag: string;
}

class Animal {
    numLegs: number;
}

class Bee extends Animal {
    keeper: BeeKeeper;
}

class Lion extends Animal {
    keeper: ZooKeeper;
}

function findKeeper<A extends Animal, K> (a: {new(): A;
    prototype: {keeper: K}}): K {

    return a.prototype.keeper;
}

findKeeper(Lion).nametag;  // typechecks!
```
