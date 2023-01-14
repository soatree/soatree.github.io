---
title: 八股文--java基础
categories:
  - 面经
tags:
  - java
date: 2023-01-14 22:02:42
---

# 前言

java基础相关面试题目整理

# 题目

## Java创建一个对象的方法有几种？

1. 使用new关键字
2. 反射
    (1)使用Class类的newInstance方法
    ```
    Employee emp2 = (Employee) Class.forName("org.programming.mitra.exercises.Employee").newInstance();
    ```
    或者(目前Class.newInstance()这种方法因为效率问题已经被标记@Deprecated)
    ```
    Employee emp2 = Employee.class.newInstance();
    ```
    (2)使用Constructor类的newInstance方法
    ```
    Constructor<Employee> constructor = Employee.class.getConstructor();
    Employee emp3 = constructor.newInstance();
    ```
3. clone
    类需要先实现Cloneable接口并实现其定义的clone方法。
    ```
    Employee emp4 = (Employee) emp3.clone();
    ```
4. 反序列化
    需要让类实现Serializable接口
    ```
    ObjectInputStream in = new ObjectInputStream(new FileInputStream("data.obj"));
    Employee emp5 = (Employee) in.readObject();
    ```

参考:
[Java中创建对象的5种方式](https://www.cnblogs.com/wxd0108/p/5685817.html)

## Java中==和equals的区别是什么？

略

## 序列化的作用是什么？常见的序列化方法是什么？Java自带的序列化是怎么实现的？

### 序列化的作用是什么？

网络传输的数据都必须是二进制数据，但是在Java中都是对象，是没有办法在网络中进行传输的，所以就需要对Java对象进行序列化。

序列化：将数据结构或对象转换成二进制串的过程；

反序列化：将在序列化过程中所生成的二进制串转换成数据结构或者对象的过程

### 常见的序列化方法是什么？

json序列化，java原生序列化等。关于选型可以参考美团技术博客，见[序列化和反序列化](https://tech.meituan.com/2015/02/26/serialization-vs-deserialization.html)。

### Java自带的序列化是怎么实现的？

1. 对于要序列化对象的类要去实现Serializable接口或者Externalizable接口
    ```
    public class TestBean implements Serializable {
        private static final long serialVersionUID = 6000894702030258313L;
        private Integer id;
        private String name;
        private Date date;
        //省去getter和setter方法和toString
    }
    ```
2. 使用JDK提供的ObjectOutputStream和ObjectInputStream来实现序列化和反序列化
    序列化：
    ```
    public static void main(String[] args) {
        TestBean testBean = new TestBean();
        testBean.setDate(new Date());
        testBean.setId(1);
        testBean.setName("zll1");
        //使用ObjectOutputStream序列化testBean对象并将其序列化成的字节序列写入test.txt文件
        try {
            FileOutputStream fileOutputStream = new FileOutputStream("D:\\test.txt");
            ObjectOutputStream objectOutputStream = new ObjectOutputStream(fileOutputStream);
            objectOutputStream.writeObject(testBean);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    ```
    反序列化：
    ```
    public static void main(String[] args) {
        try {
            FileInputStream fileInputStream = new FileInputStream("D:\\test.txt");
            ObjectInputStream objectInputStream = new ObjectInputStream(fileInputStream);
            TestBean testBean = (TestBean) objectInputStream.readObject();
            System.out.println(testBean);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
    ```
    注意：
    - 一个对象要进行序列化，如果该对象成员变量是引用类型的，那这个引用类型也一定要是可序列化的，否则会报错
    - 对于不想序列化的字段可以在字段类型之前加上transient关键字修饰（反序列化时会被赋予默认值）
    - 实现Externalizable接口需要重写两个方法writeExternal(ObjectOutput out)和readExternal(ObjectInput in)，用来自定义决定哪些字段需要序列化，哪些需要反序列化
    - 关于serialVersionUID：在进行序列化时，会把当前类的serialVersionUID写入到字节序列中（也会写入序列化的文件中），在反序列化时会将字节流中的serialVersionUID同本地对象中的serialVersionUID进行对比，一致的话进行反序列化，不一致则失败报错（报InvalidCastException异常）。如果未显式声明serialVersionUID，会隐式申明一个serialVersionUID，这个serialVersionUID和对象字段还有JVM环境有关，如果没有设置serialVersionUID做了序列化，然后修改了字段就无法进行反序列化了，所以一般会先设定一个serialVersionUID。可以基于IDEA自动生成(参见[IDEA 自动生成serialVersionUID](https://blog.csdn.net/Aphysia/article/details/80620804))，也可以基于jdk中的工具生成。

参考:
[Java几种序列化方式对比](https://zhuanlan.zhihu.com/p/361705392)
[序列化和反序列化](https://tech.meituan.com/2015/02/26/serialization-vs-deserialization.html)
[一文搞懂序列化与反序列化](https://zhuanlan.zhihu.com/p/316200445)
[IDEA 自动生成serialVersionUID](https://blog.csdn.net/Aphysia/article/details/80620804)
[serialVersionUID作用是什么以及如何生成的](https://cloud.tencent.com/developer/article/1943450)

## 重载和重写的区别？

略

## java的异常机制？结合项目描述一下你是怎么处理异常的？

### java的异常机制

- Error和Exception

Java所有异常类都是Throwable的子类。它包括Java异常处理的两个重要子类：Error和Exception。

Error：Error及其子类用来描述Java运行系统中的内部错误以及资源耗尽的错误，是程序无法处理的错误，这类错误比较严重，如OutOfMemoryError。这些异常发生时，JVM一般会选择线程终止。这些错误是不可查的，因为它们在应用程序的控制和处理能力之外，而且绝大多数是程序运行时不允许出现的状况。 

Exception：可以通过捕捉处理使程序继续执行，是程序自身可以处理的异常。根据错误发生原因可分为RuntimeException异常和除RunTimeException之外的异常，如IOException异常。RuntimeException 类及其子类表示“JVM 常用操作”引发的错误。例如，若试图使用空值对象引用、除数为零或数组越界，则分别引发运行时异常（NullPointerException、ArithmeticException和 ArrayIndexOutOfBoundException）。 

- 可查异常和不可查异常

可查异常：编译器要求必须处理的异常，这类异常的发生在一定程度上是可以预计的，除了RuntimeException及其子类以外的其它异常类都属于可查异常。这种异常的特点是Java编译器会检查它，也就是说，出现这种异常，要么用try-catch语句捕捉它，要么用throws语句声明抛出它，否则编译不通过。

不可查异常：编译器不要求强制处理的异常，包括运行时异常（RuntimeException与其子类）和错误（Error） 。

### 结合项目描述一下你是怎么处理异常的

自定义异常实现错误码接口，在特定检查处抛出包含错误码的自定义异常，

参考：
[Java的异常处理机制](https://blog.csdn.net/qq_15349687/article/details/82811581)

## Object类有哪些常用的方法

- getClass()

返回Class，native方法

- hashCode()

返回一个int，native方法

- equals(Object obj)

默认等价与==

- clone()

需要继承Coloneable接口，返回一个对象的副本，不过是对于对象的引用型字段（String和包装类除外）为浅拷贝

- toString()

默认返回 getClass().getName() + "@" + Integer.toHexString(hashCode())

- notify()、wait()

通知

- finalize()

在GC准备释放对象所占的内存空间时，将会调用finalize()，用于释放非java资源（例如打开的文件资源、数据库连接等）或调用非java方法（native方法）时分配的内存。finalize()最多只会执行一次，可以在这次执行中使对象再次被引用，使对象再活一次。

已经标记为@Deprecated，主要原因为
1. 调用时机的不确定性
2. 影响代码的可移植性，移植到其他JVM，finalize方法可能运行崩溃
3. 成本较高，如果一个类没有重载finalize方法，那么销毁时只要将堆中的内存处理一下就可以了，而如果重载了finalize方法的话，就要执行finalize方法，万一执行过程中再出现点异常或错误成本就更高了。
4. 异常丢失，万一fianlize方法中抛出了异常，那么finalize会终止运行，而抛出的这个异常也会被舍弃，最终会让对象实例处于一种半销毁半存活的僵尸状态

参考：
[Java中finalize()方法的使用](https://blog.csdn.net/CrazyLai1996/article/details/84900818)
[Java中的finalize方法已经退出历史舞台，再看它最后一眼，留点余香](https://zhuanlan.zhihu.com/p/94072959)

## java的hashcode和equals方法的作用？什么时候会用到？

hashcode在HashMap和HashSet中可以用于确定key所在桶的位置。在HashMap和HashSet中，两个key的hashcode相等，且equals为true时，可以判定为相同的key，就不会重复添加key。

## String是一个基本类型么？那java的基本类型有哪些？String和StringBuffer和StringBuilder的区别是什么？

不是;int, long, float, double, boolean, char等；String是不可变字符串，StringBuffer和StringBuilder是可变字符串，StringBuffer线程安全，StringBuilder线程不安全但是效率高。StringBuffer虽然线程安全但可能会造成字符串写入顺序问题，一般使用StringBuilder。

## String str="i"与 String str=new String("i")一样吗？不同点主要在哪里呢？

不一样。String str="i"的方式，Java 虚拟机会将其分配到常量池中，常量池指的是在编译期被确定，并被保存在已编译的.class文件中的一些数据，常量池不会重复创建对象；而 String str=new String(“i”)方式，则会被分到堆内存中。

```
    public static void main(String[] args) {
        String str1 = "i";
        String str2 = "i";
        String str3 = new String("i");
        String str4 = new String("i");
        System.out.println(str1 == str2);//ture
        System.out.println(str2 == str3);//false
        System.out.println(str3 == str4);//false
        System.out.println(str2.equals(str3));//ture
        System.out.println(str3.equals(str4));//ture
    }
```

参考：
[String str=“i“与 String str=new String(“i”)一样吗？](https://blog.csdn.net/MuYIShan1/article/details/115350086)

## 抽象类和接口的区别？抽象类一定要有抽象函数么？接口定义的变量一定是常量么？接口中可以定义函数的实现么？

### 抽象类和接口的区别？

相同点：

1. 都不能被实例化 
2. 接口的实现类或抽象类的子类都只有实现了接口或抽象类中的方法后才能实例化。

不同点：

1. 接口只有定义，不能有方法的实现，java 1.8中可以定义default方法体，而抽象类可以有定义与实现，方法可在抽象类中实现。

2. 实现接口的关键字为implements，继承抽象类的关键字为extends。一个类可以实现多个接口，但一个类只能继承一个抽象类。所以，使用接口可以间接地实现多重继承。

3. 接口强调特定功能的实现，而抽象类强调所属关系。

4. 接口成员变量默认为public static final，必须赋初值，不能被修改；其所有的成员方法都是public、abstract的。抽象类中成员变量默认default，可在子类中被重新定义，也可被重新赋值；抽象方法被abstract修饰，不能被private、static、synchronized和native等修饰，必须以分号结尾，不带花括号。

### 抽象类一定要有抽象函数么？

不需要，抽象类不一定有抽象方法；但是包含一个抽象方法的类一定是抽象类。

### 接口定义的变量一定是常量么？

是

### 接口中可以定义函数的实现么？

java 1.8中可以定义default方法体

参考：
[Java基础之接口与抽象类的区别](https://zhuanlan.zhihu.com/p/94770324)
[抽象类必须要有抽象方法吗？](https://blog.csdn.net/ZM_Crazy/article/details/113931384)

## final、static关键字在java中的作用？抽象类可以使用final修饰么？

### final、static关键字在java中的作用？

- final 变量
    变量指定的对象存储空间固定，不能修改，只读
- final 方法
    final 方法使该类的子类无法重写该方法
- final 类
    无法被子类继承
- static 
    与类相关，例如静态内部类、静态方法、静态变量、静态代码块

### 抽象类可以使用final修饰么？

不能

## final,finally,finalize分别是什么？

- final：java中的关键字，修饰符。
1. 如果一个类被声明为final，就意味着它不能再派生出新的子类，不能作为父类被继承。
2. 被声明为final的变量必须在声明时给定初值，变量指向的内存地址无法改变。
3. 被声明final的方法子类无法重写。
- finally：java的一种异常处理机制。
finally是对Java异常处理模型的补充。finally结构使代码总会执行，而不管无异常发生。使用finally可以维护对象的内部状态，并可以清理非内存资源。特别是在关闭数据库连接这方面，如果程序员把数据库连接的close()方法放到finally中，就会大大降低程序出错的几率。
- finalize：Java中的一个方法名。
Java技术使用finalize()方法在垃圾收集器将对象从内存中清除出去前，做必要的清理工作。这个方法是由垃圾收集器在确定这个对象没被引用时对这个对象调用的。它是在Object类中定义的，因此所的类都继承了它。子类覆盖finalize()方法以整理系统资源或者执行其他清理工作。目前该方法已经在JDK1.9被标记@Deprecated

参考：
[Java中final、finally、finalize的区别与用法](https://www.cnblogs.com/smart-hwt/p/8257330.html)

## Java的IO流有了解过，实现一个按行读取数据的方式。
## Java的反射原理是什么？Getclass和classforName的区别是什么？
## 如何实现一个list类型的深拷贝？Java的clone接口的作用是什么？
## Java的泛型的作用是什么？
## Java的注解有了解，其底层的实现原理是什么？怎么定义一个注解？
## Java中两个类的关系有多少种？有了解过设计模式么？
## Java的collection有几种？Collection和collections的区别是什么？
## ArrayLsit、LinkedList和vector的区别？它们是线程安全的么？如果想要线程安全应该要怎么实现？
## HashMap扩容机制？hashMap是线程安全的么？它和hashtable的区别是什么？hashMap key和value可以是null么？Hashmap的扩容一定是2^n么？1.8版本的优化点在哪里？什么时候链表转换为红黑树？什么时候红黑树转换为链表？Hashmap的get和put方法是怎么实现的？
## Queue中poll和remove方法的区别是什么？
## Iterator是什么？和splitIterator的区别是什么？
## ArrayList中怎么一边遍历一边删除？
## Treemap 和linkedHashMap有什么作用？其实现的原理是什么？
## HashSet的实现原理有了解？
## 静态内部类和内部类的区别是什么？为什么内部类可以访问外部类？
## 从安全性上说说java权限关键字private，protected，default，public的区别？
## 面向对象的概念是什么？Java的多态怎么实现？
## try {}里有一个return语句，那么紧跟在这个try后的finally {}里的code会不会被执行，什么时候被执行，在return前还是后?


# 参考

[微信公众号:我的IT技术路](https://mp.weixin.qq.com/s/54_bMeUwjxk-8DHa90heNQ)


