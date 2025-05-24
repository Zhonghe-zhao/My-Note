
给定 CPU 可以理解的所有可能的机器语言指令的集合称为**指令集** 。

每条指令被 CPU 理解为执行非常具体工作的命令，

例如： *“比较这两个数字”或“将此数字复制到该内存位置”。在计算机刚发明时，程序员必须直接用机器语*
*言编写程序，这是一件非常困难且耗时的事情。*


每个 CPU 系列都有自己的机器语言一样，每个 CPU 系列也有自己的汇编语言（旨在为同一 CPU 系列组装成机器语言）。这意味着有许多不同的汇编语言。尽管在概念上相似，但不同的汇编语言支持不同的指令，使用不同的命名约定，等等......


![[c++程序开发流程.webp]]

## 一 . 前奏


### 第 1 步：定义您要解决的问题

这是 “what” 步骤，您可以在其中弄清楚您打算解决的问题。想出你想要编程的初步想法可能是最简单的步骤，也可能是最困难的一步。但从概念上讲，这是最简单的。您所需要的只是一个可以明确定义的想法，然后您就可以为下一步做好准备。


**示例：**

	“我想编写一个程序，让我输入许多数字，然后计算平均值。”
	“我想编写一个程序来生成一个 2D 迷宫并让用户在其中导航。如果用户到达终点，他们就会获胜。
	“我想编写一个程序，该程序读取股票价格文件并预测股票是上涨还是下跌。”



### 第 2 步：确定您将如何解决问题

这是 “如何” 步骤，您可以在其中确定如何解决在步骤 1 中提出的问题。这也是软件开发中最容易被忽视的步骤。问题的症结在于解决问题的方法有很多种 —— 但是，其中一些解决方案是好的，而另一些是坏的。很多时候，程序员会得到一个想法，坐下来，然后立即开始编写解决方案。这通常会产生一个属于 bad 类别的解决方案。


**好的解决方法的特性：**

	 它们很简单（不会过于复杂或令人困惑）。
	 它们有据可查（尤其是围绕所做的任何假设或限制）。
	 它们是模块化构建的，因此部分可以重复使用或稍后更改，而不会影响程序的其他部分。
	 它们可以正常恢复或在发生意外情况时提供有用的错误消息。


### 第三步：编写程序



### 第四步： 编译源代码

**为了编译 C++ 源代码文件，我们使用 C++ 编译器。C++ 编译器按顺序遍历程序中的每个源代码 （.cpp） 文件，并执行两项重要任务**

1. 首先，编译器检查您的 C++ 代码，以确保它遵循 C++ 语言的规则。如果没有，编译器将为您提供错误（和相应的行号）以帮助确定需要修复的内容。编译过程也将中止，直到错误得到修复。

2. 其次，编译器将您的 C++ 代码转换为机器语言指令。这些指令存储在称为**对象文件的**中间文件中。对象文件还包含后续步骤中需要或有用的其他数据（包括步骤 5 中链接器所需的数据，以及步骤 7 中用于调试的数据）。

如果您的程序有 3 个 .cpp 文件，则编译器将生成 3 个目标文件


### 第 5 步：链接对象文件和库并创建所需的输出文件

编译器成功完成后，另一个称为**链接器**的程序将启动。链接器的工作是合并所有目标文件并生成所需的输出文件 此过程称为**链接** 如果链接过程中的任何步骤失败，链接器将生成一条描述问题的错误消息，然后中止。


1. 首先，链接器读取编译器生成的每个目标文件，并确保它们有效。

2. 其次，链接器确保所有跨文件依赖项都得到正确解析。例如，如果您在一个 .cpp 文件中定义某些内容，然后在不同的 .cpp 文件中使用它，则链接器会将两者连接在一起。如果链接器无法将引用连接到具有其定义的内容，则会收到链接器错误，并且链接过程将中止。

3. 第三，链接器通常在一个或多个**库文件中**链接，这些文件是预编译代码的集合，这些文件已被“打包”以供其他程序重用。

4. 最后，链接器输出所需的输出文件。通常，这将是一个可以启动的可执行文件


示例：

你调用了 `fmt.Println()`，这个函数并不在你的代码里，而是来自 Go 的标准库 `fmt`，Go 编译器会把你用到的 `fmt` 里的代码从**库文件**中找出来，并**链接**进最终的可执行程序里


![[LinkingObjects-min.webp]]


### 步骤6 & 7：测试和调试

一旦你可以运行你的程序，你就可以测试它。 **测试**是评估您的软件是否按预期工作的过程。基本测试通常涉及尝试不同的输入组合，以确保软件在不同情况下正常运行。


如果程序没有按预期运行，那么您将不得不进行一些**调试** ，这是查找和修复编程错误的过程。



## 二. 配置编译器

![[Pasted image 20250524150729.png]]

显示出必要的警告，有些警告非常有用，并且在必要时需要解决

## 三. 了解Hello World！

```C
#include <iostream>

int main()
{
	std::cout << "Hello world!";
	return 0;
}
```


第 1 行是一种特殊类型的行，称为预处理器指令。此 `#include` 预处理器指令表示我们希望使用 `iostream` 库的内容，该库是 C++ 标准库的一部分，允许我们从控制台读取和写入文本。我们需要这一行，以便在第 5 行使用 `std：：cout`。排除此行将导致第 5 行出现编译错误，否则编译器将不知道 `std：：cout` 是什么。

第 2 行为空，编译器将忽略该行。此行的存在只是为了帮助使程序对人类更具可读性（通过将 `#include` preprocessor 指令和程序的后续部分分开）。

第 3 行告诉编译器，我们将编写（定义）一个名称（标识符）为 `main` 的函数。正如您在上面所学到的，每个 C++ 程序都必须有一个 `main` 函数，否则将无法链接。此函数将生成一个类型为 `int` （整数） 的值

第 4 行和第 7 行告诉编译器哪些行是 _main_ 函数的一部分。第 4 行的左大括号和第 7 行的右大括号之间的所有内容都被视为 `main` 函数的一部分。这称为函数体。


*问题： 程序运行时会发生什么？*

`main（）` 中的语句按顺序执行。



## 四. 注释

在 C++ 中，有两种不同样式的注释，它们都有相同的目的：帮助程序员以某种方式记录代码。

1. 单行注释开头
该注释指示编译器忽略从该符号到行尾的所有内容，单行注释用于对单行代码进行快速注释。


```c
std::cout << "Hello world!\n";                 // std::cout lives in the iostream library
std::cout << "It is very nice to meet you!\n"; // this is much easier to read
std::cout << "Yeah!\n";                        // don't you think so?
```

2. 多行注释

`/*` 和 `*/` 对符号表示 C 样式的多行注释。符号之间的所有内容都将被忽


**正确使用注释**


通常，注释应该用于三件事。首先，对于给定的库、程序或函数，注释最适合_用于描述库_ 、程序或函数的作用。这些通常位于文件或库的顶部，或紧靠在函数之前。例如

*所有这些注释都使读者可以很好地了解库、程序或函数试图完成什么，而不必查看实际代码*。用户（可能是其他人，或者如果您尝试重用以前编写的代码，则可能是您）可以*一目了然地判断代码是否与他或她要完成的任务相关*。这在作为*团队的一部分*工作时尤其重要，因为不*是每个人都熟悉所有代码*。


在语句级别，应该使用注释来描述代码执行某项作_的原因_ 。错误的 statement _注释解释了代码_正在做什么。如果您编写的代码非常复杂，以至于需要一个注释_来解释语句_的作用，则可能需要重写您的语句，而不是注释它。



***“好代码说明‘做什么’，好注释说明‘为什么’。”***


## 五. 值

对值最常见的作之一是将它们*打印到屏幕*上

计算机中的主内存称为**随机存取存储器** （通常简称 **RAM**）。当我们运行一个程序时，作系统会将程序加载到 RAM 中。此时*将加载硬编码到程序本身中的任何数据*（例如，诸如 “Hello， world！” 之类的文本）。


系统还保留了一些额外的 RAM 供程序在运行时使用。此内存的常见用途是存储用户输入的值，存储从文件或网络读取的数据，或存储程序运行时计算的值（例如两个值的总和），以便以后可以再次使用。

***文本是只读值，因此无法修改其值。因此，如果我们想在内存中存储数据，我们需要其他方法来做到这一点***


```c++
char *s = "hello";   // 指向字符串常量，不能修改
s[0] = 'H';          // ❌ 未定义行为，可能崩溃
```

```c++
char s[] = "hello";  // 拷贝到数组，位于可写内存中
s[0] = 'H';          // ✅ 可以修改
```


Go中

```go
b := []byte("hello")
b[0] = 'H'           // ✅ 可以修改
s2 := string(b)
fmt.Println(s2)      // 输出 "Hello"

```

## 六. 对象和变量

在 C++ 中，不建议直接访问内存。相反，我们通过对象间接访问内存。 **对象**表示可以保存值的存储区域（通常是 RAM 或 CPU 寄存器）。对象还具有关联的属性


```c
int x; // define a variable named x (of type int)
```

编译器为我们处理有关此变量的所有其他细节，包括确定对象需要多少内存、对象将放置在哪种存储中（例如在 RAM 或 CPU 寄存器中）、它相对于其他对象的放置位置、何时创建和销毁它等

执行从 `main（）` 的顶部开始。为 `x` 分配内存。然后程序结束。

什么是对象？

`An object is a region of storage (usually memory) that can store a value.`

什么是变量

`A variable is an object that has a name.`

什么是数据

`A value is a letter (e.g. `a`), number (e.g. `5`), text (e.g. `Hello`), or instance of some other useful concept that can be represented as data.`


## 七. 变量赋值和初始化

1. 分配变量

```c
int main()
{
    int x;    // define an integer variable named x (preferred)
    int y, z; // define two integer variables, named y and z

    return 0;
}
```

2. 赋值

```c++
int width; // define an integer variable named width
width = 5; // assignment of value 5 into variable width

// variable width now has value 5
```

这被称为**复制分配**


一旦为变量指定了值，就可以通过 `std：：cout` 和 `<<` 运算符打印该变量的值

***C++ 中有 5 种常见的初始化形式：***

```c++
int a;         // default-initialization (no initializer)

// Traditional initialization forms:
int b = 5;     // copy-initialization (initial value after equals sign)
int c ( 6 );   // direct-initialization (initial value in parenthesis)

// Modern initialization forms (preferred):
int d { 7 };   // direct-list-initialization (initial value in braces)
int e {};      // value-initialization (empty braces)
```

|写法|名称|值|特点|
|---|---|---|---|
|`int a;`|默认初始化|未定义|不安全，局部变量是垃圾值|
|`int b = 5;`|复制初始化|5|支持隐式转换|
|`int c(6);`|直接初始化|6|类似函数调用|
|`int d{7};`|直接列表初始化 ✅推荐|7|安全，防止隐式类型转换|
|`int e{};`|值初始化 ✅推荐|0|初始化为零|

```c++
struct MyClass {
    MyClass(int x) { ... }          // 构造函数
    MyClass(const MyClass& other) { ... } // 拷贝构造函数
};

```


```c++
MyClass a = 5;   // copy-initialization（可能调用拷贝构造函数）
MyClass b(5);    // direct-initialization（直接调用构造函数）
```

旧的编译器可能会让 `a = 5` 先调用构造函数再调用拷贝构造函数，多一步；  
但 **现代编译器（尤其 C++17 后）会优化掉这一步**，效果基本一样。

|写法|是否推荐|备注|
|---|---|---|
|`MyClass a = x;`|✅ C++17 后可以放心用|语法清晰|
|`MyClass a(x);`|✅ 更偏向现代风格|少见歧义|
|`MyClass a{x};`|✅ 最安全，禁止隐式转换|推荐用于新项目|

**问题 : When should I initialize with { 0 } vs {}?**


```c++
struct Foo {
    Foo() : v(42) {}
    int v;
};

Foo f1{};   // ✅ 调用默认构造，v = 42
Foo f2{0};  // ❌ 编译错误：找不到接受 int 的构造函数

```

我们还注意到，最佳做法是完全避免这种语法。但是，由于您可能会遇到使用此样式的其他代码，因此多讨论一下它仍然很有用，如果不是其他原因，只是为了强调您应该避免使用它的一些原因。

```c++
int a = 5, b = 6;          // copy-initialization
int c ( 7 ), d ( 8 );      // direct-initialization
int e { 9 }, f { 10 };     // direct-list-initialization
int i {}, j {};            // value-initialization
```

```c++
int a, b = 5;     // wrong: a is not initialized to 5!
int a = 5, b = 5; // correct: a and b are initialized to 5
```



```c++
#include <iostream>

int main()
{
    [[maybe_unused]] double pi { 3.14159 };  // Don't complain if pi is unused
    [[maybe_unused]] double gravity { 9.8 }; // Don't complain if gravity is unused
    [[maybe_unused]] double phi { 1.61803 }; // Don't complain if phi is unused

    std::cout << pi << '\n';
    std::cout << phi << '\n';

    // The compiler will no longer warn about gravity not being used

    return 0;
}
```

 [[maybe_unused]]  可以接受编译而没有被使用的变量


~~~
Prefer `\n` over `std::endl` when outputting text to the console.
~~~


*缓冲区*

`std::cin` 使用的是**输入缓冲区**

- 当你输入内容后，**按下回车**，整个一行文本会被放入缓冲区中；
    
- `std::cin` 会从这个缓冲区中逐个提取数据。



*使用未初始化变量*

```c++
#include <iostream>

void doNothing(int&) // Don't worry about what & is for now, we're just using it to trick the compiler into thinking variable x is used
{
}

int main()
{
    // define an integer variable named x
    int x; // this variable is uninitialized

    doNothing(x); // make the compiler think we're assigning a value to this variable

    // print the value of x to the screen (who knows what we'll get, because x is uninitialized)
    std::cout << x << '\n';

    return 0;
}
```


## 八：标识符命名最佳实践

```c++
int value; // conventional

int Value; // unconventional (should start with lower case letter)
int VALUE; // unconventional (should start with lower case letter and be in all lower case)
int VaLuE; // unconventional (see your psychiatrist) ;)
```


```c++
int my_variable_name;   // conventional (separated by underscores/snake_case)
int my_function_name(); // conventional (separated by underscores/snake_case)

int myVariableName;     // conventional (intercapped/camelCase)
int myFunctionName();   // conventional (intercapped/camelCase)

int my variable name;   // invalid (whitespace not allowed)
int my function name(); // invalid (whitespace not allowed)

int MyVariableName;     // unconventional (should start with lower case letter)
int MyFunctionName();   // unconventional (should start with lower case letter)
```


避免使用缩写，除非它们是常见且明确的


## 九： 文字和运算符

```c++
#include <iostream>

int main()
{
    std::cout << 5 << '\n'; // print the value of a literal

    int x{ 5 };
    std::cout << x << '\n'; // print the value of a variable
    return 0;
}
```

*直接打印 值 和 变量的区别*


因此，两个 output 语句执行相同的作（打印值 5）。但是对于 Literals，可以直接打印值 `5`。对于变量，必须从*变量表示的内存中获取值*

这也解释了为什么 Literals 是 constant，而 variable 可以更改。文本的值直接放置在可执行文件中，可执行文件本身在创建后无法更改。变量的值被放置在内存中，*并且可以在可执行文件运行时更改 memory 的值。*

文本值无法更改！！！！

变量值可以在后续被其它执行代码更改！！！


## *第一个项目

返回输入数字的二倍

```c++
#include <iostream>

// worst version
int main()
{
	std::cout << "Enter an integer: ";

	int num{ };
	std::cin >> num;

	num = num * 2; // double num's value, then assign that value back to num

	std::cout << "Double that number is: " << num << '\n';

	return 0;
}
```

为什么这是不好的！

1. 丢失了原始输入，无法重用

- 比如你想再做一步 `num * 3`，但这时候 `num` 早就不是用户输入了，是翻倍后的；
    
- **原始输入已经被覆盖，无法再用**。

2.变量含义“变了”，容易让人困惑

- 原来 `num` 表示“用户输入的值”；
    
- 后来 `num` 被改成了 “输入值 × 2”；
    
- **同一个变量**，含义却在中途改变 → 看代码的人容易糊涂。


## *编程思想

编程的首要目标是使您的程序正常工作。一个不工作的程序，无论它写得有多好，都是没有用的。

但是，我喜欢一句话：“你必须写一次程序，才能知道你第一次应该怎么写它。这说明了这样一个事实，即最佳解决方案通常并不明显，而且我们对问题的第一个解决方案通常没有应有的那么好。

当我们专注于弄清楚如何使我们的程序工作时，将大量时间投入到我们甚至不知道我们是否会保留的代码上没有多大意义。所以我们走捷径。我们跳过了错误处理和注释等内容。我们在整个解决方案中散布调试代码，以帮助我们诊断问题和查找错误。我们边走边学 -- 我们认为可能有效的事情终究是行不通的，我们不得不回头尝试另一种方法。

最终结果是，我们的初始解决方案通常结构不合理、健壮（防错）、可读性或简洁性。因此，一旦您的程序开始工作，您的工作就真的没有完成（除非该程序是一次性的/一次性的）。下一步是清理代码。这包括以下内容：删除 （或注释掉） 临时/调试代码、添加注释、处理错误情况、格式化代码以及确保遵循最佳实践。即使这样，您的程序也可能没有想象中那么简单 —— 也许有可以合并的冗余逻辑，或者可以组合的多个语句，或者不需要的变量，或者可以简化的一千个其他小事情。很多时候，新程序员专注于优化性能，而他们应该优化可维护性。

这些教程中介绍的解决方案很少在第一次就表现出色。相反，它们是不断完善的结果，直到找不到其他可以改进的地方。在许多情况下，读者仍然可以找到许多其他改进建议！

所有这一切实际上是在说：如果/当您的解决方案没有从您的大脑中得到出色的优化时，请不要感到沮丧。这很正常。完美的编程是一个迭代过程（需要重复的过程）。

***写代码是先求能跑，再求好；想写出好代码，必须接受写烂再改的过程。***



#### 问题： 初始化和赋值的区别

Initialization 为变量提供初始值（在创建时）。赋值 在定义变量后为变量提供新值

由于变量只创建一次，因此只能初始化一次。可以根据需要多次为变量分配值。

#### 问题：未定义的行为？什么后果？

当程序员执行 C++ 语言未指定的内容时，将发生未定义的行为。后果几乎是任何东西，从崩溃到产生错误的答案，再到无论如何都能正常工作。


## 一. 函数

您已经知道每个可执行程序都必须有一个名为 `main（）` 的函数（这是程序运行时开始执行的位置）。然而，随着程序开始变得越来越长，将所有代码放在 `main（）` 函数中变得越来越难以管理。函数为我们提供了一种将程序拆分为小的、模块化的块的方法，这些块更易于组织、测试和使用。大多数程序使用许多功能。C++ 标准库附带了大量已编写的函数供您使用 - 但是，编写自己的函数也同样常见。您自己编写的函数称为**用户定义的函数** 。


C++ 程序可以以相同的方式工作（并借用一些相同的命名法）。当程序遇到函数调用时，它将在一个函数中按顺序执行语句。 **函数调用**告诉 CPU 中断当前函数并执行另一个函数。CPU 实质上是在当前执行点 “放置书签”，执行函数调用中指定的函数，然后**返回到**它添加书签的点并继续执行。

发起函数调用的函数是**调用方** ，被**调用** （执行）的函数是**被调用方** 。函数调用有时也称为**调用** ，调用方**调用**被调用方。


```c++
#include <iostream> // for std::cout

void doB()
{
    std::cout << "In doB()\n";
}


void doA()
{
    std::cout << "Starting doA()\n";

    doB();

    std::cout << "Ending doA()\n";
}

// Definition of function main()
int main()
{
    std::cout << "Starting main()\n";

    doA();

    std::cout << "Ending main()\n";

    return 0;
}
```


一层层封装 进入弹出

## 二. 局部性

```c++
int add(int x, int y) // x and y created and initialized here
{
    int z{ x + y };   // z created and initialized here

    return z;
}
```

就像一个人的一生被定义为他们出生和死亡之间的时间一样，一个物体的**一生**被定义为其被创造和毁灭之间的时间。请注意，*变量创建和销毁发生在程序运行时（称为运行时），而不是在编译时。因此，lifetime 是一个运行时属性。*



```c++
#include <iostream>

void doSomething()
{
    std::cout << "Hello!\n";
}

int main()
{
    int x{ 0 };    // x's lifetime begins here

    doSomething(); // x is still alive during this function call

    return 0;
} // x's lifetime ends here
```


在上面的程序中，`x` 的生命周期从定义点到函数 `main` 的末尾。这包括执行函数 `doSomething` 期间所花费的时间。

**销毁的对象将变为无效。**


在对象被销毁后，对对象的任何使用都将导致未定义的行为。  **在销毁后的某个时间点，对象使用的内存将被释放 （释放以供重用）。**


局部变量的生命周期在它超出范围时结束，因此局部变量在此时被销毁。 *请注意，并非所有类型的变量在超出范围时都会被销毁*


***"One Task"***


## 三. 命名冲突


1. a.cpp

```c++
#include <iostream>

void myFcn(int x)
{
    std::cout << x;
}
```

2. main.cpp

```c++
#include <iostream>

void myFcn(int x)
{
    std::cout << 2 * x;
}

int main()
{
    return 0;
}
```


每个文件按编译的时候 不会出现问题但是 当链接器执行时，它会将 _a.cpp_ 和 _main.cpp_ 中的所有定义链接在一起，并发现函数 `myFcn（）` 的冲突定义。然后，链接器将中止并显示错误。请注意，即使从未调用 `myFcn（），` 也会发生此错误！

在给定的范围区域内，所有标识符都必须是唯一的，否则将导致命名冲突。

## 四. 命名空间

**命名空间**提供了另一种类型的范围区域（称为**命名空间范围** ），它允许您在其中声明或定义名称以消除歧义。在命名空间中声明的名称与其他范围内声明的名称隔离，从而允许此类名称存在而不会发生冲突。

在 C++ 中，未在类、函数或命名空间中定义的任何名称都被视为隐式定义的命名空间的一部分，该命名空间称为**全局命名空间** （有时也称为**全局范围** ）。

例如，可以在不同的命名空间内定义两个具有相同声明的函数，并且不会发生命名冲突或歧义。


命名空间只能包含声明和定义。可执行语句仅允许作为定义的一部分（例如函数的一部分）。


命名空间通常用于对大型项目中的相关标识符进行分组，以帮助确保它们不会无意中与其他标识符发生冲突。*例如，如果你把所有的数学函数都放在一个名为 `math` 的命名空间中，那么你的数学函数不会与 `math` 命名空间外的同名函数发生冲突。*


1. **全局命名空间**

函数 `main（）` 和 `myFcn（）` 的两个版本都是在全局命名空间内定义的。示例中遇到的命名冲突是因为 `myFcn（）` 的两个版本最终都位于全局命名空间内*，这违反了范围区域中的所有名称都必须唯一的规则*

尽管可以在全局命名空间中定义变量，但通常应该避免这种情况

事实证明，`std：：cout` 的名字并不是真正的 `std：：cout`。它实际上只是 `cout`， `而 std` 是标识符 `cout` 所属的命名空间的名称。因为 `cout` 是在 `std` 命名空间中定义的，所以名称 `cout` 不会与我们在 `std` 命名空间之外创建的任何名为 `cout` 的对象或函数冲突（例如在全局命名空间中）。


**当您使用在非全局命名空间（例如 `std` 命名空间）中定义的标识符时，您需要告诉编译器该标识符位于命名空间内。**


2. 显式命名空间限定符 std：：

告诉编译器我们想使用 `std` 命名空间中的 `cout` 的最直接方法是显式使用 `std：：` 前缀

```c++
#include <iostream>

int main()
{
    std::cout << "Hello world!"; // when we say cout, we mean the cout defined in the std namespace
    return 0;
}
```

访问命名空间内标识符的另一种方法是使用 using 指令语句。这是我们的原始 “Hello world” 程序，带有 using 指令：

```c++
#include <iostream>

using namespace std; // this is a using-directive that allows us to access names in the std namespace with no namespace prefix

int main()
{
    cout << "Hello world!";
    return 0;
}
```

**using 指令**允许我们在不使用命名空间前缀的情况下访问命名空间中的名称。所以在上面的例子中，当编译器去确定标识符 `cout` 是什么时，它将与 `std：：cout` 匹配，由于 using 指令，它只能作为 `cout` 访问。


### Why?

```c++
#include <iostream> // imports the declaration of std::cout into the global scope

using namespace std; // makes std::cout accessible as "cout"

int cout() // defines our own "cout" function in the global namespace
{
    return 5;
}

int main()
{
    cout << "Hello, world!"; // Compile error!  Which cout do we want here?  The one in the std namespace or the one we defined above?

    return 0;
}
```


上面的程序没有编译，因为编译器现在无法判断我们想要的是定义的 `cout` 函数，还是 `std：：cout`


以这种方式使用 using 指令时，我们定义_的任何_标识符_都可能与_ `std` 命名空间中的任何同名标识符冲突

`using namespace std;`

我接下来代码中要用 std 命名空间里的所有东西，不用再加 std:: 前缀了”

等价`std::cout` → `cout`，`std::cin` → `cin`。


```c++
#include <iostream> // imports the declaration of std::cout into the global scope

using namespace std; // makes std::cout accessible as "cout"



class  mm
{
public:
	 mm();
	~ mm();
    int cout();

private:
};

int mm::cout() {
    return 5;
}

 mm:: mm()
{
}

 mm::~ mm()
{
}

int main()
{
    class mm  a;
    a.cout(); // Compile error!  Which cout do we want here?  The one in the std namespace or the one we defined above?
    cout << "11111111";
    return 0;
}
```

大体是这个意思


### 缩进

```c
#include <iostream>   // 在全局作用域中，不需要缩进

void foo()           // 也是在全局作用域中定义，不缩进
{
    std::cout << "Inside foo\n";  // 在函数作用域（foo 的内部），所以缩进一级
}

int main()           // 也是在全局作用域中定义，不缩进
{
    std::cout << "Inside main\n"; // 也是在函数作用域内部，缩进一级
    foo();                        // 继续缩进一级
    return 0;
}

```



- `#include` 和函数定义 `void foo()`、`int main()` —— 这些是**最外层的代码（global scope）**，不缩进。
    
- 函数内部的代码是函数的“嵌套作用域”（nested scope）—— 所以要**缩进一级**，表示它们**属于这个函数内部**。
    
- 缩进不仅提高可读性，还帮助你视觉上快速看出哪段代码属于哪个作用域。


## 五. 预处理器

在编译之前，每个代码 （.cpp） 文件都会经历**一个预处理**阶段。在此阶段中，称为 **preprocessor** 的程序对代码文件的文本进行各种更改。预处理器实际上不会以任何方式修改原始代码文件 —— 相反，

**预处理器所做的所有更改要么临时发生在内存中，要么使用临时文件。**


从历史上看，预处理器是独立于编译器的程序，但在现代编译器中，预处理器可能直接内置到编译器本身中

当预处理器处理完代码文件时，结果称为**翻译单元** 。此翻译单元是编译器随后编译的内容。

