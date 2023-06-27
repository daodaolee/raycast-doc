# 起步

除了作为应用程序一部分捆绑的 [Raycast API](../api-can-kao/cache.md) 之外，我们还提供了一个同级包，其中包含一组实用程序来简化扩展中使用的常见模块和操作。

![](../.gitbook/assets/utils-illustration.jpg)

## 安装

该包可以使用 npm 独立安装。

```
npm install --save @raycast/utils
```

`@raycast/utils` 对 `@raycast/api` 有对等依赖。这意味着某个版本的 utils 将需要高于某个版本的 `api` 的版本。如果不是这种情况，npm 会警告你。
