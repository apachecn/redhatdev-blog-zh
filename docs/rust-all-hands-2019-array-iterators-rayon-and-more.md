# Rust All Hands 2019:数组迭代器、Rayon 等等

> 原文：<https://developers.redhat.com/blog/2019/03/22/rust-all-hands-2019-array-iterators-rayon-and-more>

几周前，我有幸参加了 Mozilla 在柏林办公室举办的第二届年度 [Rust](https://www.rust-lang.org/) 全体会议[。与会者包括志愿者和公司员工，涵盖了 Rust 开发的所有领域，包括编译器、语言、库、文档、工具、操作和社区。虽然我确信会有一份会议的官方总结(就像](https://blog.mozilla.org/berlin/rust-all-hands-2019/)[去年的](https://blog.rust-lang.org/2018/04/06/all-hands.html))，但在这篇文章中，我将涉及几件我直接参与的事情。首先，我将介绍一个许多开发人员渴望已久的特性...

## 数组迭代器

惯用的 Rust 代码会大量使用`Iterator`特性，但不幸的是这对数组不起作用。这个特性不仅仅是一个库特性，也是编译器的一部分，因为即使普通的`for`循环也使用`IntoIterator`来转换它的输入，然后在循环的每个周期调用`Iterator::next()`。然而，`[T; N]`数组不通过值来实现`IntoIterator`，所以你能做的最好的事情就是通过引用将它作为一个片来迭代(`iter()`和`iter_mut()`)，或者将你的值放在堆上的`Vec<T>`中，而不是通过值来迭代。也可以使用像`ArrayVec`这样的第三方包装器直接迭代一个数组，但是这个功能应该是标准库中固有的。

```
for s in ["abc", "foo", "xyz"] {} // Nope, can't iterate a simple array by value
for &s in &["abc", "foo", "xyz"] {} // It's OK by reference instead.

let words: Vec<_> = my_string.split_whitespace().flat_map(|word| {
    // Can't return a temporary array for iteration at all
    [word.to_lower(), word.to_upper()]
}).collect();
```

我首先尝试在 [PR32871](https://github.com/rust-lang/rust/pull/32871) 中为数组实现`IntoIterator`。这在技术上并不太难，但是缺少常量泛型使它有点尴尬。我们没有办法为任何 N 长度实现`[T; N]`,所以目前只能为长度 0 到 32 实现特征。更不幸的是，这也出现在迭代器类型签名中。虽然我们希望它像`array::IntoIter<T, N>`一样，但是如果没有 const generics，我只能用一个约束来制作这个`IntoIter<T, A>`，这个约束是`A`必须实现一个内部`FixedSizeArray`特征。libs 团队实在不想致力于此，所以关闭了 PR。

后来 Rust 确实批准了 [RFC2000](https://github.com/rust-lang/rfcs/blob/master/text/2000-const-generics.md) 用于 const 泛型，所以我为数组迭代器开了一个新的 [PR49000](https://github.com/rust-lang/rust/pull/49000) 。(多好听的整数啊！)我们仍然没有实现 const 泛型，但我主张一条前进的道路:在不稳定`array::IntoIter`类型的情况下添加`IntoIterator`，我们可以在不久的将来清理它。这似乎是可以接受的，除了一个更棘手的问题，那就是破坏现有的`.into_iter()`调用的方法解析。目前，因为数组没有直接拥有那个方法，这样的调用将自动引用到一个*片*迭代器中。添加数组`IntoIterator`会将它分解成一个值迭代器。Clippy [PR3344](https://github.com/rust-lang/rust-clippy/pull/3344) 添加了一个 lint，用于处理在这里会被破坏的代码，但是我们仍然要非常小心地进行，因为这些情况似乎很常见。Clippy 甚至不得不在 PR 中用自己的代码修改了几个对`.iter()`的调用！

我费了好大劲才把我们带到全体会议上，我带着一个充满希望的解决方案参加了 libs 会议。我们在 PR49000 快结束时的想法是，我们可以向数组添加一个新的固有方法，保留方法解析的当前行为。编译器优先考虑固有方法而不是特征方法，但是我们仍然可以在`for`循环、`chain`、`flat_map`等中通过值获得数组的新`IntoIterator`的好处。，其中显式调用 trait 方法。我们还可以在这个固有方法上添加一条反对消息，建议使用 slice `iter()`来代替。虽然添加这个固有的方法需要一些编译器语言的帮助，但是 libs 团队对这个方法很感兴趣，我们在全体员工日总结会上宣布了这个成功。

```
// Here's the inherent method. (demonstrating with full const generics)
impl<T, const N: usize> [T; N] {
    #[deprecated(note = "use slice `iter()` to iterate by reference)]
    fn into_iter(&self) -> slice::Iter<'_, T> {
        self.iter()
    }
}

// Now the new trait impl can hide behind the inherent method.
impl<T, const N: usize> IntoIterator for [T; N] {
    type Item = T;
    type IntoIter = IntoIter<T, N>;
    fn into_iter(self) -> Self::IntoIter {
        // ...
    }
}
```

如果您更熟悉 Rust 的方法解析，您可能已经看到了这种方法的问题，这是我第二天意识到的。我们在会议期间用占位符`struct A`类型进行了试验，以确认固有方法的使用，但是我们没有适当地涵盖所有的变化。当我后来扩展这个例子时，我发现不，当直接在一个数组值上调用时，一个固有的`into_iter(&self)`实际上并不优先于`IntoIterator::into_iter(self)`。后者首先被发现，因为方法接收者是一个直接的`self`，而前者需要自动引用来调用`&self`。我们*必须*使用一个引用来为`slice::Iter`借用数组，所以看起来我们实际上没有办法在这里抢占`IntoIterator`。

遗憾的是，我们不得不撤销先前宣布的成功。如果我们愿意在这里接受一些语言杂质，也许我们仍然可以找到一种方法来覆盖编译器中的方法解析。否则，我们将回到寻找减轻添加这个新特征实现的痛苦的方法。第一步可能是将 Clippy lint 提升到主编译器中，使其具有更大的可见性，但即使这样，我们也可能希望在敢于进行实际更改之前暂时不发出警告。如果您在一个数组上使用`into_iter()`来获得一个片迭代器，请尽快切换到`iter()`,如果您有什么想法来简化这种转换，请说出来！

## 更多尽在掌握

另一个有用的会议是与 WebAssembly (Wasm)工作组的会议，在那里我代表了[Rayon](https://crates.io/crates/rayon)——一个提供窃取工作的线程池和并行迭代器的箱子。我们很想让 Rayon 与 Wasm 一起工作，在 JavaScript 下实现这种简单的并行，但是线程化仍然处于早期阶段。Wasm 的大部分`std::thread`实现都被废弃了，所以 Rayon 一试图启动线程池就会出错。我们讨论了希望等待 JavaScript 承诺的线程所面临的限制，但我们认为让 Rayon 可用于 CPU 密集型工作负载仍然是有用的。我们计划添加一个自定义的方式来启动 Rayon 的 threadpool，他们可以在那里控制产卵，我已经打开了 [rayon PR636](https://github.com/rayon-rs/rayon/pull/636) 进行实验。

在关于构建系统、基础设施和发布团队的几次会议之间，我们制定了计划来改善 Rust 贡献者的体验。在 rustbuild 中，我们讨论了一些不同的贡献者工作流，以及我们如何减轻他们的负担。计划是更积极地缓存不会影响贡献者目标的构建工件，并使使用该路径的构建选项更容易。基础设施团队讨论了 CI 在满足您的拉取请求方面的负担，因为 Rust 要求在合并任何变更之前进行一次干净的 CI 运行。完整的运行需要大约 2.5 小时，我们讨论了调整当前 CI 运行的方法，以减少冗余测试，同时仍然确保足够的覆盖率。最后，在发布团队中，我们讨论了添加更多自动化来帮助问题分类和拉式请求管理，例如一个新的机器人可以让任何人应用问题标签，还有一个机器人可以自动管理 PR 汇总来聚合 CI 运行。

除了我积极参与的会议，Rust All Hands 也是一个很好的机会来了解其他团队正在做什么。既然 2018 版已经推出了 Rust 1.31，每个人都可以从发布危机中喘口气，并开始为未来几年做计划。编译器团队讨论了一些潜在的主要重构，这些重构可以实现更好的 IDE 集成，并计划为此进行实验。lang 团队列出了一个巨大的潜在新功能列表，从几乎准备好稍微打磨一下就稳定下来的东西到可能在 Rust 中根本没有真正未来的白日梦。而且，由于几个会议室同时进行，我不可能全部看完。我希望其他与会者将他们的结果写在博客上，分享更多的信息。

*Last updated: March 18, 2019*