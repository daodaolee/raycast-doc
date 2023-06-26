---
description: 了解扩展的文件结构。
---

# 文件结构

扩展至少由一个入口点文件（例如 `src/index.ts`）和一个 `package.json` 清单文件组成。在搭建扩展脚手架时，我们添加了更多支持文件，以使用现代 JavaScript 工具简化开发。

新创建的扩展的典型目录结构如下所示：

```bash
extension
├── .eslintrc.json
├── .prettierrc
├── assets
│   └── icon.png
├── node_modules
├── package-lock.json
├── package.json
├── src
│   └── index.tsx
└── tsconfig.json
```

该目录包含所有源文件、assets 和一些支持文件。让我们逐一回顾一下：

## 源文件

将所有源文件放入  `src`  文件夹中。我们建议使用 TypeScript 作为编程语言。我们的 API 是完全类型化的，这可以帮助您在编译时而不是运行时捕获错误。支持 `ts`, `tsx`, `js` 和 `jsx` 作为文件扩展名。根据经验，对于带有 UI 的命令，请使用 tsx 或 jsx。

扩展由每个命令的入口点文件（例如 `src/index.ts`）和包含有关扩展及其命令的元数据的 `package.json` 清单文件组成。清单文件的格式与 [npm 包](https://docs.npmjs.com/cli/v7/configuring-npm/package-json) 的格式非常相似。除了一些标准属性之外，命令属性还新增了一些，比如向 Raycast 根搜索公开的命令以及它们的展示命令。

每个命令都有一个 `name` 属性，该名称映射到 `src` 文件夹中的主入口点文件。例如，`package.json` 文件中名为 `create` 的命令会映射到文件 `src/create{.ts,.tsx,.js,.jsx}`。

## 资源

可选的 `assets` 文件夹可以包含将打包到扩展存档中的图标。所有捆绑的资源都可以在运行时引用。此外，图标可以在 `package.json` 中用作扩展或命令图标。

## 支持的文件

该目录还包含一些用于设置常见 JavaScript 工具的文件：

* **.eslintrc.json** 表示 [ESLint](https://eslint.org/) 的规则，您可以使用 `npm run lint` 运行它。它提供了有关代码风格和最佳实践的建议。通常，您不必编辑此文件。
* **.prettierrc** 包含 [Prettier](https://prettier.io/) 格式化代码的默认规则。我们建议设置 [VS Code 扩展](https://prettier.io/docs/en/editors.html#visual-studio-code) 以保持代码美观。
* **node\_modules** 包含所有已安装的依赖项。您不应对此文件夹进行任何手动更改。
* **package-lock.json** 是由 npm 生成的用于安装依赖项的文件。您不应对此文件进行任何手动更改。
* **package.json** 是 [清单文件](https://developers.raycast.com/information/manifest)，其中包含有关扩展的元数据，例如其标题、命令及其依赖项。
* **tsconfig.json** 将您的项目配置为使用 TypeScript。您很可能不必编辑此文件。
