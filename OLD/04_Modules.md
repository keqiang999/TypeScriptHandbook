# 模块
这篇文章概述了在TypeScript中使用模块来组织代码的多种方式。我们将覆盖到内部和外部模块并且将讨论何时恰当的使用它们以及如何使用。我们同样将涉及一些使用扩展模块的高级话题，以及在TypeScript中使用模块时需要解决的一些常见陷阱。
** 第一步 **
让我们使用以下的代码来作为我们的例子开始。我们写了一组简单的字符串检验器，就像你在检查网页表单上的用户输入或检查外部提供的数据文件的格式时所要用到的那种。
* 单个文件中的检验器 *
```TypeScript
interface StringValidator {
    isAcceptable(s: string): boolean;
}

var lettersRegexp = /^[A-Za-z]+$/;
var numberRegexp = /^[0-9]+$/;

class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}

class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}

// Some samples to try
var strings = ['Hello', '98052', '101'];
// Validators to use
var validators: { [s: string]: StringValidator; } = {};
validators['ZIP code'] = new ZipCodeValidator();
validators['Letters only'] = new LettersOnlyValidator();
// Show whether each string passed each validator
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```
** 增加模块化 **
当我们增加更多检验器时，我们希望存在一种组织方案，能让我们追踪我们的类型而不用担心和其他对象发生名称冲突。作为把大量不同的名称放在全局命名空间中的替代，让我们把我们的对象移动到模块中。
 <br><br>
在这个例子中，我们把所有检验器相关的类型移动到了Validation模块中。因为我们希望这其中的接口和类在模块外是可以访问的，我们在它们的声明前加上export关键字。与之相反的是，变量lettersRegexp和numberRegexp是实现上的细节部分，所以不将其导出使其对模块外的代码不可见。在底部的测试代码中，如果要在模块外使用我们现在在需要修饰类型名称，比如Validation.LettersOnlyValidator。
* 模块化检验器 *
```TypeScript
module Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }

    var lettersRegexp = /^[A-Za-z]+$/;
    var numberRegexp = /^[0-9]+$/;

    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }

    export class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s);
        }
    }
}

// Some samples to try
var strings = ['Hello', '98052', '101'];
// Validators to use
var validators: { [s: string]: Validation.StringValidator; } = {};
validators['ZIP code'] = new Validation.ZipCodeValidator();
validators['Letters only'] = new Validation.LettersOnlyValidator();
// Show whether each string passed each validator
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```
## 跨文件分割
