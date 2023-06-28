# useCachedState

返回一个有状态值的钩子，以及一个更新它的函数。它类似于 React 的 `useState`，但该值将在命令运行之间保留。

{% hint style="info" %}
该值需要是 JSON 可序列化的。
{% endhint %}

## 签名

```ts
function useCachedState<T>(
  key: string,
  initialState?: T,
  config?: {
    cacheNamespace?: string;
  }
): [T, (newState: T | ((prevState: T) => T)) => void];
```

### 参数

* `key` 是 state 的唯一标识符。这可用于跨组件 和/或 命令共享状态（使用相同密钥的钩子将共享相同的状态，例如更新一个将更新其他）。

有几个配置项：

* 如果缓存中还没有任何状态，则 `initialState` 是状态的初始值。
* `config.cacheNamespace` 是一个命名空间键的字符串。

## 例子

```tsx
import { List, ActionPanel, Action } from "@raycast/api";
import { useCachedState } from "@raycast/utils";

export default function Command() {
  const [showDetails, setShowDetails] = useCachedState("show-details", false);

  return (
    <List
      isShowingDetail={showDetails}
      actions={
        <ActionPanel>
          <Action title={showDetails ? "Hide Details" : "Show Details"} onAction={() => setShowDetails((x) => !x)} />
        </ActionPanel>
      }
    >
      ...
    </List>
  );
}
```
