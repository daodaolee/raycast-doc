# Clipboard

使用 Clipboard API 可以处理剪贴板中的文本和当前选择的文本。您可以通过 [`Clipboard.copy`](clipboard.md#clipboard.copy) 将内容写入剪贴板，并通过 [`Clipboard.clear`](clipboard.md#clipboard.clear)清除内容。 [`Clipboard.paste`](clipboard.md#clipboard.paste) 函数在最前面的应用程序中的当前光标位置插入文本。

[`Action.CopyToClipboard`](user-interface/actions.md#action.copytoclipboard) 可用于将选定列表项的内容复制到剪贴板 [`Action.Paste`](user-interface/actions.md#action.paste) 可用于在最前面的应用程序中插入文本。

## API 参考

### Clipboard.copy

将文本或文件复制到剪贴板。

#### 签名

```typescript
async function copy(content: string | number | Content, options?: CopyOptions): Promise<void>;
```

#### 例子

```typescript
import { Clipboard } from "@raycast/api";

export default async function Command() {
  // copy some text
  await Clipboard.copy("https://raycast.com");

  const textContent: Clipboard.Content = {
    text: "https://raycast.com",
  };
  await Clipboard.copy(textContent);

  // copy a file
  const file = "/path/to/file.pdf";
  try {
    const fileContent: Clipboard.Content = { file };
    await Clipboard.copy(fileContent);
  } catch (error) {
    console.log(`Could not copy file '${file}'. Reason: ${error}`);
  }

// copy transient data
  await Clipboard.copy("my-secret-password", { transient: true })
}
```

#### 参数

| 名称                                        | 描述          | 类型                                                                          |
| ----------------------------------------- | ----------- | --------------------------------------------------------------------------- |
| content<mark style="color:red;">\*</mark> | 要复制到剪贴板的内容。 | `string` 或 `number` 或 [`Clipboard.Content`](clipboard.md#clipboard.content) |
| options                                   | 复制操作的选项。    | [`Clipboard.CopyOptions`](clipboard.md#clipboard.copyoptions)               |

#### 返回

当内容复制到剪贴板时解析的 Promise。

### Clipboard.paste

将文本或文件粘贴到最前面的应用程序。

#### 签名

```typescript
async function paste(content: string | Content): Promise<void>;
```

#### 例子

```typescript
import { Clipboard } from "@raycast/api";

export default async function Command() {
  await Clipboard.paste("I really like Raycast's API");
}
```

#### 参数

| 名称                                        | 描述          | 类型                                                                          |
| ----------------------------------------- | ----------- | --------------------------------------------------------------------------- |
| content<mark style="color:red;">\*</mark> | 要在光标处插入的内容。 | `string` 或 `number` 或 [`Clipboard.Content`](clipboard.md#clipboard.content) |

#### 返回

粘贴内容时解析的 Promise。

### Clipboard.clear

清除当前剪贴板内容。

#### 签名

```typescript
async function clear(): Promise<void>;
```

#### 例子

```typescript
import { Clipboard } from "@raycast/api";

export default async function Command() {
  await Clipboard.clear();
}
```

#### 返回

当剪贴板被清除时 resolves 的 Promise。

### Clipboard.read

以纯文本、文件名或 HTML 形式读取剪贴板内容。

#### 签名

```typescript
async function read(): Promise<ReadContent>;
```

#### 例子

```typescript
import { Clipboard } from "@raycast/api";

export default async () => {
  const { text, file, html } = await Clipboard.read();
  console.log(text);
  console.log(file);
  console.log(html);
};
```

#### 返回

当剪贴板内容被读取为纯文本、文件名或 HTML 时 resolves 的 Promise。

### Clipboard.readText

以纯文本形式读取剪贴板。

#### 签名

```typescript
async function readText(): Promise<string | undefined>;
```

#### 例子

```typescript
import { Clipboard } from "@raycast/api";

export default async function Command() {
  const text = await Clipboard.readText();
  console.log(text);
}
```

#### 返回

当剪贴板内容被读取为纯文本时 resolves 的 Promise。

## 类型

### Clipboard.Content

从剪贴板复制和粘贴的内容类型

```typescript
type Content =
  | {
      text: string;
    }
  | {
      file: PathLike;
    };
```

### Clipboard.ReadContent

从剪贴板读取的内容类型

```typescript
type Content =
  | {
      text: string;
    }
  | {
      file?: string;
    }
  | {
      html?: string;
    };
```

### Clipboard.CopyOptions

传递给 `Clipboard.copy` 的选项类型

```typescript
type CopyOptions = { transient: boolean }
```
