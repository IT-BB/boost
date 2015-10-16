* * *

# 第 9 章 文件系统

* * *

### 目录

*   [9.1 概述](http://zh.highscore.de/cpp/boost/filesystem.html#filesystem_general)
*   [9.2 路径](http://zh.highscore.de/cpp/boost/filesystem.html#filesystem_paths)
*   [9.3 文件与目录](http://zh.highscore.de/cpp/boost/filesystem.html#filesystem_files_and_directories)
*   [9.4 文件流](http://zh.highscore.de/cpp/boost/filesystem.html#filesystem_fstream)
*   [9.5 练习](http://zh.highscore.de/cpp/boost/filesystem.html#filesystem_exercises)


* * *

## 9.1. 概述

库 [Boost.Filesystem](http://www.boost.org/libs/filesystem/) 简化了处理文件和目录的工作。 它提供了一个名为 `boost::filesystem::path` 的类，可以对路径进行处理。 另外，还有多个函数用于创建目录或验证某个给定文件的有效性。

* * *

## 9.2. 路径

`boost::filesystem::path` 是 Boost.Filesystem 中的核心类，它表示路径的信息，并提供了处理路径的方法。

实际上，`boost::filesystem::path` 是 `boost::filesystem::basic_path<std::string>` 的一个 `typedef`。 此外还有一个`boost::filesystem::wpath` 是 `boost::filesystem::basic_path<std::wstring>` 的 `typedef`。

所有定义均位于 boost::filesystem 名字空间，定义于 `boost/filesystem.hpp` 中。

可以通过传入一个字符串至 `boost::filesystem::path` 类来构建一个路径。

~~~
#include <boost/filesystem.hpp> 

int main() 
{ 
  boost::filesystem::path p1("C:\\"); 
  boost::filesystem::path p2("C:\\Windows"); 
  boost::filesystem::path p3("C:\\Program Files"); 
} 
~~~

*   [下载源代码](http://zh.highscore.de/cpp/boost/src/9.2.1/main.cpp)

没有一个 `boost::filesystem::path` 的构造函数会实际验证所提供路径的有效性，或检查给定的文件或目录是否存在。 因此，`boost::filesystem::path` 甚至可以用无意义的路径来初始化。

~~~
#include <boost/filesystem.hpp> 

int main() 
{ 
  boost::filesystem::path p1("..."); 
  boost::filesystem::path p2("\\"); 
  boost::filesystem::path p3("@:"); 
} 
~~~

*   [下载源代码](http://zh.highscore.de/cpp/boost/src/9.2.2/main.cpp)

以上程序可以执行的原因是，路径其实只是字符串而已。 `boost::filesystem::path` 只是处理字符串罢了；文件系统没有被访问到。

`boost::filesystem::path` 特别提供了一些方法来以字符串方式获取一个路径。 有趣的是，有三种不同的方法。

~~~
#include <boost/filesystem.hpp> 
#include <iostream> 

int main() 
{ 
  boost::filesystem::path p("C:\\Windows\\System"); 
  std::cout << p.string() << std::endl; 
  std::cout << p.file_string() << std::endl; 
  std::cout << p.directory_string() << std::endl; 
} 
~~~

*   [下载源代码](http://zh.highscore.de/cpp/boost/src/9.2.3/main.cpp)

`string()` 方法返回一个所谓的可移植路径。 换句话说，就是 Boost.Filesystem 用它自己预定义的规则来正规化给定的字符串。 在以上例子中，`string()` 返回 `C:/Windows/System`。 如你所见，Boost.Filesystem 内部使用斜杠符 `/` 作为文件名与目录名的分隔符。

可移植路径的目的是在不同的平台，如 Windows 或 Linux 之间，唯一地标识文件和目录。 因此就不再需要使用预处理器宏来根据底层的操作系统进行路径的编码。 构建可移植路径的规则大多符合POSIX标准，在 [Boost.Filesystem 参考手册](http://www.boost.org/libs/filesystem/doc/reference.html) 给出。

请注意，`boost::filesystem::path` 的构造函数同时支持可移植路径和平台相关路径。 在上面例子中所使用的路径 "C:\\Windows\\System" 就不是可移植路径，而是 Windows 专用的。 它可以被 Boost.Filesystem 正确识别，但仅当该程序是在 Windows 操作系统下运行的时候！ 当程序运行于一个 POSIX 兼容的操作系统，如 Linux 时，`string()` 将返回 `C:\Windows\System`。 因为在 Linux 中，反斜杠符 `\` 并不被用作分隔符，无论是可移植格式或原生格式，Boost.Filesystem 都不会认为它是文件和目录的分隔符。

很多时候，都不能避免使用平台相关路径作为字符串。 一个例子就是，使用操作系统函数时必须要用平台相关的编码。 方法 `file_string()` 和`directory_string()` 正是为此目的而提供的。

在上例中，这两个方法都会返回 `C:\Windows\System` - 与底层操作系统无关。 在 Windows 上这个字符串是有效路径，而在一个 Linux 系统上则既不是可移植路径也不是平台相关路径，会象前面所说那样被解析。

以下例子使用一个可移植路径来初始化 `boost::filesystem::path`。

~~~
#include <boost/filesystem.hpp> 
#include <iostream> 

int main() 
{ 
  boost::filesystem::path p("/"); 
  std::cout << p.string() << std::endl; 
  std::cout << p.file_string() << std::endl; 
  std::cout << p.directory_string() << std::endl; 
} 
~~~

*   [下载源代码](http://zh.highscore.de/cpp/boost/src/9.2.4/main.cpp)

由于 `string()` 返回的是一个可移植路径，所以它与用于初始化 `boost::filesystem::path` 的字符串相同：`/`。 但是 `file_string()` 和`directory_string()` 方法则会因底层平台而返回不同的结果。 在 Windows 中，它们都返回 `\`，而在 Linux 中则都返回 `/`。

你可能会奇怪为什么会有两个不同的方法用来返回平台相关路径。 到目前为止，在所看到的例子中，`file_string()` 和 `directory_string()` 都是返回相同的值。 但是，有些操作系统可能会返回不同的结果。 因为 Boost.Filesystem 的目标是支持尽可能多的操作系统，所以它提供了两个方法来适应这种情况。 即使你可能更为熟悉 Windows 或 POSIX 系统如 Linux，但还是建议使用 `file_string()` 来取出文件的路径信息，且使用 `directory_string()` 取出目录的路径信息。 这无疑会增加代码的可移植性。

`boost::filesystem::path` 提供了几个方法来访问一个路径中的特定组件。

~~~
#include <boost/filesystem.hpp> 
#include <iostream> 

int main() 
{ 
  boost::filesystem::path p("C:\\Windows\\System"); 
  std::cout << p.root_name() << std::endl; 
  std::cout << p.root_directory() << std::endl; 
  std::cout << p.root_path() << std::endl; 
  std::cout << p.relative_path() << std::endl; 
  std::cout << p.parent_path() << std::endl; 
  std::cout << p.filename() << std::endl; 
} 
~~~

*   [下载源代码](http://zh.highscore.de/cpp/boost/src/9.2.5/main.cpp)

如果在是一个 Windows 操作系统上执行，则字符串 "C:\\Windows\\System" 被解释为一个平台相关的路径信息。 因此，`root_name()`返回 `C:`, `root_directory()` 返回 `/`, `root_path()` 返回 `C:/`, `relative_path()` 返回 `Windows/System`, `parent_path()` 返回`C:/Windows`, 而 `filename()` 返回 `System`。

如你所见，没有平台相关的路径信息被返回。 没有一个返回值包含反斜杠 `\`，只有斜杠 `/`。 如果需要平台相关信息，则要使用 `file_string()`或 `directory_string()`。 为了使用这些路径中的单独组件，必须创建一个类型为 `boost::filesystem::path` 的新对象并相应的进行初始化。

如果以上程序在 Linux 操作系统中执行，则返回值有所不同。 多数方法会返回一个空字符串，除了 `relative_path()` 和 `filename()` 会返回`C:\Windows\System`。 字符串 "C:\\Windows\\System" 在 Linux 中被解释为一个文件名，这个字符串既不是某个路径的可移植编码，也不是一个被 Linux 支持的平台相关编码。 因此，Boost.Filesystem 没有其它选择，只能将整个字符串解释为一个文件名。

Boost.Filesystem 还提供了其它方法来检查一个路径中是否包含某个特定子串。 这些方法是：`has_root_name()`,`has_root_directory()`, `has_root_path()`, `has_relative_path()`, `has_parent_path()` 和 `has_filename()`。 各个方法都是返回一个`bool` 类型的值。

还有两个方法用于将一个文件名拆分为各个组件。 它们应当仅在 `has_filename()` 返回 `true` 时使用。 否则只会返回一个空字符串，因为如果没有文件名就没什么可拆分了。

~~~
#include <boost/filesystem.hpp> 
#include <iostream> 

int main() 
{ 
  boost::filesystem::path p("photo.jpg"); 
  std::cout << p.stem() << std::endl; 
  std::cout << p.extension() << std::endl; 
} 
~~~

*   [下载源代码](http://zh.highscore.de/cpp/boost/src/9.2.6/main.cpp)

这个程序分别返回 `photo` 给 `stem()`，以及 `.jpg` 给 `extension()`。

除了使用各个方法调用来访问路径的各个组件以外，你还可以对组件本身进行迭代。

~~~
#include <boost/filesystem.hpp> 
#include <iostream> 

int main() 
{ 
  boost::filesystem::path p("C:\\Windows\\System"); 
  for (boost::filesystem::path::iterator it = p.begin(); it != p.end(); ++it) 
    std::cout << *it << std::endl; 
} 
~~~

*   [下载源代码](http://zh.highscore.de/cpp/boost/src/9.2.7/main.cpp)

如果是在 Windows 上执行，则该程序将相继输出 `C:`, `/`, `Windows` 和 `System`。 在其它的操作系统如 Linux 上，输出结果则是`C:\Windows\System`。

前面的例子示范了不同的方法来访问路径中的各个组件，以下例子则示范了修改路径信息的方法。

~~~
#include <boost/filesystem.hpp> 
#include <iostream> 

int main() 
{ 
  boost::filesystem::path p("C:\\"); 
  p /= "Windows\\System"; 
  std::cout << p.string() << std::endl; 
} 
~~~

*   [下载源代码](http://zh.highscore.de/cpp/boost/src/9.2.8/main.cpp)

通过使用重载的 `operator/=()` 操作符，这个例子将一个路径添加到另一个之上。 在 Windows 中，该程序将输出`C:\Windows\System`。 在 Linux 中，输出将会是 `C:\/Windows\System`，因为斜杠符 `/` 是文件与目录的分隔符。 这也是重载`operator/=()` 操作符的原因：毕竟，斜杠是这个方法名的一个部分。

除了 `operator/=()`，Boost.Filesystem 只提供了 `remove_filename()` 和 `replace_extension()` 方法来修改路径信息。

* * *

## 9.3. 文件与目录

`boost::filesystem::path` 的各个方法内部其实只是对字符串进行处理。 它们可以用来访问一个路径的各个组件、相互添加路径等等。

为了处理硬盘上的物理文件和目录，提供了几个独立的函数。 这些函数需要一个或多个 `boost::filesystem::path` 类型的参数，并且在其内部会调用操作系统功能来处理这些文件或目录。

在介绍各个函数之前，很重要的一点是要弄明白出现错误时会发生什么。 所有要在内部访问操作系统功能的函数都有可能失败。 在失败的情况下，将抛出一个类型为 `boost::filesystem::filesystem_error` 的异常。 这个类是派生自 `boost::system::system_error` 的，因此适用于 Boost.System 框架。

除了继承自父类 `boost::system::system_error` 的 `what()` 和 `code()` 方法以外，还有另外两个方法：`path1()` 和 `path2()`。 它们均返回一个类型为 `boost::filesystem::path` 的对象，因此在发生错误时可以很容易地确定路径信息 - 即使是对那些需要两个`boost::filesystem::path` 参数的函数。

多数函数存在两个变体：在失败时，一个会抛出类型为 `boost::filesystem::filesystem_error` 的异常，而另一个则返回类型为`boost::system::error_code` 的对象。 对于后者，需要对返回值进行明确的检查以确定是否出错。

以下例子介绍了一个函数，它可以查询一个文件或目录的状态。

~~~
#include <boost/filesystem.hpp> 
#include <iostream> 

int main() 
{ 
  boost::filesystem::path p("C:\\"); 
  try 
  { 
    boost::filesystem::file_status s = boost::filesystem::status(p); 
    std::cout << boost::filesystem::is_directory(s) << std::endl; 
  } 
  catch (boost::filesystem::filesystem_error &e) 
  { 
    std::cerr << e.what() << std::endl; 
  } 
} 
~~~

*   [下载源代码](http://zh.highscore.de/cpp/boost/src/9.3.1/main.cpp)

`boost::filesystem::status()` 返回一个 `boost::filesystem::file_status` 类型的对象，该对象可以被传递给其它辅助函数来评估。 例如，如果查询的是一个目录的状态，则 `boost::filesystem::is_directory()` 将返回 `true`。 除了 `boost::filesystem::is_directory()`


