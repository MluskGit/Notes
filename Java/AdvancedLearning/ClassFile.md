`目录 start`
 
- [Java基础](#java基础)
    - [【类和字节码】](#类和字节码)
        - [类加载和类对象](#类加载和类对象)
            - [加载和连接](#加载和连接)
            - [Class对象](#class对象)
            - [类加载器](#类加载器)
        - [方法句柄](#方法句柄)
        - [检查类文件](#检查类文件)
            - [javap](#javap)
        - [常量池](#常量池)
        - [字节码](#字节码)
            - [运行时环境](#运行时环境)
            - [操作码介绍](#操作码介绍)
            - [加载和存储操作码](#加载和存储操作码)
            - [数学运算操作码](#数学运算操作码)
            - [执行控制操作码](#执行控制操作码)
            - [调用操作码](#调用操作码)
            - [平台操作码](#平台操作码)
            - [操作码的快捷形式](#操作码的快捷形式)
            - [invokedynamic](#invokedynamic)
    - [序列化](#序列化)
        - [serialVersionUID](#serialversionuid)
        - [其他业内主流编解码框架](#其他业内主流编解码框架)
            - [MessagePack](#messagepack)
            - [Protobuf](#protobuf)
            - [Thrift](#thrift)
            - [Marshalling](#marshalling)
    - [反射的使用](#反射的使用)

`目录 end` |_2018-03-21_| [码云](https://gitee.com/kcp1104) | [CSDN](http://blog.csdn.net/kcp606) | [OSChina](https://my.oschina.net/kcp1104)
****************************************
# Java基础
## 【类和字节码】
> [个人相关代码](https://github.com/Kuangcp/JavaBase/tree/master/src/main/java/com/classfile) 

### 类加载和类对象
- 一个.class　文件定义了JVM中的类型，包括了域,方法，继承信息，注解和其他元数据

#### 加载和连接
![图](https://raw.githubusercontent.com/Kuangcp/ImageRepos/master/Tech/Book/Java7Developer/p107.jpg)

`加载`
- 这个过程就是读取字节码文件，创建一个字节数组装在这些内容，加载结束后这个对象还不能直接调用 

`连接`
- 加载完成后，类必须连接起来，分为三步：验证，准备，解析。
    - 验证：
        - 验证文件的合理性，完整性检查，检查常量池，方法的字节码检查。主要的：
        - 是否所有方法都遵守访问控制关键字的限定
        - 方法调用的参数个数和静态类型是否正确
        - 确保字节码不会试图滥用堆栈
        - 确保变量使用之前被正确初始化了
        - 检查变量是否仅被赋予恰当类型的值
    - 准备：
        - 分配内存，准备初始化类中的静态变量，但不会现在就初始化，也不会执行任何VM字节码
    - 解析：
        - 促使VM检查类文件中所引用的类型是不是都是已知的类型。如果有运行时有未知的类型，那又要引发一次类加载过程
        - 当需要加载的类全部加载解析完毕后，VM就可以初始化最初那个加载的类了。
        - 这时所有的静态变量都可以进行初始化，所有静态代码块都会运行，这一步完成后，类就能使用了

#### Class对象
- 加载和连接过程的最终结果是一个Class对象，Class对象可以和反射API一起实现对方法，域构造方法等类成员的间接访问

> 所以一个类的定义就会有一个Class对象, 但是这个对象的类型呢?怎么判断, Class对象的类型就是他的值么?

#### 类加载器
![图](https://raw.githubusercontent.com/Kuangcp/ImageRepos/master/Tech/Book/Java7Developer/p110.jpg)

- Java平台经典类加载器：
    - 根（引导）加载器： 通常在VM启动后不久就实例化，作用是加载系统的基础JAR(主要是rt.jar)，并且不做验证工作
    - 扩展类加载器： 加载安装时自带的标准扩展，一般包括安全性扩展
    - 应用或系统类加载器： 应用最广泛的类加载器，负责加载应用类，在大多SE环境中主要工作是由他完成
    - 定制类载器： 为了企业框架定制的加载器
    
*****
### 方法句柄
> 主要用于反射 用到再学

![图](https://raw.githubusercontent.com/Kuangcp/ImageRepos/master/Tech/Book/Java7Developer/p118.jpg)

******
### 检查类文件
#### javap
> 用来探视类文件内部和反汇编文件

****
### 常量池
> 常量池是为类文件中的其他常量元素提供快捷访问方式的区域。对于JVM来说常量池相当于符号表
> [参考博客](http://www.cnblogs.com/LeonNew/p/5314731.html)

- `javap -v class文件` 输出很多额外信息，# 开头的就是常量池信息
![图](https://raw.githubusercontent.com/Kuangcp/ImageRepos/master/Tech/Book/Java7Developer/p120.jpg)

*****
### 字节码

- 字节码是程序的中间表达形式，源码和机器码之间的产物
- 字节码是由源文件执行javac产生的
- 某些高级语言特性（语法糖）在字节码中给去掉了，例如循环结构，会转换成为分支指令
- 每个操作都由一个字节表示，因此叫做字节码
- 字节码是一种抽象表示方法
- 字节码进一步编译得到机器码

- `javap -c -p class文件` 反编译字节码文件，-p 能看到私有属性
    - 输出所有的属性以及类的定义信息
    - 静态块
    - 构造方法
    - 方法信息
    - 静态属性信息
    - 静态方法信息

#### 运行时环境
> 因为JVM没有CPU那样的寄存器，所以是采用的堆栈来计算的，称为操作数栈或者计算堆栈

- 当一个类被链接进运行时环境时，字节码会受到检查，其中很多验证都可以归结为对栈中类型模式的分析
- 方法需要一块内存区域作为计算堆栈来计算新值，另外每个运行的线程都需要一个调用堆栈来记录当前正在执行的方法，这两个栈会有交互

#### 操作码介绍
- 字节码由操作码 opcode 序列构成，每个指令后可能会带参数，操作码希望看到栈处于指定状态中，然后他对栈进行操作处理，把参数移走，放入结果
- 操作码表有四列：
    - 名称：操作码类型的通用名称
    - 参数：操作码的参数，以i开头的是用来作为常量池或局部变量中的查询索引的几个字节，如果有更多的参数，将会合并
        - 如果参数出现在括号里，就表明不是所有形式的操作码都会使用他
    - 堆栈布局：他展示了栈在操作码执行前后的状态。括号中的元素表示是可选的
    - 描述：描述操作码的用处


[ ] 下面的内容需要继续阅读Java7程序员修炼之道
#### 加载和存储操作码
#### 数学运算操作码
#### 执行控制操作码
#### 调用操作码
#### 平台操作码
#### 操作码的快捷形式

#### invokedynamic
> 这个特性是针对 框架开发和非Java语言准备的

****************
## 序列化
> [码农翻身:序列化： 一个老家伙的咸鱼翻身](https://mp.weixin.qq.com/s?__biz=MzAxOTc0NzExNg==&mid=2665513589&idx=1&sn=d402d623d9121453f1e570395c7f99d7&chksm=80d67a36b7a1f32054d4c779dd26e8f97a075cf4d9ed1281f16d09f1df50a29319cd37520377&scene=21#wechat_redirect) `对象转化为二进制流`

### serialVersionUID
> 简单的说就是类的版本控制, 标明类序列化时的版本, 版本一致表明这两个类定义一致  
> 在进行反序列化时, JVM会把传来的字节流中的serialVersionUID与本地相应实体（类）的serialVersionUID进行比较，如果相同就认为是一致的，可以进行反序列化，否则就会出现序列化版本不一致的异常。(InvalidCastException)  
[参考博客](http://swiftlet.net/archives/1268)

- serialVersionUID有两种显示的生成方式： 
    -  一个是默认的1L
    -  一个是根据类名、接口名、成员方法及属性等来生成一个64位的哈希字段

> 当你一个类实现了Serializable接口，如果没有定义serialVersionUID，Eclipse会提供这个提示功能告诉你去定义 。
在Eclipse中点击类中warning的图标一下，Eclipse就会自动给定两种生成的方式。
如果不想定义它，在Eclipse的设置中也可以把它关掉的，设置如下：
Window ==> Preferences ==> Java ==> Compiler ==> Error/Warnings ==>Potential programming problems
将Serializable class without serialVersionUID的warning改成ignore即可。

******************************

### 其他业内主流编解码框架
> 因为Java序列化的性能和存储开销都表现不好,而且不能跨语言, 所以一般不使用Java的序列化而是使用以下流行的库

#### MessagePack
> [Github:msgpack](https://github.com/msgpack)

[参考博客: MessagePack：一种高效二进制序列化格式](http://hao.jobbole.com/messagepack/)

#### Protobuf
> Google开源的库 全称 Google Protocol Buffers

- 他将数据结构以 proto后缀的文件进行描述, 通过代码生成工具, 可以生成对应数据结构的POJO对象和Protobuf相关的方法和属性
    - 特点:
        - 结构化数据存储格式: XML JSON等
        - 高效的编解码性能
        - 语言无关, 平台无关, 扩展性好
        - 官方支持 Java C++ Python三种语言, 并且Js的支持也比较好[](https://github.com/dcodeIO/ProtoBuf.js/)
    - 数据描述文件和代码生成机制优点:
        - 文本化的数据结构描述语言, 可以实现语言和平台无关, 特别适合异构系统间的集成
        - 通过标识字段的顺序, 可以实现协议的前向兼容 _在不同版本的数据结构进程间进行数据传递_
        - 自动代码生成, 不需要手工编写同样数据结构的C++和Java版本;
        - 方便后续的管理和维护,相比于代码, 结构化的文档更容易管理和维护

#### Thrift
> 源于Facebook, 支持多种语言: C++ C# Cocoa Erlang Haskell Java Ocami Perl PHP Python Ruby Smalltalk

- 它支持数据(对象)序列化和多种类型的RPC服务, Thrift适用于静态的数据交换, 需要预先确定好他的数据结构, 当数据结构发生变化时,需要重新编辑IDL文件

#### Marshalling
> JBOSS 内部使用的编解码框架

**********************
## 反射的使用

