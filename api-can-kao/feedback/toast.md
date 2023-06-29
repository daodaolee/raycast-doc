# Toast

当发生异步操作或引发错误时，通常最好让用户了解情况。Toast 就是为此而做的。

此外，Toast 可以有一些与其相关操作相关的操作。例如，您可以提供一种方法来取消异步操作、撤消操作或复制错误的堆栈跟踪。

![](../../.gitbook/assets/toast.png)

## API 参考

### showToast

创建并显示具有给定 [选项](toast.md#toast.options) 的 Toast。

#### 签名

```typescript
async function showToast(options: Toast.Options): Promise<Toast>;
```

#### 例子

```typescript
import { showToast, Toast } from "@raycast/api";

export default async function Command() {
  const success = false;

  if (success) {
    await showToast({ title: "Dinner is ready", message: "Pizza margherita" });
  } else {
    await showToast({
      style: Toast.Style.Failure,
      title: "Dinner isn't ready",
      message: "Pizza dropped on the floor",
    });
  }
}
```

当显示动画 Toast 时，您可以稍后更新它：

```typescript
import { showToast, Toast } from "@raycast/api";
import { setTimeout } from "timers/promises";

export default async function Command() {
  const toast = await showToast({
    style: Toast.Style.Animated,
    title: "Uploading image",
  });

  try {
    // upload the image
    await setTimeout(1000);

    toast.style = Toast.Style.Success;
    toast.title = "Uploaded image";
  } catch (err) {
    toast.style = Toast.Style.Failure;
    toast.title = "Failed to upload image";
    if (err instanceof Error) {
      toast.message = err.message;
    }
  }
}
```

#### 参数

| 名称                                        | 描述             | 类型                                        |
| ----------------------------------------- | -------------- | ----------------------------------------- |
| options<mark style="color:red;">\*</mark> | 自定义 Toast 的选项。 | [`Alert.Options`](alert.md#alert.options) |

#### 返回

展示 Toast  状态为 resolves 的 Promise， Toast 可用于更改或隐藏它。

## 类型

### Toast

具有特定样式、标题和消息的 Toast。

使用 [showToast](toast.md#showtoast) 创建并显示 Toast。

#### 属性

| 名称                                                | 描述                                   | 类型                                                          |
| ------------------------------------------------- | ------------------------------------ | ----------------------------------------------------------- |
| message<mark style="color:red;">\*</mark>         | Toast 的附加消息。有助于显示更多信息，例如新创建的资源的标识符。  | `string`                                                    |
| primaryAction<mark style="color:red;">\*</mark>   | 用户将鼠标悬停在 Toast 上时可以执行的 primary 操作。   | [`Alert.ActionOptions`](alert.md#alert.actionoptions)       |
| secondaryAction<mark style="color:red;">\*</mark> | 用户将鼠标悬停在 Toast 上时可以执行的 secondary 操作。 | [`Alert.ActionOptions`](alert.md#alert.actionoptions)       |
| style<mark style="color:red;">\*</mark>           | Toast 的样式。                           | [`Action.Style`](../user-interface/actions.md#action.style) |
| title<mark style="color:red;">\*</mark>           | Toast 的标题，显示在顶部。                     | `string`                                                    |

#### 方法

| 名称   | 类型                    | 描述       |
| ---- | --------------------- | -------- |
| hide | `() => Promise<void>` | 隐藏 Toast |
| show | `() => Promise<void>` | 显示 Toast |

### Toast.Options

创建 Toast 的配置项。

#### 例子

```typescript
import { showToast, Toast } from "@raycast/api";

export default async function Command() {
  const options: Toast.Options = {
    style: Toast.Style.Success,
    title: "Finished cooking",
    message: "Delicious pasta for lunch",
    primaryAction: {
      title: "Do something",
      onAction: (toast) => {
        console.log("The toast action has been triggered");
        toast.hide();
      },
    },
  };
  await showToast(options);
}
```

#### 属性

| 名称                                      | 描述                                   | 类型                                                          |
| --------------------------------------- | ------------------------------------ | ----------------------------------------------------------- |
| title<mark style="color:red;">\*</mark> | Toast 的标题，显示在顶部。                     | `string`                                                    |
| message                                 | Toast 的附加消息。有助于显示更多信息，例如新创建的资源的标识符。  | `string`                                                    |
| primaryAction                           | 用户将鼠标悬停在 Toast 上时可以执行的 primary 操作。   | [`Alert.ActionOptions`](alert.md#alert.actionoptions)       |
| secondaryAction                         | 用户将鼠标悬停在 Toast 上时可以执行的 secondary 操作。 | [`Alert.ActionOptions`](alert.md#alert.actionoptions)       |
| style                                   | Toast 的样式。                           | [`Action.Style`](../user-interface/actions.md#action.style) |

### Toast.Style

定义 Toast 的样式。

使用 [Toast.Style.Success](toast.md#toast.style) 进行确认，使用 [Toast.Style.Failure](toast.md#toast.style) 显示错误。当您的 Toast 显示到进程完成时，请使用 [Toast.Style.Animated](toast.md#toast.style)。您可以稍后使用 Toast.hide 隐藏它或更新现有 Toast 的属性。

#### 枚举成员

| 名称       | 值                                             |
| -------- | --------------------------------------------- |
| Animated | ![](../../.gitbook/assets/toast-animated.png) |
| Success  | ![](../../.gitbook/assets/toast-success.png)  |
| Failure  | ![](../../.gitbook/assets/toast-failure.png)  |

### Toast.ActionOptions

用于创建 Toast 操作的选项。

#### 属性

| 名称                                         | 描述          | 类型                                                      |
| ------------------------------------------ | ----------- | ------------------------------------------------------- |
| title<mark style="color:red;">\*</mark>    | 操作的标题。      | `string`                                                |
| onAction<mark style="color:red;">\*</mark> | 触发操作时调用的回调。 | `(toast:` [`Toast`](toast.md#toast)`) => void`          |
| shortcut                                   | 操作的键盘快捷键。   | [`Keyboard.Shortcut`](../keyboard.md#keyboard.shortcut) |
