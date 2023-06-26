# 后台刷新

扩展的命令可以配置为在后台自动运行，无需用户手动打开它们。后台刷新可用于：

* 动态更新 Raycast 根搜索中命令的子标题
* 刷新菜单栏命令
* 主要命令的其他支持功能

本指南可帮助您了解何时以及如何使用后台刷新并了解限制。

## 调度命令

Raycast 支持以配置的时间间隔使用 `no-view` 和 `menu-bar` 的调度命令。

### Manifest

将新的属性 `interval`  添加到 [manifest](../manifest.md#command-properties) 中的命令

例子:

```json
{
    "name": "unread-notifications",
    "title": "Show Unread Notifications",
    "description": "Shows the number of unread notifications in root search",
    "mode": "no-view",
    "interval": "10m"
},
```

间隔指定命令应在后台每 X 秒 (s)、分钟 (m)、小时 (h) 或天 (d) 启动。示例：`10m` 、`12h` 、`1d`。最小值为 10 秒 (`10s`)，应谨慎使用，另请参阅最佳实践部分。

请注意，实际调度并不准确，可能会在容差范围内变化。 macOS 确定运行命令的最佳时间以优化能耗，并且在使用电池运行时，调度时间也会有所不同。为了防止同一命令在后台重叠启动，命令会在根据时间间隔动态调整的超时后终止。

## 在后台运行

从后台启动时，命令的入口点保持不变。对于无视图命令，命令将运行直到主异步函数的 Promise 解析为止。菜单栏命令渲染 React 组件并运行，直到 `isLoading` 属性设置为 `false`。

You can use the global `environment.launchType` in your command to determine whether the command has been launched by the user (`LaunchType.UserInitiated`) or via background refresh (`LaunchType.Background`).

您可以在命令中使用全局 `environment.launchType` 来确定该命令是由用户(`LaunchType.UserInitiated`)启动还是通过后台刷新(`LaunchType.Background`)启动。

```typescript
import { environment, updateCommandMetadata } from "@raycast/api";

async function fetchUnreadNotificationCount() {
  return 10;
}

export default async function Command() {
  console.log("launchType", environment.launchType);
  const count = await fetchUnreadNotificationCount();
  await updateCommandMetadata({ subtitle: `Unread Notifications: ${count}` });
}
```

如果命令超过其最大执行时间，Raycast 会自动终止该命令。如果您的命令保存了与其他命令共享的某些状态，请确保使用防御性编程，即在存储的状态不完整或无法访问时添加对错误和数据抢用的处理。

## 开发调试

对于正在开发的本地命令，错误会像往常一样通过控制台显示。根搜索中的两个开发操作可帮助您运行和调试计划的命令：

* 在后台运行：这会立即运行命令，并将 `environment.launchType` 设置为`LaunchType.Background`。
* 显示错误：如果无法加载命令或引发未捕获的运行时异常，则可以在 Raycast 错误叠加中显示完整错误。此操作还会向已安装的 Store 命令的用户显示，并提供复制和报告生产错误覆盖上的错误操作。

![](../../.gitbook/assets/background-refresh-error.png)

当后台运行导致错误时，用户还将在根搜索命令上看到警告图标，以及通过操作面板显示错误提示的工具提示。命令子标题上的工具可以提示显示上次运行时间。

您可以启动内置根搜索命令 “Extension Diagnostics” 来查看哪些命令在后台运行以及它们上次运行的时间。

## 首选项

对于计划命令，Raycast 会自动添加命令首选项，为用户提供启用和禁用后台刷新的选项。首选项还显示命令的上次运行时间。

![](../../.gitbook/assets/background-refresh-preferences.png)

当用户通过应用商店安装命令时，后台刷新最初是被禁用的，并在用户第一次打开命令或在首选项中启用后台刷新时的激活。 （这是为了避免在用户不知情的情况下在后台自动运行命令。）

## 最佳实践

* 确保该命令在用户手动启动或在后台启动时有用
* 选择尽可能高的间隔值 - 低值意味着命令将更频繁地运行并消耗更多能量
* 如果您的命令执行网络请求，请检查服务的速率限制并适当处理错误（例如稍后自动重试）
* 确保命令尽快完成；对于菜单栏命令，请确保 `isLoading` 尽早设置为 false
* 如果扩展的命令之间共享状态，则使用防御性编程并处理潜在的数据抢用和不可访问的数据
