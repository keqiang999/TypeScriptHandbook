# 引言

为了让程序可用，我们需要能够处理一些最简单的数据单元：数字、字符串、结构体、布尔值等等。
在TypeScript中，我们支持如你所期望那样的与JavaScript中同样多的类型，以及额外一个便利的枚举类型来帮助处理一些事情。

# Boolean（布尔类型）

最基础的数据类型是简单的true/false值，它在JavaScript和TypeScript中被称为`boolean`值。

```ts
let isDone: boolean = false;
```

# Number（数字类型）

与在JavaScript中同样，在TypeScript中所有的数字都是浮点数值。
这些浮点数值被称为`number`类型。
除了十进制和十六进制文本，TypeScript同样支持在ECMAScript 2015中引入的二进制和八进制数。


```ts
let decimal: number = 6;
let hex: number = 0xf00d;
let binary: number = 0b1010;
let octal: number = 0o744;
```

# String（字符串类型）

在为网页和服务器创建JavaScript程序的时，另一个基本部分是处理文字数据。
和其他编程语言中一样，我们使用`string`类型来指代这些文字数据类型。
TypeScript和JavaScript一样使用双引号(`"`)或单引号(`'`)来包围字符串数据。

```ts
let color: string = "blue";
color = 'red';
```

你同样可以使用 *模板字符串* 来跨行或嵌入表达式。
这些字符串被反引号(`` ` ``)字符包围，而嵌入的表达式则基于`${ expr }`的形式。

```ts
let fullName: string = `Bob Bobbington`;
let age: number = 37;
let sentence: string = `Hello, my name is ${ fullName }.

I'll be ${ age + 1 } years old next month.`
```

这和这样声明`sentence`的方式是等价的：

```ts
let sentence: string = "Hello, my name is " + fullName + ".\n\n" +
    "I'll be " + (age + 1) + " years old next month."
```

# Array（数组类型）

TypeScript如同JavaScript那样，允许你处理值的数组。
数组类型可以以两种方式书写。
第一种方式，你可以在元素类型后跟上`[]`来表示该元素类型的数组：

```ts
let list: number[] = [1, 2, 3];
```

第二种方式是使用一个泛型数组类型`Array<elemType>`：

```ts
let list: Array<number> = [1, 2, 3];
```

# 元组

元组类型允许你表达一个元素数量固定而类型不必完全一致的数组类型。
比如，你可能想要由一个`string`值和一个`number`值的组合来表示一个值：

```ts
// Declare a tuple type
let x: [string, number];
// Initialize it
x = ["hello", 10]; // OK
// Initialize it incorrectly
x = [10, "hello"]; // Error
```

当根据已知索引来访问元素时，它正确的类型将会被返回：

```ts
console.log(x[0].substr(1)); // OK
console.log(x[1].substr(1)); // Error, 'number' does not have 'substr'
```

当访问一个已知索引外的元素时，将会使用联合类型来做为替代：

```ts
x[3] = "world"; // OK, 'string' can be assigned to 'string | number'

console.log(x[5].toString()); // OK, 'string' and 'number' both have 'toString'

x[6] = true; // Error, 'boolean' isn't 'string | number'
```

联合类型将做为高级话题在我们之后的章节提到。

# Enum（枚举类型）

在JavaScript的标准数据类型基础上额外增加的一种实用类型是`enum`类型。
和C#等其他语言一样，枚举类型是给予一组数字值更友好名称的一种方式。

```ts
enum Color {Red, Green, Blue};
let c: Color = Color.Green;
```

默认情况下，枚举类型从`0`开始编号其成员。
你可以通过手动某个成员的值来改变这点。
比如，我们让上一个例子从`1`开始编号，而不是从`0`开始：

```ts
enum Color {Red = 1, Green, Blue};
let c: Color = Color.Green;
```

更甚至于可以手动设定枚举对象中所有成员的值：

```ts
enum Color {Red = 1, Green = 2, Blue = 4};
let c: Color = Color.Green;
```

枚举类型的一个便利特性是你可以通过数字值来得到枚举类型中该值所应的名称。
例如，当我们有一个值`2`但并不确定它在上面的`Color`枚举对象中的映射关系，我们可以查找其对应的名称：

```ts
enum Color {Red = 1, Green, Blue};
let colorName: string = Color[2];

alert(colorName);
```

# Any（任意类型）

我们可能会需要描述一个在编程时并不知道具体类型的变量。
这些变量值可能来源于动态内容，比如来自于用户或者第三方库。
在这种情况下，我们希望停用类型检查并让变量值通过编译时的检查。
为了达到这个目的，我们将它们标记为`any`类型：

```ts
let notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // okay, definitely a boolean
```

`any`类型是在使用现有JavaScript时一种强力的方式，它允许你在编译过程中逐步选择和退出类型检查。
你可能认为`Object`能起到类似的作用，正如它在其他语言中所做的那样。
但是`Object`变量仅允许你将任意值附加上去而你却不能调用任意方法，哪怕该方法真实存在：

```ts
let notSure: any = 4;
notSure.ifItExists(); // okay, ifItExists might exist at runtime
notSure.toFixed(); // okay, toFixed exists (but the compiler doesn't check)

let prettySure: Object = 4;
prettySure.toFixed(); // Error: Property 'toFixed' doesn't exist on type 'Object'.
```

`any`类型在当你知道某个类型的一部分，但也许并不是全部的时候同样便利。
例如，可能有一个数组但是该数组由不同的类型混合而成：

```ts
let list: any[] = [1, true, "free"];

list[1] = 100;
```

# Void（空类型）

`void`类型有点像相反的`any`类型：没有任何类型的类型。
你会在不返回任何值的函数返回类型上更多的看到它：

```ts
function warnUser(): void {
    alert("This is my warning message");
}
```

声明类型为`void`变量并不实用，因为你仅能为它们附上`undefined`值或`null`值：

```ts
let unusable: void = undefined;
```

# Null和Undefined

在TypeScript中，`undefined`和`null`事实上都有它们自己的类型，分别叫做`undefined`和`null`。
和`void`很像，它们对于自身来说并无多大用处：

```ts
// Not much else we can assign to these variables!
let u: undefined = undefined;
let n: null = null;
```

在默认情况下`null`和`undefined`是所有其他类型的字类型。
这意味着你可以给其他类型，比如`number`指定`null`或`undefined`类型。

然而，当使用`--strictNullChecks`标识的时候，`null`和`undefined`仅能分配给`void`和它们各自的类型。
这有助于避免*许多*常见错误。
在当你想要传入一个`string`、`null`、`undefined`三者之一时，你可以使用联合类型`string | null | undefined`。
再一次，我们会在之后更多关注联合类型。

> 说明：我们鼓励尽可能使用`--strictNullChecks`，但是出于这本手册的目的，我们将假定这个标识是关闭的。

# 类型断言

有时候你可能会碰到对于一个值，你所知道的多于TypeScript所做的情况。
这经常发生于当你知道一些实体的类型比它当前的类型更为具体时。

*类型断言* 是一种告诉编译器“相信我，我知道我在做什么。”的一种方式。
一个类型断言类似于一个在其他语言中扮演的类型，但不进行特殊的检查或调整数据。
它在运行时没有影响，而仅仅被编译器所实用。
TypeScript假定你做为程序员已经完成了所有你所需做到的特殊检查。

类型断言有两种形式。
一种是“尖括号”语法：

```ts
let someValue: any = "this is a string";

let strLength: number = (<string>someValue).length;
```

另一种是`as`语法：

```ts
let someValue: any = "this is a string";

let strLength: number = (someValue as string).length;
```

这两个例子是等价的。
选用何种方法更多的基于喜好；无论如何，当配合JSX使用TypeScript时，只有`as`风格断言是被允许的。

# 关于`let`的说明

你可能已经注意到了，我们使用了`let`关键字而不是JavaScript中更为熟悉的`var`关键字。
这个`let`关键字事实上是一个更新的JavaScript结构体，且由TypeScript来让它可用。
我们将会在之后讨论具体细节，但是JavaScript中许多通用问题会通过使用`let`来缓解，所以你应当尽可能的使用它来替代`let`。