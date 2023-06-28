# Navigation

## API 参考

### useNavigation

一个钩子，可让您在导航堆栈中推送和弹出视图组件。 您很可能不会经常使用这个钩子。要推送新组件，请使用推送操作。当用户按 ESC 时，我们会自动弹出到上一个组件。

#### 签名

```typescript
function useNavigation(): Navigation;
```

#### 例子

```typescript
import { Action, ActionPanel, Detail, useNavigation } from "@raycast/api";

function Ping() {
  const { push } = useNavigation();

  return (
    <Detail
      markdown="Ping"
      actions={
        <ActionPanel>
          <Action title="Push" onAction={() => push(<Pong />)} />
        </ActionPanel>
      }
    />
  );
}

function Pong() {
  const { pop } = useNavigation();

  return (
    <Detail
      markdown="Pong"
      actions={
        <ActionPanel>
          <Action title="Pop" onAction={pop} />
        </ActionPanel>
      }
    />
  );
}

export default function Command() {
  return <Ping />;
}
```

#### 返回

具有 [Navigation.push](navigation.md#navigation) 和 [Navigation.pop](navigation.md#navigation) 功能的导航对象。使用这些函数可以更改导航堆栈。

## 类型

### Navigation

用于执行推送和弹出操作的 [useNavigation](navigation.md#usenavigation) 钩子的返回类型。

#### 属性

| 名称                                     | 描述              | 类型                                     |
| -------------------------------------- | --------------- | -------------------------------------- |
| pop<mark style="color:red;">\*</mark>  | 从导航堆栈中弹出当前视图组件。 | `() => void`                           |
| push<mark style="color:red;">\*</mark> | 将新的视图组件推送到导航堆栈。 | `(component: React.ReactNode) => void` |
