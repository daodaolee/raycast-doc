---
description: 本指南介绍了如何查找和修复扩展中的错误。
---

# 调试一个扩展

错误是不可避免的。因此，有一种简单的方法来发现和修复它们非常重要。本指南向您展示如何查找扩展中的问题。

## Console

使用 `console` 进行简单的调试，例如记录变量、函数调用或其他有用的消息。在 [开发模式](../zi-liao/tools/cli.md#development) 下，所有日志都显示在终端中。这里有一些例子：

```typescript
console.log("Hello World"); // Prints: Hello World

const name = "Thomas";
console.debug(`Hello ${name}`); // Prints: Hello Thomas

const error = new Error("Boom 💥");
console.error(error); // Prints: Boom 💥
```

有关更多信息，请查看 [Node.js 文档](https://nodejs.org/dist/latest-v16.x/docs/api/console.html)。

我们自动禁用商店扩展的控制台日志记录。

## Visual Studio Code

对于更复杂的调试，您可以安装 [VSCode 扩展](https://marketplace.visualstudio.com/items?itemName=tonka3000.raycast)，以便能够将 node.js 调试器附加到正在运行的 Raycast 程序。

1. 通过 `npm run dev` 或通过 VSCode 命令 `Raycast: Start Development Mode`
2. 启动 VSCode 命令 `Raycast: Attach Debugger`
3. 像在任何其他 Node.js 基础项目中一样设置断点
4. 激活你的命令

## 未处理的异常和 Promise rejections

所有未处理的异常和 Promise rejections 都在 Raycast 中显示错误叠加。

![开发模式下未处理的异常](../.gitbook/assets/basics-unhandled-exception.png)

在开发过程中，我们会显示堆栈跟踪并添加一个操作来跳转到错误，以便于修复它。在生产中，仅显示错误消息。您应该为所有预期的错误 [显示一个 toast](../api-can-kao/feedback/toast.md)，例如失败的网络请求。

## React Developer Tools

我们支持开箱即用的 [React 开发者工具](https://github.com/facebook/react/tree/main/packages/react-devtools)。使用这些工具检查和更改 React 组件的 props，并立即在 Raycast 中查看结果。这对于具有很多状态的复杂命令特别有用。

![React Developer Tools](../.gitbook/assets/basics-react-developer-tools.png)

首先，将 `react-devtools` 添加到您的扩展中。打开终端，导航到您的扩展目录并运行以下命令：

```typescript
npm install --save-dev react-devtools@4.24.6
```

然后使用 `npm run dev` 重新构建扩展，打开要在 Raycast 中调试的命令，然后使用 `⌘` `⌥` `D` 启动 React 开发者工具。现在选择一个 React 组件，更改右侧边栏中的 prop，然后点击进入。您会立即注意到 Raycast 中的变化。

### 替代方案：全局安装 React Developer Tools

如果您希望全局安装 `react-devtools`，您可以执行以下操作：

```bash
npm install -g react-devtools@4.24.6
```

然后，您可以从终端运行 `react-devtools`来启动独立的 DevTools 应用程序。 Raycast 会自动连接，您将可以开始调试组件树。

## 环境

默认情况下，从商店安装的扩展在 Node 生产模式下运行，开发扩展在开发模式下运行。在开发模式下，CLI 输出会向您显示其他错误和警告（例如，当您缺少列表项的 React  `key` 属性时会出现臭名昭著的警告）；在生产模式下运行时，性能通常会更好。您可以通过打开 Raycast Preferences > Advanced >“Use Node Productionenvironment” 来强制开发扩展在 Node 生产模式下运行。

在运行时，您可以检查正在运行的 Node 环境：

```typescript
if (process.env.NODE_ENV === "development") {
  // running in development Node environment
}
```

要检查您运行的是线上版本还是本地开发版本：

```typescript
import { environment } from "@raycast/api";

if (environment.isDevelopment) {
  // running the development version
}
```
