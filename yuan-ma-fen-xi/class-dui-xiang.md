# Class对象

## 区分java中的对象

Java中存在两种对象，一种叫实例对象，一种叫Class对象

每个类的运行时的类型信息就是用Class对象表示的。它包含了与类有关的信息。其实我们的实例对象就通过Class对象来创建的。Java使用Class对象执行其RTTI（运行时类型识别，Run-Time Type Identification），多态是基于RTTI实现的

## Class对象获取方式

1. Class.forName("类的全限定名")
2. 实例对象.getClass()
3. 类名.class （类字面常量）

## 类加载机制

虚拟机把描述类的数据从Class文件中加载到内存，并对数据进行校验、转换解析和初始化，最终形成可被虚拟机直接使用的Java类型，这就是虚拟机的类加载机制。

## 类加载过程

### 加载阶段

```
通过一个类的全限定名来获取定义此类的二进制字节流。
将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口
```

### 链接阶段

#### 验证

连接阶段的第一步，确保Class文件中的字节流信息符合当前虚拟机的要求

```
文件格式验证
元数据验证
字节码验证
符号引用验证
```

#### 准备

正式为类变量分配内存并设置类变量初始值的阶段，这些变量所使用的内存都将在方法区中进行分配

注：类变量仅包含static变量，不包括实例变量。赋初值并不是赋值阶段。

#### 解析

虚拟机将常量池内的符号引用替换为直接引用的过程。

```
符号引用：符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义的定位到目标即可。符号引用的字面量形式明确定义在Java虚拟机的规范当中。
直接引用：可以是直接指向目录的指针、相对偏移量或是一个能间接定位到目录的句柄，如果有了直接引用，那引用的目标必定已经在内存中存在。
```

### 初始化阶段

类初始化阶段是类加载过程的最后一步，前面的类加载过程中，除了在加载阶段用户应用程序可以通过自定义类加载器参与之外，其余动作完全由虚拟机进行主导和控制，\
到了初始化阶段，才真正开始执行类中定义的java程序代码。

```
初始化阶段是执行类构造器<clinit>()方法的过程
```

什么是clinit方法

Java在编译时，当类有静态变量赋值操作或者静态代码块时，编译后会为这个类生成clinit方法来执行这些语句

```
1、<clinit>()方法是由编译器自动收集类中所有类变量（静态变量）和赋值动作和静态语句块（static块）中的语句合并产生的，手机顺序是由语句在源文件中出现的顺序所决定的。
2、<clinit>()方法与类的构造函数（实例构造器<init>()方法}）不同，它不需要显示的调用父类构造器，虚拟机会保证子类的<clinit>()方法执行前，父类的<clinit>()方法已经被执行了
3、父类的静态语句块优先与子类静态语句块变量的赋值操作
4、<clinit>()方法不是必须的，如果类中没有静态语句块，也没有对静态变量的赋值操作，编译器可以不生成<clinit>()方法
5、接口中不能使用静态语句块，但仍然有变量初始化的赋值操作，所以也可能会生成<clinit>()方法
6、虚拟机会保证<clinit>()方法在多线程环境中被正确的加锁和同步
```

什么是init方法\
Java在编译时，会生成init方法。将变量，语句块初始化，调用父类的构造器等操作收敛到方法

方法的的执行时间一定早于，因为方法是在类初始化过程中执行的，而方法是在对象实例化时执行的。

## 类加载器

虚拟机设计团队把类加载阶段中的"通过一个类的全限定名来获取描述此类的二进制字节流"这个动作放到Java虚拟机外部去实现，以便让应用程序自己决定如何去获取所需要的类，执行这个动作的代码模块称为”类加载器“。

从Java虚拟机的角度讲，存在两种不同的类加载器：一种是启动类加载器，这个类加载器使用c++语言实现，是虚拟机自身的一部分。另一种就是所有的其他的类加载器，\
这些加载器都由Java语言实现，独立于虚拟机外部，并且全部继承自抽象类java.lang.ClassLoader

从开发人员角度讲，类加载器还可以区分的更细致一下

```
启动类加载器
扩展类加载器：由sun.misc.Launchear$ExtClassLoader实现，负责加载<JAVA_HOME>\lib\ext目录中的，或者被java.ext.dirs指定的路径中的所有的类库
应用程序类加载器：由sun.misc.Launchear$AppClassLoader实现，负责加载用户类路径上指定的类库
自定义类加载器：我们可以编写类加载器
```

## Class对象内部方法

### 判断类型

```
isInstance(Object obj)
isAssignableFrom(Class<?> cls)
isInterface()
isArray()
isPrimitive()
isAnnotation()
isSynthetic()
```

### 获取构造器

```
getConstructor(Class<?>... parameterTypes)：返回指定参数类型、public访问权限的构造器
getDeclaredConstructor(Class<?>... parameterTypes)：返回指定参数类型、所有访问权限的构造器
getDeclaredConstructor()：返回所有访问权限的构造器
```

### 获取字段

```
getFields：获取public修饰的所有Field对象，返回一个Field数组（包括父类的）
getDeclaredFields：获取所有Field对象数组
getField：传入一个参数（属性名），获取单个Field对象，只能获取public修饰的
getDeclaredField：传入一个参数（属性名），获取单个Field对象
```

### 获取方法

```
getMethods：获取所有的public修饰的Method对象数组，包括父类的
getDeclaredMethods：获取所有的Method对象数组，不包括父类
getMethod：传入一个参数（方法名），返回一个Method对象，只能获取到public修饰的
getDeclared：传入一个参数（方法名），返回一个Method对象
```

## 简单示例

```java
package com.code.source.curriculum1;

import java.lang.reflect.Constructor;
import java.util.Arrays;

public class A {
    public String count = "1";

    public static void main(String[] args) throws Exception {
        A a = new A();
        Class<?> aClass = a.getClass();

        //获取构造器
        Constructor<?> aClassConstructor = aClass.getConstructor();
        A a1 = (A) aClassConstructor.newInstance();
        if(a == a1){
            System.out.println("a == a1");
        } else {
            System.out.println("a != a1");
        }

        //获取字段
        Arrays.stream(aClass.getDeclaredFields()).forEach(field -> System.out.println(field.getName()));

        //获取方法
        Arrays.stream(aClass.getDeclaredMethods()).forEach(method -> System.out.println(method.getName()));
    }
}

```

<figure><img src="../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>
