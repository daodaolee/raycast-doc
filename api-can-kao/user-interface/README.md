# 用户界面

Raycast 使用 React 渲染用户界面，并将支持的元素渲染到我们的本机 UI。该 API 附带了一组 UI 组件，您可以使用它们来构建扩展。可以将其视为一个设计系统。下面是高级组件：

* [List](list.md) 显示多个相似的项目，例如您未完成的待办事项列表。
* [Grid](grid.md) 类似于 list，但有更多的底部空间来显示每个项目的图像，例如图标的集合。
* [Detail](detail.md) 提供更多信息，例如 GitHub 拉取请求的详细信息。
* [Form](form.md) 创建新内容，例如提交错误报告。\
  每个组件都可以通过 [ActionPanel](https://developers.raycast.com/api-reference/user-interface/action-panel) 提供交互。该面板有一个 [Actions](https://developers.raycast.com/api-reference/user-interface/actions) 列表，其中每个操作都可以与[键盘快捷键](https://developers.raycast.com/api-reference/keyboard)关联。快捷方式允许用户无需使用鼠标即可使用 Raycast。

## 渲染

要渲染用户界面，您需要执行以下操作：

* 在 `package.json` manifest file 中设置 `mode` 为 `view`
* 从命令输入文件导出 React 组件

从一般经验法则的角度出发，您应该尽快渲染某些内容。这保证了您的命令具有响应能力。如果没有可显示的数据，则可以在顶级组件（例如  [`<Detail>`](detail.md)、 [`<Form>`](form.md)或 [`<List>`](list.md) ）上将 `isLoading` 属性设置为 true。它在 Raycast 顶部显示一个 loading 指示器。

以下示例显示命令启动后 2 秒的 loading 指示器：

```typescript
import { List } from "@raycast/api";
import { useEffect, useState } from "react";

export default function Command() {
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    setTimeout(() => setIsLoading(false), 2000);
  }, []);

  return <List isLoading={isLoading}>{/* Render your data */}</List>;
}
```
