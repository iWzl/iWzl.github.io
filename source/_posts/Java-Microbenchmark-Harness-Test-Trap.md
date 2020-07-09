---
title: Java Microbenchmark Harness微基准测试陷阱
date: 2020-07-09 22:15:59
tags:
	- Java
	- 基准测试
	- 系统调优
	- 误区和陷阱
	- JMH
categories:
    - 系统调优
---

> **天下之事，闻者不如见者知之为详，见者不如居者知之为尽。——宋-陆游**
>
> **你要知道梨子的滋味，就要亲口尝一下。——毛泽东**

俗话说，没有实践就没有发言权，自己实践才能知之为尽，Benchmark 为应用提供了数据支持，是评价和比较方法好坏的基准，Benchmark 的准确性，多样性便显得尤为重要。JMH为Java方法的基准水平提供了很好的量化标准，也为剖析系统性能和技术选型提供的更为有效切可靠的实现手段。

以下记录在使用Java Microbenchmark Harness做微基准测试容易遇见的误区和相关陷阱，便于之后的查找和说明。

<!-- more -->

## 代码预热

> **Warmup = waiting for the transient responses to settle down**

随着JVM虚拟机的优化，JIT的存在，代码的执行往往前期执行结果没有后期的执行结果好。 `Benchmark` 产生更可靠的结果的原因是，它只度量稳定状态下`方法任务`的执行时间，而不理会最初的性能。大多数 Java 实现具有复杂的性能生命周期。一般来说，最初的性能往往相当低，然后性能显著提高（常常出现几次性能跃升），直到到达稳定状态。

### 类装载

JVM 通常只在类的第一次使用类时装载它们。所以,方法任务的第一次执行时间包含装载它使用的所有类的时间（如果这些类还没有装载的话）。因为类装载往往涉及磁盘 I/O、解析和检验，这会显著增加`方法任务`的第一次执行时间。**常常**可以通过多次执行`方法任务`来消除这种影响。

> PS:  **常常**而不是**总是**，这是因为 `方法任务` 可能具有复杂的分支行为，这可能导致它在任何给定的执行过程中并不使用所有可能用到的类。幸运的是，如果执行任务足够多次，就可能经历所有分支，因此很快就会装载所有相关类）。

如果使用定制的类装载器，就有另一个问题：JVM 可能认为一些类已经成了垃圾，因此决定卸载它。这不太可能严重影响性能，但是仍然会使基准测试结果产生偏差。

### 混合模式

在执行即时（Just-in-time，JIT）编译之前，现代的 JVM 通常会运行代码一段时间（常常是纯解释式运行），从而收集剖析信息。这对基准测试的影响在于，任务可能需要执行许多次，才能达到稳定状态。

一般来说对稳定状态下的基准测试至少需要以下步骤：

1. 执行 `方法任务` 一次，以便装载所有类。
2. 执行 `方法任务` 足够多次，以确保出现稳定状态的执行数据。

## 陷阱一: 死码消除

[死码消除]([https://zh.wikipedia.org/zh-cn/%E6%AD%BB%E7%A2%BC%E5%88%AA%E9%99%A4](https://zh.wikipedia.org/zh-cn/死碼刪除))(Dead code elimination）是一种编译最优化技术,在某些情况下，编译器可以判断出某些代码根本不影响输出，所以编译器会消除这些代码,例如注释的代码，不可达的代码块，可达但不被使用的代码，会被判断为死码而在Javac的时候被消除。

```java
public class ErrorBenchmark {
    private double PI = Math.PI;
  
    @Benchmark
    public void benchmarkNothing(){
        // 19873732.412 ± 6783114.266  ops/ms
        //do nothing 
    }
  
    @Benchmark
    public void benchmarkWrong(){
        //  20988076.131 ± 7282548.202  ops/ms
        Math.log(PI);  // DCE 会被判断为死码而被消除
    }

    @Benchmark
    public double benchmarkRight(){
       // 306740.041 ±   52692.696  ops/ms
        return Math.log(PI);
    }

}
```

在做测试时，需要注意方法会不会有死码的存在，否者可能会带来一些不合理的测试结果和意外。对于会被判断为死码但又需要进行执行测试方法来说，可以想办法去除孤立的方法执行，例如增加方法返回值，或者使用JMH的提供的API**Blackhole**。

```java
@Benchmark
public void benchmarkRight(Blackhole bh) {
    bh.consume(Math.log(PI));
}
```

## 陷阱二：常量折叠与常量传播

[常数折叠]([https://zh.wikipedia.org/wiki/%E5%B8%B8%E6%95%B8%E6%8A%98%E7%96%8A#%E5%B8%B8%E6%95%B8%E5%82%B3%E6%92%AD](https://zh.wikipedia.org/wiki/常數折疊#常數傳播))（Constant folding）以及常数传播（constant propagation）都是[编译器最佳化](https://zh.wikipedia.org/w/index.php?title=編譯器最佳化&action=edit&redlink=1)技术。是一个在编译时期简化常数的一个过程。常数在表示式中仅仅代表一个简单的数值，就算一个变数从未被修改也可作为常数，或者直接将一个变数被明确地被标注为常数。

```java
long number = 2 * 600 * 200;
```

多数的现代编译器不会真的产生两个乘法的指令再将结果储存下来，取而代之的，他们会辨识出语句的结构，并在编译时期将数值直接计算出来。常数折叠的时机取决于编译器，有的在编译前期完成，有的在较后期进行。

```java
private double x = Math.PI;

// 编译器会对 final 变量特殊处理 
private final double wrongX = Math.PI;

@Benchmark
public double baseline() {
    // 2.220 ± 0.352 ns/op
    return Math.PI;
}

@Benchmark
public double measureWrong_1() { 
    // 2.220 ± 0.352 ns/op
    // 错误，结果可以被预测，会发生常量折叠
    return Math.log(Math.PI);
}

@Benchmark
public double measureWrong_2() { 
    // 2.220 ± 0.352 ns/op
    // 错误，结果可以被预测，会发生常量折叠
    return Math.log(wrongX);
}

@Benchmark
public double measureRight() { 
    // 22.590 ± 2.636  ns/op
    return Math.log(x);
}
```

由于发生了常量折叠，相同实现下的执行销量完全不同，这个测试在一定程度上说明了final的定义对于方法执行结果的影响。

此外**常数传播 (**Constant propagation**)** 是一个替代表示式中已知常数的过程，也是在编译时期进行，包含前述所定义，内建函数也适用于常数。

```java
int x = 520;
int y = 260 - 520 / 2;
return y * (1314 / x + 2);
```

传播可以理解变量的替换，如果进行持续传播，上式则可写成如下

```java
int x = 14;
int y = 0;
return 0;
```

## 陷阱三：不要在测试中写循环





## 为什么要使用JMH

对于传统的接口调用测试，可能会使用以下的方式测试

```java
long startTime = System.currentTimeMillis();
benchmarkTestMethod();
System.out.println(System.currentTimeMillis()-startTime);
```

而JMH则使用

```java
@Benchmark
public void benchmarkTestMethod(){
  // do 
}
```

---

## 参考和引用

* [IBM Developer Java 代码基准测试的问题](https://www.ibm.com/developerworks/cn/java/j-benchmark1.html)
* [JAVA 拾遗 — JMH 与 8 个测试陷阱](https://www.cnkirito.moe/java-jmh/)

