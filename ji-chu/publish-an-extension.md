---
description: 了解如何与我们的社区分享您的扩展。
---

# 发布一个扩展

在发布扩展程序之前，请查看 [如何为发布做准备](https://developers.raycast.com/basics/prepare-an-extension-for-store)。确保遵循指南是帮助您的扩展通过审核的最佳方式。

### 验证您的扩展

打开终端，导航到扩展目录，然后运行 ​​`npm run build` 来打包您的扩展。保证该命令顺利完成且没有任何错误。

{% hint style="info" %}
npm run build 验证您的扩展以进行打包，而无需将其发布到商店。在 [这里](https://developers.raycast.com/information/tools/cli#build) 阅读更多相关信息。
{% endhint %}

### 发布您的扩展

要与其他人共享您的扩展，请导航到您的扩展目录，然后运行 `​​npm run publish` 来发布您的扩展。系统会要求您通过 GitHub 进行身份验证，因为脚本会自动在我们的 [存储库](https://github.com/raycast/extensions) 中打开拉取请求。

{% hint style="info" %}
如果有人为您的扩展做出了贡献，则运行 npm run publish 将失败，直到您运行

```bash
npx @raycast/api@latest pull-contributions
```

在您的 git 存储库中。这会将贡献与您的代码合并，并要求您修复冲突（如果有）。
{% endhint %}

打开 PR 后，您可以通过再次运行 `npm run publish` 继续向其推送更多提交。

#### 替代方式

如果您想更好地控制发布过程，您可以手动执行 `npm run publish` 的操作。您需要在我们的 [存储库](https://github.com/raycast/extensions) 中打开 PR。为此，[Fork 我们的存储库](https://docs.github.com/en/get-started/quickstart/fork-a-repo)，将您的扩展添加到您的 fork 中，再推送您的更改，然后 [通过 GitHub Web 界面](https://docs.github.com/en/github/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request-from-a-fork) 将 PR 打开到我们的 `main` 中。

### 等待审核

在您打开一个 PR 后，我们将审核您的扩展并在需要时请求更改。一旦接受，PR 将被合并，您的扩展将自动发布到 [Raycast 商店](https://raycast.com/store)。

{% hint style="info" %}
我们仍在修复 bug 并更新我们的指南。如果有任何不清楚的地方，请在我们的 [社区](https://raycast.com/community) 中告诉我们。
{% endhint %}

### 分享您的扩展

一旦您的扩展在 Raycast 商店中发布，您就可以与我们的社区分享。打开 Manage Extensions 命令，搜索您的扩展并按 `⌘` `⌥` `.`复制链接。

![管理您的扩展](../.gitbook/assets/basics-manage-extensions.png)

🚀 现在是时候分享您的作品了！在 Twitter 上发布有关您的扩展的信息，与我们的 [Slack 社区](https://raycast.com/community) 分享或发送给您的队友。
