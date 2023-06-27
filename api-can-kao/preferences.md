# Preferences

使用 Preferences API 使您的扩展可配置。

Preferences 在每个命令的 [manifest](https://developers.raycast.com/information/manifest#preference-properties) 中配置或在扩展的上下文中共享。

在打开命令之前，用户需要设置所需的首选项。它们是确保扩展程序的用户正确设置所有内容的好方法。

## API 参考

### getPreferenceValues

用于访问已传递给命令的首选项值的函数。

每个首选项名称都映射到其值，并且定义的默认值作为备用值。

#### 签名

```typescript
function getPreferenceValues(): { [preferenceName: string]: any };
```

#### 例子

```typescript
import { getPreferenceValues } from "@raycast/api";

interface Preferences {
  name: string;
  bodyWeight?: string;
  bodyHeight?: string;
}

export default async function Command() {
  const preferences = getPreferenceValues<Preferences>();
  console.log(preferences);
}
```

#### 返回

一个对象，其中首选项名称作为属性键，键入的值作为属性值。

根据首选项的类型，其值的类型会有所不同。

| 首选项类型       | 值类型                                       |
| ----------- | ----------------------------------------- |
| `textfield` | `string`                                  |
| `password`  | `string`                                  |
| `checkbox`  | `boolean`                                 |
| `dropdown`  | `string`                                  |
| `appPicker` | [`Application`](utilities.md#application) |
| `file`      | `string`                                  |
| `directory` | `string`                                  |

### openExtensionPreferences

打开扩展程序的首选项面板。

#### 签名

```typescript
export declare function openExtensionPreferences(): Promise<void>;
```

#### 例子

```typescript
import { ActionPanel, Action, Detail, openExtensionPreferences } from "@raycast/api";

export default function Command() {
  const markdown = "API key incorrect. Please update it in extension preferences and try again.";

  return (
    <Detail
      markdown={markdown}
      actions={
        <ActionPanel>
          <Action title="Open Extension Preferences" onAction={openExtensionPreferences} />
        </ActionPanel>
      }
    />
  );
}
```

#### 返回

打开扩展首选项面板时，promise 为 resolve。

### openCommandPreferences

打开命令的首选项面板。

#### 签名

```typescript
export declare function openCommandPreferences(): Promise<void>;
```

#### 例子

```typescript
import { ActionPanel, Action, Detail, openCommandPreferences } from "@raycast/api";

export default function Command() {
  const markdown = "API key incorrect. Please update it in command preferences and try again.";

  return (
    <Detail
      markdown={markdown}
      actions={
        <ActionPanel>
          <Action title="Open Extension Preferences" onAction={openCommandPreferences} />
        </ActionPanel>
      }
    />
  );
}
```

#### 返回

打开命令的首选项面板时，promise 为 resolve。

## 类型

### Preferences

命令通过 [`getPreferenceValues`](https://developers.raycast.com/api-reference/preferences#getpreferencevalues) 函数接收其首选项的值。它是一个对象，其中首选项的名称作为键，其值作为属性的值。

根据首选项的类型，其值的类型会有所不同。

| 首选项类型       | 值类型                                       |
| ----------- | ----------------------------------------- |
| `textfield` | `string`                                  |
| `password`  | `string`                                  |
| `checkbox`  | `boolean`                                 |
| `dropdown`  | `string`                                  |
| `appPicker` | [`Application`](utilities.md#application) |
| `file`      | `string`                                  |
| `directory` | `string`                                  |

{% hint style="info" %}
Raycast 提供了一个名为 `Preferences` 的全局 TypeScript 命名空间，其中包含扩展的所有命令的首选项类型。

例如，如果名为 `show-todos` 的命令有一些首选项，则可以使用 `getPreferenceValues<Preferences.ShowTodos>()` 指定其 `getPreferenceValues` 的返回类型。这将确保命令中使用的类型与 manifest 保持同步。
{% endhint %}
