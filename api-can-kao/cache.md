# Cache

缓存抽象，将数据存储在磁盘上并支持 LRU（最近最少使用）访问。由于扩展只能消耗最大数量。堆内存大小，缓存仅在内存中维护轻量级索引，并将实际数据存储在扩展支持目录中磁盘上的单独文件中。

## API 参考

### Cache

`Cache` 类提供 CRUD 样式的方法（get、set、remove）来基于键同步更新和检索数据。数据必须是字符串，由客户端决定使用哪种序列化格式。典型的用例是使用 `JSON.stringify` 和 `JSON.parse`。

默认情况下，缓存在扩展的命令之间共享。如果需要，请使用 [Cache.Options](cache.md#cache.options) 为每个命令配置命名空间（例如，将其设置为 [`environment.commandName`](environment.md)）。

#### 签名

```typescript
constructor(options: Cache.Options): Cache
```

#### 例子

```typescript
import { List, Cache } from "@raycast/api";

type Item = { id: string; title: string };
const cache = new Cache();
cache.set("items", JSON.stringify([{ id: "1", title: "Item 1" }]));

export default function Command() {
  const cached = cache.get("items");
  const items: Item[] = cached ? JSON.parse(cached) : [];

  return (
    <List>
      {items.map((item) => (
        <List.Item key={item.id} title={item.title} />
      ))}
    </List>
  );
}
```

#### 属性

| 名称                                        | 描述                           | 类型        |
| ----------------------------------------- | ---------------------------- | --------- |
| isEmpty<mark style="color:red;">\*</mark> | 缓存是空就返回 `true` ，否则为 `false`  | `boolean` |

#### 方法

| 方法                                                                                        |
| ----------------------------------------------------------------------------------------- |
| [`get(key: string): string \| undefined`](cache.md#cache-get)                             |
| [`has(key: string): boolean`](cache.md#cache-has)                                         |
| [`set(key: string, data: string): void`](cache.md#cache-set)                              |
| [`remove(key: string): boolean`](cache.md#cache-remove)                                   |
| [`clear(options = { notifySubscribers: true }): void`](cache.md#cache-clear)              |
| [`subscribe(subscriber: Cache.Subscriber): Cache.Subscription`](cache.md#cache-subscribe) |

### Cache#get

返回给定键的数据。如果该键没有数据，则返回 `undefined`。如果您只想检查密钥是否存在，请使用  [has](cache.md#cache-has)。

#### 签名

```typescript
get(key: string): string | undefined
```

#### 参数

| 名称                                    | 描述         | 类型       |
| ------------------------------------- | ---------- | -------- |
| key<mark style="color:red;">\*</mark> | 缓存条目的 key。 | `string` |

### Cache#has

如果键的数据存在则返回 `true`，否则返回 `false`。您可以使用此方法来检查条目，而不会影响 LRU 访问。

#### 签名

```typescript
has(key: string): boolean
```

#### 参数

| 名称                                    | 描述         | 类型       |
| ------------------------------------- | ---------- | -------- |
| key<mark style="color:red;">\*</mark> | 缓存条目的 key。 | `string` |

### Cache#set

设置给定 key 的数据。如果数据超出配置的容量，则最近最少使用的条目将被删除。这也会通知注册订阅者（请参考 [subscribe](cache.md#cache-subscribe)）。

#### 签名

```typescript
set(key: string, data: string)
```

#### 参数

| 名称                                     | 描述           | Type     |
| -------------------------------------- | ------------ | -------- |
| key<mark style="color:red;">\*</mark>  | 缓存条目的 key。   | `string` |
| data<mark style="color:red;">\*</mark> | 缓存条目的字符串化数据。 | `string` |

### Cache#remove

删除给定键的数据。这也会通知注册订阅者（请参考 [subscribe](cache.md#cache-subscribe)）。如果删除了键的数据，则返回 `true`，否则返回 `false`。

#### 签名

```typescript
remove(key: string): boolean
```

### Cache#clear

清除所有存储的数据。这也会通知注册订阅者（请参考 [subscribe](cache.md#cache-subscribe)），除非 `notifySubscribers` 选项设置为 `false`。

#### 签名

```typescript
clear((options = { notifySubscribers: true }));
```

#### 参数

| 名称      | 描述                                                            | 类型       |
| ------- | ------------------------------------------------------------- | -------- |
| options | 具有 `notifySubscribers` 属性的选项。默认为 `true`；设置为 `false` 会禁用订阅者通知。 | `object` |

### Cache#subscribe

注册一个新订阅者，当缓存数据被设置或删除时，该订阅者会收到通知。返回一个可以调用以删除订阅者的函数。

#### 签名

```typescript
subscribe(subscriber: Cache.Subscriber): Cache.Subscription
```

#### 参数

|  名称        | 描述                                                                       | 类型                                              |
| ---------- | ------------------------------------------------------------------------ | ----------------------------------------------- |
| subscriber | 更新 Cache 时调用的函数。该函数接收两个值：清除 Cache 时更新或未定义的 Cache 条目的 `key`，以及关联的 `data`。 | [`Cache.Subscriber`](cache.md#cache.subscriber) |

## 类型

### Cache.Options

用于创建新缓存的选项。

#### 属性

| 名称        | 描述                                                     | 类型       |
| --------- | ------------------------------------------------------ | -------- |
| capacity  | 容量（以字节为单位）。如果存储的数据超出容量，则删除最近最少使用的数据。默认容量为 10 MB。       | `number` |
| namespace | 如果设置，缓存将通过子目录命名。这对于分离扩展的各个命令的缓存很有用。默认情况下，缓存在扩展的命令之间共享。 | `string` |

### Cache.Subscriber

订阅参数的函数。

```typescript
type Subscriber = (key: string | undefined, data: string | undefined) => void;
```

### Cache.Subscription

订阅返回的函数。

```typescript
type Subscription = () => void;
```

