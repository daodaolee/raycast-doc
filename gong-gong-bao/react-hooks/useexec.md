# useExec

执行命令并返回与命令执行相对应的  [AsyncState](useexec.md#asyncstate)  的 钩子。命令运行期间会保留最后一个值。

## 签名

有两种使用钩子的方法。

&#x20;执行单个文件时应首选第一个。文件及其参数不必转义。

```ts
function useExec<T, U>(
  file: string,
  arguments: string[],
  options?: {
    shell?: boolean | string;
    stripFinalNewline?: boolean;
    cwd?: string;
    env?: NodeJS.ProcessEnv;
    encoding?: BufferEncoding | "buffer";
    input?: string | Buffer;
    timeout?: number;
    parseOutput?: ParseExecOutputHandler<T>;
    initialData?: U;
    keepPreviousData?: boolean;
    execute?: boolean;
    onError?: (error: Error) => void;
    onData?: (data: T) => void;
    onWillExecute?: (args: string[]) -> void;
  }
): AsyncState<T> & {
  revalidate: () => void;
  mutate: MutatePromise<T | U | undefined>;
};
```

第二个可用于执行更复杂的命令。文件和参数在单个命令字符串中指定。例如，`useExec('echo', ['Raycast'])` 与 `useExec('echo Raycast')` 相同。

如果文件或参数包含空格，则必须使用反斜杠对其进行转义。如果命令不是常量而是变量，例如使用`environment.supportPath` 或 `process.cwd()`，这一点尤其重要。除空格外，不需要 转义/引用。

如果命令使用特定于 `shell` 的功能（例如 `&&` 或 `||`），则必须使用 shell 选项，而不是使用简单文件后跟其参数。

```ts
function useExec<T, U>(
  command: string,
  options?: {
    shell?: boolean | string;
    stripFinalNewline?: boolean;
    cwd?: string;
    env?: NodeJS.ProcessEnv;
    encoding?: BufferEncoding | "buffer";
    input?: string | Buffer;
    timeout?: number;
    parseOutput?: ParseExecOutputHandler<T>;
    initialData?: U;
    keepPreviousData?: boolean;
    execute?: boolean;
    onError?: (error: Error) => void;
    onData?: (data: T) => void;
    onWillExecute?: (args: string[]) -> void;
  }
): AsyncState<T> & {
  revalidate: () => void;
  mutate: MutatePromise<T | U | undefined>;
};
```

### 参数

* `file` 是要执行的文件的路径。
* `arguments` 是作为参数传递给文件的字符串数组。

或

* `command` 是要执行的字符串。

有几个配置项：

*   `options.shell` 是一个布尔值或字符串，用于指示是否在 shell 内运行命令。如果为 `true`，则使用 `/bin/sh`。可以将不同的 shell 指定为字符串。 shell 应该理解 `-c` 开关。

    我们建议不要使用此选项，因为它是：

    * 不跨平台，鼓励特定于 shell 的语法。
    * 速度较慢，因为有额外的 shell 解析。
    * 不安全，可能被命令注入。
* `options.stripFinalNewline` 是一个布尔值，告诉钩子从输出中删除最后一个换行符。默认情况下，它会做。
* `options.cwd` 是一个字符串，用于指定子进程的当前工作目录。默认情况下，它将是 `process.cwd()`。
* `options.env` 是设置为子进程环境的键值对。它将自动从 `process.env` 扩展。
* `options.encoding` 是一个字符串，用于指定用于解码 `stdout` 和 `stderr` 输出的字符编码。如果设置为“buffer”，则 `stdout` 和 `stderr` 将是 Buffer 而不是字符串。
* `options.input` 是要写入文件的 `stdin` 字符串或 buffer。
* `options.timeout` 是一个数字。如果大于 0，则当子进程运行时间超过超时毫秒时，父进程将发送信号 `SIGTERM`。默认情况下，执行将在 10000 毫秒（例如 10 秒）后超时。
* `options.parseOutput` 是一个函数，它接受子进程的输出作为参数，并返回钩子将返回的数据 - 请参阅 [ParseExecOutputHandler](useexec.md#parseexecoutputhandler)。默认情况下，该钩子将返回 `stdout`

包括 [useCachedPromise](usecachedpromise.md) 的选项:

* `options.keepPreviousData` 是一个布尔值，告诉钩子保留以前的结果，而不是在缓存中没有新参数的情况下返回初始值。当用于列表的数据以避免闪烁时，这特别有用。有关详细信息，请参阅 [依赖于用户输入的参数](useexec.md#yi-lai-yu-yong-hu-shu-ru-de-can-shu)。

包括 [useCachedState](usecachedstate.md) 的选项：

* `options.initialData` 是状态的初始值（如果缓存中还没有任何状态）。

包括 [usePromise](usepromise.md) 的选项：

* `options.execute` 是一个布尔值，指示是否实际执行该函数。 React 要求在渲染器上定义每个钩子，所以此标志使您能够在当前定义钩子，但要等到所有参数准备好才能执行该函数。
* `options.onError` 是执行失败时调用的函数。默认情况下，它将记录错误并显示失败 toast 以及重试操作。
* `options.onData` 是执行成功时调用的函数。
* `options.onWillExecute` 是一个在执行开始时调用的函数。

### 返回

返回一个对象，其  [AsyncState](useexec.md#asyncstate)  对应于函数的执行以及操作它的几个方法。

* `data`, `error`, `isLoading` - 查看 [AsyncState](useexec.md#asyncstate).
* `revalidate` 是一个用相同参数再次手动调用函数的方法。
* `mutate` 是一个包装异步更新的方法，可以控制在更新过程中如何更新 `usePromise` 数据。默认情况下，更新完成后数据将重新验证（例如，该函数将再次调用）。有关更多信息，请参阅 [变更和优化更新](useexec.md#bian-geng-he-you-hua-geng-xin)。

## 例子

```tsx
import { List } from "@raycast/api";
import { useExec } from "@raycast/utils";
import { cpus } from "os";
import { useMemo } from "react";

const brewPath = cpus()[0].model.includes("Apple") ? "/opt/homebrew/bin/brew" : "/usr/local/bin/brew";

export default function () {
  const { isLoading, data } = useExec(brewPath, ["info", "--json=v2", "--installed"]);
  const results = useMemo<{ id: string; name: string }[]>(() => JSON.parse(data || "{}").formulae || [], [data]);

  return (
    <List isLoading={isLoading}>
      {results.map((item) => (
        <List.Item key={item.id} title={item.name} />
      ))}
    </List>
  );
}
```

## 依赖于用户输入的参数

默认情况下，当传递给钩子的参数发生更改时，该函数将再次执行，并且这些参数的缓存值将立即返回。这意味着在尚未使用新参数的情况下，将返回初始数据。

此行为可能会导致一些闪烁（初始数据 -> 获取的数据 -> 参数更改 -> 初始数据 -> 获取的数据等）。为了避免这种情况，我们可以将 `keepPreviousData` 设置为 `true`，如果新参数的缓存为空（初始数据 -> 获取的数据 -> 参数更改 -> 获取的数据），钩子将保留最新获取的数据。

```tsx
import { useState } from "react";
import { Detail, ActionPanel, Action } from "@raycast/api";
import { useFetch } from "@raycast/utils";

const Demo = () => {
  const [searchText, setSearchText] = useState("");
  const { isLoading, data } = useExec("brew", ["info", searchText]);

  return <Detail isLoading={isLoading} markdown={data} />;
};
```

{% hint style="info" %}
将用户输入传递给命令时，请务必小心使用 `shell` 选项，因为它可能存在潜在危险。
{% endhint %}

## 变更和优化更新

在优化更新中，UI 的行为就像在收到服务器的确认之前就成功拿到结果了一样 - 它认为它最终会得到正常的返回结果而不是错误。这可以提供更灵敏的用户体验。

您可以指定一个`optimisticUpdate` 函数来改变数据，从而异步更新引入的更改。

执行此操作时，您可以指定 `rollbackOnError` 函数，以便在异步更新失败时恢复数据。如果不指定，数据将自动回滚到之前的值（优化更新之前）。

```tsx
import { Detail, ActionPanel, Action, showToast, Toast } from "@raycast/api";
import { useFetch } from "@raycast/utils";

const Demo = () => {
  const { isLoading, data, revalidate } = useExec("brew", ["info", "--json=v2", "--installed"]);
  const results = useMemo<{}[]>(() => JSON.parse(data || "[]"), [data]);

  const installFoo = async () => {
    const toast = await showToast({ style: Toast.Style.Animated, title: "Installing Foo" });
    try {
      await mutate(
        // we are calling an API to do something
        installBrewCask("foo"),
        {
          // but we are going to do it on our local data right away,
          // without waiting for the call to return
          optimisticUpdate(data) {
            return data?.concat({ name: "foo", id: "foo" });
          },
        }
      );
      // yay, the API call worked!
      toast.style = Toast.Style.Success;
      toast.title = "Foo installed";
    } catch (err) {
      // oh, the API call didn't work :(
      // the data will automatically be rolled back to its previous value
      toast.style = Toast.Style.Failure;
      toast.title = "Could not install Foo";
      toast.message = err.message;
    }
  };

  return (
    <List isLoading={isLoading}>
      {(data || []).map((item) => (
        <List.Item
          key={item.id}
          title={item.name}
          actions={
            <ActionPanel>
              <Action title="Install Foo" onAction={() => installFoo()} />
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

一种包装异步更新的方法，可以控制在更新过程中如何更新  `useFetch` 数据。

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

### ParseExecOutputHandler

接受子进程的输出作为参数，返回钩子将返回的数据的函数。

```ts
export type ParseExecOutputHandler<T> = (args: {
  /** The output of the process on stdout. */
  stdout: string | Buffer; // depends on the encoding option
  /** The output of the process on stderr. */
  stderr: string | Buffer; // depends on the encoding option
  error?: Error | undefined;
  /** The numeric exit code of the process that was run. */
  exitCode: number | null;
  /** The name of the signal that was used to terminate the process. For example, SIGFPE. */
  signal: NodeJS.Signals | null;
  /** Whether the process timed out. */
  timedOut: boolean;
  /** The command that was run, for logging purposes. */
  command: string;
  /** The options passed to the child process, for logging purposes. */
  options?: ExecOptions | undefined;
}) => T;
```
