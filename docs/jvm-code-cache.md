# JVM 代码缓存简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jvm-code-cache>

## 1。简介

在本教程中，我们将快速浏览并了解 JVM 的代码[缓存内存](/web/20221006220648/https://www.baeldung.com/cs/cache-memory)。

## 2。什么是代码缓存？

简单来说， **JVM 代码缓存是 JVM 存储其编译成本机代码**的字节码的区域。我们将可执行本地代码的每个块称为`nmethod`。`nmethod `可能是一个完整的或者内嵌的 Java 方法。

实时(JIT)编译器是代码缓存区域的最大消费者。这就是为什么有些开发人员称这种内存为 JIT 代码缓存。

## 3。代码缓存调整

**代码缓存有固定的大小**。一旦写满，JVM 将不会编译任何额外的代码，因为 JIT 编译器现在是关闭的。此外，我们将收到`“CodeCache is full… The compiler has been disabled`警告消息。结果，我们的应用程序最终会出现性能下降。为了避免这种情况，我们可以使用以下大小选项来调整代码缓存:

*   `InitialCodeCacheSize`–初始代码缓存大小，默认为 160K
*   **`ReservedCodeCacheSize`–默认最大大小为 48MB**
*   `CodeCacheExpansionSize`–代码缓存的扩展大小，32KB 或 64KB

增加`ReservedCodeCacheSize`可能是一个解决方案，但这通常只是一个临时的解决方法。

幸运的是，JVM 提供了一个 **`UseCodeCacheFlushing`选项来控制代码缓存区**的刷新。其默认值为 false。当我们启用它时，它**在满足以下条件时释放被占用的区域:**

*   代码缓存已满；**如果该区域的大小超过某个阈值，则刷新该区域**
*   自上次清理以来，经过了特定的时间间隔
*   预编译代码不够热。对于每个编译过的方法，JVM 跟踪一个特殊的热度计数器。如果这个计数器的值小于计算出的阈值，JVM 就会释放这段预编译代码

## 4。代码缓存使用情况

为了监控代码缓存的使用情况，我们需要跟踪当前使用的内存大小。

**为了获得代码缓存使用的信息，我们可以指定`–XX:+PrintCodeCache` JVM 选项**。运行我们的应用程序后，我们将看到类似的输出:

```
CodeCache: size=32768Kb used=542Kb max_used=542Kb free=32226Kb 
```

让我们看看这些值的含义:

*   输出中的`size `显示内存的最大大小，与`ReservedCodeCacheSize`相同
*   `used`是当前正在使用的内存的实际大小
*   `max_used`是已经使用的最大尺寸
*   `free`是尚未被占用的剩余内存

`PrintCodeCache` 选项非常有用，因为我们可以:

*   看看什么时候冲水
*   确定我们是否达到了关键的内存使用点

## 5.分段代码缓存

从 [Java 9](https://web.archive.org/web/20221006220648/https://openjdk.java.net/jeps/197) 开始，JVM 将代码缓存分成三个不同的部分，每个部分包含一种特定类型的编译代码。更具体地说，有三个部分:

*   非方法段包含 JVM 内部相关代码，比如字节码解释器。默认情况下，该段大约为 5 MB。此外，还可以通过`-XX:NonNMethodCodeHeapSize `调谐标志配置段大小
*   分析代码段包含轻度优化的代码，其生存期可能很短。尽管默认情况下段的大小约为 122 MB，但我们可以通过`-XX:ProfiledCodeHeapSize `调优标志来更改它
*   非概要分析段包含完全优化的代码，具有潜在的长生存期。同样，默认情况下大约是 122 MB。当然，该值可通过`-XX:NonProfiledCodeHeapSize `调谐标志进行配置

这种新结构对不同类型的编译代码进行了不同的处理，从而带来了更好的整体性能。

例如，将短期编译代码与长期代码分开可以提高方法清理器的性能，这主要是因为它需要扫描的内存区域更小。

## 6。结论

这篇短文简要介绍了 JVM 代码缓存。

此外，我们还提供了一些使用和调优选项来监控和诊断这个内存区域。