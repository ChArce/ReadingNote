## JAVA 编程思想

### 第14章 类型信息


Class对象只在需要的时候才会加载，static的初始化在类加载的时候进行.


如果运行时需要使用类型信息，首先要获得对恰当的Class对象的引用，Class.forName()可以不需要为了获得
Class引用而持有该类型的对象。当然如果此时你有一个感兴趣的类型的对象，那么可以使用getClass()方法来
获取Class引用.


```java
Class t = Class.forName(T);
t.newInstance();
```

```java
T t = new T();
```
上述两个创建对象的效果是一样子的，但是机制不一样
   1. newInstance()必须保证类已经加载并且已经链接，而new可以没有被加载
   2. newInstance()只能调用无参的构造函数，而new生成对象没有这个限制。

另外还有一种方法来生成对Class对象的引用,如 `FancyToy.class`, 这种方法在编译时就会受到检查，也更高效。
对于基本数据类型，还有一个标准字段TYPE。如:
| int.class | Integer.TYPE |
| byte.class | Byte.TYPE |


```java
Class<? extends Number> bounded = int.class;
bounded = double.class;
bounded = Number.class;
```

对于泛型来说 newInstance()是可以返回该对象的确切类型的，但如果持有的是超累，那么此时newInstance()返回的不是精确类型，而只是Object

