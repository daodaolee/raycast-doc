# Actions

我们的 API 包括一些可用于常见交互的内置操作，例如打开链接或将某些内容复制到剪贴板。通过使用它们，您确保遵循我们的人机界面指南。如果您需要自定义内容，请使用 [`Action`](actions.md#action) 组件。所有内置操作都只是其之上的抽象。

## API 参考

### Action

用户可以执行的特定于上下文的操作。&#x20;

为项目分配键盘快捷键，以便用户更轻松地执行常用操作。

#### 例子

```typescript
import { ActionPanel, Action, List } from "@raycast/api";

export default function Command() {
  return (
    <List navigationTitle="Open Pull Requests">
      <List.Item
        title="Docs: Update API Reference"
        subtitle="#1"
        actions={
          <ActionPanel title="#1 in raycast/extensions">
            <Action.OpenInBrowser url="https://github.com/raycast/extensions/pull/1" />
            <Action.CopyToClipboard title="Copy Pull Request Number" content="#1" />
            <Action title="Close Pull Request" onAction={() => console.log("Close PR #1")} />
          </ActionPanel>
        }
      />
    </List>
  );
}
```

#### 参数

| 名称                                      | 描述                                                        | 类型                                                            | 默认                                              |
| --------------------------------------- | --------------------------------------------------------- | ------------------------------------------------------------- | ----------------------------------------------- |
| title<mark style="color:red;">\*</mark> | Action 显示的标题。                                             | `string`                                                      | -                                               |
| autoFocus                               | 当父 ActionPanel（或 Actionpanel.Submenu）打开时，Action 是否自动获得焦点。 | `boolean`                                                     | -                                               |
| icon                                    | Action 显示的图标。                                             | [`Image.ImageLike`](icons-and-images.md#image.imagelike)      | -                                               |
| shortcut                                | Action 的键盘快捷键。                                            | [`Keyboard.Shortcut`](../keyboard.md#keyboard.shortcut)       | -                                               |
| style                                   | Action 的样式。                                               | [`Alert.ActionStyle`](../feedback/alert.md#alert.actionstyle) | [Action.Style.Regular](actions.md#action.style) |
| onAction                                | Action 触发的回调。                                             | `() => void`                                                  | -                                               |

### Action.CopyToClipboard

将内容复制到剪贴板的操作。&#x20;

主窗口关闭，内容复制到剪贴板后会显示 HUD。

#### 例子

```typescript
import { ActionPanel, Action, Detail } from "@raycast/api";

export default function Command() {
  return (
    <Detail
      markdown="Press `⌘ + .` and share some love."
      actions={
        <ActionPanel>
          <Action.CopyToClipboard content="I ❤️ Raycast" shortcut={{ modifiers: ["cmd"], key: "." }} />
        </ActionPanel>
      }
    />
  );
}
```

#### 参数

| 名称                                        | 描述              | 类型                                                                                                  | 默认                                         |
| ----------------------------------------- | --------------- | --------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| content<mark style="color:red;">\*</mark> | 将复制到剪贴板的内容。     | `string` 或 `number` 或[`Clipboard.Content`](../clipboard.md#clipboard.content)                       | -                                          |
| icon                                      | Action 显示的可选图标。 | [`Image.ImageLike`](icons-and-images.md#image.imagelike)                                            | [Icon.Clipboard](icons-and-images.md#icon) |
| shortcut                                  | Aciton 的键盘快捷键。  | [`Keyboard.Shortcut`](../keyboard.md#keyboard.shortcut)                                             | -                                          |
| title                                     | Action 的可选标题。   | `string`                                                                                            | `"Copy to Clipboard"`                      |
| transient                                 | 是否应将内容暂时复制到剪贴板。 | `boolean`                                                                                           | false                                      |
| onCopy                                    | 内容复制到剪贴板时的回调。   | `(content: string \| number \|` [`Clipboard.Content`](../clipboard.md#clipboard.content)`) => void` | -                                          |

### Action.Open

使用特定应用程序打开文件或文件夹的操作，就像双击文件的图标一样。&#x20;

文件打开后主窗口关闭。

#### 例子

```typescript
import { ActionPanel, Detail, Action } from "@raycast/api";

export default function Command() {
  return (
    <Detail
      markdown="Check out your extension code."
      actions={
        <ActionPanel>
          <Action.Open title="Open Folder" target={__dirname} />
        </ActionPanel>
      }
    />
  );
}
```

#### 参数

| 名称                                       | 描述               | 类型                                                       | 默认                                      |
| ---------------------------------------- | ---------------- | -------------------------------------------------------- | --------------------------------------- |
| target<mark style="color:red;">\*</mark> | 要打开的文件、文件夹或 URL。 | `string`                                                 | -                                       |
| title<mark style="color:red;">\*</mark>  | Action 的标题。      | `string`                                                 | -                                       |
| application                              | 用于打开文件的应用程序名称。   | `string` 或 [`Application`](../utilities.md#application)  | -                                       |
| icon                                     | Action 显示的图标。    | [`Image.ImageLike`](icons-and-images.md#image.imagelike) | [Icon.Finder](icons-and-images.md#icon) |
| shortcut                                 | Action 的键盘快捷键。   | [`Keyboard.Shortcut`](../keyboard.md#keyboard.shortcut)  | -                                       |
| onOpen                                   | 文件或文件夹打开时的回调。    | `(target: string) => void`                               | -                                       |

### Action.OpenInBrowser

在默认浏览器中打开 URL 的操作。&#x20;

在浏览器中打开 URL 后，主窗口将关闭。

#### 例子

```typescript
import { ActionPanel, Detail, Action } from "@raycast/api";

export default function Command() {
  return (
    <Detail
      markdown="Join the gang!"
      actions={
        <ActionPanel>
          <Action.OpenInBrowser url="https://raycast.com/jobs" />
        </ActionPanel>
      }
    />
  );
}
```

#### 参数

| 名称                                    | 描述                          | 类型                                                       | 默认                                     |
| ------------------------------------- | --------------------------- | -------------------------------------------------------- | -------------------------------------- |
| url<mark style="color:red;">\*</mark> | 要打开的网址。                     | `string`                                                 | -                                      |
| icon                                  | Action 显示的图标。               | [`Image.ImageLike`](icons-and-images.md#image.imagelike) | [Icon.Globe](icons-and-images.md#icon) |
| shortcut                              | Action 的可选键盘快捷键。            | [`Keyboard.Shortcut`](../keyboard.md#keyboard.shortcut)  | -                                      |
| title                                 | <p>Action</p><p> 的可选标题。</p> | `string`                                                 | `"Open in Browser"`                    |
| onOpen                                | 在浏览器中打开 URL 时的回调。           | `(url: string) => void`                                  | -                                      |

### Action.OpenWith

使用特定应用程序打开文件或文件夹的操作。&#x20;

该操作将打开一个子菜单，其中包含可以打开该文件或文件夹的所有应用程序。在指定应用程序中打开文件后，主窗口将关闭。

#### 例子

```typescript
import { ActionPanel, Detail, Action } from "@raycast/api";
import { homedir } from "os";

const DESKTOP_DIR = `${homedir()}/Desktop`;

export default function Command() {
  return (
    <Detail
      markdown="What do you want to use to open your desktop with?"
      actions={
        <ActionPanel>
          <Action.OpenWith path={DESKTOP_DIR} />
        </ActionPanel>
      }
    />
  );
}
```

#### 参数

| Prop                                   | Description    | Type                                                     | Default                                 |
| -------------------------------------- | -------------- | -------------------------------------------------------- | --------------------------------------- |
| path<mark style="color:red;">\*</mark> | 要打开的路径。        | `string`                                                 | -                                       |
| icon                                   | Action 显示的图标。  | [`Image.ImageLike`](icons-and-images.md#image.imagelike) | [Icon.Upload](icons-and-images.md#icon) |
| shortcut                               | Action 的键盘快捷键。 | [`Keyboard.Shortcut`](../keyboard.md#keyboard.shortcut)  | -                                       |
| title                                  | Action 的标题。    | `string`                                                 | `"Open With"`                           |
| onOpen                                 | 文件或文件夹打开时的回调。  | `(path: string) => void`                                 | -                                       |

### Action.Paste

将内容粘贴到最前置的应用程序的操作。&#x20;

将内容粘贴到最前置的应用程序后，主窗口将关闭。

#### 例子

```typescript
import { ActionPanel, Detail, Action } from "@raycast/api";

export default function Command() {
  return (
    <Detail
      markdown="Let us know what you think about the Raycast API?"
      actions={
        <ActionPanel>
          <Action.Paste content="api@raycast.com" />
        </ActionPanel>
      }
    />
  );
}
```

#### 参数

| 名称                                        | 描述                  | 类型                                                                                                  | 默认                                         |
| ----------------------------------------- | ------------------- | --------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| content<mark style="color:red;">\*</mark> | 将粘贴到最前置的应用程序的内容。    | `string` 或 `number` 或[`Clipboard.Content`](../clipboard.md#clipboard.content)                       | -                                          |
| icon                                      | Action 显示的图标。       | [`Image.ImageLike`](icons-and-images.md#image.imagelike)                                            | [Icon.Clipboard](icons-and-images.md#icon) |
| shortcut                                  | Action 的键盘快捷键。      | [`Keyboard.Shortcut`](../keyboard.md#keyboard.shortcut)                                             | -                                          |
| title                                     | Action 的可选标题。       | `string`                                                                                            | `"Paste in Active App"`                    |
| onPaste                                   | 将内容粘贴到最前置的应用程序时的回调。 | `(content: string \| number \|` [`Clipboard.Content`](../clipboard.md#clipboard.content)`) => void` | -                                          |

### Action.Push

将新 view 推送到导航堆栈的操作。

#### 例子

```typescript
import { ActionPanel, Detail, Action } from "@raycast/api";

function Ping() {
  return (
    <Detail
      markdown="Ping"
      actions={
        <ActionPanel>
          <Action.Push title="Push Pong" target={<Pong />} />
        </ActionPanel>
      }
    />
  );
}

function Pong() {
  return <Detail markdown="Pong" />;
}

export default function Command() {
  return <Ping />;
}
```

#### 参数

| 名称                                       | 描述                | 类型                                                       | 默认 |
| ---------------------------------------- | ----------------- | -------------------------------------------------------- | -- |
| target<mark style="color:red;">\*</mark> | 将推送到导航堆栈的目标 view。 | `React.ReactNode`                                        | -  |
| title<mark style="color:red;">\*</mark>  | Action 显示的标题。     | `string`                                                 | -  |
| icon                                     | Action 显示的图标。     | [`Image.ImageLike`](icons-and-images.md#image.imagelike) | -  |
| shortcut                                 | Action 显示的快捷键     | [`Keyboard.Shortcut`](../keyboard.md#keyboard.shortcut)  | -  |
| onPush                                   | 推送目标 view 时的回调。   | `() => void`                                             | -  |

### Action.ShowInFinder

在 Finder 中显示文件或文件夹的操作。&#x20;

文件或文件夹在 Finder 中显示后，主窗口将关闭。

#### 例子

```typescript
import { ActionPanel, Detail, Action } from "@raycast/api";
import { homedir } from "os";

const DOWNLOADS_DIR = `${homedir()}/Downloads`;

export default function Command() {
  return (
    <Detail
      markdown="Are your downloads pilling up again?"
      actions={
        <ActionPanel>
          <Action.ShowInFinder path={DOWNLOADS_DIR} />
        </ActionPanel>
      }
    />
  );
}
```

#### 参数

| 名称                                     | 描述                      | 类型                                                         | 默认                                      |
| -------------------------------------- | ----------------------- | ---------------------------------------------------------- | --------------------------------------- |
| path<mark style="color:red;">\*</mark> | 要打开的路径。                 | [`PathLike`](../utilities.md#pathlike)                     | -                                       |
| icon                                   | Action 显示的可选图标。         | [`Image.ImageLike`](icons-and-images.md#image.imagelike)   | [Icon.Finder](icons-and-images.md#icon) |
| shortcut                               | Action 显示的快捷键           | [`Keyboard.Shortcut`](../keyboard.md#keyboard.shortcut)    | -                                       |
| title                                  | Action 的可选标题。           | `string`                                                   | `"Show in Finder"`                      |
| onShow                                 | 文件或文件夹显示在 Finder 中时的回调。 | `(path:` [`PathLike`](../utilities.md#pathlike)`) => void` | -                                       |

### Action.SubmitForm

添加用于捕获表单值的提交处理程序的操作。

#### 例子

```typescript
import { ActionPanel, Form, Action } from "@raycast/api";

export default function Command() {
  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit Answer" onSubmit={(values) => console.log(values)} />
        </ActionPanel>
      }
    >
      <Form.Checkbox id="answer" label="Are you happy?" defaultValue={true} />
    </Form>
  );
}
```

#### 参数

| 名称       | 描述                          | 类型                                                                                               | 默认              |
| -------- | --------------------------- | ------------------------------------------------------------------------------------------------ | --------------- |
| icon     | Action 显示的图标。               | [`Image.ImageLike`](icons-and-images.md#image.imagelike)                                         | -               |
| shortcut | Action 显示的快捷键               | [`Keyboard.Shortcut`](../keyboard.md#keyboard.shortcut)                                          | -               |
| title    | Action 显示的标题                | `string`                                                                                         | `"Submit Form"` |
| onSubmit | 表单提交时的回调。该处理程序接收包含用户输入的值对象。 | `(input:` [`Form.Values`](form.md#form.values)`) => boolean \| void \| Promise<boolean \| void>` | -               |

### Action.Trash

将文件或文件夹移至废纸篓的操作。

#### 例子

```typescript
import { ActionPanel, Detail, Action } from "@raycast/api";
import { homedir } from "os";

const FILE = `${homedir()}/Downloads/get-rid-of-me.txt`;

export default function Command() {
  return (
    <Detail
      markdown="Some spring cleaning?"
      actions={
        <ActionPanel>
          <Action.Trash paths={FILE} />
        </ActionPanel>
      }
    />
  );
}
```

#### 参数

| 名称                                      | 描述               | 类型                                                                                                        | 默认                                     |
| --------------------------------------- | ---------------- | --------------------------------------------------------------------------------------------------------- | -------------------------------------- |
| paths<mark style="color:red;">\*</mark> | 要移至垃圾箱的一个或多个项。   | [`PathLike`](../utilities.md#pathlike) or [`PathLike`](../utilities.md#pathlike)`[]`                      | -                                      |
| icon                                    | Action 显示的可选图标。  | [`Image.ImageLike`](icons-and-images.md#image.imagelike)                                                  | [Icon.Trash](icons-and-images.md#icon) |
| shortcut                                | Action 的可选键盘快捷键。 | [`Keyboard.Shortcut`](../keyboard.md#keyboard.shortcut)                                                   | -                                      |
| title                                   | Action 的可选标题。    | `string`                                                                                                  | `"Move to Trash"`                      |
| onTrash                                 | 当所有项目都移至垃圾箱时回调。  | `(paths:` [`PathLike`](../utilities.md#pathlike) `\|` [`PathLike`](../utilities.md#pathlike)`[]) => void` | -                                      |

### Action.CreateSnippet

导航到 “Create Snippet” 命令并预填充字段的操作

#### 例子

```typescript
import { ActionPanel, Detail, Action } from "@raycast/api";

export default function Command() {
  return (
    <Detail
      markdown="Test out snippet creation"
      actions={
        <ActionPanel>
          <Action.CreateSnippet snippet={{ text: "DE75512108001245126199" }} />
        </ActionPanel>
      }
    />
  );
}
```

#### 参数

| 名称                                        | 描述                                                                                    | 类型                                                       | 默认 |
| ----------------------------------------- | ------------------------------------------------------------------------------------- | -------------------------------------------------------- | -- |
| snippet<mark style="color:red;">\*</mark> |                                                                                       | [`Snippet`](actions.md#snippet)                          | -  |
| icon                                      | Action 显示的可选图标。有关支持的格式和类型，请参阅 [Image.ImageLike](icons-and-images.md#image.imagelike)。 | [`Image.ImageLike`](icons-and-images.md#image.imagelike) | -  |
| shortcut                                  | Action 的快捷键                                                                           | [`Keyboard.Shortcut`](../keyboard.md#keyboard.shortcut)  | -  |
| title                                     | Action 的标题                                                                            | `string`                                                 | -  |

### Action.CreateQuicklink

导航到 “Create Quicklink” 命令并预填充字段的操作。

#### 例子

```typescript
import { ActionPanel, Detail, Action } from "@raycast/api";

export default function Command() {
  return (
    <Detail
      markdown="Test out quicklink creation"
      actions={
        <ActionPanel>
          <Action.CreateQuicklink quicklink={{ link: "https://duckduckgo.com/?q={Query}" }} />
        </ActionPanel>
      }
    />
  );
}
```

#### 参数

| 名称                                          | 描述                                                                                     | 类型                                                       | 默认 |
| ------------------------------------------- | -------------------------------------------------------------------------------------- | -------------------------------------------------------- | -- |
| quicklink<mark style="color:red;">\*</mark> | 要创建的 [Quicklink](actions.md#quicklink) 。                                               | [`Quicklink`](actions.md#quicklink)                      | -  |
| icon                                        | Action 显示的可选图标。有关支持的格式和类型，请参阅 [Image.ImageLike](icons-and-images.md#image.imagelike) 。 | [`Image.ImageLike`](icons-and-images.md#image.imagelike) | -  |
| shortcut                                    | Action 的快捷键                                                                            | [`Keyboard.Shortcut`](../keyboard.md#keyboard.shortcut)  | -  |
| title                                       | Action 的可选标题。                                                                          | `string`                                                 | -  |

### Action.ToggleQuickLook

切换 “Quick Look” 以预览文件的操作。

#### 例子

```typescript
import { ActionPanel, List, Action } from "@raycast/api";

export default function Command() {
  return (
    <List>
      <List.Item
        title="Preview me"
        quickLook={{ path: "~/Downloads/Raycast.dmg", name: "Some file" }}
        actions={
          <ActionPanel>
            <Action.ToggleQuickLook shortcut={{ modifiers: ["cmd"], key: "y" }} />
          </ActionPanel>
        }
      />
    </List>
  );
}
```

#### 参数

| 名称       | 描述             | 类型                                                       | 默认                                   |
| -------- | -------------- | -------------------------------------------------------- | ------------------------------------ |
| icon     | Action 显示的图标.  | [`Image.ImageLike`](icons-and-images.md#image.imagelike) | [Icon.Eye](icons-and-images.md#icon) |
| shortcut | Action 的键盘快捷键。 | [`Keyboard.Shortcut`](../keyboard.md#keyboard.shortcut)  | -                                    |
| title    | Action 的标题。    | `string`                                                 | `"Quick Look"`                       |

### Action.PickDate

选择日期的 Action。

#### 例子

```typescript
import { ActionPanel, List, Action } from "@raycast/api";

export default function Command() {
  return (
    <List>
      <List.Item
        title="Todo"
        actions={
          <ActionPanel>
            <Action.PickDate title="Set Due Date…" />
          </ActionPanel>
        }
      />
    </List>
  );
}
```

#### 参数

| 名称                                         | 描述              | 类型                                                        | 默认                                        |
| ------------------------------------------ | --------------- | --------------------------------------------------------- | ----------------------------------------- |
| title<mark style="color:red;">\*</mark>    | Action 的标题.     | `string`                                                  | -                                         |
| onChange<mark style="color:red;">\*</mark> | 选择日期时的回调        | `(date: Date) => void`                                    | -                                         |
| icon                                       | Action 显示的可选图标。 | [`Image.ImageLike`](icons-and-images.md#image.imagelike)  | [Icon.Calendar](icons-and-images.md#icon) |
| max                                        | 允许选择的最长日期（含）。   | `Date`                                                    | -                                         |
| min                                        | 允许选择的最短日期（含）。   | `Date`                                                    | -                                         |
| shortcut                                   | Action 的键盘快捷键。  | [`Keyboard.Shortcut`](../keyboard.md#keyboard.shortcut)   | -                                         |
| type                                       | 可以选择哪些类型的日期组件   | [`Action.PickDate.Type`](actions.md#action.pickdate.type) | -                                         |

## 类型

### Snippet

#### 属性

| 名称                                     | 描述          | Type     |
| -------------------------------------- | ----------- | -------- |
| text<mark style="color:red;">\*</mark> | 代码片段内容。     | `string` |
| keyword                                | 触发代码片段的关键字。 | `string` |
| name                                   | 代码片段名称。     | `string` |

### Quicklink

#### 属性

| 名称                                     | 描述                                                          | 类型                                                      |
| -------------------------------------- | ----------------------------------------------------------- | ------------------------------------------------------- |
| link<mark style="color:red;">\*</mark> | URL 或文件路径，可以选择包含占位符，例如“https://google.com/search?q={Query}” | `string`                                                |
| application                            | 在其中打开快速链接的应用程序。                                             | `string` 或 [`Application`](../utilities.md#application) |
| name                                   | 快链名称                                                        | `string`                                                |

### Action.Style

定义操作的样式。

使用 [Action.Style.Regular](actions.md#action.style) 显示常规操作。当您的操作包含用户应该注意的内容时，请使用  [Action.Style.Destructive](actions.md#action.style) 。如果操作正在执行用户无法恢复的操作，请使用 [Alert](../feedback/alert.md)。

### Action.PickDate.Type

用户可以使用  `Action.PickDate` 选择日期组件的类型。

#### 枚举成员

| 名称       | 描述                 |
| -------- | ------------------ |
| DateTime | 除了年、月、日之外，还可以选择时、秒 |
| Date     | 只能选择年、月、日          |
