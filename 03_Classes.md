# 类
传统JavaScript侧重于函数和基于原型的继承来作为建立可复用组件的基本手段，但这会让那些更熟悉于面向对象方法的程序员感到尴尬，在面向对象中类继承的功能和对象是基于类构建的。从ECMAScript6开始，作为下一个版本的JavaScript，JavaScript程序员将可以使用面向对象的基于类的方法来构建应用程序。在TypeScript中，我们允许开发者马上使用这些技术，并且将它们编译成可在所有主流浏览器和平台上运行的JavaScript代码而不需要等待下一个版本的JavaScript。
</br>
## 类
让我们来看一个基于类的简单示例：
```TypeScript
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

var greeter = new Greeter("world");
```
如果你使用过C#和Java，那两者的语法应该看上去是相近的。我们声明了一个新的类“Greeter”。这个类有三个成员，包括一个叫“greeting”的属性，一个结构体和一个方法“greet”。
</br>
你需要注意到，在类中当我们引用一个类成员是我们使用了“this”前缀。这表示这是对一个成员的访问。
</br>
在最后一行我们使用“new”关键字构造了一个Greeter类的实例。这将会调用我们之前定义的构造体，它创建了一个新的Greeter对象并运行构造体函数来将其初始化。
</br>
## 继承
在TypeScript中，我们使用通用的面向对象的模式。当然，在基于类编程的最基本模式之一是允许使用继承来扩展已有的类来创建新的类。
</br>
让我们来看看这个例子：
```TypeScript
class Animal {
    name:string;
    constructor(theName: string) { this.name = theName; }
    move(meters: number = 0) {
        alert(this.name + " moved " + meters + "m.");
    }
}

class Snake extends Animal {
    constructor(name: string) { super(name); }
    move(meters = 5) {
        alert("Slithering...");
        super.move(meters);
    }
}

class Horse extends Animal {
    constructor(name: string) { super(name); }
    move(meters = 45) {
        alert("Galloping...");
        super.move(meters);
    }
}

var sam = new Snake("Sammy the Python");
var tom: Animal = new Horse("Tommy the Palomino");

sam.move();
tom.move(34);
```
这个例子覆盖了TypeScript中一些和其他语言共同的继承特性。这里我们看到了使用“extend”关键字来创建一个子类。你可以看到“Horse”和“Snake”作为“Animal”的子类并能访问它的特性。
</br>
这个例子同样展示了可以在专门的子类中覆盖基类的方法。“Snake”和“Horse”都创建了“move”方法并覆盖了“Animal”类中的该方法，给予它各自类的功能。
</br>
## 私有、公有修饰符
