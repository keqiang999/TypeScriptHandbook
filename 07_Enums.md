# 枚举

枚举允许我们定义一组命名的数字常量
我们使用 `enum` 关键字来定义一个枚举类型。

```ts
enum Direction {
    Up = 1,
    Down,
    Left,
    Right
}
```

枚举体可以包含一个或多个枚举成员
枚举成员有一个与之相关联的数值，其既可以是*常量*也可以是*计算得出的值*。
当满足如下条件时，枚举成员的值被认为是不变的:

* 其未被初始化且前一个值为常量时
    在这种情况下，当前的枚举成员的值为前一枚举成员值加1。
    但如果第一个元素未被初始化，则它将被初始化为 `0`。
* 当枚举成员被初始化为一个常量表达式时。
    由于常量表达式是 TypeScript 表达式的一个子集， 因此可以在编译时求值
    如果一个表达式是一个常量枚举表达式，那么它必须是以下情形之一:
    * 数值型
    * 引用之前定义的常量枚举成员（可以是在不同的枚举类型中定义的）。
         如果这个成员是在同一个枚举中，可以使用非限定名来引用
    * 带括号的常量枚举表达式
    * `+`, `-`, `~` 单目运算符常量表达式
    * `+`, `-`, `*`, `/`, `%`, `<<`, `>>`, `>>>`, `&`, `|`, `^` 双目运算符且作为操作数
    倘若运算后的值为`NaN` 或者 `Infinity`，则将会在编译时报错。

在其他情况下枚举值被视为计算获得。

```ts
enum FileAccess {
    // constant members
    None,
    Read    = 1 << 1,
    Write   = 1 << 2,
    ReadWrite  = Read | Write,
    // computed member
    G = "123".length
}
```

枚举是在运行时真实存在的对象
原因之一是保证从枚举值到枚举名能进行反向映射。

```ts
enum Enum {
    A
}
let a = Enum.A;
let nameOfA = Enum[Enum.A]; // "A"
```

编译后:

```js
var Enum;
(function (Enum) {
    Enum[Enum["A"] = 0] = "A";
})(Enum || (Enum = {}));
var a = Enum.A;
var nameOfA = Enum[Enum.A]; // "A"
```

编译后的枚举代码是一个双向映射的对象，包含正向 (`name` -> `value`) 和反向(`value` -> `name`) 。
引用枚举成员总是发出属性访问且从不内联。
在很多情况下，这是一个理想的解决方案。
然而有时要求会更严格。
当访问枚举值时，为了避免产生额外的代码和间接引用，我们可以使用常量枚举
常量枚举是指在`enum`关键字之前添加 `const`修饰。

```ts
const enum Enum {
    A = 1,
    B = A * 2
}
```

常量枚举只能使用常量表达式且会在编译时被删除，并在使用的地方进行内联。
这是因为常量枚举不能有计算成员。

```ts
const enum Directions {
    Up,
    Down,
    Left,
    Right
}

let directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right]
```

编译后的代码如下所示

```js
var directions = [0 /* Up */, 1 /* Down */, 2 /* Left */, 3 /* Right */];
```

# 外部枚举

外部枚举是用来声明对外部代码库中的枚举类型进行描述。

```ts
declare enum Enum {
    A = 1,
    B,
    C = 2
}
```

外部和非外部枚举之间的一个重要区别在于，普通枚举成员未被初始化时被认为是一个常量。
对于非常量外部枚举未初始化时被认为是计算出来的。
