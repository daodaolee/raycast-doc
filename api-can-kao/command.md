# Command

该公共包可与 Raycast 命令配合使用。

## API 参考

### launchCommand

启动另一个命令。如果该命令不存在，或者未启用，则会抛出错误。如果该命令是另一个扩展的一部分，则会向用户显示权限警报。如果您的命令需要根据用户交互打开另一个命令，或者要立即触发后台刷新（例如，当命令需要更新关联的菜单栏命令时），请使用此方法。

#### 签名

```typescript
export async function launchCommand(options: LaunchOptions): Promise<void>;
```

#### 例子

```typescript
import { launchCommand, LaunchType } from "@raycast/api";

export default async function Command() {
  await launchCommand({ name: "list", type: LaunchType.UserInitiated, context: { foo: "bar" } });
}
```

#### 参数

| 名称                                        | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | 类型                                          |
| ----------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------- |
| options<mark style="color:red;">\*</mark> | <p>具有以下属性的参数对象： <code>name</code>：扩展清单中定义的命令名称<br> <code>type</code>：<a href="https://developers.raycast.com/api-reference/environment#launchtype">LaunchType.UserInitiated</a> 或 <a href="https://developers.raycast.com/api-reference/environment#launchtype">LaunchType.Background</a> 参数：扩展清单中定义的参数属性和值的可选对象，例如： <code>{ "argument1" : "value1" }</code> </p><p><code>context</code>：自定义数据的任意对象，应传递给命令并可作为<code>environment.launchContext</code>访问；该对象必须是 JSON 可序列化的（支持日期和缓冲区）</p> | [`LaunchOptions`](command.md#launchoptions) |

#### 返回

当命令启动时解析的 Promise。 （请注意，这并不表示启动的命令已完成执行。）

### updateCommandMetadata

更新当前命令 manifest 中声明的​​属性值。请注意，目前仅支持 `subtitle`。传递 `null` 以清除自定义 `subtitle`。

{% hint style="info" %}
实际的 manifest 文件不会被修改，因此只要命令保持安装状态，更新就会适用。
{% endhint %}

#### 签名

```typescript
export async function updateCommandMetadata(metadata: { subtitle?: string | null }): Promise<void>;
```

#### 例子

```typescript
import { updateCommandMetadata } from "@raycast/api";

async function fetchUnreadNotificationCount() {
  return 10;
}

export default async function Command() {
  const count = await fetchUnreadNotificationCount();
  await updateCommandMetadata({ subtitle: `Unread Notifications: ${count}` });
}
```

#### 返回

当命令的元数据更新时解析的 Promise。

## 类型

### LaunchContext

表示命令启动传递的上下文对象。

### LaunchOptions

用于决定应启动哪个命令以及应接收哪些数据（参数、上下文）的参数对象。

#### IntraExtensionLaunchOptions

从同一扩展启动命令时可以使用的选项。

| 名称                                     | 描述                                                                                                          | 类型                                                                   |
| -------------------------------------- | ----------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| name<mark style="color:red;">\*</mark> | 扩展 manifest 中定义的命令名称                                                                                        | `string`                                                             |
| type<mark style="color:red;">\*</mark> | [LaunchType.UserInitiated](environment.md#launchtype) 或者 [LaunchType.Background](environment.md#launchtype) | [`LaunchType`](environment.md#launchtype)                            |
| arguments                              | 扩展 manifest 中定义的参数属性和值的可选对象，例如： `{ "argument1": "value1" }`                                                 | [`Arguments`](../zi-liao/lifecycle/arguments.md#arguments) 或者 `null` |
| context                                | 应传递给命令并可作为`environment.launchContext`访问的自定义数据的任意对象；该对象必须是 JSON 可序列化的（支持日期和缓冲区）                              | [`LaunchContext`](command.md#launchcontext) 或者 `null`                |
| fallbackText                           | 作为备用文本发送到命令的可选字符串                                                                                           | `string` 或者 `null`                                                   |

#### InterExtensionLaunchOptions

从不同扩展启动命令时可以使用的选项。

| 名称                                                  | 描述                                            | 类型       |
| --------------------------------------------------- | --------------------------------------------- | -------- |
| extensionName<mark style="color:red;">\*</mark>     | 从不同的扩展启动命令时，扩展名称（如扩展 manifest 中定义）是必需的        | `string` |
| ownerOrAuthorName<mark style="color:red;">\*</mark> | 从不同的扩展程序启动命令时，所有者或作者（如扩展程序 manifest 中所定义）是必需的 | `string` |
