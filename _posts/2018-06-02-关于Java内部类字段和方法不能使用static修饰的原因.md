---
layout: mypost
title: 关于Java内部类字段和方法不能使用static修饰的原因
categories: [Java]
---


昨天的文章中，遗留了一个问题就是，为什么Java内部类字段和方法不能使用static修饰。

先下下面一段代码：

```
class OuterClass {

 public int age=20;

 class InnerClass {
  static int i = 100; // compile error
  static void f() { } // compile error
 }
}
```


上面的内部类的成员变量和方法，只要加上了static修饰，就会出现编译错误。

原因：

简单的来说，内部类是外部类的实例，与外部类的的成员变量是一样的，每个实例化出来的对象，它的成员变量赋值都是独立的不会相互影响。

看下面一个例子：

```
class Employee {
    public String name;
}
```

上面是一个成员变量，现在我们创建两个实例并赋值：
```
Employee a = new Employee();
a.name = "Oscar";

Employee b = new Employee();
b.name = "jcyang";
```
上面的代码是没问题的。

ok，现在继续看，假如我们允许内部类出现静态成员，会出现什么情况：

```
class Employee {
    public String name;
    class InnerData {
       public static count; // ??? count of which ? a or b?
     }
}
```
这个时候，我们给内部类实例的静态字段count赋值，就会发生混乱：

```
Employee a = new Employee();
a.name = "Oscar";
a.new InnerData().count=3

Employee b = new Employee();
b.name = "jcyang";
b.new InnerData().count=4
```

现在已经分不清到底是修改类count字段，逻辑上来说成员变量是对象各自独立的属性读写都是不依赖的，但如果上面的场景成立，就会自相矛盾。所以这就是为什么内部类里面不允许存在静态成员的原因。

其实归根结底，还是类与对象的区别，静态属性不依赖于对象，因为它保存在jvm的静态区，所以访问修改的时候不需要依赖当前有没有存活的对象，在虚拟机加载的时候也是优先于实例生成的。而实例对象则是保存在jvm的堆内存中，想要访问内部类，必须先实例化外部类，然后通过外部类才能访问内部类。内部类其实也可以认为是外部类的一个成员变量，只要是成员变量，各个对象都是不依赖的，静态属性的出现破坏了这一逻辑，所以java语言在语义层面不允许我们那么做，这其实不是技术问题，是一个语言的逻辑和语义问题。

最近开通了知识星球服务，欢迎大家加入一起学习。
![image](http://dl2.iteye.com/upload/attachment/0129/9751/e6046a0e-bc81-3f0a-9940-8621b067b83f.jpeg)