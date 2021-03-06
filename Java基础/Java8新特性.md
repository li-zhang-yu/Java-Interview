# 一、Lamada

## 1.1 实现匿名内部类

## 1.2 语法格式

```java
/**
 * 总结：lambda表达式
 * 左侧：lambda表达式的参数列表————————接口的参数列表
 * 右侧：接口的方法实现
 *
 * 语法格式1：无参数无返回值方法 () -> System.out.println("runnable");
 * 语法格式2：一个参数，无返回值 (x) -> System.out.println(x)
 * 语法格式3：一个参数，括号可以不写
 * 语法格式4：多个参数，有返回值，lambda有多条语句，有返回值 使用{}
 * 语法格式5：多个参数，有返回值，lambda中只有一条语句，return可以省略
 * 语法格式6：lambda参数列表中的参数类型可以省略不写，因为JVM编译器，通过上下文推断出数据类型，即“类型推断”
 */
```

## 1.3 Java内置的四大核心函数式接口

消费类型接口：Consumer<T>

供给型接口：Supplier<T>

函数型接口：Function<T,R>

断定型接口：Predicate<T>

![](http://mycsdnblog.work/201919121408-c.png)

## 1.4 lambda表达式的作用

**不仅仅是语法糖**

lambda带给我们的，不仅仅只是语法上的简洁，而是一种新的编程方式，可以**将函数当做参数传递**，有点C#里面委托的意思。

> 委托是一个类，它定义了方法的类型，使得可以将方法当作另一个方法的参数来进行传递
>
> public delegate int MethodtDelegate(int x, int y);表示有两个参数，并返回int型。

所谓“函数参数”，虽然本质上也是用内部类实现。但是，那又怎样呢？我们完全可以将其理解为函数参数来简化我们的编程，不能将lambda简单理解为少写代码的语法糖。

**函数式接口：只有一个方法的接口**

使用此类接口的时候（Runable，Comparator），要么我们定义一个类来实现该接口，要么使用匿名内部类。对于匿名内部类，编译器会为每一个匿名内部类创建相应的类文件。一般的程序，往往回调接口会有很多。这样就会生成很多的类文件，因为类在使用之前需要加载类文件并进行验证，这个过程就会影响应用的性能。

# 二、流

流是数据渠道，用于操作数据源（集合、数组等）所生成的元素序列。“集合讲的是数据，流讲的是计算”。

注意：

Stream自己不会存储元素

Stream不会改变源对象。相反，他们会返回一个持有结果的新Stream。

Stream操作是延迟执行的，这意味着他们会等到需要结果的时候才去执行。

## 2.1 操作步骤

1. 创建流

2. 中间操作

   ```
   /**
    * 中间操作
    *
    * 筛选与切片
    * filter——接收lambda，从流中排除某些元素
    * limit——截断流，使用元素不超过给定数量
    * skip——跳过元素
    * distinct——筛选
    */
   
   /**
    * map 接收lambda，将元素转换成其它形式或提取信息。接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素
    * flatMap 接收一个函数作为参数，将流中的每个值都转换成另一个流，然后把所有流连接成一个流
    */
    
   /**
    * 排序
    * sorted()自然排序
    * sorted(Comparator com)定制排序
    */
   ```

3. 终止操作

   ```
   /**
    * Stream的终止操作：
    * allMatch 检查是否匹配所有元素
    * anyMatch 检查是否至少匹配一个元素
    * noneMatch 检查是否没有匹配所有元素
    * findFirst 返回第一个元素
    * findAny 返回当前流中的任意元素
    * count 返回流中的元素个数
    * max 返回流中的最大值
    * min 返回流中的最小值
    */
   ```

**多个中间操作可以连接起来形成一个流水线，除非流水线上触发终止操作，否则中间操作不会执行任何的处理！而在终止操作时一次性全部处理，称为“惰性求值”。内部迭代**

## 2.2 并行流和顺序流

Fork/Join框架

将大任务进行拆分(fork)成若干个小任务（不可拆分时），再将一个个的小任务的运算结果进行join汇总

![](http://mycsdnblog.work/201919121632-v.png)

采用“工作窃取”模式：

当执行新的任务时它可以将其拆分成更小的任务执行，并将小任务加到线程队列中，然后再从一个随机线程的队列中偷一个并把它放到自己的队列中

相对于一般的线程池实现，fork/join框架的优势体现在对其中包含的任务的处理方式上，在一般的线程池中，如果一个线程正在执行的任务由于某些原因无法继续运行，那么该线程会处于等待状态，而在fork/join框架实现中，如果某个子问题由于等待另一个子问题的完成而无法继续执行，那么处理该子问题的线程会主动寻找其它尚未运行的子问题来执行。这种方式减少了线程等待的时间，提高了性能。

Java8中将并行进行了优化，我们可以很容易的对数据进行并行操作。Stream API可以声明性地通过parrallel()与sequential()在并行流与顺序流之间进行切换。

ForkJoin框架采用的是“工作窃取模式”，传统线程在处理任务时，假设有一个大任务被分解成了20个小任务，并由四个线程A,B,C,D处理，理论上来讲一个线程处理5个任务，每个线程的任务都放在一个队列中，当B,C,D的任务都处理完了，而A因为某些原因阻塞在了第二个小任务上，那么B,C,D都需要等待A处理完成，此时A处理完第二个任务后还有三个任务需要处理，可想而知，这样CPU的利用率很低。而ForkJoin采取的模式是，当B,C,D都处理完了，而A还阻塞在第二个任务时，B会从A的任务队列的末尾偷取一个任务过来自己处理，C和D也会从A的任务队列的末尾偷一个任务，这样就相当于B,C,D额外帮A分担了一些任务，提高了CPU的利用率。

2、ForkJoin框架的使用方式，

- 编写计算类继承RecursiveTask<T>接口并重写T compute方法；
- 使用fork方法拆分任务，join合并计算结果；
- 使用ForkJoinPool调用invoke方法来执行一个任务。

