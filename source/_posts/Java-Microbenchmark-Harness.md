---
title: Java Microbenchmark Harness微基准拾遗
date: 2020-07-07 18:24:35
tags:
	- Java
	- 基准测试
	- 系统调优
categories:
    - 系统调优
---

> **If you cannot measure it, you cannot improve it.	–Lord Kelvin**

[Java Microbenchmark Harness](http://openjdk.java.net/projects/code-tools/jmh/) 是专门进行代码的微基准测试的一套工具API。 为应用提供了数据支持，是评价和比较方法好坏的基准。一般说JMH，是在 **Method 层面上的 Benchmark**，精度可以精确到微秒级。以下记录JMH下的一些东西，便于之后查找和学习。

Benchmark 作为应用框架，产品的基准画像，存在统一的标准，避免了不同测评对象自说自话的尴尬，应用框架各自使用有利于自身场景的测评方式必然不可取。

<!-- more -->

## Hello JHM

### Maven依赖

在项目中使用Maven,只需要添加如下依赖：

```xml
<!-- JMH-->
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>${jmh.version}</version>
</dependency>
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>${jmh.version}</version>
    <scope>provided</scope>
</dependency>
```

### 性能测试设计

测试比较Spring和StringBuilder的完成字符串拼接的性能

```java
/**
 * 比较字符串直接相加和StringBuilder的效率
 *
 * @author Leo Wang
 * @version 1.0
 * @date 2020/7/7 16:44
 */
@BenchmarkMode(Mode.Throughput)
@Warmup(iterations = 1,time = 1, timeUnit = TimeUnit.SECONDS)
@Measurement(iterations = 10,time = 10,timeUnit = TimeUnit.SECONDS)
@Threads(8)
@Fork(2)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
@State(Scope.Thread)
public class StringBuilderBenchmark {
    @Benchmark
    public void testStringAdd() {
        String a = "";
        for (int i = 0; i < 10; i++) {
            a += i;
        }
        print(a);
    }

    @Benchmark
    public void testStringBuilderAdd() {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 10; i++) {
            sb.append(i);
        }
        print(sb.toString());
    }
  
    private void print(String a) {
    }
```

### 性能测试执行

对于JMH来说，其执行方式主要有两种

#### 直接IDE运行

对于体量小的测试，可以直接在IDE中完成相关的测试。如上的测试来说，可以直接运行，然后查看相关结果，执行的结果的Main函数如下，创建*Options*对象，传入需要执行的测试和测试报告的输出地址。直接执行Main方法

```java
    public static void main(String[] args) throws RunnerException {
        String userDirPath = System.getProperty("user.dir");
        String benchmarkLogPath = String.format("%s/%s",userDirPath,"/StringBenchmark.log");
        Options options = new OptionsBuilder()
                .include(StringBuilderBenchmark.class.getSimpleName())
                .output(benchmarkLogPath)
                .build();
        new Runner(options).run();
    }
```

在使用IDE进行测试时，需要注意不能使用**Dubug**模式启动，否则不能正常完成测试。

#### 打包成Jar,其他机器上执行

一般对于大型的测试，需要测试时间比较久，线程比较多，就需要去写好了丢到远端的Linux系统环境中里执行， 不然会在本机执行很久并且需要的性能需求可能达不到测试需求。

```bash
mvn clean package
java -jar StringBuilderBenchmark.jar
```

### 测试结果

当正常跑完项目测试以后，JHM会在指定的文件夹下输出一下的测试结果

```shell
# JMH version: 1.23
# VM version: JDK 1.8.0_251, Java HotSpot(TM) 64-Bit Server VM, 25.251-b08
# VM invoker: /Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/jre/bin/java
# VM options: -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=63118:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8
# Warmup: 1 iterations, 1 s each
# Measurement: 10 iterations, 10 s each
# Timeout: 10 min per iteration
# Threads: 8 threads, will synchronize iterations
# Benchmark mode: Throughput, ops/time
# Benchmark: com.upuphub.lake.skylake.benchmark.StringBuilderBenchmark.testStringAdd

# Run progress: 0.00% complete, ETA 00:06:44
# Fork: 1 of 2
# Warmup Iteration   1: 9014.340 ops/ms
Iteration   1: 21302.297 ops/ms
Iteration   2: 21807.763 ops/ms
Iteration   3: 21812.419 ops/ms
Iteration   4: 21840.912 ops/ms
Iteration   5: 21985.020 ops/ms
Iteration   6: 22066.751 ops/ms
Iteration   7: 22006.021 ops/ms
Iteration   8: 19239.509 ops/ms
Iteration   9: 10515.274 ops/ms
Iteration  10: 11758.987 ops/ms

# Run progress: 25.00% complete, ETA 00:05:21
# Fork: 2 of 2
# Warmup Iteration   1: 5273.829 ops/ms
Iteration   1: 18880.356 ops/ms
Iteration   2: 22225.847 ops/ms
Iteration   3: 22017.665 ops/ms
Iteration   4: 22036.969 ops/ms
Iteration   5: 22080.422 ops/ms
Iteration   6: 22262.118 ops/ms
Iteration   7: 22153.187 ops/ms
Iteration   8: 22105.884 ops/ms
Iteration   9: 21613.504 ops/ms
Iteration  10: 22029.923 ops/ms


Result "com.upuphub.lake.skylake.benchmark.StringBuilderBenchmark.testStringAdd":
  20587.041 ±(99.9%) 2921.754 ops/ms [Average]
  (min, avg, max) = (10515.274, 20587.041, 22262.118), stdev = 3364.697
  CI (99.9%): [17665.287, 23508.796] (assumes normal distribution)


# JMH version: 1.23
# VM version: JDK 1.8.0_251, Java HotSpot(TM) 64-Bit Server VM, 25.251-b08
# VM invoker: /Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/jre/bin/java
# VM options: -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=63118:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8
# Warmup: 1 iterations, 1 s each
# Measurement: 10 iterations, 10 s each
# Timeout: 10 min per iteration
# Threads: 8 threads, will synchronize iterations
# Benchmark mode: Throughput, ops/time
# Benchmark: com.upuphub.lake.skylake.benchmark.StringBuilderBenchmark.testStringBuilderAdd

# Run progress: 50.00% complete, ETA 00:03:34
# Fork: 1 of 2
# Warmup Iteration   1: 50084.373 ops/ms
Iteration   1: 67510.457 ops/ms
Iteration   2: 42202.643 ops/ms
Iteration   3: 41633.858 ops/ms
Iteration   4: 43352.405 ops/ms
Iteration   5: 43748.063 ops/ms
Iteration   6: 45176.476 ops/ms
Iteration   7: 44649.922 ops/ms
Iteration   8: 40872.340 ops/ms
Iteration   9: 40520.724 ops/ms
Iteration  10: 38853.095 ops/ms

# Run progress: 75.00% complete, ETA 00:01:47
# Fork: 2 of 2
# Warmup Iteration   1: 45279.748 ops/ms
Iteration   1: 71985.226 ops/ms
Iteration   2: 43291.826 ops/ms
Iteration   3: 44149.181 ops/ms
Iteration   4: 43297.043 ops/ms
Iteration   5: 40614.460 ops/ms
Iteration   6: 40444.594 ops/ms
Iteration   7: 40912.490 ops/ms
Iteration   8: 41428.454 ops/ms
Iteration   9: 43022.557 ops/ms
Iteration  10: 43368.455 ops/ms


Result "com.upuphub.lake.skylake.benchmark.StringBuilderBenchmark.testStringBuilderAdd":
  45051.713 ±(99.9%) 7496.158 ops/ms [Average]
  (min, avg, max) = (38853.095, 45051.713, 71985.226), stdev = 8632.587
  CI (99.9%): [37555.555, 52547.872] (assumes normal distribution)


# Run complete. Total time: 00:07:08

REMEMBER: The numbers below are just data. To gain reusable insights, you need to follow up on
why the numbers are the way they are. Use profilers (see -prof, -lprof), design factorial
experiments, perform baseline and negative tests that provide experimental control, make sure
the benchmarking environment is safe on JVM/OS/HW level, ask for reviews from the domain experts.
Do not assume the numbers tell you what you want them to tell.

Benchmark                                     Mode  Cnt      Score      Error   Units
StringBuilderBenchmark.testStringAdd         thrpt   20  20587.041 ± 2921.754  ops/ms
StringBuilderBenchmark.testStringBuilderAdd  thrpt   20  45051.713 ± 7496.158  ops/ms
```

整个测试报告由三个部分组成，首先分别是**testStringAdd**的测试结果然后是**testStringBuilderAdd**的测试结果，最后时两个测试结果之间的结果汇总和对应的比较。前两个部分的结果是类似的，会列出测试环境的一些基本信息，包括JHM的版本、虚拟机版本和相关一些配置等的信息以及测试的一些配置和设置，然后就是预热迭代执行（Warmup Iteration）， 然后是正常的迭代执行（Iteration），最后是结果（Result）的信息输出。一般来说最关注第三部分，也就是汇总结果。

> Tips: 对于汇总结果部分的输出,Error是没有数据的，这里是Score过长挤过去的

可以看出StringBuilder在做字符串拼接的速度比String的直接评价速度好两倍以上。

## JHM的注解和功能

### *@BenchmarkMode*

基准测试类型。这里选择的是Throughput也就是吞吐量。吞吐量会得到单位时间内可以进行的操作数。

- Throughput: 整体吞吐量，例如“1秒内可以执行多少次调用”。
- AverageTime: 调用的平均时间，例如“每次调用平均耗时xxx毫秒”。
- SampleTime: 随机取样，最后输出取样结果的分布，例如“99%的调用在xxx毫秒以内，99.99%的调用在xxx毫秒以内”
- SingleShotTime: 以上模式都是默认一次 iteration 是 1s，唯有 SingleShotTime 是只运行一次。往往同时把 warmup 次数设为0，用于测试冷启动时的性能。
- All(“all”, “All benchmark modes”): 执行所有模式。

### *@Warmup*

在进行基准测试前需要进行预热。一般前几次进行程序测试的时候都会比较慢， 所以要让程序进行几轮预热，保证测试的准确性。其中的参数iterations就是预热轮数。

> Tips: 因为 JVM 的 JIT 机制的存在，如果某个函数被调用多次之后，JVM 会尝试将其编译成为机器码从而提高执行速度。所以为了让 benchmark 的结果更加接近真实情况就需要进行预热

### *@Measurement*

度量，一些基本的测试参数。

1. iterations 进行测试的轮次
2. time 每轮进行的时长
3. timeUnit 时长单位

可以根据具体情况调整。一般比较重的东西可以进行大量的测试，放到服务器上运行。

### *@Threads*

每个进程中的测试线程，根据具体情况选择，一般为cpu乘以2。

### *@Fork*

进行 fork 的次数。如果 fork 数是2的话，则 JMH 会 fork 出两个进程来进行测试。

### *@OutputTimeUnit*

基准测试结果的时间类型。一般选择秒、毫秒、微秒。

### *@Benchmark*

方法级注解，表示该方法是需要进行 benchmark ，用法和 JUnit 的 @Test 类似。

### *@Param*

属性级注解，@Param 用来指定某项参数的多种情况。适合用来测试一个函数在不同的参数输入的情况下的性能。

### *@Setup*

方法级注解，需要在测试之前进行一些准备工作，比如对一些数据的初始化。

### *@TearDown*

方法级注解，在测试之后进行一些结束工作，比如关闭线程池，数据库连接等的，主要用于资源的回收等。

### *@State*

当使用@Setup参数的时候，必须在类上加这个参数，不然会提示无法运行。

State 用于声明某个类是一个“状态”，然后接受一个 Scope 参数用来表示该状态的共享范围。 很多 benchmark 会需要一些表示状态的类，JMH 允许你把这些类以依赖注入的方式注入到 benchmark 函数里。Scope 主要分为三种。

1. Thread: 该状态为每个线程独享。
2. Group: 该状态为同一个组里面所有线程共享。
3. Benchmark: 该状态在所有线程间共享。

## 补充说明

大地撒娇

---

### 参考和来源

* [Java微基准测试框架JMH](https://www.xncoding.com/2018/01/07/java/jmh.html)
* [Java使用JMH进行简单的基准测试Benchmark](http://irfen.me/java-jmh-simple-microbenchmark/)
* [Java 并发编程笔记：JMH 性能测试框架](http://blog.dyngr.com/blog/2016/10/29/introduction-of-jmh/)
* [JMH - Java Microbenchmark Harness](http://tutorials.jenkov.com/java-performance/jmh.html)

