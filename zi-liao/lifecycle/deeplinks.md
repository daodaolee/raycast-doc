# Deeplinks

Deeplinks 是 Raycast 特定的 URL，您可以使用它来启动任何命令，只要它在 Raycast 中安装并启用即可。&#x20;

它们遵循以下格式：

```
raycast://extensions/<author-or-owner>/<extension-name>/<command-name>
```

| 名称              | 描述                                                                                                                                                        | 类型       |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| author-or-owner | 对于商店扩展，它是扩展 [mainfest](https://developers.raycast.com/information/manifest) 中 `owner`或 `author` 字段的值。对于内置扩展（例如 `Calendar`），它始终是 `raycast`。                | `string` |
| extension-name  | 对于商店扩展，它是扩展清单中扩展 `name` 字段的值。对于内置扩展（例如 `Calendar`），它叫 “slugified”。                                                                                        | `string` |
| command-name    | 对于商店扩展，它是扩展 [mainfest](https://developers.raycast.com/information/manifest) 中命令 `name` 字段的值。对于内置命令（例如“`My Schedule`”），在 `my-schedule` 下命令名称叫 “slugified”。 | `string` |

为了更轻松地获取命令的 Deeplink，Raycast 根中的每个命令现在都有一个 `Copy Deeplink` 操作。

{% hint style="info" %}
每当使用 Deeplink 启动命令时，Raycast 都会要求您确认是否要运行该命令。这是为了确保您了解正在运行的命令。
{% endhint %}

![](../../.gitbook/assets/deeplink-confirmation.png)

## 查询参数

| 名称           | 描述                                                                                                          | 类型                               |
| ------------ | ----------------------------------------------------------------------------------------------------------- | -------------------------------- |
| launchType   | 在后台运行命令，跳过将 Raycast 带到前台。                                                                                   |  `userInitiated` 或者 `background` |
| arguments    | 如果命令接收 [arguments](https://developers.raycast.com/information/lifecycle/arguments)，则可以使用此查询参数传递它们。          | URL-encoded JSON object.         |
| context      | 如果该命令使用 [LaunchContext](https://developers.raycast.com/api-reference/command#launchcontext)，则可以使用此查询参数进行传递。 | URL-encoded JSON object.         |
| fallbackText | 用于预填充搜索栏或命令的第一个文本输入的一些文本                                                                                    | `string`                         |
