# 引言

软件工程的主要部分不仅仅是构建定义清晰并且拥有统一API的组件，还要确保这些组件是可以重复使用的。
确保组件可用于处理当前的数据及将来的数据会让你在构建大型软件系统时拥有最大的灵活度。

在C#和Java这类语言中，创建可复用组件的主要工具之一是*泛型*，也就是说，你可以创建一个在各种数据类型中都可以使用的组件。
这样就可以让用户在他们自建的类型中使用这类组件。

# 泛型的初次实践

让我们开始做第一个泛型示例：身份函数。
身份函数是一个无论任意输入什么都原样返回的函数。
你可以把认为它和`echo`命令相似。

在不使用泛型的情况下，我们要给身份函数一个特定类型：

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

虽然使用`any`可以接受任意和`arg`中的全部类型，但事实上，当函数返回时，我们失去了原始数据的类型信息。
如果我们传递一个数字类型给函数，我们只会知道函数会返回一个any类型。

因此，我们需要一种既可以捕获参数类型又可以表示返回值的方法。
在这里，我们将使用*类型变量*，一种可以作用在类型上而不仅仅是值上的特殊变量。

```ts
function identity<T>(arg: T): T {
    return arg;
}
```

我们现在给身份函数添加了一个类型变量`T`。
这个`T`可以让我们捕获用户提供的变量类型（比如`number`），然后我们就在稍后使用这些信息。
在这里，我们使用`T`作为返回类型。回顾一下，我们现在可以看到参数和返回类型用的是同样的类型。
这让我们可以监控数据从进入到走出函数时的类型信息。

我们称这个版本的`identity`函数是一个泛型，因为它可以在多种类型中正常运作。
不同于使用`any`类型，这次的`identity`函数就和第一次的函数一样准确（也就是说，它不会丢失任何信息），因为它使用了数字类型作为参数和返回值的类型。

当我们完成了身份函数的泛型之后，我们可以通过两种方式来调用它。
第一种方式是向函数传递所有的参数，包括类型参数：

```ts
let output = identity<string>("myString");  //  type of output will be 'string'
```

在这里我们明确地将`T`设置为`string`并作为调用函数时的参数之一，使用`<>`而不是`()`标记在参数的周围。

第二种方式可能也是最常用的方式。在这里我们使用*参数类型推断*——也就是说，我们想要编译器根据我们传入的参数类型自动为`T`赋值：

```ts
let output = identity("myString");  // type of output will be 'string'
```

请注意我们并不一定要明确的用尖括号(`<>`)来传递类型；编译器只需要查看值`"myString"`，便会根据这个值设置`T`的类型。
虽然参数类型推断是一个可以让代码变得尽可能短而且易读的实用工具，在编译器不能推断类型的时候，比如一些更复杂的例子中，你可能还是要像第一个例子那样明确地传递参数类型。

# 使用泛型类型的变量

在你刚开始使用泛型的时候，你会发现当你创建了一个类似`identity`函数的泛型时，编译器会强制你在函数体中正确使用任意的一般类型参数。
也就是说，你需要考虑到这些参数可以是任意一种类型。

让我们重新回顾一下之前的`identity`函数：

```ts
function identity<T>(arg: T): T {
    return arg;
}
```

那么如果我们想要在每次调用的时候在控制器上记录参数`arg`的长度呢？
我们可能想这么写：

```ts
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T doesn't have .length
    return arg;
}
```

当我们这么做的时候，编译器会返回一个错误，我们想使用`arg`的成员`.length`，但是我们没有在任何地方说过`arg`拥有这个成员。
请记住，我们之前说过这类变量可以是任何类型，所以某个使用这个函数的人可能传入一个`number`，但它并没有`.length`这个成员成员。

假设我们使用了`T`数组来实现了这个方法，而不是直接使用`T`。因为我们在和数组打交道，所以`.length`成员现在是可用的。
我们可以像创建一个其他类型的数组一样来描述它：

```ts
function loggingIdentity<T>(arg: T[]): T[] {
    console.log(arg.length);  // Array has a .length, so no more error
    return arg;
}
```

你可以看到`loggingIdentity`的类型是“泛型函数`loggingIdentity` 有一个类型参数`T`，还有一个作为数组`T`存在的参数`arg`，最后返回数组`T`”。
如果我们传入一组数字，我们会得到一组数字作为返回，因为`T`是绑定为`number`类型的。
这允许我们使用自己的泛型类型变量`T`作为我们正在使用的类型中的一部分，而非整个类型，这么做会带来更好的灵活性。

我们也可以把示例用这种方法来实现：

```ts
function loggingIdentity<T>(arg: Array<T>): Array<T> {
    console.log(arg.length);  // Array has a .length, so no more error
    return arg;
}
```

你可能已经对其他语言中的这类类型形式相当熟悉。
在下一章节，我们会告诉你如何创建类似`Array<T>`这样的自定义泛型类型。

# 泛型类型

在之前的章节中，我们创建了可以在大量类型中使用的泛型身份函数。
在本节中，我们会探索类型函数并且知道如何创建泛型接口。

泛型函数的类型就和这些非泛型函数一样，把类型参数列在最前面，和函数定义相似：

```ts
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: <T>(arg: T) => T = identity;
```

我们也可以在类型中对泛型类型参数进行不同的命名，只要类型变量的数目和最终的使用是一一对应的。

```ts
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: <U>(arg: U) => U = identity;
```

我们也可以把泛型类型作为一个对象字面量类型的调用签名：

```ts
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: {<T>(arg: T): T} = identity;
```

这样我们就可以写第一个泛型接口了。
让我们把上一个例子中的对象字面量放到一个接口中：

```ts
interface GenericIdentityFn {
    <T>(arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn = identity;
```

在一个类似的例子中，我们可能想要把泛型参数变成整个接口的参数。
这样我们就能看到我们泛型了什么类型（例如：`Dictionary<string>`而不仅仅是`Dictionary`）。
在这种接口中，类型参数对其他成员都是可见的。

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
不同于描述一个泛型函数，我们现在拥有一个非泛型函数签名，而它是一个泛型类型的一部分。
当我们使用`GenericIdentityFn`的时候，我们也需要确定对应参数的类型（在这里是`number`），有效地确定底层的调用签名会使用什么类型。
正确理解何时该把调用签名直接放在类型参数上还是放在接口本身对于描述泛型部分的类型的非常有用的。

除了泛型接口，我们还可以创建泛型类。
请注意泛型枚举和泛型命名空间是不可创建的。

# 泛型类

泛型类的结构和泛型接口是类似的。
泛型类的泛型类型参数是跟在类名之后的尖括号 (`<>`) 里的。

```ts
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

在`GenericNumber`类中有一种相当文字化的使用方法，但是你你能也注意到了在使用`number`类的时候并没有任何限制。
我们甚至可以使用`string`或者更加复杂的对象。

```ts
let stringNumeric = new GenericNumber<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function(x, y) { return x + y; };

alert(stringNumeric.add(stringNumeric.zeroValue, "test"));
```

和接口一样，把类型参数直接传入类本身可以保证类的所有属性都有相同的类型。

就像我们在[类的章节](04_Classes.md)中说过的那样，一个类有两种类型：静态类型和实例类型。
泛型类只是在实例类型而非静态类型中使用泛型，所以在使用泛型类的时候，静态成员不可以使用类的类型参数。

# 泛型约束

如果你还记得早前的例子，你也许会想到去写一个可以在某些类型中作用的泛型函数，而这个函数在具体类型中的功能是你所知道的。
在我们`loggingIdentity`的例子中，我们希望可以获得`arg`的`.length`属性，但是编译器不能证明每种类型都有`.length`属性，所以它警告我们不能做这样的假设。

```ts
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T doesn't have .length
    return arg;
}
```

除了在所有类型中可用，我们希望这个函数的约束也可以作用在所有拥有`.length`属性的类型中。
只要某一类型拥有这个成员，我们就可以使用，但这个类型至少要拥有这个成员。
为了达到这个目的，我们需要把希望T可以做到的事情列出并做成一个约束。

所以，我们要创建一个接口来描述这个约束。
在这里，我们会创建一个只拥有`.length`属性的接口，然后我们会用这个接口和关键字`extends`来表示我们的约束：

```ts
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);  // Now we know it has a .length property, so no more error
    return arg;
}
```

因为泛型函数现在被约束了，它不再能作用于任意类型：

```ts
loggingIdentity(3);  // Error, number doesn't have a .length property
```

因此，我们需要传入其类型满足所有所需属性的值：

```ts
loggingIdentity({length: 10, value: 3});
```

## 在泛型约束中使用类型参数

你可以声明一个被其他类型参数限制的类型参数。
比如，这里我们有两个对象，我们需要把属性从一个对象传递给另一个。
我们希望确保不会从`source`中意外地写入任何额外的属性，所以我们在两个类型中放置了一个约束：

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

## 在泛型中使用类类型

当我们在TypeScript中使用泛型创建工厂的时候，必须要参考对应的构造函数的类类型。比如，

```ts
function create<T>(c: {new(): T; }): T {
    return new c();
}
```

这是一个更高级的，使用原型属性来推断和约束构造函数和实例类型之间的类类型关系的例子。

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
