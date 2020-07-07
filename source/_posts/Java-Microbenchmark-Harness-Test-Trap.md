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

在某些情况下，编译器可以判断出某些代码根本不影响输出，所以编译器会消除这些代码,例如注释的代码，不可达的代码块，可达但不被使用的代码，都会被判断为死码而被Javac的时候被消除。

```java
public class ErrorBenchmark {
    private double PI = Math.PI;
  
    @Benchmark
    public void benchmarkNothing(){
        //do nothing
    }
  
    @Benchmark
    public void benchmarkWrong(){
        Math.log(PI);
    }

    @Benchmark
    public double benchmarkRight(){
        return Math.log(PI);
    }

}
```



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
* [JAVA 拾遗 — JMH 与 8 个测试陷阱](cnkirito.moe/java-jmh/)

