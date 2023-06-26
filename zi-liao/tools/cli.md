---
description: Raycast CLI 允许您构建、开发和检查您的扩展。
---

# CLI

CLI 是 `@raycast/api` 包的一部分，在安装过程中会自动安装在您的扩展目录中。要获取可用 CLI 命令的列表，请在扩展目录中运行以下命令：

```sh
 npx ray -h
```

## Build

`npx ray build` 创建扩展的优化生产版本以进行发布。我们的 CI 使用此命令将您的扩展发布到商店。

您可以使用 `npx ray build -e dist` 验证您的扩展是否正确构建。

## Development

`npx ray develop` 在开发模式下启动您的扩展。该模式包括以下内容:

* 扩展程序显示在根搜索的顶部，以便快速访问
* 保存更改时，Command 会自动重新加载（您可以通过 Raycast Preferences > Advanced > "Auto-reload on save" 切换自动重新加载）
* 错误覆盖包括详细的堆栈跟踪，以加快调试速度
* 显示在终端中的日志消息
* 状态指示器在命令的导航标题中可见，以指示构建错误
* 将扩展导入到 Raycast（如果之前没有）

## Lint

`npx ray lint` 对 `src` 目录中的所有文件运行  [ESLint](http://eslint.org) 。

## Migrate

`npx ray migrate` 将您的扩展 [迁移](https://developers.raycast.com/migration) 到最新版本的  `@raycast/api`

## Publish

`npx ray publish` 验证、构建和发布扩展。

如果扩展是私有的（例如，有 `owner` 并且没有公共访问 `access`），它将被发布到组织的私有存储中。此命令仅适用于属于该组织的用户。点击 [此处](https://developers.raycast.com/teams/getting-started) 了解详情。
