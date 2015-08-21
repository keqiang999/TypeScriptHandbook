## 基础类型
为了让程序更为有用，我们需要使用一些数据的最小单元：数字、字符串、结构体、布尔值等等。在TypeScript中，除了支持JavaScript中已知的类型，还在此基础上引入了实用的枚举类型。
</br>
### 布尔类型（Boolean）
最基本的数据类型便是true/false值，它在JavaScript和TypeScript（当然包括其他语言）中被称为“布尔”值。
```TypeScript
var isDone: boolean = false;
```
</br>
### 数字类型（Number）
与在JavaScript中一样，TypeScript中所有的数字都是浮点数。这些浮点数被称为“数字”类型。
```TypeScript
var height: number = 6;
```
</br>
### 字符串（String）
另一种在为网页和服务器编写JavaScript程序的基本是使用文本数据。在其他语言中，我们使用“字符串”类型来表达这些文字数据。和JavaScript一样，TypeScript使用双引号（")或单引号（'）来包围文字数据。
```TypeScript
var name: string = "bob";
name = 'smith';
```
</br>
### 数组（Array）
TypeScript类似于JavaScript，允许你使用数组类型。数组类型可以用以下两种方式书写。第一种方式是特定类型后跟上“[]”来表示该种类型的数组：
```TypeScript
var list:number[] = [1, 2, 3];
```
第二种方式是使用通用数组类型，Array<元素类型>：
```TypeScript
var list:Array<number> = [1, 2, 3];
```
</br>
### 枚举（Enum）
在JavaScript标准数据类型之外增加的一种实用类型是“枚举”类型。类似C#之类的语言，枚举类型是一种使用一个友好名称来表示一组数值的方法。
```TypeScript
enum Color {Red, Green, Blue};
var c: Color = Color.Green;
```
默认情况下，枚举类型从0开始编号。你可以通过手动设定一个枚举成员的编号值来改变这点。例如，我们可以让之前的例子从1开始编号而不是0：
```TypeScript
enum Color {Red = 1, Green, Blue};
var c: Color = Color.Green;
```
甚至于你可以手动设定所有枚举成员的编号值：
```TypeScript
enum Color {Red = 1, Green = 2, Blue = 4};
var c: Color = Color.Green;
```
枚举类型一个便利的特性是你可以通过一个数值得到它在枚举类型中对应的名称。例如，如果我们有一个值2但我们不确定它在上述Color枚举类型中对应的名称，我们可以通过以下方法查找它所对应的名称：
```TypeScript
enum Color {Red = 1, Green, Blue};
var colorName: string = Color[2];

alert(colorName);
```
</br>
### 任意类型（Any）
我们有时会需要声明一个在我们编程时并不知道具体类型的变量。它们的值可能来自于动态内容，比如来自于用户输入或者第三方库。在这种情况下，我们需要跳过类型检查并且让这些值通过编译时检测。为了达到这个目的，我们将它们标为“任意”类型：
```TypeScript
var notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // okay, definitely a boolean
```
“任意”类型在使用已有JavaScript代码时十分有用，它允许你在编译过程中逐渐选择使用或跳过类型检查。
“任意”类型在你知道某类型的一部分但或许并不是全部的时同样便利。例如你在使用一个混合数据类型的数组时：
```TypeScript
var list:any[] = [1, true, "free"];

list[1] = 100;
```
</br>
### 空类型（Void）
在某些情况下相对于“任何”的是“空”,当不属于任何具体类型时。你可能更多的是在无返回值函数的返回类型上见到它：
```TypeScript
function warnUser(): void {
    alert("This is my warning message");
}
```
