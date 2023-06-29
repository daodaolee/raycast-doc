# Alert

当用户执行重要操作时（例如，不可逆地删除某些内容时），您可以使用 `confirmAlert` 请求确认。

![](../../.gitbook/assets/alert.png)

## API 参考

### confirmAlert

创建并显示带有给定 [选项](alert.md#alert.options) 的确认告警。

#### 签名

```typescript
async function confirmAlert(options: Alert.Options): Promise<boolean>;
```

#### 例子

```typescript
import { confirmAlert } from "@raycast/api";

export default async function Command() {
  if (await confirmAlert({ title: "Are you sure?" })) {
    console.log("confirmed");
    // do something
  } else {
    console.log("canceled");
  }
}
```

#### 参数

| 名称                                        | 描述         | 类型                                        |
| ----------------------------------------- | ---------- | ----------------------------------------- |
| options<mark style="color:red;">\*</mark> | 用于创建告警的选项。 | [`Alert.Options`](alert.md#alert.options) |

#### 返回

当用户触发其中一个操作时解析为布尔值的 Promise。对于 primary 操作，它为 `true`；对于 dismiss 操作，它为 `false`。

## 类型

### Alert.Options

创建告警的选项。

#### 例子

```typescript
import { Alert, confirmAlert } from "@raycast/api";

export default async function Command() {
  const options: Alert.Options = {
    title: "Finished cooking",
    message: "Delicious pasta for lunch",
    primaryAction: {
      title: "Do something",
      onAction: () => {
        // while you can register a handler for an action, it's more elegant
        // to use the `if (await confirmAlert(...)) { ... }` pattern
        console.log("The alert action has been triggered");
      },
    },
  };
  await confirmAlert(options);
}
```

#### 属性

| 名称                                      | 描述                              | 类型                                                                         |
| --------------------------------------- | ------------------------------- | -------------------------------------------------------------------------- |
| title<mark style="color:red;">\*</mark> | 告警的标题。显示在图标下方。                  | `string`                                                                   |
| dismissAction                           | 消除告警的操作。当用户执行此操作时通常不应该有任何副作用。   | [`Alert.ActionOptions`](alert.md#alert.actionoptions)                      |
| icon                                    | 用于说明操作的告警图标。显示在顶部。              | [`Image.ImageLike`](../user-interface/icons-and-images.md#image.imagelike) |
| message                                 | 告警的附加消息。有助于显示更多信息，例如破坏性操作的确认消息。 | `string`                                                                   |
| primaryAction                           | 用户可以采取的 primary操作。              | [`Alert.ActionOptions`](alert.md#alert.actionoptions)                      |

### Alert.ActionOptions

用于创建告警操作的选项。

#### 属性

| 名称                                      | 描述      | 类型                                                |
| --------------------------------------- | ------- | ------------------------------------------------- |
| title<mark style="color:red;">\*</mark> | 操作的标题   | `string`                                          |
| style                                   | 操作的样式   | [`Alert.ActionStyle`](alert.md#alert.actionstyle) |
| onAction                                | 触发操作的回调 | `() => void`                                      |

### Alert.ActionStyle

定义告警操作的视觉样式。

使用 [Alert.ActionStyle.Default](alert.md#alert.actionstyle) 确认  positive 操作。使用 [Alert.ActionStyle.Destructive](alert.md#alert.actionstyle) 确认 destructive  操作（例如删除文件）。

#### 枚举成员

| 名称          | 值                                                       |
| ----------- | ------------------------------------------------------- |
| Default     | ![](../../.gitbook/assets/alert-action-default.png)     |
| Destructive | ![](../../.gitbook/assets/alert-action-destructive.png) |
| Cancel      | ![](../../.gitbook/assets/alert-action-cancel.png)      |
