---
description: 了解如何导入扩展以与他人协作。
---

# 贡献一个扩展

所有已发布的扩展都是开源的，可以在 [此存储库](https://github.com/raycast/extensions) 中找到。这使得多个开发人员可以轻松协作。本指南解释了如何导入扩展（包括修复错误、添加新功能或以其他方式做出贡献）。

## 获取源代码

首先，您需要找到扩展的源代码。最简单的方法是在 Raycast 的根搜索中使用 Fork Extension 操作。

![Fork 一个扩展](../.gitbook/assets/fork-extension.png)

## 修改扩展

在本地获得源代码后，打开终端并导航到扩展的文件夹。到达那里后，从终端中的扩展文件夹运行 `npm install && npm run dev` 以便开始修改扩展。

![打开已导入的扩展](../.gitbook/assets/basics-open-command.png) ![图标列表命令](../.gitbook/assets/basics-icon-list.png)

您会在根搜索的顶部看到您 fork 出的扩展，并且可以打开其命令。完成编辑扩展后，请确保将自己添加到其 [manifest](../zi-liao/manifest.md) 的贡献者部分，然后运行 `​​npx @raycast/api@latestpublish`。
