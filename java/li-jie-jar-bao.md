# 理解jar包

## 最简单的jar包示例

Teacher.java

```java
public class Teacher {
    public static void greeting() {
        System.out.println("Welcome!");
    }
}
```

Welcome.java

```java
public class Welcome {
    public static void main(String[] args) {
        Teacher.greeting();
    }
}
```

使用javac将这两个java文件构建成class文件

<div align="left"><figure><img src="../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure></div>

将两个class文件打成jar包，执行如下命令

```
jar -cvf test.jar Teacher.class Welcome.class
```

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

修改jar包里面的MANIFEST.MF文件，新增Main-Class入口配置

<div align="left"><figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

执行jar包

<div align="left"><figure><img src="../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

## 如果带有包名

包名即目录，本地需要存在目录，打的jar包当中会按照目录去构建

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

修改jar包里面的的MANIFEST.MF文件，添加入口为com.test.Welcome

<div align="left"><figure><img src="../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

执行jar包

<div align="left"><figure><img src="../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure></div>

## 如果存在依赖jar包

需要配置Class-Path参数

将主类文件、依赖jar和配置文件都打进jar包

```
jar -cvf test.jar Prefix.class lib/mysql-connector-java-8.0.20.jar prefix.properties
```

修改里面的META-INF/MANIFEST.MF文件

```
Manifest-Version: 1.0
Created-By: 1.8.0_412 (Red Hat, Inc.)
Main-Class: Prefix
Class-Path: lib/mysql-connector-java-8.0.20.jar
```

新增的参数如下：

<mark style="color:red;">Main-Class: Prefix</mark>\ <mark style="color:red;">Class-Path: lib/mysql-connector-java-8.0.20.jar</mark>

执行jar包运行

<div align="left"><figure><img src="../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure></div>

