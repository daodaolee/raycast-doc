# Storage

Storage API 可用于将数据存储在 Raycast 的 [本地加密数据库](../zi-liao/security.md#shu-ju-cun-chu) 中。

扩展中的所有命令都可以共享对存储数据的访问。扩展程序无法访问其他扩展程序的存储。

可以通过  [`LocalStorage.getItem`](storage.md#localstorage.getitem)、 [`LocalStorage.setItem`](storage.md#localstorage.setitem)或 [`LocalStorage.removeItem`](storage.md#localstorage.removeitem)  等函数来管理值。典型的用例是存储与用户相关的数据，例如输入的待办事项。

{% hint style="info" %}
这个 API 并不意味着存储大量数据。如果你想，请使用 [Node 的内置 API 来写入文件](https://nodejs.dev/learn/writing-files-with-nodejs)，例如到扩展的 [支持目录](environment.md#environment)。
{% endhint %}

## API 参考

### LocalStorage.getItem

获取给定键的值。

#### 签名

```typescript
async function getItem(key: string): Promise<Value | undefined>;
```

#### 例子

```typescript
import { LocalStorage } from "@raycast/api";

export default async function Command() {
  await LocalStorage.setItem("favorite-fruit", "apple");
  const item = await LocalStorage.getItem<string>("favorite-fruit");
  console.log(item);
}
```

#### 参数

| 名称                                    | 描述         | 类型       |
| ------------------------------------- | ---------- | -------- |
| key<mark style="color:red;">\*</mark> | 您想要获取其值的键。 | `string` |

#### 返回

获取给定键的值，是一个 Promise。如果键不存在，则返回 undefined。

### LocalStorage.setItem

存储给定键的值。

#### 签名

```typescript
async function setItem(key: string, value: Value): Promise<void>;
```

#### 例子

```typescript
import { LocalStorage } from "@raycast/api";

export default async function Command() {
  await LocalStorage.setItem("favorite-fruit", "apple");
  const item = await LocalStorage.getItem<string>("favorite-fruit");
  console.log(item);
}
```

#### 参数

| 名称                                      | 描述             | 类型                                                    |
| --------------------------------------- | -------------- | ----------------------------------------------------- |
| key<mark style="color:red;">\*</mark>   | 您想要新建或更新其值的键。  | `string`                                              |
| value<mark style="color:red;">\*</mark> | 您要为给定键新建或更新的值。 | [`LocalStorage.Value`](storage.md#localstorage.value) |

#### 返回

存储值，是一个 promise。

### LocalStorage.removeItem

删除给定键的值。

#### 签名

```typescript
async function removeItem(key: string): Promise<void>;
```

#### 例子

```typescript
import { LocalStorage } from "@raycast/api";

export default async function Command() {
  await LocalStorage.setItem("favorite-fruit", "apple");
  console.log(await LocalStorage.getItem<string>("favorite-fruit"));
  await LocalStorage.removeItem("favorite-fruit");
  console.log(await LocalStorage.getItem<string>("favorite-fruit"));
}
```

#### 参数

| 名称                                    | 描述      | Type     |
| ------------------------------------- | ------- | -------- |
| key<mark style="color:red;">\*</mark> | 删除其值的键。 | `string` |

#### 返回

删除值，是一个 promise。

### LocalStorage.allItems

获取扩展本地存储中的所有存储值。

#### 签名

```typescript
async function allItems(): Promise<Values>;
```

#### 例子

```typescript
import { LocalStorage } from "@raycast/api";

interface Values {
  todo: string;
  priority: number;
}

export default async function Command() {
  const items = await LocalStorage.allItems<Values>();
  console.log(`Local storage item count: ${Object.entries(items).length}`);
}
```

#### 返回

包含所有 Values 的对象，是一个 promise。

### LocalStorage.clear

删除所有值，是一个 promise。

#### 签名

```typescript
async function clear(): Promise<void>;
```

#### 例子

```typescript
import { LocalStorage } from "@raycast/api";

export default async function Command() {
  await LocalStorage.clear();
}
```

#### 返回

A Promise that resolves when all values are removed.

当所有值都被删除时，promise 为 resolve。

## 类型

### LocalStorage.Values

本地存储的值。

对于 type-safe 值，您可以定义自己的接口。使用本地存储项的键作为属性名称。

#### 属性

| 名称             | 类型    | 描述         |
| -------------- | ----- | ---------- |
| \[key: string] | `any` | 给定键的本地存储值。 |

### LocalStorage.Value

```typescript
Value: string | number | boolean;
```

支持的存储值类型。

#### 例子

```typescript
import { LocalStorage } from "@raycast/api";

export default async function Command() {
  // String
  await LocalStorage.setItem("favorite-fruit", "cherry");

  // Number
  await LocalStorage.setItem("fruit-basket-count", 3);

  // Boolean
  await LocalStorage.setItem("fruit-eaten-today", true);
}
```
