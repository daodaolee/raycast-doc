---
description: 最常见问题的解答。
---

# FAQ

<details>

<summary> <a href="https://github.com/raycast/script-commands">script commands</a> 和扩展有什么区别？</summary>

脚本命令是扩展 Raycast 的第一种方法。它们是执行 shell 脚本并在 Raycast 中显示一些有限输出的简单方法。扩展是我们扩展 Raycast 的下一次迭代。虽然脚本几乎可以用任何脚本语言编写，但扩展是用 TypeScript 编写的。

它们可以显示丰富的用户界面，例如列表和表单，但也可以是“headless”的，仅运行简单的脚本即可。 扩展可以通过我们的商店与我们的社区共享。这使得那些 Mac 上没有自制软件或其他 shell 集成的技术不太熟练的人很容易发现和使用它们。

</details>

<details>

<summary>为什么我不能使用 <code>react-dom</code>?</summary>

即使您编写 JS/TS 代码，所有内容都会在 Raycast 中本地渲染。不涉及任何 HTML 或 CSS。因此，您不需要 `react-dom` 包提供的特定于DOM的方法。

相反，我们实现了一个自定义 [reconciler](https://reactjs.org/docs/reconciliation.html)，它将您的 React 组件树转换为 Raycast 理解的渲染树。渲染树本身用于构造由 [Apple's AppKit](https://developer.apple.com/documentation/appkit/) 支持的视图层次结构。这  [React Native](https://reactnative.dev) 的工作原理类似。

</details>
