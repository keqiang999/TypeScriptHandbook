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
