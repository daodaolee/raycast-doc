---
description: 此示例演示如何将多个脚本绑定到单个扩展中。
---

# Spotify Controls

{% hint style="info" %}
该示例的源代码可以在 [这里](https://github.com/raycast/extensions/tree/main/extensions/spotify-controls#readme) 找到。您可以在 [此处](https://www.raycast.com/thomas/spotify-controls) 安装它。
{% endhint %}

此示例演示如何构建在 Raycast 中不显示 UI 的命令。这种类型的命令对于与其他应用程序交互非常有用，例如跳过 Spotify 中的歌曲或只是简单地运行一些不需要视觉展示的脚本。

![示例：从 Raycast 控制 Spotify macOS 应用程序](../.gitbook/assets/example-spotify-controls.png)

## 控制 Spotify macOS 应用程序

Spotify's macOS app supports AppleScript. This is great to control the app without opening it. For this, we use the [`run-applescript`](https://www.npmjs.com/package/run-applescript) package. Let's start by toggling play pause:

Spotify 的 macOS 应用程序支持 AppleScript。这非常适合在不打开应用程序的情况下控制它。为此，我们使用 [`run-applescript`](https://www.npmjs.com/package/run-applescript) 包。让我们从切换播放暂停开始：

```typescript
import { runAppleScript } from "run-applescript";

export default async function Command() {
  await runAppleScript('tell application "Spotify" to playpause');
}
```

## 关闭 Raycast 主窗口

执行此命令时，您会注意到 Raycast 会切换 Spotify macOS 应用程序的播放暂停状态，但 Raycast 主窗口保持打开状态。理想情况下，该窗口会在运行命令后关闭。然后你就可以继续之前所做的事情。

以下是关闭主窗口的方法：

```typescript
import { closeMainWindow } from "@raycast/api";
import { runAppleScript } from "run-applescript";

export default async function Command() {
  await closeMainWindow();
  await runAppleScript('tell application "Spotify" to playpause');
}
```

请注意，我们在运行 AppleScript 之前调用 `closeMainWindow` 函数。

只需不到 10 行代码，您就可以执行脚本并控制 Raycast 的 UI。下一步，您可以添加更多命令来跳过曲目。
