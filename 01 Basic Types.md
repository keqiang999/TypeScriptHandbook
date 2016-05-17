# 引言

为了让程序可用，我们需要能够处理一些最简单的数据单元：数字、字符串、结构体、布尔值等等。
在TypeScript中，我们支持正如你期望的与JavaScript中同样多的类型，以及额外一个便利的枚举类型来帮助处理一些事情。

# 布尔

最基础的数据类型是简单的true/false值，它在JavaScript和TypeScript中被称为`boolean`值。

```ts
let isDone: boolean = false;
```

# 数字

与在JavaScript中同样，在TypeScript中所有的数字都是浮点数值。
这些浮点数值被称为`number`类型。
除了十进制和十六进制文本，TypeScript同样支持在ECMAScript 2015中引入的二进制和八进制数。


```ts
let decimal: number = 6;
let hex: number = 0xf00d;
let binary: number = 0b1010;
let octal: number = 0o744;
```

# 字符串

在为网页和服务器创建JavaScript程序的时，另一个基本部分是处理文字数据。
和其他编程语言中一样，我们使用`string`类型来指代这些文字数据类型。
TS和JS一样使用双引号(`"`)或单引号(`'`)来包围字符串数据。

```ts
let color: string = "blue";
name = 'red';
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

# 数组

TS如同JS那样，允许你处理值的数组。
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
x = ['hello', 10]; // OK
// Initialize it incorrectly
x = [10, 'hello']; // Error
```

当根据已知索引来访问元素时，它正确的类型将会被返回：

```ts
console.log(x[0].substr(1)); // OK
console.log(x[1].substr(1)); // Error, 'number' does not have 'substr'
```

当访问一个已知索引外的元素时，将会使用联合类型来做为替代：

```ts
x[3] = 'world'; // OK, string can be assigned to (string | number)

console.log(x[5].toString()); // OK, 'string' and 'number' both have toString

x[6] = true; // Error, boolean isn't (string | number)
```

联合类型将做为高级话题在我们之后的章节提到。

# 枚举

A helpful addition to the standard set of datatypes from JavaScript is the `enum`.
As in languages like C#, an enum is a way of giving more friendly names to sets of numeric values.


```ts
enum Color {Red, Green, Blue};
let c: Color = Color.Green;
```

By default, enums begin numbering their members starting at `0`.
You can change this by manually setting the value of one of its members.
For example, we can start the previous example at `1` instead of `0`:

```ts
enum Color {Red = 1, Green, Blue};
let c: Color = Color.Green;
```

Or, even manually set all the values in the enum:

```ts
enum Color {Red = 1, Green = 2, Blue = 4};
let c: Color = Color.Green;
```

A handy feature of enums is that you can also go from a numeric value to the name of that value in the enum.
For example, if we had the value `2` but weren't sure what that mapped to in the `Color` enum above, we could look up the corresponding name:

```ts
enum Color {Red = 1, Green, Blue};
let colorName: string = Color[2];

alert(colorName);
```

# Any

We may need to describe the type of variables that we do not know when we are writing an application.
These values may come from dynamic content, e.g. from the user or a 3rd party library.
In these cases, we want to opt-out of type-checking and let the values pass through compile-time checks.
To do so, we label these with the `any` type:

```ts
let notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // okay, definitely a boolean
```

The `any` type is a powerful way to work with existing JavaScript, allowing you to gradually opt-in and opt-out of type-checking during compilation.
You might expect `Object` to play a similar role, as it does in other languages.
But variables of type `Object` only allow you to assign any value to them -- you can't call arbitrary methods on them, even ones that actually exist:

```ts
let notSure: any = 4;
notSure.ifItExists(); // okay, ifItExists might exist at runtime
notSure.toFixed(); // okay, toFixed exists (but the compiler doesn't check)

let prettySure: Object = 4;
prettySure.toFixed(); // Error: Property 'toFixed' doesn't exist on type 'Object'.
```

The `any` type is also handy if you know some part of the type, but perhaps not all of it.
For example, you may have an array but the array has a mix of different types:

```ts
let list: any[] = [1, true, "free"];

list[1] = 100;
```

# Void

`void` is a little like the opposite of `any`: the absence of having any type at all.
You may commonly see this as the return type of functions that do not return a value:

```ts
function warnUser(): void {
    alert("This is my warning message");
}
```

Declaring variables of type `void` is not useful because you can only assign `undefined` or `null` to them:

```ts
let unusable: void = undefined;
```

# Type assertions

Sometimes you'll end up in a situation where you'll know more about a value than TypeScript does.
Usually this will happen when you know the type of some entity could be more specific than its current type.

*Type assertions* are a way to tell the compiler "trust me, I know what I'm doing."
A type assertion is like a type cast in other languages, but performs no special checking or restructuring of data.
It has no runtime impact, and is used purely by the compiler.
TypeScript assumes that you, the programmer, have performed any special checks that you need.

Type assertions have two forms.
One is the "angle-bracket" syntax:

```ts
let someValue: any = "this is a string";

let strLength: number = (<string>someValue).length;
```

And the other is the `as`-syntax:

```ts
let someValue: any = "this is a string";

let strLength: number = (someValue as string).length;
```

The two samples are equivalent.
Using one over the other is mostly a choice of preference; however, when using TypeScript with JSX, only `as`-style assertions are allowed.

# A note about `let`

You may've noticed that so far, we've been using the `let` keyword instead of JavaScript's `var` keyword which you might be more familiar with.
The `let` keyword is actually a newer JavaScript construct that TypeScript makes available.
We'll discuss the details later, but many common problems in JavaScript are alleviated by using `let`, so you should use it instead of `var` whenever possible.
