---
title: 关于正则表达式的简单总结
date: 2018-12-18 16:03:31
tags:
- regex
categories:
- coding
keywords:
- regex
- regular expression
description:
---

本文记录一下正则表达式的一些语法和常见用法。以备后查。

在线regex测试网站推荐[regex101](https://regex101.com)。

有关regex的资料推荐: 

- [regular-expressions.info](https://www.regular-expressions.info/quickstart.html)
- [runoob.com](http://www.runoob.com/regexp/regexp-metachar.html)

<!--more-->

## 基础概念语法

### Text patterns and Matches

pattern表示一个规则。在本文中将正则表达式的Text pattern使用`/`包裹起来。如`/cat/`

### Literal Characters

字面量字符，最普通的一种情况。比如`/cat/`, 输入字符串`he catch a catfish for his cat.` 在字符串的第4个位置找到匹配。

如果想继续匹配。则需要设置为全局模式。`/cat/g`，则会对输入字符串进行全局搜索。

在正则表达式中有12个字符有着特殊的含义，分别是：

> the backslash `\`, the caret `^`, the dollar sign `$`, the period or dot `.`, the vertical bar or pipe symbol `|`, the question mark `?`, the asterisk or star `*`, the plus sign `+`, the opening parenthesis `(`, the closing parenthesis `)`, the opening square bracket `[`, and the opening curly brace `{`

它们也叫做`metacharacters`。这些字符如果想被当成普通字符使用，需要使用`\`进行转译。如如果你想匹配`1+1=2`，你需要的Text pattern是`/1\+1=2/`

### Character Classes or Character Sets

Character Class 只能在几个字符中匹配到其中一个。比如`/gr[ae]y/`可以匹配`gray`, `grey`, 但不能匹配`graey`。可以使用hyphen（连字符-）来表示一个候选范围，如`/[0-9]/`可匹配一个数字。`/[a-zA-Z]/`可用来匹配`bH`, `cD`等。也可以单字符和连字符一起使用，如`/[x0-9]/`用来匹配字符`x`或`0-9`的数字。

在方括号里使用`^`用来表示取否。如`/[^abc]/`可匹配非a非b非c的字符，`/[^ ]/`可匹配非空格（blank space）的字符。

### Shorthand Character Classes

正则表达式提供了一些更简便的方式（Shorthand）来匹配一些常用的字符串。注意，这些shorthand具体包含的字符跟不同的regex flavour有关。不同的flavour可能还有其他的一些shorthand。这里讲一些通用的shorhand放在下表中。

| Shorthand | Equals       | Match                                            |
| --------- | ------------ | ------------------------------------------------ |
| \d        | [0-9]        | digit: 0-9的数字                                 |
| \w        | [a-zA-Z0-9_] | Word character: 用于组成单词的字符，下划线和数字 |
| \s        | [ \t\r\n\f]  | Whitespace ：空格，tab，换行符, form feed        |

Shorthand也可以有shorthand。一般用大写字母来表示取否。

| Negated Shorthand | Equals |
| ----------------- | ------ |
| \D                | [^\d]  |
| \W                | [^\w]  |
| \S                | [^\s]  |

在中括号中使用Negated Shorthand需要注意。如`[\D\S]`与`[^\d\s]`表达的含义不一样。`[^\d\s]`用来匹配即不是White space又不是数字的字符，而`[\D\S]`两个条件（\D或\S）满足一个即可匹配，换句话说，`[\D\S]`可以匹配数字，whitespace或者其他字符。这里面涉及的是逻辑上与或非关系。

可以在[regex101](https://regex101.com)上进行在线测试每个shorthand代表的简写。

{% asset_img regex101.png %}

### Non-Printable Characters

在正则表达式中可以使用特殊字符来匹配不可打印字符。

| pattern | match                 | pattern | match                 |
| ------- | --------------------- | ------- | --------------------- |
| \t      | tab字符（ASCII 0x09） | \r      | carriage return(0x0D) |
| \n      | line feed(0x0A)       | \f      | form feed(0x0C)       |

需要注意的是，在windows上使用`\r\n`来用作line break, 而unix上是`\n`.

### The Dot matches (Almost) any character

`.`可以匹配除了line break之外的所有字符。如`/gr.y/`可以匹配`gray`, `grey`, `/gr%y`. 谨慎的使用dot 字符。因为使用Character class匹配的速度回更快也更精确.

### Anchors

> Anchors do not match any characters. They match a position.

Anchors（锚）用来匹配一个位置。`^`用于匹配输入字符串的开始位置，`$`用于匹配输入字符串的结束位置。在`multi-line`模式下，`^`用来匹配每一行的开始位置，`$`用来匹配每一行的结束位置。如`/^a/`可以匹配`abc`中的a，但是再`bca`中找不到匹配。

`\b`用来匹配word boundary。

> A word boundary is a position between a character that can be matched by `\w` and a character that cannot be matched by `\w`

`\B`正好相反，用来匹配`\b`无法匹配的地方。

更多的内容相关请参考: [Learn more about anchors](https://www.regular-expressions.info/anchors.html).

### Alternation

Alternation相当于正则表达式中的"或"。如`/cat|dog/`匹配`cat and dogs`中的`cat`,如果再次匹配（如使用global模式），才会再次匹配到`dog`.

Alternation在正则表达式中的操作符中优先级最低。如`cat|dog food`可以用来匹配`cat`或者`dog food`。如果想要匹配`cat food`或者`dog food`，需要使用捕获组`(cat|dog)food`.

### Repetition

- quantifier `?`用来使得前面的token最多可以出现一次（0次或一次）。如`colou?r`匹配`colour`或`color` .

- quantifier `*`用来使得前面的token出现0次，一次，或多次。

- quentifier `+`用来使得前面的token至少出现一次（一次或多次）。

- 使用大括号`{}`用来表示前面的token出现特定次数。如`/\b[1-9][0-9]{3}\b/`匹配`1000-9999`的数字.

`/\b[1-9][0-9]{2,4}\b/`匹配100-9999.

### Greedy and Lazy Repetition

Repetition的匹配符都是“贪婪”的，他们会尽可能多的匹配，返回尽可能长的匹配字符串。如`/<.+>/`在输入字符串为`This is a <EM>first</EM> test`中返回的匹配字符串为`<EM>first</EM>`。

在`+`之后使用`?`使得匹配变为lazy. 使用`/<.+?>/`来匹配上面的字符串，则返回的是`<EM>`.

更好的方式是使用dot 运算符。

> Use `<[^<>]+>` to quickly match an HTML tag without regard to attributes. The negated character class is more specific than the dot, which helps the regex engine find matches quickly.

### Grouping and Capturing

使用`()`来对多个字符进行分组，可以使用repetition运算符来修饰分组。如`/set(value)?/ 可以匹配`set`或`setvalue`.

使用小括号分组即创建了一个捕获组。上面的例子有一个分组，如果匹配set，则group1什么都没有，如果匹配setvalue，则group1是`value`。group0总是整个匹配的字符串。至于如何访问group，这个跟你使用的编程语言有关。

创建捕获组之后，可以使用backreference来引用捕获组。backreference即使用backslash和group number来表示捕获组的内容。

如`/([abc])=\1/`可以匹配`a=a`,或`b=b`，或`c=c`

如果你只想分组不想创建捕获组可以使用`(？:)?`。如上面的例子，`set(?:value)?`, 这样只会创建分组，但不会创建捕获组。这样做的目的可以使得正则表达式匹配更快，性能更高。


