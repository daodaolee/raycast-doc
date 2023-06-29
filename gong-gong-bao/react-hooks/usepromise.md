# usePromise

一个钩子，它包装一个异步函数或返回 Promise 函数，并返回与函数执行相对应的 [AsyncState](usepromise.md#asyncstate)。

{% hint style="info" %}
该函数被假定为不可变的（例如，更改它不会触发重新验证）。
{% endhint %}

## 签名

```ts
type Result<T> = `type of the returned value of the returned Promise`;

function usePromise<T>(
  fn: T,
  args?: Parameters<T>,
  options?: {
    abortable?: MutableRefObject<AbortController | null | undefined>;
    execute?: boolean;
    onError?: (error: Error) => void;
    onData?: (data: Result<T>) => void;
    onWillExecute?: (args: Parameters<T>) -> void;
  }
): AsyncState<Result<T>> & {
  revalidate: () => void;
  mutate: MutatePromise<Result<T> | undefined>;
};
```

### 参数

* `fn` 是一个异步函数或返回 Promise 的函数。
* `args` 是传递给函数的参数数组。每次它们发生变化时，该函数都会再次执行。如果函数不需要任何参数，则可以省略该数组。

有几个配置项：

* `options.abortable` 是对 [`AbortController`](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) 的引用，用于在触发新调用时取消先前的调用。
* `options.execute` 是一个布尔值，表示是否执行该函数。React 要求在渲染器上定义每个钩子，所以此标识使您能够在当前定义钩子，但要等到所有参数准备好才能执行该函数。
* `options.onError` 是执行失败时调用的函数。默认情况下，它将记录错误并显示失败 toast 以及重试操作。
* `options.onData` 是执行成功时调用的函数。
* `options.onWillExecute` 是一个在执行开始时调用的函数。

### 返回

返回一个对象，其 [AsyncState](usepromise.md#asyncstate) 对应于函数的执行以及操作它的几个方法。

* `data`, `error`, `isLoading` - 查看 [AsyncState](usepromise.md#asyncstate)。
* `revalidate` 是一个用相同参数再次手动调用函数的方法。
* `mutate` 是一个包装异步更新的方法，可以控制在更新过程中如何更新 `usePromise` 数据。默认情况下，更新完成后数据将重新验证（例如，该函数将再次调用）。有关更多信息，请参阅 [变更和优化更新](usepromise.md#bian-geng-he-you-hua-geng-xin)。

## 例子

```tsx
import { Detail, ActionPanel, Action } from "@raycast/api";
import { usePromise } from "@raycast/utils";

const Demo = () => {
  const abortable = useRef<AbortController>();
  const { isLoading, data, revalidate } = usePromise(
    async (url: string) => {
      const response = await fetch(url, { signal: abortable.current?.signal });
      const result = await response.text();
      return result;
    },
    ["https://api.example"],
    {
      abortable,
    }
  );

  return (
    <Detail
      isLoading={isLoading}
      markdown={data}
      actions={
        <ActionPanel>
          <Action title="Reload" onAction={() => revalidate()} />
        </ActionPanel>
      }
    />
  );
};
```

## 变更和优化更新

在优化更新中，UI 的行为就像在收到服务器的确认之前就成功拿到结果了一样 - 它认为它最终会得到正常的返回结果而不是错误。这可以提供更灵敏的用户体验。

您可以指定一个`optimisticUpdate` 函数来改变数据，从而异步更新引入的更改。

执行此操作时，您可以指定 `rollbackOnError` 函数，以便在异步更新失败时恢复数据。如果不指定，数据将自动回滚到之前的值（优化更新之前）。

```tsx
import { Detail, ActionPanel, Action, showToast, Toast } from "@raycast/api";
import { usePromise } from "@raycast/utils";

const Demo = () => {
  const { isLoading, data, mutate } = usePromise(
    async (url: string) => {
      const response = await fetch(url);
      const result = await response.text();
      return result;
    },
    ["https://api.example"]
  );

  const appendFoo = async () => {
    const toast = await showToast({ style: Toast.Style.Animated, title: "Appending Foo" });
    try {
      await mutate(
        // we are calling an API to do something
        fetch("https://api.example/append-foo"),
        {
          // but we are going to do it on our local data right away,
          // without waiting for the call to return
          optimisticUpdate(data) {
            return data + "foo";
          },
        }
      );
      // yay, the API call worked!
      toast.style = Toast.Style.Success;
      toast.title = "Foo appended";
    } catch (err) {
      // oh, the API call didn't work :(
      // the data will automatically be rolled back to its previous value
      toast.style = Toast.Style.Failure;
      toast.title = "Could not append Foo";
      toast.message = err.message;
    }
  };

  return (
    <Detail
      isLoading={isLoading}
      markdown={data}
      actions={
        <ActionPanel>
          <Action title="Append Foo" onAction={() => appendFoo()} />
        </ActionPanel>
      }
    />
  );
};
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
  data: T,
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
  data: T | undefined,
  error: Error | undefined
}
```

### MutatePromise

一种包装异步更新的方法，可以控制在更新过程中如何更新 `usePromise` 数据

```ts
export type MutatePromise<T> = (
  asyncUpdate?: Promise<any>,
  options?: {
    optimisticUpdate?: (data: T) => T;
    rollbackOnError?: boolean | ((data: T) => T);
    shouldRevalidateAfter?: boolean;
  }
) => Promise<any>;
```
