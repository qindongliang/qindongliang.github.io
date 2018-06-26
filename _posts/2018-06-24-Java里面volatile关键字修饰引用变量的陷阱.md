---
layout: mypost
title: Java里面volatile关键字修饰引用变量的陷阱
categories: [Java]
---



# Java里面volatile关键字修饰引用变量的陷阱

如果我现在问你volatile的关键字的作用，你可能会回答对于一个线程修改的变量对其他的线程立即可见。这种说法没多大问题，但是不够严谨。

严谨的回答应该是volatile关键字对于基本类型的修改可以在随后对多个线程的读保持一致，但是对于引用类型如数组，实体bean，仅仅保证引用的可见性，但并不保证引用内容的可见性。

下面这些数据结构都属于引用类型，即使使用volatile关键字修饰，也不能保证修改后的数据会立即对其他的多个线程保持一致：

```java
volatile int [] data;
valatile boolean [] flags;
volatile Person  person;
```

如何证明？看下面的一段代码：


```java
   private static volatile Data data;

    public static void setData(int a, int b) {
        data = new Data(a, b);
    }

    private static class Data {
        private int a;
        private int b;

        public Data(int a, int b) {
            this.a = a;
            this.b = b;
        }

        public int getA() {
            return a;
        }

        public int getB() {
            return b;
        }
    }

    public static void main(String[] args) throws InterruptedException {

        for (int i = 0; i < 10000; i++) {
            int a = i;
            int b = i;
            //writer
            Thread writerThread = new Thread(() -> {setData(a, b);});
            //reader
            Thread readerThread = new Thread(() -> {
                while (data == null) {}
                int x = data.getA();
                int y = data.getB();
                if (x != y) {
                    System.out.printf("a = %s, b = %s%n", x, y);
                }
            });

            writerThread.start();
            readerThread.start();
            writerThread.join();
            readerThread.join();
        }
        System.out.println("finished");
    }
```

上面的代码，有个实体类Data，它有两个字段，分别是a和b，然后在我们的main方法中，我们声明了
一个for循环1万次，在循环体里面我们先声明了一个写入线程，每次给实体类赋值，接着又声明了一个读取线程，当实体不为null的时候，打印如果有不一致的时候，其字段的值。接着同时启动两个线程，并在主线程中分别等待其结束。

在我的mac系统上，运行了第三次的时候出现了不一致：

```
a = 2760, b = 2761
a = 3586, b = 3587
finished
```
原因是对于属性a和b我们都是分别的读取，所以缺乏了happens-before关系的约束。

如何解决这种情况？


（1）去掉独立的getA和getB方法，使用int数组，一次返回两个属性


```
    public int[] getValues() {
            return new int[]{a, b};
        }
```

（2）使用java并发包下面的基于CAS的原子结构：
AtomicReference


```
//修改1
   private static AtomicReference<Data> data = new AtomicReference<>();

//修改2
    public static void setData(int a, int b) {
        data.compareAndSet(null, new Data(a, b));
    }

//修改3
    Thread readerThread = new Thread(() -> {
                while (data.get() == null) {}
                int x = data.get().getA();
                int y = data.get().getB();
                if (x != y) {
                    System.out.printf("a = %s, b = %s%n", x, y);
                }
            });
```


总结：

本篇文章主要讲述了关于volatile修饰引用变量的问题即它只能保证引用本身的可见性，并不能保证内部字段的可见性，如果想要保证内部字段的可见性最好使用CAS的数据结构，这里还需要说明的的一点是volatile有时候修饰引用类型如boolean数组可能结果是没问题的，大家可以看我在Stack Overflow上提问的一个问题：

https://stackoverflow.com/questions/50967448/about-java-volatile-array

在编程的世界里面，对于不确定的事情，我们始终都要以最坏的打算来看待，所以请记住：尽量避免使用volatile关键字修饰引用变量。

