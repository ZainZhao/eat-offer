## Basic

- 属于 `java.lang`包

- Java导包的两种机制

  - 单类型导入（single-type-import）：`import java.io.File`
    - 好处：提高编译速度、避免命名冲突
    - 坏处：import语句看起来很长
  - 按需类型导入（type-import-on-demand）：`import java.io.*`
    - 并非导入整个包，而是导入当前类需要使用的类

  > 不能随意使用按需类型导入？因为单类型导入和按需类型导入对类文件的定位算法是不一样的
  >
  > 对于单类型导入很简单，因为包明和文件名都已经确定，所以可以一次性查找定位。
  >
  > 对于按需类型导入则比较复杂，编译器会把包名和文件名进行排列组合，然后对所有的可能性进行类文件查找定位。
  >
  > eg..
  >
  > ```java
  > package com;
  > import java.io.*;
  > import java.util.*;
  > 
  > 查找File类，会查找：
  > File
  > com.File
  > java.lang.File
  > java.io.File
  > java.util.File
  > ```
  >
  > 找到类之后不会停止下一步查找，而要把所有的可能性都查找完以确定是否有类导入冲突。如果在查找完成后，编译器发现了两个同名的类，那么就会报错

  注：按需导入不会降低Java代码的执行效率，但会影响代码的编译速度



- instanceof 

  - 测试一个对象是否为一个类的实例

  - obj 必须为引用类型，不能是基本类型

  - obj 为 null    `null instanceof Object => false`

  - obj 为 class 接口的实现类   

    ```java
    ArrayList arrayList = new ArrayList();
    System.out.println(arrayList instanceof List);//true
    // 反过来也是返回 true
    List list = new ArrayList();
    System.out.println(list instanceof ArrayList);//true
    ```

  - obj 为 class 类的直接或间接子类

    ```java
    public class Person {}
    public class Man extends Person{}
    
    Person p1 = new Person();
    Person p2 = new Man();
    Man m1 = new Man();
    System.out.println(p1 instanceof Man);//false Man是Person的子类
    System.out.println(p2 instanceof Man);//true
    System.out.println(m1 instanceof Man);//true
    ```

## 常用方法

**equals()**

- equals 与 == 的区别
  - == 用来比较基本类型的值是否相等或者两个对象的引用是否相等
  - equals 用来比较两个对象是否相等
- Object 中的 equals 与 == 是等价的
- 重写的 equals 方法应该遵守 自反性、对称性、传递性、一致性 
- 如果 equals 的语义在每个子类中有所改变，就使用`getClass`检测。如果所有子类都有统一的定义，那么使用`instanceof`

**public final native Class<?>getClass()**

- native 修饰的方法是由操作系统帮忙实现的
- calss是类的一个属性，能获取该类编译时的类对象。没有多态的，是静态解析的，编译时可以确定类型信息。
- getClass() 是一个类的方法，它获取该类的类对象。有多态能力，运行时可以返回子类的类型信息。
- ` Class<?>` 这样也能编译通过 `Class<? extends String> c = "".getClass()`

**public native inthashCode()**

- 为避免hash冲突，也就是算法获得的元素要尽量均匀分布
- JDK中许多包装类也重写了hashCode方法
- 算法获得的元素应该尽量均匀分布
- 为什么不直接通过hashCode产生唯一索引？
  - 很难有这种算法
  - 生成的hashCode非常大，可能会超出Java所能表示的范围

- String 类的 hashCode

  ```java
  public int hashCode() {
      int h = hash; // 默认值是0
      if (h == 0 && value.length > 0) {
          char val[] = value; // 字符串对应的char数组
  
          for (int i = 0; i < value.length; i++) {
              h = 31 * h + val[i]; // val[0]*31^(n-1) + val[1]*31^(n-2) + ... + val[n-1]
          }
          hash = h;
      }
      return h;
  }
  ```

- 对于需要大量并且快速的对比，如果都用equals()去实现显然效率太低，所以可以考虑先比较hashCode

**toString()**

- `getClass().getName() +"@"+ Integer.toHexString(hashCode())`

**notify() notifyAll() wait()**

**finalize()**

**private static native void registerNatives()**