---
layout: mypost
title: JDK8中LinkedList的工作原理剖析
categories: [Java]
---

LinkedList虽然在日常开发中使用频率并不是很多，但作为一种和数组有别的数据结构，了解它的底层实现还是很有必要的。

在这之前我们先来复习下ArrayList的优缺点，ArrayList基于数组的动态管理实现的，数组在内存中是一块连续的存储地址并且数组的查询和遍历是非常快的；缺点在于在添加和删除元素时，需要大幅度拷贝和移动数据，还要考虑是否需要扩容操作，所以效率比较低。


正是由于上面的不足，才出现了链表的这种数据结构，首先链表在内存中并不是连续的，而是通过引用来关联所有元素的，所以链表的优点在于添加和删除元素比较快，因为只是移动指针，并且不需要判断是否扩容，缺点是查询和遍历效率比较低下。


首先，我们先看下LinkedList的继承结构：
````java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
````

从上面的源码里面可以看到：


LinkedList继承了AbstractSequentialList是一个双向链表，可以当做队列，堆栈，或者是双端队列。

实现了List接口可以有队列操作。

实现了Deque接口可以有双端队列操作

实现了Cloneable接口既可以用来做浅克隆

实现了Serializable接口可以用来做网络传输和持久化，同时可以使用序列化来做深度克隆。


下面我们先来看下LinkedList的核心数据结构：
````java
    private static class Node<E> {
        E item; //当前节点
        Node<E> next;//后置节点
        Node<E> prev;//前置节点

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
````

然后在看下其成员变量和构造方法：

````java
     
 `   transient int size = 0;//当前存储元素的个数

    transient Node<E> first;//首节点

    transient Node<E> last;//末节点

    //无参数构造方法 
    public LinkedList() {
    }

   //有参数构造方法
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }


````

从上面可以看到LinkedList有两个构造函数，一个无参，一个有参，有参的构造函数的功能是通过一个集合参数并把它里面所有的元素给插入到LinkedList中，注意这里为什么说是插入，而不是说初始化添加，因为LinkedList并非线程安全，完全有可能在this()方法调用之后，已经有其他的线程向里面插入数据了。

（一）addAll方法分析

下面我们看下addAll方法的调用链：

````java
`   //1
    public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c);
    }
    
  //2
    public boolean addAll(int index, Collection<? extends E> c) {
        //校验是否越界
        checkPositionIndex(index);
        //将集合参数转换为数组
        Object[] a = c.toArray();
        //新增的元素个数
        int numNew = a.length;
        if (numNew == 0)
            return false;

        //临时index节点的前置节点和后置节点
        Node<E> pred, succ;
        //说明是尾部插入
        if (index == size) {
            succ = null;//后置节点同时为最后一个节点的值总是为null
            pred = last;//前置节点是当前LinkList节点的最后一个节点
        } else {//非尾部插入
            succ = node(index);//找到index节点作为作为后置节点
            pred = succ.prev;//index节点的前置节点赋值当前的前置节点
        }

        //遍历对象数组
        for (Object o : a) {
            @SuppressWarnings("unchecked") E e = (E) o;//转为指定的泛型类
            //当前插入的第一个节点的初始化
            Node<E> newNode = new Node<>(pred, e, null);
            //如果节点为null，那么说明当前就是第一个个节点
            if (pred == null)
                first = newNode;
            else//不为null的情况下，把pred节点的后置节点赋值为当前节点
                pred.next = newNode;
            //上面的操作完成之后，就把当前的节点赋值为临时的前置节点
            pred = newNode;
        }
        //循环追加完毕后
        //判断如果是尾部插入
        if (succ == null) {//如果最后一个节点是null，说明是尾部插入，那么尾部节点的前置节点，就会被赋值成最后节点
            last = pred;
        } else {//非尾部插入
            pred.next = succ;//尾部节点等于succ也就是index节点
            succ.prev = pred;//succ也就是index节点的前置节点，就是对象数组里面最后一个节点
        }
        //size更新
        size += numNew;
        modCount++;//修改次数添加
        return true;
    }


    
    //3 给一个index查询该位置上的节点
    Node<E> node(int index) {
        // assert isElementIndex(index);
         //如果index小于当前size的一半，就从首节点开始遍历查找
        if (index < (size >> 1)) {
            Node<E> x = first;//初始化等于首节点
            for (int i = 0; i < index; i++)
                x = x.next;//依次向后遍历index次，并将查询结果返回
            return x;
        } else {//否则，就从末尾节点向前遍历
            Node<E> x = last;//初始化等于末尾节点
            for (int i = size - 1; i > index; i--)
                x = x.prev;//依次向前遍历index次，并将查询结果返回
            return x;
        }
    }


````

这里面我们看到主要方法有两个：

首先是addAll(int index, Collection c)方法，这里面先判断了是否会出现索引越界的可能，然后分别初始化了两个临时节点pred和succ，分别作为index节点的前置节点和后置节点，如果不是在第一次初始化插入的情况下，这段代码的工作原理，大家可以理解为一个木棒一刀两断之后，第一段的末尾处就是前置节点，而第二段木棒的开始处就是后置节点，我们插入的数据就类似于放在两个木棒之间，然后在依次追加进来，最后把前置节点连接上和后置节点连接上，就相当于插入完成，变成了一根更长的木棒，这个过程大家用笔画一下，还是比较容易理解的。

然后大家看到还有一个方法node(int index)，这个方法的主要功能是找到index个数上的Node节点，虽然源码里面已经做过遍历优化，折半查询，如果index小于size的一半，就从头开始向后遍历查询，否则就从后向前遍历查询，即使这样，遍历和查询仍然是链表的缺点，可以看成是O(n)操作。



（二）add方法分析

add方法无疑是操作链表经常用的方法，它的源码如下：
````java
    //1
    public boolean add(E e) {
        linkLast(e);
        return true;
    }

//2
    void linkLast(E e) {
        //获取当前链表的最后一个节点
        final Node<E> l = last;
        //构造出一个新的节点，它的前置节点是当前链表的最后一个节点
        final Node<E> newNode = new Node<>(l, e, null);
        //然后把新节点作为当前链表的最后一个节点
        last = newNode;
        //首次插入
        if (l == null)
            first = newNode;
        else//非首次插入就把最后一个节点指向新插入的节点
            l.next = newNode;
        size++;//size更新
        modCount++;
    }



````

从上面我们看到add方法每次都会把新增节点放在链表的最后的一位，正是因为放在链表的末位，所以链表的添加性能可以看成O(1)操作。



（二）remove方法分析

移除方法有常用的有两个，一个是根据index移除，另外一个根据Object移除，源码如下：

````java
   //1
    public E remove(int index) {
        checkElementIndex(index);
        return unlink(node(index));
    }
    
    //2
        public boolean remove(Object o) {
        //移除的数据为null
        if (o == null) {
            //遍历找到第一个为null的节点，然后移除掉
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            //移除的数据不为null，就遍历找到第一条相等的数据，然后移除掉
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
    
    
    //3
        E unlink(Node<E> x) {
        // assert x != null;
        //移除的数据
        final E element = x.item;
        //移除节点后置节点
        final Node<E> next = x.next;
        //移除节点前置节点
        final Node<E> prev = x.prev;

        //如果前置节点为null，那么后置节点就赋值给首节点
        if (prev == null) {
            first = next;
        } else {//前置节点的后置节点为当前节点的后置节点
            prev.next = next;
            x.prev = null;//当前节点的前置节点置位null
        }
        //如果后置节点为null，末尾节点就为当前节点的前置节点
        if (next == null) {
            last = prev;
        } else {//否则后置节点的前置节点为移除本身的前置节点
            next.prev = prev;
            x.next = null;//移除节点的末尾为null
        }
        //移除的数据置位null，便于gc
        x.item = null;
        size--;
        modCount++;
        return element;
    }
    
````


从上面的源码里可以看到移除根据index移除里面调用了node(index)方法来查找需要移除的节点，而根据Object移除的时候，则是进行了整个链表的遍历，然后再卸载节点。


除此之外链表还有没有任何参数的remove，removeFirst，removeLast方法，其中remove方法本质是调用了removeFirst方法。


这里能够总结出来链表基于首尾节点的删除可以看成是O(1)操作，而非首尾的删除最坏的情况下能够达到O(n)操作，因为要遍历查询指定节点，所以性能较差。


（三）get方法分析

get系方法有三个，分别是get(index)，getFirst()，getLast()，其中get(index)方法如下：
````java
    public E get(int index) {
        checkElementIndex(index);//是否越界
        return node(index).item;//折半遍历查询
    }
````

我们看到get(index)方法本质调用了node(index)方法，这个方法在前面分析过了性能一半，其他的getFirst和getLast不用多说了O(1)操作。

（四）set方法分析

源码如下：

````java
    public E set(int index, E element) {
        checkElementIndex(index);//是否越界
        Node<E> x = node(index);//折半查询
        E oldVal = x.item;// 查询旧值
        x.item = element;//放入新值
        return oldVal;//返回旧值
    }
````

set方法依旧是调用的node方法，所以链表在指定位置更新数据，性能也一般。

（四）clear方法分析

````java
    public void clear() {
         //遍历所有的数据，置位null      
        for (Node<E> x = first; x != null; ) {
            Node<E> next = x.next;
            x.item = null;
            x.next = null;
            x.prev = null;
            x = next;
        }
        first = last = null;
        size = 0;
        modCount++;
    }
````
clear方法比较简单，就是所有的节点的数据置位null，方便垃圾回收


（五）toArray方法分析
````java
    public Object[] toArray() {
      //声明长度一样的数组
        Object[] result = new Object[size];
        int i = 0;
        for (Node<E> x = first; x != null; x = x.next)
            result[i++] = x.item;
        return result;
    }

````

声明一个长度一样的数组，依次遍历所有数据放入数组。



（六）序列化和反序列方法分析
````java
//序列化
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
        s.defaultWriteObject();
        //先写入大小
        s.writeInt(size);
        //再依次遍历链表写入字节流中
        for (Node<E> x = first; x != null; x = x.next)
            s.writeObject(x.item);
    }
    
    //反序列化
        private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        // Read in any hidden serialization magic
        s.defaultReadObject();

         //先读取大小
        int size = s.readInt();
        //再依次读取元素，每次都追加到链表末尾
        for (int i = 0; i < size; i++)
            linkLast((E)s.readObject());
    }

````

这里我们看到链表中也自定义了序列化和反序列化的方法，在序列化时只写入x.item而并不是整个Node，这样做避免java自带的序列化机制会把整个Node的数据给写入序列化，并且如果Node还是双端链表的数据结构，那么无疑会导致重复两倍的空间浪费。

在反序列化时我们看到先读取size，然后根据size依次循环读取item，并重新生成双端链表的数据结构，依次追加到链表的尾部。




（六）关于操作队列或者堆栈的方法

文章开头说了LinkedList可以当双端队列或者堆栈使用，这里面有一系列的方法，这里只简单列举几个常用的方法，因为原理比较简单所以不在叙述：
````java
pop()//移除第一个元素并返回
push(E e)//添加一个元素在队列首位
poll()  //移除第一个元素
offer(E e)//添加一个元素在队列末位
peek()//返回第一个元素，没有移除动作
````



总结：

   本文介绍了JDK8中LinkedList的工作原理，并对其常用的方法进行了分析，LinkedList底层是一个链表，链表在内存中不是一块连续的地址，而是用多少就会申请多少，所以它比ArrayList更加节省空间，
   此外它的添加方法和首末位的删除操作非常快，但是查询和遍历操作比较耗时。最后LinkedList还可以用来当做双端队列和堆栈容器，需要特别注意的是LinkedList也并非是线程安全的，如果需要
   请使用其他并发工具包下提供的类。
   







