# useFetch

获取 URL 并返回与执行获取相对应的  [AsyncState](usefetch.md#asyncstate)  的 Hook。命令运行期间会保留最后一个值。

## 签名

```ts
function useFetch<T, U>(
  url: string,
  options?: RequestInit & {
    parseResponse?: (response: Response) => Promise<T>;
    initialData?: U;
    keepPreviousData?: boolean;
    execute?: boolean;
    onError?: (error: Error) => void;
    onData?: (data: T) => void;
    onWillExecute?: (args: [string, RequestInit]) -> void;
  }
): AsyncState<T> & {
  revalidate: () => void;
  mutate: MutatePromise<T | U | undefined>;
};
```

### 参数

* `url` 是要获取的 URL 的字符串。

有几个配置项：

* `options` 扩展了 [`RequestInit`](https://github.com/nodejs/undici/blob/v5.7.0/types/fetch.d.ts#L103-L117)  ，允许指定要应用于请求的 body、headers 等。
* `options.parseResponse` 是一个函数，它接受 Response 作为参数并返回钩子将返回的数据。默认情况下，如果响应带有 JSON `Content-Type` 标头，则挂钩将返回 `response.json()`，否则返回 `response.text()`。

包括 [useCachedPromise](usecachedpromise.md) 的选项：

* `options.keepPreviousData` 是一个布尔值，告诉钩子保留以前的结果，而不是在缓存中没有新参数的情况下返回初始值。当用于列表的数据以避免闪烁时，这特别有用。有关详细信息，请参阅 [依赖于列表搜索文本的参数](usefetch.md#yi-lai-yu-lie-biao-sou-suo-wen-ben-de-can-shu)。

包括 [useCachedState](usecachedstate.md) 的选项：

* `options.initialData` 是状态的初始值（如果缓存中还没有任何状态）。

包括 [usePromise](usepromise.md) 的选项：

* `options.execute` 是一个布尔值，指示是否实际执行该函数。 React 要求在渲染器上定义每个钩子，所以此标志使您能够在当前定义钩子，但要等到所有参数准备好才能执行该函数。
* `options.onError` 是执行失败时调用的函数。默认情况下，它将记录错误并显示失败 toast 以及重试操作。
* `options.onData` 是执行成功时调用的函数。
* `options.onWillExecute` 是一个在执行开始时调用的函数。

### 返回

返回一个对象，其  [AsyncState](usefetch.md#asyncstate)  对应于函数的执行以及操作它的几个方法。

* `data`, `error`, `isLoading` - 查看 [AsyncState](usefetch.md#asyncstate).
* `revalidate` 是一个用相同参数再次手动调用函数的方法。
* `mutate` 是一个包装异步更新的方法，可以控制在更新过程中如何更新 `usePromise` 数据。默认情况下，更新完成后数据将重新验证（例如，该函数将再次调用）。有关更多信息，请参阅 [变更和优化更新](usefetch.md#bian-geng-he-you-hua-geng-xin)。

## 例子

```tsx
import { Detail, ActionPanel, Action } from "@raycast/api";
import { useFetch } from "@raycast/utils";

const Demo = () => {
  const { isLoading, data, revalidate } = useFetch("https://api.example");

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

## 依赖于列表搜索文本的参数

默认情况下，当传递给钩子的参数发生更改时，该函数将再次执行，并且这些参数的缓存值将立即返回。这意味着在尚未使用新参数的情况下，将返回初始数据。

此行为可能会导致一些闪烁（初始数据 -> 获取的数据 -> 参数更改 -> 初始数据 -> 获取的数据等）。为了避免这种情况，我们可以将 `keepPreviousData` 设置为 `true`，如果新参数的缓存为空（初始数据 -> 获取的数据 -> 参数更改 -> 获取的数据），钩子将保留最新获取的数据。

```tsx
import { useState } from "react";
import { List, ActionPanel, Action } from "@raycast/api";
import { useFetch } from "@raycast/utils";

const Demo = () => {
  const [searchText, setSearchText] = useState("");
  const { isLoading, data } = useFetch(`https://api.example?q=${searchText}`, {
    // to make sure the screen isn't flickering when the searchText changes
    keepPreviousData: true,
  });

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
import { useFetch } from "@raycast/utils";

const Demo = () => {
  const { isLoading, data, mutate } = useFetch("https://api.example");

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

一种包装异步更新的方法，可以控制在更新过程中如何更新 `useFetch` 数据。

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
