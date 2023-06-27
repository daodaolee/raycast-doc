---
description: 了解如何构建您的第一个扩展并在 Raycast 中使用它。
---

# 创建您的第一个扩展

## 创建一个新的扩展

打开 “Create Extension” 命令，将扩展命名为 “Hello World”，然后选择  “Detail template”模板。在 Location 字段中选择一个父文件夹，然后按 `⌘` `↵` 继续。

![在 Raycast 中创建扩展命令](../.gitbook/assets/hello-world.png)

{% hint style="info" %}
要创建私人扩展，请在第一个下拉列表中选择您的组织。您需要登录并且是组织的一部分才能查看下拉列表。在 [此处](https://developers.raycast.com/teams/getting-started) 了解有关 Raycast for Teams 的更多信息。
{% endhint %}

接下来，您需要按照屏幕上的说明来构建扩展。

## 构建您的扩展

打开终端，导航到扩展目录并运行 `npm install && npm run dev`。打开 Raycast，您会在根搜索的顶部看到您的扩展。按 `↵` 打开它。

![您的第一个扩展](../.gitbook/assets/hello-world-2.png)

## 开发您的扩展

要更改扩展，请打开扩展目录中的 `./src/index.tsx` 文件，更改 `markdown` 文本并保存。然后，再次在 Raycast 中打开命令并查看更改。

{% hint style="info" %}
npm run dev 在开发模式下启动扩展，具有热重载、错误报告和[其他功能](https://developers.raycast.com/information/tools/cli#development)。
{% endhint %}

## 使用您的扩展

现在，您可以在终端中按 `⌃` `C` 来停止 `npm run dev`。该扩展保留在 Raycast 中，当搜索扩展名称 “Hello World” 或命令名称 “Render Markdown” 时，您可以在根目录中找到其命令。

![在根搜索中找到您的扩展](../.gitbook/assets/hello-world-2.png)

🎉 恭喜！您构建了您的第一个扩展。去做想做的吧！

{% hint style="info" %}
当您想要更改扩展中的某些内容时，不要忘记再次运行 `npm run dev` 。
{% endhint %}
