# System Utilities

该公共包暴露了 Raycast 的一些本机功能，以允许深度集成到用户的设置中。例如，您可以使用应用程序 API 检查是否安装了桌面应用程序，然后提供一个操作来深层链接到该应用程序。

## API 参考

### getApplications

返回可以打开该文件的所有应用程序。

#### 签名

```typescript
async function getApplications(path?: PathLike): Promise<Application[]>;
```

#### 例子

```typescript
import { getApplications } from "@raycast/api";

export default async function Command() {
  const installedApplications = await getApplications();
  console.log("The following applications are installed on your Mac:");
  console.log(installedApplications.map((a) => a.name).join(", "));
}
```

#### 参数

| 名称   | 描述                                       | 类型                                  |
| ---- | ---------------------------------------- | ----------------------------------- |
| path | 要获取应用程序的文件或文件夹的路径。如果未指定路径，则返回所有已安装的应用程序。 | [`PathLike`](utilities.md#pathlike) |

#### 返回

一个 [Application](utilities.md#application) 数组。

### getDefaultApplication

返回打开文件的默认应用程序。

#### 签名

```typescript
async function getDefaultApplication(path: PathLike): Promise<Application>;
```

#### 例子

```typescript
import { getDefaultApplication } from "@raycast/api";

export default async function Command() {
  const defaultApplication = await getDefaultApplication(__filename);
  console.log(`Default application for JavaScript is: ${defaultApplication.name}`);
}
```

#### 参数

| 名称                                     | 描述                   | 类型                                  |
| -------------------------------------- | -------------------- | ----------------------------------- |
| path<mark style="color:red;">\*</mark> | 要获取默认应用程序的文件或文件夹的路径。 | [`PathLike`](utilities.md#pathlike) |

#### 返回

打开文件的默认  [Application](utilities.md#application) ， 是一个 Promise。如果没有找到，promise 为 rejected。

### getFrontmostApplication

返回最前面的应用程序。

#### 签名

```typescript
async function getFrontmostApplication(): Promise<Application>;
```

#### 例子

```typescript
import { getFrontmostApplication } from "@raycast/api";

export default async function Command() => {
  const defaultApplication = await getFrontmostApplication();
  console.log(`The frontmost application is: ${frontmostApplication.name}`);
};
```

#### 返回

最前面的  [Application](utilities.md#application)，是一个 Promise。如果没有找到，promise 为 rejected。

### showInFinder

在 Finder 中显示文件或目录。

#### 签名

```typescript
async function showInFinder(path: PathLike): Promise<void>;
```

#### 例子

```typescript
import { showInFinder } from "@raycast/api";
import { homedir } from "os";
import { join } from "path";

export default async function Command() {
  await showInFinder(join(homedir(), "Downloads"));
}
```

#### 参数

| 名称                                     | 描述               | 类型                                  |
| -------------------------------------- | ---------------- | ----------------------------------- |
| path<mark style="color:red;">\*</mark> | 在 Finder 中显示的路径。 | [`PathLike`](utilities.md#pathlike) |

#### 返回

是一个 promise，当该项目在 Finder 中显示时状态为 resolve。

### trash

将文件或目录移至废纸篓。

#### 签名

```typescript
async function trash(path: PathLike | PathLike[]): Promise<void>;
```

#### 例子

```typescript
import { trash } from "@raycast/api";
import { writeFile } from "fs/promises";
import { homedir } from "os";
import { join } from "path";

export default async function Command() {
  const file = join(homedir(), "Desktop", "yolo.txt");
  await writeFile(file, "I will be deleted soon!");
  await trash(file);
}
```

#### 参数

| 名称                                     |  描述 | 类型                                                                            |
| -------------------------------------- | --- | ----------------------------------------------------------------------------- |
| path<mark style="color:red;">\*</mark> |     | [`PathLike`](utilities.md#pathlike) 或 [`PathLike`](utilities.md#pathlike)`[]` |

#### 返回

是一个promise，当所有文件都移至垃圾箱时状态为 resolve。

### open

使用默认应用程序或指定应用程序打开目标。

#### 签名

```typescript
async function open(target: string, application?: Application | string): Promise<void>;
```

#### 例子

```typescript
import { open } from "@raycast/api";

export default async function Command() {
  await open("https://www.raycast.com", "com.google.Chrome");
}
```

#### 参数

| 名称                                       | 描述                                                                                | 类型                                                   |
| ---------------------------------------- | --------------------------------------------------------------------------------- | ---------------------------------------------------- |
| target<mark style="color:red;">\*</mark> | 要打开的文件、文件夹或 URL。                                                                  | `string`                                             |
| application                              | 用于打开文件的应用程序名称。如果没有指定应用程序，则使用系统确定的默认应用程序打开指定的文件。请注意，您可以使用应用程序名称、应用程序标识符或应用程序的绝对路径。 | `string` 或 [`Application`](utilities.md#application) |

#### 返回

是一个 promise，当目标被打开时变为 resolve。

## 类型

### Application

代表系统上本地安装的应用程序对象。

它可用于在特定应用程序中打开文件或文件夹。使用 [getApplications](utilities.md#getapplications) 或 [getDefaultApplication](utilities.md#getdefaultapplication) 获取可以打开特定文件或文件夹的应用程序。

#### 属性

| 名称                                     | 描述                                         | 类型       |
| -------------------------------------- | ------------------------------------------ | -------- |
| name<mark style="color:red;">\*</mark> | 应用程序的显示名称。                                 | `string` |
| path<mark style="color:red;">\*</mark> | 应用程序包的绝对路径，例如 `/Applications/Raycast.app`, | `string` |
| bundleId                               | 应用程序的包标识符，例如`com.raycast.macos`。           | `string` |
|                                        |                                            |          |

### PathLike

```typescript
PathLike: string | Buffer | URL;
```

支持的路径类型。
