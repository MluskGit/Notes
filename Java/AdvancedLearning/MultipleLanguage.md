`目录 start`
 
- [JVM上的多语言使用](#jvm上的多语言使用)
    - [语言生态学](#语言生态学)
            - [重新实现的语言和原生语言](#重新实现的语言和原生语言)
    - [JVM上的多语言编程](#jvm上的多语言编程)
        - [编译器小说](#编译器小说)

`目录 end` *目录创建于2018-01-14*
****************************************
# JVM上的多语言使用
> 先把Java熟练先

## 语言生态学
- 大致讨论 解释型和编译型， 动态和静态， 命令式和函数式
- Java是运行时编译，静态类型的命令式语言。强调安全性，代码清晰，性能，并表现出一定程度的繁琐和死板（例如部署）

`解释型和编译型`
- 在80 90 年代，边界较为清晰，类C语言是编译型，Perl和Python是解释型。但Java是两者都有
- 基于JVM来划分的边界是：该语言是否将源码编译为类文件并且执行，不产生类文件的语言会由解释器逐行执行。有些语言既有编译器又有解释器，有些是既有解释器又有产生字节码的即时编译器JIT

`动态和静态类型`
- 动态类型语言，变量在不同的时间可能会有不同的类型 动态类型语言是跟踪变量的值的类型信息，静态类型语言是跟踪变量的类型信息
- 静态类型适合做编译型语言

`命令式和函数式`
- Java是典型的命令式语言，命令式语言把程序的运行状态建模为可修改的数据，用一系列的指令来改变状态。因此在命令式语言中，程序状态是核心概念
- 命令式语言主要分为两类，一种是面向过程语言，一种是面向对象语言
    - 面向过程：Basic Fortran 这种语言将代码和数据完全分离开，有简单的代码操作数据范式
    - 面向对象：数据和代码（方法形式）封装在对象中，面向对象语言中或多或少会存在元数据（比如：类信息）引入的额外结构
- 函数式语言他把计算本身当成最重要的概念。函数式语言和过程式语言一样对值进行操作，但他不会修改输入，而是像数学函数一样返回新值
    - 函数被看成是一个小处理机，输入值并输出值，他们没有自己的状态，并且将他们和任何外部状态绑定在一起也没有意义
- `Groovy带一点函数式风格，Scala对FP的利用更为充分，Clojure是纯粹的函数式语言，没有丁点儿面向对象特性`

#### 重新实现的语言和原生语言
> 一般来说，以JVM为目标的语言较重新实现的语言能将自己的类型系统和JVM的类型系统结合的更紧密

- 重新实现已有语言的JVM语言：
    - JRuby：Ruby是一个动态类型的面向对象语言，有些函数式特性，在JVM上基本算解释型的
    - Jython：动态的面向对象语言。运行方式是先生成Python字节码再转化成JVM字节码。这使得他能以看起来像是Python的典型解释型模式下运行
    - Rhino：他在JVM上提供了一个JavaScript实现，既支持编译模式，也支持解释模式
    
## JVM上的多语言编程
- 非Java技术的作用可以分为三个层次 特定领域层，动态层，稳定层，多语言编程金字塔：
![p178金字塔](https://raw.githubusercontent.com/Kuangcp/ImageRepos/master/Tech/Book/Java7Developer/p178.jpg)

- 静态类型语言更倾向于稳定层的任务，能力不是那么强，通用性较低的技术在金字塔的顶部更好用

`为什么非要用Java语言`
- Java 作为一种通用，静态类型的编译型语言，实现稳定层方便，但是放到金字塔上层就成为负担
    - 编译耗时
    - 静态类型不够灵活，重构时间长
    - 部署麻烦
    - 语法不适合生产DSL（领域专用语言 domain specific language）

`最有影响力的JVM语言`
- Groovy
    - James Strachan 于2003年发明，可以看作动态层语言，擅长DSL构建
- Scala
    - Martin Odersky 于2003年意外产生，一门支持函数式编程的面向对象语言
    - 有一个非常好的ScalaTest测试框架，比Junit更简洁，
- clojure
    - Rich Hickey设计的属于Lisp家族的语言，动态类型的函数式语言，编译型语言但是通常以源码发布
    
![p180 分类](https://raw.githubusercontent.com/Kuangcp/ImageRepos/master/Tech/Book/Java7Developer/p180.jpg)


`JVM对备选语言的支持`
- 一种语言要在JVM上运行的两种方式：
    - 一个产生类文件的编译器
    - 一个用JVM字节码实现的解释器
![p183.jpg](https://raw.githubusercontent.com/Kuangcp/ImageRepos/master/Tech/Book/Java7Developer/p183.jpg)
- 有一种评估语言运行时环境复杂度的简单方法，看运行实现中Jar的大小，Clojure相对较轻量，JRuby就显得重

### 编译器小说
> 语言的某些特性是由编程环境和高层语言合成的，在底层JVM中不存在，这种特性就称为编译器小说

- Java中的编译器小说还包括检查型异常和内部类（通常内部类都会转换成带有特殊合成访问方法的顶层类），如果jar -cvf看jar包，能看到很多含`$`的类，这些就是被取出转换成`常规类`的内部类
`备选语言的编译器小说`
![p184.jpg](https://raw.githubusercontent.com/Kuangcp/ImageRepos/master/Tech/Book/Java7Developer/p184.jpg)
- 函数一等值：
    - 这个就是说可以将函数当成其他普通值一样操作，Java只能把类当做最小的代码和功能单元。解决这种差异的方法是，因为对象只是把数据和操作数据的方法绑定在一起，只要有一个没有状态只有一个方法的对象。
    - 这似乎就是Java8的lambda表达式的存在条件，单方法的实现用操作符 `->`
- 多继承：
    - 在Java和JVM中无法实现多继承，只能使用接口，但是接口又没有任何具体的方法
    - 在Scala中特性机制 trait 允许将方法的实现混合到类中，所以提供了不同的继承视图，这种行为必须由Scala编译器和运行时合成，在VM层面不提供这种特性
  
