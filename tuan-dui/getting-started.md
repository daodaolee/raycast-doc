---
description: 本指南帮助您设置团队 Raycast。
---

# 开始

Raycast for Teams 允许您在私人商店中构建、共享和发现扩展。该商店仅可供您组织的成员访问。

## 创建您的团队

首先，创建您的组织。指定组织名称、handle（在链接中使用，例如 `https://raycast.com/your_org/some_extension`）并可选择上传头像。

![创建一个团队](../.gitbook/assets/teams-create-organization.png)

{% hint style="info" %}
您可以稍后使用 “Manage Organization”  命令编辑组织的信息。
{% endhint %}

## 创建您的私人扩展存储

创建组织后，就可以为您的扩展程序设置私人商店了。

### 初始化本地存储库

首先，选择一个文件夹来为您的私有扩展存储创建本地存储库。我们创建一个包含入门扩展的文件夹。我们建议将团队的所有扩展存储在单个存储库中。这使得协作变得容易。

![创建本地存储库](../.gitbook/assets/teams-create-repository.png)

### 构建入门扩展

创建本地存储库后，导航到 `getting-started` 文件夹。该文件夹包含一个简单的扩展，其中包含一个命令，该命令显示带有一些有用链接的列表。在文件夹中运行 `npm run dev` 以构建扩展并启动开发模式。 打开 Raycast ，您可以在根搜索中看到一个新的 “Development” 部分。该部分显示了正在开发的所有命令。你可以打开命令并打开一些链接。

![构建入门扩展](../.gitbook/assets/teams-develop-extension.png)

{% hint style="info" %}
有关如何创建扩展的更详细指南，请参阅 [创建您的第一个扩展](../ji-chu/create-your-first-extension.md)。
{% endhint %}

### 发布该扩展

现在，我们与您的组织共享该扩展。在扩展文件夹中执行 `npm run publish`。该命令验证、构建扩展并将其发布到您的私有扩展存储。该扩展程序仅可供您组织的成员访问。

![发布该扩展](../.gitbook/assets/teams-publish-extension.png)

🎉 恭喜！您构建并发布了您的第一个私人扩展。现在是时候在您的组织中发布这一信息了。

## 邀请你的队友

使用 Raycast 中的 “Copy Organization Invite Link” 命令来共享对您的组织的访问权限。将链接发送给您的队友。当有人加入您的组织时，您会收到一封电子邮件。您可以使用 “Manage Organization” 命令查看谁属于您的组织、重置邀请链接并编辑您的组织详细信息。

下一步，请按照 [本指南](collaborate-on-private-extensions.md) 将本地存储库推送到源代码控制系统。这使您可以与队友就扩展进行协作。
