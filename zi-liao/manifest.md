# Manifest

`package.json`  manifest 文件是 npm `package.json` 文件的超集。这样，您只需要一个文件来配置您的扩展。本文档仅涵盖 Raycast 特定领域。有关其他内容，请参阅[ npm 的文档](https://docs.npmjs.com/cli/v7/configuring-npm/package-json)。

这是一个典型的 manifest 文件：

```javascript
{
  "name": "my-extension",
  "title": "My Extension",
  "description": "My extension that can do a lot of things",
  "icon": "icon.png",
  "author": "thomas",
  "categories": ["Fun", "Communication"],
  "license": "MIT",
  "commands": [
    {
      "name": "index",
      "title": "Send Love",
      "description": "A command to send love to each other",
      "mode": "view"
    }
  ]
}
```

## 扩展属性

所有 Raycast 相关的扩展属性。

| 属性           | 必填  | 描述                                                                                                                          |
| ------------ | --- | --------------------------------------------------------------------------------------------------------------------------- |
| name         | Yes | 扩展名的唯一名称。这在指向您的扩展程序的商店链接中使用，因此请保持简短且 URL 兼容。                                                                                |
| title        | Yes | 在 Store 中向用户显示的扩展程序的标题以及首选项。使用此标题可以很好地描述您的扩展，以便用户可以在商店中找到它。                                                                 |
| description  | Yes | Store 中显示的扩展的完整描述。                                                                                                          |
| icon         | Yes | 对资源文件夹中图标文件的引用。使用 `png` 格式，尺寸为 512 x 512 像素。要支持浅色和深色主题，请添加两个图标，其中一个以 `@dark` 作为后缀，例如`icon.png` 和 `icon@dark.png`。           |
| author       | Yes | 您的 Raycast Store 名称（用户名）                                                                                                    |
| categories   | Yes | 您的扩展程序所属的一系列类别。                                                                                                             |
| commands     | Yes | 扩展公开的命令数组，请参阅 [命令属性](manifest.md#ming-ling-shu-xing)。                                                                       |
| contributors | No  | 一个数组，Raycast 存储对此扩展做出贡献的人员的字段（用户名）。                                                                                         |
| keywords     | No  | 可以在 Raycast 中搜索扩展的关键字数组。                                                                                                    |
| preferences  | No  | 扩展可以提供 Raycast Preferences > Extensions 中显示的首选项。您可以使用配置值和密码或个人 access token 的首选项，请参阅[首选项属性](../api-can-kao/preferences.md)。 |
| external     | No  | 应从构建中排除的包或文件名的数组。该包不会被捆绑，但导入会被保留并在运行时进行评估。                                                                                  |

## 命令属性

所有命令属性

| 属性                | 必填  | 描述                                                                                                                                                                               |
| ----------------- | --- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| name              | Yes | 命令的唯一 ID。该名称直接映射到命令的入口点文件。因此，名为“index”的命令将映射到 `index.ts`（或任何其他受支持的 TypeScript 或 JavaScript 文件扩展名，例如 `.tsx`、`.js`、`.jsx`）。                                                        |
| title             | Yes | 命令的名称，在 Raycast 中向用户显示。                                                                                                                                                          |
| subtitle          | No  | 根搜索中命令的可选子标题。通常，这是与您的命令关联的服务或域。您可以使用  [`updateCommandMetadata`](../api-can-kao/command.md#updatecommandmetadata) 动态更新此属性。                                                        |
| description       | Yes | 它可以帮助用户了解该命令的作用。它将显示在 Store 和首选项中。                                                                                                                                               |
| icon              | No  | 对资源文件夹中图标文件的可选引用。使用尺寸至少为 512 x 512 像素的 png 格式。要支持浅色和深色主题，请添加两个图标，其中一个以 `@dark` 作为后缀，例如 `icon.png` 和 `icon@dark.png`。 如果未指定图标，则将使用扩展图标。                                           |
| mode              | Yes | `view` 值表示该命令执行时将显示主视图。 `no-view` 意味着该命令不会将视图推送到 Raycast 中的主导航堆栈。后者可以方便地直接打开 URL 或其他不需要用户界面的 API 功能。 `menu-bar` 表示该命令将返回一个 [Menu Bar Extra](../api-can-kao/menu-bar-commands.md) |
| interval          | No  | 该值指定应每隔 X 秒 (s)、分钟 (m)、小时 (h) 或天 (d) 在后台启动`no-view` 或者 `menu-bar` 命令。示例：90s、1m、12h、1d。最小值为 1 分钟 (1m)。                                                                            |
| keywords          | No  | 可以在 Raycast 中搜索命令的可选关键字数组。                                                                                                                                                       |
| arguments         | No  | 调用命令时用户请求的可选参数数组，请参阅 [Argument 属性](manifest.md#argument-shu-xing)。                                                                                                               |
| preferences       | No  | 命令可以选择提供在选择命令时显示在 Raycast Preferences > Extensions 中的首选项。您可以使用配置值和密码或个人访问令牌的首选项，请参阅 [首选项属性](../api-can-kao/preferences.md)。命令自动“继承”扩展首选项，并且还可以覆盖具有相同名称的条目。                       |
| disabledByDefault | No  | <p>指定默认情况下是否启用该命令。默认情况下，所有命令均已启用，但在某些情况下，您可能希望包含其他命令并让用户在需要时启用它们。<br><em>请注意，此标志仅在安装新扩展或有新命令时使用。</em></p>                                                                        |

## 偏好属性

扩展或特定于命令的首选项的所有属性。使用 [首选项 API](../api-can-kao/preferences.md) 访问它们的值。

| 属性          | 必填                                  | 描述                                                                                                                          |
| ----------- | ----------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| name        | Yes                                 | 首选项的唯一 ID。                                                                                                                  |
| description | Yes                                 | 它可以帮助用户了解偏好的作用。将鼠标悬停在其上时，它将显示为工具提示。                                                                                         |
| type        | Yes                                 | 偏好类型。我们目前支持 `"textfield"` 和 `"password"`（用于安全输入）、`"checkbox"`、`"dropdown"`、 `"appPicker"`、 `"file"`和 `"directory"`          |
| required    | Yes                                 | 指示该值是否是必需的并且必须由用户输入才能使用扩展。                                                                                                  |
| title       | 当 `type` 为 `checkbox` 时为No，否则为 Yes。 | <p>Raycast 首选项中显示的首选项的显示名称。 对于复选框，它显示为复选框本身上方的部分标题。</p><p>如果要将多个复选框分组到一个部分中，请设置第一个复选框的 <code>title</code> 并将其他复选框的标题留空。</p> |
| placeholder | No                                  | 未输入值时在首选项字段中显示的文本。                                                                                                          |
| default     | No                                  | 该字段的可选默认值。对于文本字段，这是一个字符串值；对于复选框，布尔值；对于下拉列表，数据数组中对象的值；对于 appPickers，则是应用程序名称、包 ID 或路径。                                       |
| data        | 当 `type` 为 `dropdown` 时为Yes，否则为 No。 | 具有 `title` 和 `value` 属性的对象数组，例如：`[{"title": "Item 1", "value": "1"}]`                                                       |
| label       | 当 `type` 为 `checkbox` 时为Yes，否则为 No。 | 必需，仅限复选框：复选框的标签。显示在复选框旁边。                                                                                                   |

## Argument 属性

命令 argument 里的所有属性。使用 [Arguments API](manifest.md#argument-shu-xing) 来访问它们的值。

| 属性          | 必填  | 描述                                                                                       |
| ----------- | --- | ---------------------------------------------------------------------------------------- |
| name        | Yes | 参数的唯一 ID。该值将作为[顶级 prop](lifecycle/arguments.md#can-shu) 传递的对象中的键。                        |
| type        | Yes | 参数类型。我们目前支持 `text` 和 `password`（用于安全输入）。当类型为密码时，输入的文本将替换为星号。最常见的用例 - 将密码或 secrets 传递给命令。 |
| placeholder | Yes | 参数输入字段的占位符。                                                                              |
| required    | No  | 指示该值是否是必需的并且必须由用户在打开命令之前输入。该值的默认值为 `false`。                                              |
