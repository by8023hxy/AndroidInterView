**第1题，什么是kotlin？**

kotlin是静态类型的编程语言，运行于jvm之上。

**第2题， 是谁开发了kotlin?**

kotlin是由jetbrains开发的。

**第3题， 为什么我们应该从Java转到kotlin?**

首先，kotlin比Java要简单。它去除了很多Java里面的冗余代码。kotlin提供了很多Java不具有的特性。

**第4题, 说一下使用kotlin的三大好处。**

kotlin比较容易学，因为它跟Java很接近。

kotlin是功能性编程语言，是基于jvm上的。

kotlin的代码更易读，更容易理解。

**第5题, 解释一下extension函数。**

extension函数用来对class的扩展，而不需要从class进行派生。

**第6题, kotlin中的null safety是什么意思?**

null safety的特性是为了去除null pointer exception在实时运行中的出现风险。它也用来区分空引用和非空引用。

**第7题, 为什么kotlin跟Java具有互相的操作性?**

因为这两门语言，对于jvm来说没有区别。它们都是编译成byte code, 然后在jvm上运行的。

**第8题，在kotlin中是否存在三元条件操作符?**

不存在， 在kotlin中没有三元条件操作符。

**第9题, 在kotlin中如何声明一个变量?**

1 定义只读局部变量使用关键字 val 定义。只能为其赋值一次,

2 定义全局变量使用关键字var定义,

**第10题，在kotlin中有多少构造函数？**

有两种，一种是primary构造函数，一种是secondary构造函数。

**第11题， kotlin支持哪种编程类型?**

一种是procedural编程, 另一种是面向对象的编程。

**第12题，说一下kotlin中对Java.io.file的的扩展方法。**

bufferedReader.

readBytes.

readText

forEachLine

readLines

**第13题, 在kotlin中如何处理null异常?**

使用elvis操作符来处理null异常。

**第14题，有哪些特点， kotlin有，但是Java没有?**

null safety.

Operator overloading.

Coroutines.

Range expressions.

Smart casts.

Companion objects.

**第15题, 解释一下kotlin中数据类的作用。**

数据类包含基本的数据类型, 它不包含任何功能函数。

**第16题， 我们能把Java代码转成kotlin代码吗?**

是的，我们可以用jetbrains ide把Java代码转成kotlin，也可以用Android studio转。

**第17题, kotlin允许macros吗?**

不允许。kotlin不支持宏。

**第18题，说一下kotlin类的缺省行为。**

kotlin类缺省是final的。因为kotlin支持多重类继承。开放类代价要比final类高很多。

**第19题， kotlin是否支持原始数据类型?**

不支持，kotlin不支持原始数据类型。

**第20题, 什么是range操作符?**

Range操作符用来遍历一个范围。用两个点来表示的。

for(i in 1..15)

print(i)

**第21题, kotlin对标准的Java库和类提供额外的功能吗?**

kotlin程序是跑在标准的Java虚拟机上的。所以kotlin跟Java在这一层级几乎没有区别。Java代码还可以直接在kotlin程序中使用。

**第22题, 在kotlin中定义一个volatile变量。**

Volatile var x:Long?=null

**第23题, kotlin中的抽象有什么作用?**

抽象是面向对象编程中最重要的概念。抽象类的特点是，你知道这个类会有什么功能，但是你不知道它具体如何实现这些功能和实现哪些功能。

**第24题，在kotlin中如何比较两个字符串?**

第1种方法你可以用双等号来比较两个字符串。

第2种方法用String.compareTo，这个扩展函数来比较两个字符串。

**第25题， 下面这段代码是干什么用的?**

bar{

System.out.println("haha")

}

bar作为一个函数，正在接收一个表达式为参数，这个表达式用来打印一行字符串。



**Kotlin和Java有什么区别？**

- Kotlin更安全；

如空引用由类型系统控制，你不会再遇到NullPointerException。

- 简洁，可靠，有趣

如你可以用Lambda 表达式；可以减少了很多模版代码。

- 函数式支持；
- 扩展函数；

Kotlin同C#类似，能够扩展一个类的新功能而无需继承该类或使用像装饰者这样的任何类型的设计模式。Kotlin支持扩展函数和扩展属性。

- Kotlin中没有静态成员；