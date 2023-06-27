# 执行 AppleScript

执行 AppleScript 脚本的函数。

## 签名

有两种方法可以使用该功能。

执行静态脚本时应首选第一个。

```ts
function runAppleScript<T>(
  script: string,
  options?: {
    humanReadableOutput?: boolean;
    language?: "AppleScript" | "JavaScript";
    signal?: AbortSignal;
    timeout?: number;
    parseOutput?: ParseExecOutputHandler<T>;
  }
): Promise<T>;
```

第二个可用于将参数传递给脚本。

```ts
function runAppleScript<T>(
  script: string,
  arguments: string[],
  options?: {
    humanReadableOutput?: boolean;
    language?: "AppleScript" | "JavaScript";
    signal?: AbortSignal;
    timeout?: number;
    parseOutput?: ParseExecOutputHandler<T>;
  }
): Promise<T>;
```

### 参数

* `script` 是要执行的脚本。
* `arguments` 是作为参数传递给脚本的字符串数组。

有几个选项：

* `options. humanReadableOutput` 是一个布尔值，告诉脚本以什么形式输出。默认情况下，`runAppleScript` 以人类可读的形式返回其结果：字符串周围没有引号，字符不会转义，列表和记录的大括号被省略等。这通常更有用，但可能会引起歧义。例如，列表 `{"foo", "bar"}` 和 `{{"foo", {"bar"}}}` 都将显示为 ‘foo, bar’。要以可重新编译为相同值的明确形式查看结果，请将 `humanReadableOutput` 设置为 `false`。
* `options.language` 是一个字符串，用于指定脚本是使用 [`AppleScript`](https://developer.apple.com/library/archive/documentation/AppleScript/Conceptual/AppleScriptLangGuide/introduction/ASLR\_intro.html#//apple\_ref/doc/uid/TP40000983) 还是 JavaScript。默认情况下，它会假设它使用 `AppleScript`。
* `options.signal` 是一个 Signal 对象，允许您在需要时通过 AbortController 对象中止请求。
* `options.timeout` 是一个数字。如果大于 0，则当脚本运行时间超过超时毫秒时，父级将发送信号 \`SIGTERM\`。默认情况下，执行将在 10000 毫秒（例如 10 秒）后超时。
* `options.parseOutput` 是一个函数，它接受脚本的输出作为参数并返回钩子将返回的数据 - 请参阅 [ParseExecOutputHandler](runapplescript.md#parseexecoutputhandler)。默认情况下，该函数将以字符串形式返回 `stdout`。

### 返回

返回一个默认解析为字符串的 Promise。您可以通过传递 `options.parseOutput` 来控制它返回的内容。

## 例子

```tsx
import { showHUD } from "@raycast/api";
import { runAppleScript } from "@raycast/utils";

export default async function () {
  const res = await runAppleScript(
    `
on run argv
  return "hello, " & item 1 of argv & "."
end run
`,
    ["world"]
  );
  await showHUD(res);
}
```

## 类型

### ParseExecOutputHandler

接受脚本输出作为参数并返回函数将返回的数据的函数。

```ts
export type ParseExecOutputHandler<T> = (args: {
  /** The output of the script on stdout. */
  stdout: string;
  /** The output of the script on stderr. */
  stderr: string;
  error?: Error | undefined;
  /** The numeric exit code of the process that was run. */
  exitCode: number | null;
  /** The name of the signal that was used to terminate the process. For example, SIGFPE. */
  signal: NodeJS.Signals | null;
  /** Whether the process timed out. */
  timedOut: boolean;
  /** The command that was run, for logging purposes. */
  command: string;
  /** The options passed to the script, for logging purposes. */
  options?: ExecOptions | undefined;
}) => T;
```
