# useCachedPromise

一个钩子，它包装一个异步函数或返回 Promise 的函数，并返回与函数执行相对应的 [AsyncState](usecachedpromise.md#asyncstate)。命令运行期间会保留最后一个值。

{% hint style="info" %}
该值需要是 JSON 可序列化的。该函数被假定为不可变的（例如，更改它不会触发重新验证）。
{% endhint %}

## 签名

```ts
type Result<T> = `type of the returned value of the returned Promise`;

function useCachedPromise<T, U>(
  fn: T,
  args?: Parameters<T>,
  options?: {
    initialData?: U;
    keepPreviousData?: boolean;
    abortable?: MutableRefObject<AbortController | null | undefined>;
    execute?: boolean;
    onError?: (error: Error) => void;
    onData?: (data: Result<T>) => void;
    onWillExecute?: (args: Parameters<T>) -> void;
  }
): AsyncState<Result<T>> & {
  revalidate: () => void;
  mutate: MutatePromise<Result<T> | U>;
};
```

### 参数

* `fn` 是一个异步函数或返回 Promise 的函数。
* `args` 是传递给函数的参数数组。每次它们发生变化时，该函数都会再次执行。如果函数不需要任何参数，则可以省略该数组。

有几个配置项：

* `options.keepPreviousData` 是一个布尔值，告诉钩子保留以前的结果，而不是在缓存中没有新参数的情况下返回初始值。当用于列表的数据以避免闪烁时，这特别有用。有关详细信息，请参阅[依赖于列表搜索文本的 Promise 参数](https://developers.raycast.com/utilities/react-hooks/usecachedpromise#promise-argument-dependent-on-list-search-text)。

包括 [useCachedState](https://developers.raycast.com/utilities/react-hooks/usecachedstate) 的选项：

* `options.initialData` 是状态的初始值（如果缓存中还没有任何状态）。

包括 [usePromise](https://developers.raycast.com/utilities/react-hooks/usepromise) 的选项：

* `options.abortable` 是对 [`AbortController`](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) 的引用，用于在触发新调用时取消先前的调用。
* `options.execute` 是一个布尔值，指示是否实际执行该函数。 React 要求在渲染器上定义每个钩子，所以此标志使您能够在当前定义钩子，但要等到所有参数准备好才能执行该函数。
* `options.onError` 是执行失败时调用的函数。默认情况下，它将记录错误并显示失败 toast 以及重试操作。
* `options.onData` 是执行成功时调用的函数。
* `options.onWillExecute` 是一个在执行开始时调用的函数。

### 返回

返回一个对象，其  [AsyncState](usecachedpromise.md#asyncstate)  对应于函数的执行以及操作它的几个方法。

* `data`, `error`, `isLoading` - 查看 [AsyncState](usecachedpromise.md#asyncstate).
* `revalidate` 是一个用相同参数再次手动调用函数的方法。
* `mutate` 是一个包装异步更新的方法，可以控制在更新过程中如何更新 `usePromise` 数据。默认情况下，更新完成后数据将重新验证（例如，该函数将再次调用）。有关更多信息，请参阅 [变更和优化更新](https://developers.raycast.com/utilities/react-hooks/usepromise#mutation-and-optimistic-updates)。

## 例子

```tsx
import { Detail, ActionPanel, Action } from "@raycast/api";
import { useCachedPromise } from "@raycast/utils";

const Demo = () => {
  const abortable = useRef<AbortController>();
  const { isLoading, data, revalidate } = useCachedPromise(
    async (url: string) => {
      const response = await fetch(url);
      const result = await response.text();
      return result;
    },
    ["https://api.example"],
    {
      initialData: "Some Text",
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

## 取决于列表搜索文本的 Promise 参数

默认情况下，当传递给钩子的参数发生更改时，该函数将再次执行，并且这些参数的缓存值将立即返回。这意味着在尚未使用新参数的情况下，将返回初始数据。

此行为可能会导致一些闪烁（初始数据 -> 获取的数据 -> 参数更改 -> 初始数据 -> 获取的数据等）。为了避免这种情况，我们可以将 `keepPreviousData` 设置为 `true`，如果新参数的缓存为空（初始数据 -> 获取的数据 -> 参数更改 -> 获取的数据），钩子将保留最新获取的数据。

```tsx
import { useState } from "react";
import { List, ActionPanel, Action } from "@raycast/api";
import { useCachedPromise } from "@raycast/utils";

const Demo = () => {
  const [searchText, setSearchText] = useState("");
  const { isLoading, data } = useCachedPromise(
    async (url: string) => {
      const response = await fetch(url);
      const result = await response.text();
      return result;
    },
    ["https://api.example"],
    {
      // to make sure the screen isn't flickering when the searchText changes
      keepPreviousData: true,
    }
  );

  return (
    <List isLoading={isLoading} searchText={searchText} onSearchTextChange={setSearchText} throttle>
      {(data || []).map((item) => (
        <List.Item key={item.id} title={item.title} />
      ))}
    </List>
  );
};
```

## 变更和优化更新

在优化更新中，UI 的行为就像在收到服务器的确认之前就成功拿到结果了一样 - 它认为它最终会得到正常的返回结果而不是错误。这可以提供更灵敏的用户体验。

您可以指定一个`optimisticUpdate` 函数来改变数据，从而异步更新引入的更改。

执行此操作时，您可以指定 `rollbackOnError` 函数，以便在异步更新失败时恢复数据。如果不指定，数据将自动回滚到之前的值（优化更新之前）。

```tsx
import { Detail, ActionPanel, Action, showToast, Toast } from "@raycast/api";
import { useCachedPromise } from "@raycast/utils";

const Demo = () => {
  const { isLoading, data, mutate } = useCachedPromise(
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

一种包装异步更新的方法，可以控制在更新过程中如何更新 `useCachedPromise` 数据

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
