---
description: 了解如何在组织的私有扩展程序存储中共享扩展程序
---

# 发布私人扩展

要发布扩展，请在扩展目录中运行 `npm run publish`。该命令验证、构建扩展并将其发布到所有者的商店。该扩展仅适用于该组织的成员。您的扩展程序的链接将被复制到您的剪贴板，以便与您的队友共享。🥳

要将扩展标记为私有，您需要将 `package.json` 中的所有者字段设置为您的组织 handle。如果您不知道您的 handle，请打开 “Manage Organization” 命令，在右上角的下拉列表中选择您的组织，然后执行 “Copy Organization Handle” 操作  (`⌘` `⇧` `.`)。

{% hint style="info" %}
使用 Create Extension 命令为您的组织创建专用扩展。
{% endhint %}

为了能够向组织发布私有扩展，您需要登录。Raycast 也会帮助您使用 CLI 登录。如果您尚未登录或需要切换帐户，可以运行 `npx ray login` 和 `npx ray logout`。
