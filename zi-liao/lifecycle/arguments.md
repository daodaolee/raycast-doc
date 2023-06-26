# 参数

Raycast 支持命令传递参数，以便用户可以在打开命令之前直接从根搜索输入值。

![](../../.gitbook/assets/arguments.png)

参数在每个命令的 [mainfest](https://developers.raycast.com/information/manifest#argument-properties) 中配置。

{% hint style="info" %}
* 最大参数数量：3（如果您有需要更多参数的用例，请通过反馈或在 Slack 社区 中告知我们）
* mainfest 中指定的参数顺序很重要，并且由根搜索中显示的字段作出反馈。为了提供更好的用户体验，请将必需的参数放在可选参数之前。
{% endhint %}

## 例子

假设我们想要一个带有两个参数的命令。它的 `package.json` 看起来像这样：

```json
{
  "name": "arguments",
  "title": "API Arguments",
  "description": "Example of Arguments usage in the API",
  "icon": "command-icon.png",
  "author": "mattisssa",
  "license": "MIT",
  "commands": [
    {
      "name": "my-command",
      "title": "Arguments",
      "subtitle": "API Examples",
      "description": "Demonstrates usage of arguments",
      "mode": "view",
      "arguments": [
        {
          "name": "title",
          "placeholder": "Title",
          "type": "text",
          "required": true
        },
        {
          "name": "subtitle",
          "placeholder": "Subtitle",
          "type": "text"
        }
      ]
    }
  ],
  "dependencies": {
    "@raycast/api": "1.38.0"
  },
  "scripts": {
    "dev": "ray develop",
    "build": "ray build -e dist",
    "lint": "ray lint"
  }
}
```

命令本身将通过 `arguments` prop 接收参数的值：

```typescript
import { Form, LaunchProps } from "@raycast/api";

export default function Todoist(props: LaunchProps<{ arguments: Arguments.MyCommand }>) {
  const { title, subtitle } = props.arguments;
  console.log(`title: ${title}, subtitle: ${subtitle}`);

  return (
    <Form>
      <Form.TextField id="title" title="Title" defaultValue={title} />
      <Form.TextField id="subtitle" title="Subtitle" defaultValue={subtitle} />
    </Form>
  );
}
```

## 类型

### 参数

命令通过名为 `arguments` 的顶级 prop 接收其参数的值。它是一个对象，其中参数的 `name` 作为键，参数的值作为属性的值。

根据参数的 `type`，其值的类型会有所不同。

| 参数类型       | 值类型      |
| ---------- | -------- |
| `text`     | `string` |
| `password` | `string` |

{% hint style="info" %}
Raycast 提供了一个名为 Arguments 的全局 TypeScript 命名空间，其中包含扩展的所有命令的参数类型。

例如，如果名为 show-todos 的命令接受参数，则其 LaunchProps 可以描述为 LaunchProps<{argus: Arguments.ShowTodos }>。这将确保命令中使用的类型与 mainfest 保持同步。
{% endhint %}
