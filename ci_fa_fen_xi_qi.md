* * *

# 第 12 章 词法分析器

* * *

### 目录

*   [12.1 概述](http://zh.highscore.de/cpp/boost/parser.html#parser_general)
*   [12.2 扩展BNF范式](http://zh.highscore.de/cpp/boost/parser.html#parser_ebnf)
*   [12.3 语法](http://zh.highscore.de/cpp/boost/parser.html#parser_grammar)
*   [12.4 动作](http://zh.highscore.de/cpp/boost/parser.html#parser_actions)
*   [12.5 练习](http://zh.highscore.de/cpp/boost/parser.html#parser_exercises)

* * *

## 12.1. 概述

词法分析器用于读取各种格式的数据，这些数据可以具有灵活但可能非常复杂的结构。 关于"格式"的一个最好的例子就是 C++ 代码。 编译器的词法分析器必须理解 C++ 的各种可能的语言结构组合，以将它们翻译为某种二进制形式。

开发词法分析器的主要问题是所分析的数据的组成结构具有大量的规则。 例如，C++ 支持很多的语言结构，开发一个相应的词法分析器可能需要无数个 `if` 表达式来识别任意所能想象到的 C++ 代码是否有效。

本章所介绍的 [Boost.Spirit](http://www.boost.org/libs/spirit/) 库将词法分析器的开发放到了桌面上来。 无需将明确的规则转换为代码并使用无数的 `if` 表达式来验证代码，Boost.Spirit 可以使用一种被称为扩展BNF范式的东西来表示规则。 通过使用这些规则，Boost.Spirit 就可以对一个 C++ 源代码进行分析。

Boost.Spirit 的基本思想类似于正则表达式。 即不用 `if` 表达式来查找指定模式的文本，而是将模式以正则表达式的方式指定出来。 然后就可以使用象 Boost.Regex 这样的库来执行相应的查找国，因此开发者无需关心其中的细节。

本章将示范如何用 Boost.Spirit 来读入正则表达式不再适用的复杂格式。 由于 Boost.Spirit 是一个功能非常全的库，引入了多个不同的概念，所以在本章我们将开发一个 [JSON 格式](http://www.json.org/) 的简单的词法分析器。 JSON 是被如 Ajax 一类的应用程序用于在程序之间交换数据的格式，类似于 XML，可以在不同平台上运行。

虽然 Boost.Spirit 简化了词法分析器的开发，但是还没有人能够成功地基于这个库写出一个 C++ 词法分析器。 这类词法分析器的开发仍然是 Boost.Spirit 的一个长期目标，不过，由于 C++ 语言的复杂性，目前还未能实现。 Boost.Spirit 目前还不能很好地适用于这种复杂性或二进制格式。

* * *

## 12.2. 扩展BNF范式

Backus-Naur 范式，简称 BNF，是一种精确描述规则的语言，被应用于多种技术规范。 例如，众多互联网协议的许多技术规范，称为 RFC，除了文字说明以外，都包含了以 BNF 编写的规则。

Boost.Spirit 支持扩展BNF范式(EBNF)，可以用比 BNF 更简短的方式来指定规则。 EBNF 的主要优点就是简短，从而写法更简单。

请注意，EBNF 有几种不同的变体，它们的语法可能有些差异。 本章以及 Boost.Spirit 所使用的 EBNF 语法类似于正则表达式。

要使用 Boost.Spirit，你必须懂得 EBNF。 多数情况下，开发者已经知道 EBNF，因此才会选择 Boost.Spirit 来重用以前用 EBNF 表示的规则。 以下是对 EBNF 的一个简短介绍；如果需要对本章当中以及 Boost.Spirit 所使用的语法有一个快速的参考，请查看 W3C XML 规范，其中包含了一个 [短摘要](http://www.w3.org/TR/2004/REC-xml-20040204/#sec-notation)。

| `digit` | = | "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9" |

严格地讲，EBNF 以生成规则来表示规则。 可以将任意数量的生成规则组合起来，描述一个特定的格式。 以上格式只包含一个生成规则。 它定义了一个 `digit` 是由0至9之间的任一数字组成。

象 `digit` 这样的定义被称为非终结符号。 以上定义中的数字 0 到 9 则被称为终结符号。 这些符号不具有任意特定意义，而且很容易识别出来，因为它们是用双引号引起来的。

所有数字值是用竖直符相连的，其意义与 C++ 中的 `||` 操作符一样：多选一。

一句话，这个生成规则指明了0至9之间的任一数字都是一个 `digit`。

| `integer` | = | ("+" | "-")? `digit`+ |

这个新的非终结符 `integer` 包含至少一个 `digit`，而且可选地以一个加号或减号开头。

`integer` 的定义用到了多个新的操作符。 圆括号用于创建一个子表达式，就象它在数学中的作用。 其它操作符可应用于这些子表达式。 问号表示这个子表达式只能出现一次或不出现。

`digit` 之后的加号表示相应的表达式必须出现至少一次。

这个新的生成规则定义了一个任意的正或负的整数。 一个 `digit` 正好是一个数字，而一个 `integer` 则可以由多个数字组成，且可以被标记为无符号的或有符号的。 因此 5 即是一个 `digit` 也是一个 `integer`，而 +5 则只是一个 `integer`。 同样地，169 或 -8 也只是 `integer`。

通过定义和组合各种非终结符，可以创建越来越复杂的生成规则。

| `real` | = | `integer` "." `digit`* |

`integer` 的定义表示的是整数，而 `real` 的定义则表示了浮点数。 这个规则基于前面已定义的非终结符 `integer` 和 `digit`，以一个句点号分隔。 `digit` 之后的星类表示点号之后的数字是可选的：可以有任意多个数字或没有数字。

浮点数如 1.2, -16.99 甚至 3\. 都符合 `real` 的定义。 但是，当前的定义不允许浮点数不带前导的零，如 .9。

正如本章开始的时候所提到的，接下来我们要用 Boost.Spirit 开发一个 JSON 格式的词法分析器。 为此，需要用 EBNF 来给出 JSON 格式的规则。

| `object` | = | "{" `member` ("," `member`)* "}" |
| `member` | = | `string` ":" `value` |
| `string` | = | '"' `character`* '"' |
| `value` | = | `string` | `number` | `object` | `array` | "true" | "false" | "null" |
| `number` | = | `integer` | `real` |
| `array` | = | "[" `value` ("," `value`)* "]" |
| `character` | = | "a" | "b" | "c" | "d" | "e" | "f" | "g" | "h" | "i" | "j" | "k" | "l" | "m" | "n" | "o" | "p" | "q" | "r" | "s" | "t" | "u" | "v" | "w" | "x" | "y" | "z" |

JSON 格式基于一些包含了键值和值的成对的对象，它们被花括号括起来。 其中键值是普通的字符串，而值可以是字符串、数字值、数组、其它对象或是字面值 `true`, `false` 或 `null`。 字符串是由双引号引起来的连续字符。 数字值可以是整数或浮点数。 数组包含以逗号分隔的值，并以方括号括起来。

请注意，以上定义并不完整。 一方面，`character` 的定义缺少了大写字母以及其它字符；另一方面，JSON 还特别支持 Unicode 或控制字符。 这些现在都可以先忽略掉，因为 Boost.Spirit 定义了常用的非终结符号，如字母数字字符，以减少你打字的次数。 另外，稍后在代码中，字符串被定义为除双引号以外的任意字符的连续串。 由于双引号用于结束一个字符串，所以其它所有字符都在字符串中使用。 上述 EBNF 并不如此表示，因为 EBNF 要求定义一个包含除单个字符外的所有字符的非终结符号，应该定义一个例外来排除。

以下是使用了上述定义的 JSON 格式的一个例子。

~~~
{
  "Boris Schäling" :
  {
    "Male": true,
    "Programming Languages": [ "C++", "Java", "C#" ],
    "Age": 31
  }
}
~~~

整个对象由最外层的花括号给出，它包含了一个键-值对。 键值是 "Boris Schäling"，值则是一个新的对象，包含多个键-值对。 其中所有键值均为字符串，而值则分别为字面值 `true`，一个包含几个字符串的数组，以及一个数字值。

以上所定义的 EBNF 规则现在就可用于通过 Boost.Spirit 开发一个可以读取以上 JSON 格式的词法分析器。

* * *

## 12.3. 语法

继上一节中以 EBNF 为 JSON 格式定义了相关规则后，现在要将这些规则与 Boost.Spirit 一起使用。 Boost.Spirit 实际上允许以 C++ 代码来定义 EBNF 规则，方法是重载各个由 EBNF 使用的不同操作符。

请注意，EBNF 规则需要稍作修改，才能创建出合法的 C++ 代码。 在 EBNF 中各个符号是以空格相连的，在 C++ 中需要用某个操作符来连接。 此外，象星号、问号和加号这些操作符，在 EBNF 中是置于对应的符号后面的，在 C++ 中必须置于符号的前面，才能作为单参操作符来使用。

以下是在 Boost.Spirit 中为表示 JSON 格式，用 C++ 代码写的 EBNF 规则。

~~~
#include <boost/spirit.hpp> 

struct json_grammar 
  : public boost::spirit::grammar<json_grammar> 
{ 
  template <typename Scanner> 
  struct definition 
  { 
    boost::spirit::rule<Scanner> object, member, string, value, number, array; 

    definition(const json_grammar &self) 
    { 
      using namespace boost::spirit; 
      object = "{" >> member >> *("," >> member) >> "}"; 
      member = string >> ":" >> value; 
      string = "\"" >> *~ch_p("\"") >> "\""; 
      value = string | number | object | array | "true" | "false" | "null"; 
      number = real_p; 
      array = "[" >> value >> *("," >> value) >> "]"; 
    } 

    const boost::spirit::rule<Scanner> &start() 
    { 
      return object; 
    } 
  }; 
}; 

int main() 
{ 
} 
~~~

*   [下载源代码](http://zh.highscore.de/cpp/boost/src/12.3.1/main.cpp)

为了使用 Boost.Spirit 中的各个类，需要包含头文件 `boost/spirit.hpp`。 所有类均位于名字空间 `boost::spirit` 内。

为了用 Boost.Spirit 创建一个词法分析器，除了那些定义了数据是如何构成的规则以外，还必须创建一个所谓的语法。 在上例中，就创建一个 `json_grammar` 类，它派生自模板类 `boost::spirit::grammar`，并以该类的名字来实例化。 `json_grammar` 定义了理解 JSON 格式所需的完整语法。

语法的一个最重要的组成部分就是正确读入结构化数据的规则。 这些规则在一个名为 `definition` 的内层类中定义 - 这个名字是强制性的。 这个类是带有一个模板参数的模板类，由 Boost.Spirit 以一个所谓的扫描器来进行实例化。 扫描器是 Boost.Spirit 内部使用的一个概念。 虽然强制要求 `definition` 必须是以一个扫描器类型作为其模板参数的模板类，但是对于 Boost.Spirit 的日常使用来说，这些扫描器是什么以及为什么要定义它们，并不重要。

`definition` 必须定义一个名为 `start()` 的方法，它会被 Boost.Spirit 调用，以获得该语法的完整规则和标准。 这个方法的返回值是`boost::spirit::rule` 的一个常量引用，它也是一个以扫描器类型实例化的模板类。

`boost::spirit::rule` 类用于定义规则。 非终结符号就以这个类来定义。 前面所定义的非终结符号 `object`, `member`, `string`, `value`,`number` 和 `array` 的类型均为 `boost::spirit::rule`。

所有这些对象都被定义为 `definition` 类的属性，这并非强制性的，但简化了定义，尤其是当各个规则之间有递归引用时。 正如在上一节中所看到的 EBNF 例子那样，递归引用并不是一个问题。

乍一看，在 `definition` 的构造函数内的规则定义非常类似于在上一节中看到的 EBNF 生成规则。 这并不奇怪，因为这正是 Boost.Spirit 的目标：重用在 EBNF 中定义的生成规则。

由于是用 C++ 代码来组成 EBNF 中建立的规则，为了写出合法的 C++，其实是有一点点差异的。 例如，所有符号间的连接是通过 `>>` 操作符完成的。 EBNF 中的一些操作符，如星号，被置于相应符号的前面而非后面。 尽管有这样一些语法上的修改，Boost.Spirit 还是尽量在将 EBNF 规则转换至 C++ 代码时不进行太多的修改。

`definition` 的构造函数使用了由 Boost.Spirit 提供的两个类：`boost::spirit::ch_p` 和 `boost::spirit::real_p`。 这些以分析器形式提供的常用规则可以很方便地重用。 一个例子就是 `boost::spirit::real_p`，它可以用于保存正或负的整数或浮点数，无需定义象 `digit`



