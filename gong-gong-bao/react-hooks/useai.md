# useAI

一个钩子，要求 AI 回答问题并返回与查询执行相对应的 [AsyncState](useai.md#asyncstate) 。

## 签名

```ts
function useAI(
  prompt: string,
  options?: {
    creativity?: AI.Creativity;
    model?: AI.Model;
    stream?: boolean;
    execute?: boolean;
    onError?: (error: Error) => void;
    onData?: (data: T) => void;
    onWillExecute?: (args: string[]) -> void;
  }
): AsyncState<String> & {
  revalidate: () => void;
};
```

### 参数

* `prompt`是询问AI的提示。

有几个配置项：

* `options.creativity` 是一个 0 到 2 之间的数字，用于控制答案的多变性。具体任务（例如修复语法）需要较少的多变性，而开放式问题（例如产生想法）则需要更多的多变性。
* \`options.model\` 是一个字符串，决定使用哪个 AI 模型来回答。
* `options.stream` 是一个布尔值，表示是流式传输答案还是仅在收到整个答案时更新数据。默认情况下，数据将被流式传输。

包括 [usePromise](usepromise.md) 的选项：

* `options.execute` 是一个布尔值，指示是否实际执行该函数。 React 要求在渲染器上定义每个钩子，所以此标志使您能够在当前定义钩子，但要等到所有参数准备好才能执行该函数。
* `options.onError` 是执行失败时调用的函数。默认情况下，它将记录错误并显示失败 toast 以及重试操作。
* `options.onData` 是执行成功时调用的函数。
* `options.onWillExecute` 是一个在执行开始时调用的函数。

### 返回

返回一个对象，其  [AsyncState](useai.md#asyncstate)  对应于函数的执行以及操作它的几个方法。

* `data`, `error`, `isLoading` - 查看 [AsyncState](useai.md#asyncstate).
* `revalidate` 是一个用相同参数再次手动调用函数的方法。

## 例子

```tsx
import { Detail } from "@raycast/api";
import { useAI } from "@raycast/utils";

export default function Command() {
  const { data, isLoading } = useAI("Suggest 5 jazz songs");

  return <Detail isLoading={isLoading} markdown={data} />;
}
```

## 类型

### AsyncState

与函数的执行状态对应的对象。

```ts
// Initial State
{
  isLoading: true, // or `false` if `options.execute` is `false`
  data: undefined,
  error: undefined
}

// Success State
{
  isLoading: false,
  data: string,
  error: undefined
}

// Error State
{
  isLoading: false,
  data: undefined,
  error: Error
}

// Reloading State
{
  isLoading: true,
  data: string | undefined,
  error: Error | undefined
}
```
