# Rust 如何让 Rayon 的数据并行变得神奇

> 原文：<https://developers.redhat.com/blog/2021/04/30/how-rust-makes-rayons-data-parallelism-magical>

Rayon 是一个用于 Rust 编程语言的数据并行库。开始使用 Rayon 的程序员的普遍反应表达了它是多么神奇:“我修改了一行代码，现在我的代码可以并行运行了！”作为 Rayon 的作者之一，我当然很高兴看到快乐的用户，但我想消除一些魔力，并给予它应有的信任——让 Rust 自己。

## Rust 如何支持人造丝的平行性

最好的“神奇”场景通常是这样的，带有顺序迭代器:

```
let total = foo_vector.iter_mut()
    .filter(|foo| foo.is_interesting())
    .map(|foo| foo.heavy_computation())
    .sum();
```

要用 Rayon 使它成为一个并行迭代器，只需修改第一行来调用`par_iter_mut()`，如下例所示，并观察它点亮所有的 CPU！没有问题的是，`foo_vector`是堆栈上的局部变量，或者计算可能会改变值。整个集合在多个线程中自动拆分，独立处理更新，不会出现问题:

```
let total = foo_vector.par_iter_mut()
    .filter(|foo| foo.is_interesting())
    .map(|foo| foo.heavy_computation())
    .sum();
```

作为一个 Rust 开发者，仅仅去快是不够的。还必须检查所有内容的内存和线程安全性。人造丝也是如此。假设顺序迭代器被写成这样:

```
let mut total = 0;
foo_vector.iter_mut()
    .filter(|foo| foo.is_interesting())
    .for_each(|foo| total += foo.heavy_computation());
```

这段代码按顺序是没问题的，但是如果你试图让它与`par_iter_mut()`并行，编译器将返回一个错误:

```
cannot assign to `total`, as it is a captured variable in a `Fn` closure

```

如果多个线程试图同时更新`total`，就会出现数据竞争。您可以通过对`total`使用原子类型来解决这个问题，或者通过使用 Rayon 的内置并行`sum()`或您自己在迭代器上定制的 fold+reduce 来解决这个问题。

但是 Rayon 是一个普通的库，没有任何编译器集成，所以它怎么知道有数据竞争呢？神奇之处在于 Rust 语言本身，它来自表达正确的通用约束的 Rayon。

## 《铁锈》中的所有权和借款

Rust 对于任何类型的值的所有权都有很强的语义。编译器的静态借用检查器跟踪一个值在哪里被借用作为引用，或者作为`&mut T`或者`&T`。当一个价值被拥有而没有任何借贷时，拥有者可以用这个价值做任何他想做的事情。借用值作为`&mut T`是*独占访问*，只有借用者可以对该值做任何事情——但即使是借用者也必须让底层的`T`处于有效状态。借用值为`&T`是*不可变共享访问*，借用者和拥有者都只能读取值。借用检查器在静态确定的代码区域(生存期)中强制执行所有这些操作，其中只有未借用的`T`所有者或独占的`&mut T`借用者才被允许进行任何修改。唯一一次可以修改`&T`的是基于`UnsafeCell`构建的类型，比如 atomics 或`Mutex`，它们增加了运行时同步。

我推荐阅读 [Rust:一个独特的视角](https://limpet.net/mbrubeck/2019/02/07/rust-a-unique-perspective.html)来了解更多关于这个话题的信息。

## 线程安全特征

有两个自动特征控制 Rust 的所有线程安全: [`Send`](https://doc.rust-lang.org/stable/std/marker/trait.Send.html) 和 [`Sync`](https://doc.rust-lang.org/stable/std/marker/trait.Sync.html) 。自动化意味着这些特征是根据它们的组成推断出来的:只有当一个结构的所有字段都实现了`Send`或`Sync`时，这个结构才会实现它们。如果某个字段没有，比如一个原始指针，那么特征就不适用于该结构，除非一个特征被添加了一个`unsafe impl`声明，在该声明中作者断言线程安全将以某种其他方式得到维护(存在未定义行为的风险)。

`Send`意味着`T`可以将控制权转移到另一个线程。对于拥有的值，这仅仅意味着值可以完全移动，原始线程不再有访问权。但是我们也可以寄推荐信。对于唯一的`&mut T`，如果满足`T: Send`，就可以发送引用，将唯一的借位传递给另一个线程。对于共享的`&T`，只能基于另一个特征`T: Sync`发送引用，这表明`T`可以安全地与另一个线程共享。

Rustonomicon 有一个关于[发送和同步](https://doc.rust-lang.org/stable/nomicon/send-and-sync.html)的详细章节。

## 功能特征

与函数调用方式相关的特性有三种:[`FnOnce`](https://doc.rust-lang.org/stable/std/ops/trait.FnOnce.html)[`FnMut`](https://doc.rust-lang.org/stable/std/ops/trait.FnMut.html)[`Fn`](https://doc.rust-lang.org/stable/std/ops/trait.Fn.html)。普通函数自动实现所有三个特征，但是闭包实现这些特征取决于闭包如何使用它们捕获的状态。如果一个闭包要移动或消耗其状态的任何部分，它只实现通过值用`self`调用的`FnOnce`，因为它没有剩余的状态来第二次移动或消耗。如果一个闭包修改了它的状态，它实现了用`&mut self`调用的`FnMut`，也可以作为`FnOnce`调用。如果一个闭包只是读取它的状态，或者像普通函数一样根本没有状态，它就实现用`&self`调用的`Fn`，以及`FnMut`和`FnOnce`。

博客文章 [Closures: Magic Functions](https://rustyyato.github.io/rust/syntactic/sugar/2019/01/17/Closures-Magic-Functions.html) 详细介绍了这个实现。

## 人造丝的一般限制

有了 Rust 语言中这些强大的工具，Rayon 只需要指定它的约束。并行迭代器及其项必须实现`Send`，因为它们将在线程间发送。迭代器方法如 [`filter`](https://docs.rs/rayon/1.5.0/rayon/iter/trait.ParallelIterator.html#method.filter) 、 [`map`](https://docs.rs/rayon/1.5.0/rayon/iter/trait.ParallelIterator.html#method.map) 、`[for_each](https://docs.rs/rayon/1.5.0/rayon/iter/trait.ParallelIterator.html#method.for_each)`对它们的回调函数/闭包`F`有更多的约束:

*   它必须实现`Send`，这样你才能把它发送到线程池。
*   它必须实现`Sync`，这样你就可以在多个线程间共享对那个回调的`&F`引用。
*   它必须实现`Fn`,这样就可以通过共享引用从任何线程中调用它。

因此人造丝要求`F: Send + Sync + Fn`。

让我们再来看看编译器会拒绝的例子:

```
let mut total = 0;
foo_vector.par_iter_mut()
    .filter(|foo| foo.is_interesting())
    .for_each(|foo| total += foo.heavy_computation());
```

我们将理所当然地认为这个向量中的类型`Foo`实现了`Send`，所以在线程间发送`&mut Foo`引用作为并行迭代器项也是非常好的。`for_each`闭包会捕获对累积的`total`变量的可变引用。假设这个数字有一个简单的类型，比如`i32`，那么用`&mut i32`对另一个线程进行`Send`闭包是可以接受的。使用`Sync`甚至可以在线程间共享对闭包的引用。然而，变异会使它成为`FnMut`，要求`&mut self`实际调用它。编译器的错误现在应该有意义了:

```
cannot assign to `total`, as it is a captured variable in a `Fn` closure

```

因为 Rayon 在这里需要`Fn`，它被编译器锁定，不允许你做任何使它成为`FnMut`的事情。如果您将总数更改为诸如`AtomicI32`这样的类型，它允许使用共享引用进行更新，代码将会编译并正常工作:

```
let mut total = AtomicI32::new(0);
foo_vector.par_iter_mut()
    .filter(|foo| foo.is_interesting())
    .for_each(|foo| total.fetch_add(foo.heavy_computation(), Ordering::Relaxed));
```

这就是 Rust 的[无畏并发](https://doc.rust-lang.org/book/ch16-00-concurrency.html)(或并行)的影响——不是说你永远不会在线程代码中编写 bug，而是说编译器会在 bug 伤害你之前抓住它们。Rayon 可以让您非常容易地接触到并行，感觉几乎是不可思议的，但事实上 Rayon 对您的代码一无所知:它只是指定简单的约束，并让 Rust 编译器做证明它的艰苦工作。

更多关于 Rust 的文章，请访问[红帽的专题页面](https://developers.redhat.com/blog/category/rust/)。

*Last updated: October 14, 2022*