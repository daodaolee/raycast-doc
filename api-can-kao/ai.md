# AI

AI API 为开发人员提供了对 AI 功能的无缝访问，无需 API 密钥、配置或额外的依赖项。

{% hint style="info" %}
某些用户可能无权访问此 API。如果用户无权访问 Raycast AI，当您的扩展程序调用 AI API 时，系统会询问他们是否想要访问权限。如果用户不希望获得访问权限，API 调用将引发错误。

您可以使用 [`environment.canAccess(AI)`](environment.md)检查用户是否有权访问该API。
{% endhint %}

## API 参考

### AI.ask

向 AI 询问任何你想要的东西，可以在“no-view” 命令、effects 或回调中使用它。在 React 组件中，您可能想使用 [`useAI` util hook](../gong-gong-bao/react-hooks/useai.md) 代替。

#### 签名

```typescript
async function ask(prompt: string, options?: AskOptions): Promise<string> & EventEmitter;
```

#### 例子

{% tabs %}
{% tab title="基础用法" %}
```typescript
import { AI, Clipboard } from "@raycast/api";

export default async function command() {
  const answer = await AI.ask("Suggest 5 jazz songs");

  await Clipboard.copy(answer);
}
```
{% endtab %}

{% tab title="错误处理" %}
```typescript
import { AI, showToast } from "@raycast/api";

export default async function command() {
  try {
    await AI.ask("Suggest 5 jazz songs");
  } catch (error) {
    // Handle error here, eg: by showing a Toast
    await showToast({
      style: Toast.Style.Failure,
      title: "Failed to generate answer",
    });
  }
}
```
{% endtab %}

{% tab title="流式回答" %}
```typescript
import { AI, getSelectedFinderItems, showHUD } from "@raycast/api";
import fs from "fs";

export default async function main() {
  let allData = "";
  const [file] = await getSelectedFinderItems();

  const answer = AI.ask("Suggest 5 jazz songs");

  // Listen to "data" event to stream the answer
  answer.on("data", async (data) => {
    allData += data;
    await fs.promises.writeFile(`${file.path}`, allData.trim(), "utf-8");
  });

  await answer;

  await showHUD("Done!");
}
```
{% endtab %}

{% tab title="用户反馈" %}
```typescript
import { AI, getSelectedFinderItems, showHUD } from "@raycast/api";
import fs from "fs";

export default async function main() {
  let allData = "";
  const [file] = await getSelectedFinderItems();

  // If you're doing something that happens in the background
  // Consider showing a HUD or a Toast as the first step
  // To give users feedback about what's happening
  await showHUD("Generating answer...");

  const answer = await AI.ask("Suggest 5 jazz songs");

  await fs.promises.writeFile(`${file.path}`, allData.trim(), "utf-8");

  // Then, when everythig is done, notify the user again
  await showHUD("Done!");
}
```
{% endtab %}

{% tab title="检查访问权限" %}
```typescript
import { AI, getSelectedFinderItems, showHUD, environment } from "@raycast/api";
import fs from "fs";

export default async function main() {
  if (environment.canAccess(AI)) {
    const answer = await AI.ask("Suggest 5 jazz songs");
    await Clipboard.copy(answer);
  } else {
    await showHUD("You don't have access :(");
  }
}
```
{% endtab %}
{% endtabs %}

#### 参数

| 名称                                       | 描述 | 类型                                     |
| ---------------------------------------- | -- | -------------------------------------- |
| prompt<mark style="color:red;">\*</mark> |    | `string`                               |
| options                                  |    | [`AI.AskOptions`](ai.md#ai.askoptions) |

#### 返回

一个resolves 的 Promise。

## 类型

### AI.Creativity

具体任务（例如修复语法）需要较少的创造力，而开放式问题（例如产生想法）则需要更多创造力。

```typescript
type Creativity = "none" | "low" | "medium" | "high" | "maximum" | number;
```

如果传递一个数字，则该数字需要在 0-2 范围内。对于较大的值，将使用 2。对于较低的值，将使用 0。

### AI.Model

用于回答提示的 AI 模型。默认为  `"text-davinci-003"`。

```typescript
type Model = "text-davinci-003" | "gpt-3.5-turbo";
```

### AI.AskOptions

#### 属性

| 名称         | 描述                                                                                              | 类型                                     |
| ---------- | ----------------------------------------------------------------------------------------------- | -------------------------------------- |
| creativity | 具体任务（例如修复语法）需要较少的创造力，而开放式问题（例如产生想法）则需要更多创造力。如果传递一个数字，则该数字需要在 0-2 范围内。对于较大的值，将使用 2。对于较低的值，将使用 0。 | [`AI.Creativity`](ai.md#ai.creativity) |
| model      | 用于回答提示的 AI 模型。                                                                                  | [`AI.Model`](ai.md#ai.model)           |
| signal     | 取消请求的中止标识。                                                                                      | `AbortSignal`                          |
