# ESLint

Raycast 可以使用 CLI 的 lint 命令 (`ray lint`) 轻松对扩展进行 lint 检测。

Raycast 默认提供一个 [ESLint 配置](https://github.com/raycast/eslint-config/blob/main/index.js)，其中包括检查 Raycast 扩展所需的一切。默认配置非常简单：

```json
{ 
  "root": true,
  "extends": [
    "@raycast"
  ]
}
```

它抽象出了用于 Raycast 扩展的不同 ESLint 依赖项，并包含不同的规则集。

它还包括 Raycast 自己的 ESLint 插件规则集，使您在构建扩展时更容易遵循最佳实践。例如，有一条 [规则](https://github.com/raycast/eslint-plugin/blob/main/docs/rules/prefer-title-case.md) 可以帮助您遵循操作组件的标题大小写约定。

您可以直接在 [存储库文档](https://github.com/raycast/eslint-plugin#rules) 上查看 Raycast 的 ESLint 插件规则。

## 自定义

您可以自由打开/关闭规则或添加您认为适合您的扩展的新插件。例如，您可以为您的扩展添加规则  [`@raycast/prefer-placeholders`](https://github.com/raycast/eslint-plugin/blob/main/docs/rules/prefer-placeholders.md) ：

```json
{
  "root": true,
  "extends": [
    "@raycast"
  ],
  "rules": {
    "@raycast/prefer-placeholders": "warn"
  }
}
```

## 迁移

从版本 1.48.8 开始，使用 `Create Extension` 命令创建新扩展时会自动包含 ESLint 配置。如果您的扩展是在此版本之前创建的，您可以按照 [v1.48.8](https://developers.raycast.com/migration/v1.48.8) 页面上列出的步骤进行迁移。
