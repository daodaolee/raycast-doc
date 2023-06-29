# HUD

当用户执行的操作会产生关闭 Raycast 的副作用时（例如在 [Clipboard](../clipboard.md) 中复制某些内容时），您可以使用 HUD 来确认该操作是否正常工作。

![](../../.gitbook/assets/hud.png)

## API 参考

### showHUD

HUD 会自动隐藏主窗口并在屏幕底部显示一条简短的消息。

#### 签名

```typescript
async function showHUD(title: string): Promise<void>;
```

#### 例子

```typescript
import { showHUD } from "@raycast/api";

export default async function Command() {
  await showHUD("Hey there 👋");
}
```

#### 参数

| 名称                                      | 描述             | 类型       |
| --------------------------------------- | -------------- | -------- |
| title<mark style="color:red;">\*</mark> | 将在 HUD 中显示的标题。 | `string` |

#### 返回

显示 HUD 时 resolves 的 Promise。
