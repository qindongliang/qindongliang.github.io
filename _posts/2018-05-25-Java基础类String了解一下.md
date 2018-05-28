---
layout: mypost
title: Java基础类String了解一下
categories: [Java]
---



### 前言

当你路过一些商场或者地铁口的时候，有没有被千篇一律的"xx健身，了解一下" 所烦到。

无论在什么编程语言里面，字符串类型一直都是我们使用频率非常高的一个类型，在Java语言里面也不例外，今天我们不打广告而是重新认识一下我们的老朋友String类。

String类被封装在java.lang包里面，在Java里面每一个创建出来的字符串它的类型都是String，它最大的特点就是不可变（immutable ），这意味String类一旦创建就不能再修改，如果看过其源码就会发现String Class类是用关键字final修饰的并且其主要的成员变量也都是使用final修饰的。

### 什么是不可变对象
不可变对象一旦创建它的状态就不能再修改，我们常用的基本类型的包装类如Integer，Byte, Short, Float, Double都是不可变的。


下面看一个不可变类的例子：

````
public final class MyString
{
 final String str;
 MyString(String s)
 {
  this.str = s;
 }
 public String get()
 {
  return str;
 }
}
````

在上面这个例子中MyString类就是不可变的，一旦实例化之后str的值就不能变化。

当然Java里面的String底层是用char数组+UTF-16编码存储的，这个在后面会提到。


### 创建String类的常用方法

（1）使用字面量创建，是我们最常用的方法

````
String str1 = "Hello";
````

（2）通过一个String对象

````
String str2 = new String(str1);
````

（3）使用new关键词
````
String str3 = new String("Java");
````

（4）使用+号操作符
````
String str4 = str1 + str2;
OR
String str5 = "hello"+"Java";
````

### 创建一个String类时发生了什么

每当我们创建一个String字面量时，jvm（java虚拟机）都会先检查字符串池（string pool）里面是否已经存在该字符串，如果已经存在，jvm会返回这个实例的引用，如果在池里不存在那么新的字符串就会被创建，然后添加到string池中，这个string池我们可以理解它是一块常量的字符串池，在JDK7之后是被放在java堆内存里面，当然避免创建重复的实例是jvm层面对字符串创建的一个优化。

下面看下字符串对象时怎么被存储的：

````
String str= "Hello";
````
上面的这段代码创建了一个字符串字面量，如果是第一次创建，这个实例会被存放到字符串常量吃里面：

![image](https://www.studytonight.com/java/images/java-string1.jpg)

紧接着，当我们再次创建一个的字符串时，jvm实际上会把已经存在实例的引用赋值给新的字符串：
````
String str2="Hello";
// check
System.out.println(str==str2); //true
````
![image](https://www.studytonight.com/java/images/java-string2.jpg)

 当然，如果我们改变字符串的内容，那么他的引用也会对应变化：
````
 String str2=str.concat("world");
````

![image](https://www.studytonight.com/java/images/java-string3.jpg)


 ### 字符串连接

 这里有2种方法可以连接两个或者更多的字符串：

 （1）使用 concat() 方法

````
 string s = "Hello";
string str = "Java";
string str2 = s.concat(str);
String str1 = "Hello".concat("Java");    //works with string literals too.
````


 （2）使用 + 操作符

````
 string str = "Rahul";
string str1 = "Dravid";
string str2 = str + str1;
string st = "Rahul"+"Dravid";
````

 ### 字符串比较

 字符串比较这里有3种方式：

 （1）使用equals方法

 注意equals方法比较特殊，因为这个Object类里面的方法，Java里面所有的类都直接或者间接的继承了Object类，如果继承之后没有重写equals方法，那么默认比较的两个对象的内存地址。
通过观察源码我们能发现String里面的equals方法已经被重写过，它比较的是字符串的内容。

JDK8中的源码如下：
````
    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
````

 根据源码，我们知道内容相等返回true，否则就是false
````
 String s = "Hell";
String s1 = "Hello";
String s2 = "Hello";
s1.equals(s2);    //true
s.equals(s1) ;   //false
````

  （2）使用==方法

==方法比较的是两个对象的引用，也就是内存地址,如果内存地址一样，就返回true，否则就返回false

下面看个例子：
````
String s1 = "Java";
String s2 = "Java";
String s3 = new string ("Java");
test(s1 == s2)     //true
test(s1 == s3)      //false
````

细心的朋友可能已经看出来s1和s3有一样的内容，但是却返回false，是因为他们的内存地址不一样，上面说过基于字符串字面量是放在字符串常量池里面，但是如果使用new出来的实例是放在堆内存里面，所以他们的内存指针是不一样的。



   ![image](https://www.studytonight.com/java/images/not-equals-operator.png)

   （3）使用compareTo方法

   compareTo方法是基于自然顺序（通常指字母表的顺序）比较两个字符串的先后，它返回是int类型的值分别是：-1（排名靠前）,0（排名相等）,1（排名靠后）
````
String s1 = "Abhi";
String s2 = "Viraaj";
String s3 = "Abhi";
s1.compareTo(S2);     //return -1 because s1 < s2
s1.compareTo(S3);     //return 0 because s1 == s3
s2.compareTo(s1);     //return 1 because s2 > s1
````

### 总结：

本篇文章介绍了Java语言里面String类的特点和一些常见的基础用法，此外还介绍了创建一个String实例时其在JVM里面的是如何优化执行和存储的，这里面主要涉及关于字符串常量池的知识，这个会在下一篇文章中单独介绍。