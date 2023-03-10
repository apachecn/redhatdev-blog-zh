# 谢南多厄气相色谱在 JDK 14，第 1 部分:自固定屏障

> 原文：<https://developers.redhat.com/blog/2020/03/04/shenandoah-gc-in-jdk-14-part-1-self-fixing-barriers>

在即将到来的 [JDK 14](https://openjdk.java.net/projects/jdk/14/) 中，谢南多厄垃圾收集器 (GC)的开发已经有了显著的改进。这里介绍的第一种方法(自修复屏障)旨在减少屏障中速和慢速路径中的本地延迟。第二部分将介绍并发根处理和并发类卸载。

## 自固定屏障

自固定护栏的改进建立在 JDK 13 号使用的[负载参考护栏](https://developers.redhat.com/blog/2019/06/27/shenandoah-gc-in-jdk-13-part-1-load-reference-barriers/)的基础上。在从引用字段或数组元素加载之后，在将加载的对象给予应用程序代码的其余部分之前，使用加载引用屏障。在伪代码中，屏障看起来像这样:

```
T load_reference_barrier(T* addr) {
  T obj = *addr;

  // Fast-path: accesses thread-local flag
  if (!is_gc_active()) return obj;

  // Mid-path 1: accesses small bitmap (byte per region, handful of KBs?)
  if (!is_in_collection_set(obj)) return obj;

  // Mid-path 2: accesses fwdptrs in object (entire heap?)
  T fwd = resolve_forwardee(obj);
  if (obj != fwd) return fwd;

  // Slow-path: call to runtime, once per location with non-forwarded object
  return load_reference_barrier_slowpath(obj);
}

```

结果是，每当我们在 GC 处于活动状态时加载一个对象，并且如果该对象是对收集组的引用，我们就开始解析该对象，并且可能进入运行时的较慢部分来进行实际的撤离。有可能对象会被 GC 自己清空，屏障会在路径 2 中途检查时发现这一点，然后从那里返回。在最坏的情况下，屏障会调用运行时并完成所有的事情。

这个故事中对性能敏感的部分是，我们将一直在中间路径 2 检查重定位的对象，直到 GC 周期结束，GC 更新感兴趣的引用。如果我们在热循环中进行访问，那么在 GC 运行时，我们总是会深入到屏障中。并且中间路径 2 检查是昂贵的，因为它到达很远很广。

自我修复障碍背后的想法是，当我们解析了对象并发现了转发的副本时，我们也可以就在那里更新位置。因为我们正在更新对收集组中对象副本*而不是*的引用，所以在下一次 barrier 调用时，我们将从中间路径 1 退出。

这是这些变化后屏障的样子:

```
T load_reference_barrier(T* addr) {
  T obj = *addr;

  // Fast-path: accesses thread-local flag
  if (!is_gc_active()) return obj;

  // Mid-path 1: accesses small bitmap (byte per region, handful of KBs?)
  if (!is_in_collection_set(obj)) return obj;

  // Mid-path 2: accesses fwdptrs in objects (entire heap?)
  T fwd = resolve_forwardee(obj);

  if (obj != fwd) {
    // Can do the update here
    CAS(addr, fwd, obj);
    return fwd;
  }

  // Slow-path: call to runtime, once per location with non-forwarded object
  fwd = load_reference_barrier_slowpath(obj);
  // Can do the update here
  CAS(addr, fwd, obj);
  return fwd;
}

```

换句话说，一旦我们得到了`forwardee`，我们就进入慢速路径，在这里我们可以将对新副本的引用标记回原始地址。我们使用比较和设置操作来实现这一点，以避免另一个我们不能覆盖的 Java 线程对同一字段的潜在竞争更新。

现在，请注意，对于每个未更新的位置，我们只失败了一次中间路径 2 检查。当我们没有通过这项检查时，我们可以修复它，然后再也不会访问屏障的黑暗部分。考虑到这一点，我们可以通过将整个更新移到慢路径本身来简化障碍:

```
T load_reference_barrier(T* addr) {
  T obj = *addr;

  // Fast-path: accesses thread-local flag
  if (!is_gc_active()) return obj;

  // Mid-path 1: accesses small bitmap (byte per region, handful of KBs?)
  if (!is_in_collection_set(obj)) return obj;

  // Slow-path: call to runtime, once per non-updated location
  return load_reference_barrier_slowpath(obj, addr); // Update is actually here
}

```

现在我们有了更简单的变异体侧屏障。复杂之处在于将`addr`传递到慢路径，这需要摆弄译员、C1 和 C2——这就是为什么一开始不这样做的原因。此外，请注意警告。对于未更新的位置，我们通常在中途路径 2 检查时提前退出。现在，我们为它们输入运行时。虽然这种行为在代码中看起来更糟，但对收集组中的热对象进行多次中间路径 2 检查比在每个位置进入运行时进行一次修复要昂贵得多。

这些机制也减少了 GC 工作人员在更新引用阶段的工作量。

## 结论

希望您现在对 JDK 14 的谢南多厄 GC 版本中的自修复障碍如何帮助减少障碍中间和慢速路径中的局部延迟有了更好的了解。在下一篇文章中，我将介绍并发根处理和并发类卸载。总之，这些特性通过将 GC 工作从暂停阶段转移到并发阶段，减少了 GC 暂停时间，从而减少了全局延迟。

*Last updated: June 29, 2020*