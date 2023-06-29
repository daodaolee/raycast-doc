# useSQL

一个钩子，它在本地 SQL 数据库上执行查询并返回与查询执行相对应的 [AsyncState](usesql.md#asyncstate) 。命令运行期间会保留最后一个值。

## 签名

```ts
function useSQL<T>(
  databasePath: string,
  query: string,
  options?: {
    permissionPriming?: string;
    execute?: boolean;
    onError?: (error: Error) => void;
    onData?: (data: T) => void;
    onWillExecute?: (args: string[]) -> void;
  }
): AsyncState<T> & {
  revalidate: () => void;
  mutate: MutatePromise<T | U | undefined>;
  permissionView: React.ReactNode | undefined;
};
```

### 参数

* `databasePath`是本地 SQL 数据库的路径。
* `query` 是在数据库上运行的 SQL 查询。

有几个配置项：

* `options.permissionPriming` 是一个字符串，用来解释为什么扩展需要完全磁盘访问。例如，Apple Notes 扩展使用 “这是搜索您的 Apple Notes 所必需的”。虽然它是可选的，但我们建议设置它以帮助用户理解。

包括 [usePromise](usepromise.md) 的选项：

* `options.execute` 是一个布尔值，指示是否实际执行该函数。 React 要求在渲染器上定义每个钩子，所以此标志使您能够在当前定义钩子，但要等到所有参数准备好才能执行该函数。
* `options.onError` 是执行失败时调用的函数。默认情况下，它将记录错误并显示失败 toast 以及重试操作。
* `options.onData` 是执行成功时调用的函数。
* `options.onWillExecute` 是一个在执行开始时调用的函数。

### 返回

返回一个对象，其 [AsyncState](usesql.md#asyncstate) 对应于函数的执行以及操作它的几个方法。

* `data`, `error`, `isLoading` - 查看 [AsyncState](usesql.md#asyncstate).
* `permissionView`是一个 React Node，当它存在时应该返回。它将提示用户授予完全磁盘访问权限（这是钩子运行所必需的）。
* `revalidate`是一种使用相同参数再次手动调用函数的方法。
* \`mutate\` 是一种包装异步更新的方法，在更新过程中对 `useSQL` 的数据进行操作。默认情况下，更新完成后数据将重新验证（例如，该函数将再次调用）。有关更多信息，请参阅 [变更和优化更新](usesql.md#bian-geng-he-you-hua-geng-xin)。

## 例子

```tsx
import { useSQL } from "@raycast/utils";
import { resolve } from "path";
import { homedir } from "os";

const NOTES_DB = resolve(homedir(), "Library/Group Containers/group.com.apple.notes/NoteStore.sqlite");
const notesQuery = `SELECT id, title FROM ...`;
type NoteItem = {
  id: string;
  title: string;
};

const Demo = () => {
  const { isLoading, data, permissionView } = useSQL<NoteItem>(NOTES_DB, notesQuery);

  if (permissionView) {
    return permissionView;
  }

  return (
    <List isLoading={isLoading}>
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
import { useSQL } from "@raycast/utils";

const NOTES_DB = resolve(homedir(), "Library/Group Containers/group.com.apple.notes/NoteStore.sqlite");
const notesQuery = `SELECT id, title FROM ...`;
type NoteItem = {
  id: string;
  title: string;
};

const Demo = () => {
  const { isLoading, data, mutate, permissionView } = useFetch("https://api.example");

  if (permissionView) {
    return permissionView;
  }

  const createNewNote = async () => {
    const toast = await showToast({ style: Toast.Style.Animated, title: "Creating new Note" });
    try {
      await mutate(
        // we are calling an API to do something
        somehowCreateANewNote(),
        {
          // but we are going to do it on our local data right away,
          // without waiting for the call to return
          optimisticUpdate(data) {
            return data?.concat([{ id: "" + Math.random(), title: "New Title" }]);
          },
        }
      );
      // yay, the API call worked!
      toast.style = Toast.Style.Success;
      toast.title = "Note created";
    } catch (err) {
      // oh, the API call didn't work :(
      // the data will automatically be rolled back to its previous value
      toast.style = Toast.Style.Failure;
      toast.title = "Could not create Note";
      toast.message = err.message;
    }
  };

  return (
    <List isLoading={isLoading}>
      {(data || []).map((item) => (
        <List.Item
          key={item.id}
          title={item.title}
          actions={
            <ActionPanel>
              <Action title="Create new Note" onAction={() => createNewNote()} />
            </ActionPanel>
          }
        />
      ))}
    </List>
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

一种包装异步更新的方法，可以控制在更新过程中如何更新 `useSQL` 数据

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
