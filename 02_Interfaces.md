## 接口
TypeScript的核心原则之一便是着重于值所拥有的“形”的类型检查。这在有时被称为“鸭子类型”或“结构型亚型”。在TypeScript中，接口充当了命名这些类型的角色，同时也是和你自己及项目外代码定义约定的有效方式。
</br>
### 我们的第一个接口
了解接口是如何运作最简单的方式就是看一个简单的例子：
```TypeScript
function printLabel(labelledObj: {label: string}) {
  console.log(labelledObj.label);
}

var myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```
类型检查器检查了对“printLabel”的调用。这个“printLabel”函数有一个需要在传入的对象中有一个叫“label”的字符串类型的属性的变量。要注意的是我们的对象实际上包含比这更多的属性，但是编译器只检测那些目前需要用到的那些并匹配那些需要的类型。
</br>
我们可以再写一个例子，这次使用接口来描述需要“label”属性是一个字符串的需求：
```TypeScript
interface LabelledValue {
  label: string;
}

function printLabel(labelledObj: LabelledValue) {
  console.log(labelledObj.label);
}

var myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```
接口“LabelledValue”是一个我们现在可以用于描述之前需求的名字。它依然匹配有一个叫做“label”的字符串类型属性。注意我们并没有确切说这个传递给“printLabel”的对象继承了这个接口，而在其他语言中的确会这样。这里，它只是一个形式。如果我们传递给函数的对象满足了函数列出的需求，那么它就会被通过。
</br>
值得一提的是类型检测并不需要这些属性是有序排列的，除非接口需要的属性要求匹配和拥有需要的类型。
</br>
### 可选属性
在一个接口中并非全部属性都是必需的。有些属性存在于特定条件之下而另一些则不是。这些可选属性在创建一些用户传递给函数的对象只有几个属性被接受的模式比如“可选包”时会受到欢迎。
</br>
以下是该模式的例子：
```TypeScript
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
  var newSquare = {color: "white", area: 100};
  if (config.color) {
    newSquare.color = config.color;
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

var mySquare = createSquare({color: "black"});
```
包含可选属性的接口的写法类似于其他接口，是在每个可选属性名称后跟“?”来作为属性申明的一部分。
</br>
可选属性的又是在于你可以描述这些可能可用的属性的同时捕获那些你所知并不希望可用的属性。比如当我们拼错了我们传递给“createSquare”的属性的属性名称，我们会得到一条错误信息：
```TypeScript
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
  var newSquare = {color: "white", area: 100};
  if (config.color) {
    newSquare.color = config.collor;  // Type-checker can catch the mistyped name here
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

var mySquare = createSquare({color: "black"});
```
</br>
### 函数类型
