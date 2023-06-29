# 生命周期

命令一般会启动，运行一段时间，然后卸载。

## 启动

当在 Raycast 中启动命令时，命令代码会立即执行。如果扩展导出默认函数，则会自动调用该函数。如果您在导出的默认函数中返回一个 React 组件，它将自动呈现为根组件。对于不需要用户界面的命令（清单中的`mode` 属性设置为 `“no-view”`），您可以导出异步函数并使用 async/await 执行 API 方法。

{% tabs %}
{% tab title="视图命令" %}
```typescript
import { Detail } from "@raycast/api";

// Returns the main React component for a view command
export default function Command() {
  return <Detail markdown="# Hello" />;
}
```
{% endtab %}

{% tab title="无视图命令" %}
```typescript
import { showHUD } from "@raycast/api";

// Runs async. code in a no-view command
export default async function Command() {
  await showHUD("Hello");
}
```
{% endtab %}
{% endtabs %}

有多种方法可以启动命令：

* 用户在根搜索中搜索该命令并执行它
* 用户注册该命令的别名并按下它
* 另一个命令通过 [`launchCommand`](../../api-can-kao/command.md#launchcommand) 启动该命令
* 该命令在 [后台](background-refresh.md) 启动
* [表单的草稿](../../api-can-kao/user-interface/form.md#cao-gao) 已保存并由用户执行
* 用户将该命令注册为 [fallback 命令](https://manual.raycast.com/fallback-commands)，并在根搜索中没有结果时执行它
* 用户单击 [深度链接](deeplinks.md)

根据命令的启动方式，会不同的参数传递给导出的默认函数。

```typescript
import { Detail, LaunchProps } from "@raycast/api";

// Access the different launch properties via the argument passed to the function
export default function Command(props: LaunchProps) {
  return <Detail markdown={props.fallbackText || "# Hello"} />;
}
```

### launch 参数

| 属性                                           | 描述                                                  | 类型                                                                    |
| -------------------------------------------- | --------------------------------------------------- | --------------------------------------------------------------------- |
| arguments<mark style="color:red;">\*</mark>  | 使用这些值来填充命令的初始状态。                                    | [`Arguments`](arguments.md#arguments)                                 |
| launchType<mark style="color:red;">\*</mark> | 命令的启动类型（用户启动或后台）。                                   | [`LaunchType`](../../api-can-kao/environment.md#launchtype)           |
| draftValues                                  | 当用户通过草稿输入命令时，该对象将包含保存为草稿的用户输入。使用它的值来填充表单的初始状态。      | [`Form.Values`](../../api-can-kao/user-interface/form.md#form.values) |
| fallbackText                                 | 当该命令作为备用命令启动时，该字符串包含根搜索的文本。                         | `string`                                                              |
| launchContext                                | 当通过 launchCommand 以编程方式启动命令时，该对象包含传递给  `context`的值。 | [`LaunchContext`](../../api-can-kao/command.md#launchcontext)         |

## 卸载

卸载命令时（通常通过弹出回根搜索查看命令或在脚本完成无视图命令后），Raycast 将从内存中卸载整个命令。请注意，命令有内存限制，如果超出这些限制，命令将被终止，并且用户将看到一条错误消息。
