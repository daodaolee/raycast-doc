# 迁移

本部分包含帮助您将扩展迁移到较新版本 API 的指南。

## 如何自动迁移您的扩展

只要有可能，我们都会提供工具来使用 [codemod](https://github.com/facebook/jscodeshift) 自动迁移到较新版本的 API。&#x20;

要运行 codemods，请在扩展目录中运行以下命令：

```
npx ray migrate
```

或者

```
npx @raycast/migration@latest .
```

它将检测您之前使用的 API 版本，并应用此后可用的所有迁移。&#x20;

运行后，请检查更新的文件并确保没有任何损坏 - 毕竟总会有特殊情况。
