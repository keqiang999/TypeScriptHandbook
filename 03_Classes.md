# 类
传统JavaScript侧重于函数和基于原型的继承来作为建立可复用组件的基本手段，但这会让那些更熟悉于面向对象方法的程序员感到尴尬，在面向对象中类继承的功能和对象是基于类构建的。从ECMAScript6开始，作为下一个版本的JavaScript，JavaScript程序员将可以使用面向对象的基于类的方法来构建应用程序。在TypeScript中，我们允许开发者马上使用这些技术，并且将它们编译成可在所有主流浏览器和平台上运行的JavaScript代码而不需要等待下一个版本的JavaScript。
<br><br>
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
<br><br>
你需要注意到，在类中当我们引用一个类成员是我们使用了“this”前缀。这表示这是对一个成员的访问。
<br><br>
在最后一行我们使用“new”关键字构造了一个Greeter类的实例。这将会调用我们之前定义的构造体，它创建了一个新的Greeter对象并运行构造体函数来将其初始化。
<br><br>
## 继承
在TypeScript中，我们使用通用的面向对象的模式。当然，在基于类编程的最基本模式之一是允许使用继承来扩展已有的类来创建新的类。
<br><br>
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
<br><br>
这个例子同样展示了可以在专门的子类中覆盖基类的方法。“Snake”和“Horse”都创建了“move”方法并覆盖了“Animal”类中的该方法，给予它各自类的功能。
<br><br>
## 私有、公有修饰符
### 默认为公有
你可能注意到了在上面的例子里，我们并非必须使用“public”关键词来让类中的成员可见。像C#之类的语言中需要强制类成员标记“public”关键字来使其可见。在Typescript中，每个成员默认是公有的。
<br><br>
你同样可以将类成员标为私有，这样你可以控制你的类哪些成员是对外可见的。我们可以将前一部分中的“Animal”类这样改写：
```TypeScript
class Animal {
    private name:string;
    constructor(theName: string) { this.name = theName; }
    move(meters: number) {
        alert(this.name + " moved " + meters + "m.");
    }
}
```
### 理解私有
TypeScript是一个结构化类型系统。当我们比较两种不同类型时，无论他们来自哪里，当其中每个成员的类型相互匹配的时候，我们就成这两种类型是相互匹配的。
<br><br>
当我们比较有私有成员的类型时，我们对待它们的方式会有所不同。为了让两种类型能够匹配，当它们中的一个有一个私有成员，那另一个则必须有源自同一个声明的的私有成员。
<br><br>
让我们来看一个例子来更好的理解这是怎么回事：
```TypeScript
class Animal {
    private name:string;
    constructor(theName: string) { this.name = theName; }
}

class Rhino extends Animal {
	constructor() { super("Rhino"); }
}

class Employee {
    private name:string;
    constructor(theName: string) { this.name = theName; }	
}

var animal = new Animal("Goat");
var rhino = new Rhino();
var employee = new Employee("Bob");

animal = rhino;
animal = employee; //error: Animal and Employee are not compatible
```
在这个例子中，我们有“Animal”和“Rhino”类，而“Rhino”类是“Animal”的一个子类。同时我们有一个叫“Employee”的类且有和“Animal”类同样的结构。我们穿件了一些这些类的实例并试图让他们进行匹配来看看会发生什么。因为“Animal”和“Rhino”在同样的定义中共享了私有部分：“Animal”类中的“private name: string”，它们是匹配的。然而，这在“Employee”的情况下却不是。当我们试图从一个“Employees”匹配“Animal”时我们得到了一个类型不匹配的错误，即使“Employee”同样有一个叫做“name”的私有成员，但它并不是那个由“Animal”创建的成员。
### 参数属性
在创建参数属性时，关键字“public”和“private”同样可以作为给你创建和初始化类中成员的简写方式。这种属性让你可以一步创建和初始化一个成员。这里是之前例子的一个进一步修改版本。注意我们如何把“theName”丢在一起并且只是使用简写的“private name: string”结构体参数来创建和初始化“name”成员。
```TypeScript
class Animal {
    constructor(private name: string) { }
    move(meters: number) {
        alert(this.name + " moved " + meters + "m.");
    }
}
```
以这种方式使用“private”来创建和初始化一个私有成员，“public”时也是类似的。
<br><br>
## 访问器
TypeScript支持getter/setter方法来作为一个拦截到一个对象成员访问的方式。这给你一个获得更好控制如何访问每个对象中成员的方式。
<br><br>
让我们使用‘get’和‘set’来转换一个简单的类。首先，让我们用一个未使用访问器器的例子开始。
```TypeScript
class Employee {
    fullName: string;
}

var employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    alert(employee.fullName);
}
```
虽然允许人们能随意地直接设定fullName变量十分便利，但这会在别人带有恶意地修改名字时带来麻烦。
<br><br>
在这个版本中，我们在允许用户修改职员信息前进行了检查以确保用户拥有一个有效的验证码。我们将直接访问fullName变量的方式替换为一个会检查验证码的‘set’函数。我们增加一个对应的‘get’函数来允许之前的例子能无缝运作。
```TypeScript
var passcode = "secret passcode";

class Employee {
    private _fullName: string;

    get fullName(): string {
        return this._fullName;
    }
	
    set fullName(newName: string) {
        if (passcode && passcode == "secret passcode") {
            this._fullName = newName;
        }
        else {
            alert("Error: Unauthorized update of employee!");
        }
    }
}

var employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    alert(employee.fullName);
}
```
为了自我验证我们的访问器现在会检查验证满，我们可以修改验证码，之后便能看到当验证码不匹配时我们会看到一个警告框告诉我们没有权限去更新职员信息。
<br><br>
注意：访问器需要你设置编译器输出为ECMAScript5.
## 静态属性
