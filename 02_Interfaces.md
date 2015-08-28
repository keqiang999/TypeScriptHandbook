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
可选属性的优势在于你可以描述这些可能可用的属性的同时捕获那些你所知并不希望可用的属性。比如当我们拼错了我们传递给“createSquare”的属性的属性名称，我们会得到一条错误信息：
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
接口能够描述JavaScript可以做到的广泛形式。除了能用属性来描述对象，接口同样能够描述函数类型。
</br>
为了用接口来描述函数类型，我们给接口一个调用签名。这就像是一个只有参数列表和返回值的函数定义。
```TypeScript
interface SearchFunc {
  (source: string, subString: string): boolean;
}
```
当完成定义后，我们可以使用其他接口一样使用这个函数类型接口。在这里，我们将展示如何创建一个函数类型变量并分配它一个相同类型的函数值。
```TypeScript
var mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
  var result = source.search(subString);
  if (result == -1) {
    return false;
  }
  else {
    return true;
  }
}
```
当函数类型正确通过类型检查，变量名称就不需要被匹配。我们可以例如把上面的例子这样来写：
```TypeScript
var mySearch: SearchFunc;
mySearch = function(src: string, sub: string) {
  var result = src.search(sub);
  if (result == -1) {
    return false;
  }
  else {
    return true;
  }
}
```
函数参数在某个时间被逐个检查，并检查相互间参数位置的类型。在这里，同样的，我们函数表达式的返回值是被它返回的值所隐含的（这里是false和true）。当函数表达式返回数字或字符串类型时，类型检查器会警告我们这个返回值和SearchFunc接口的返回值并不匹配。
</br>
### 数组类型
和我们用接口来描述函数类型的方式相似，我们同样可以使用接口来描述数组类型。数组类型包含一个“索引”类型来描述允许用于索引对象的类型，连同用于访问索引对应的返回类型。
```TypeScript
interface StringArray {
  [index: number]: string;
}

var myArray: StringArray;
myArray = ["Bob", "Fred"];
```
这里有两种支持的索引类型：字符串和数字。这两种索引类型是可以被同时支持的，限制在于用数字索引返回的类型必须是字符串索引返回类型的子类型。
</br>
索引签名是一个描述数组和“字典”模式的有效方式，他们同样强制所有属性匹配他们的返回类型。在这个例子中，属性并不匹配更普遍的索引，然后类型检查器报了一个错误：
```TypeScript
interface Dictionary {
  [index: string]: string;
  length: number;    // error, the type of 'length' is not a subtype of the indexer
} 
```
### 类类型
