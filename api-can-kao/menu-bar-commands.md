# Menu Bar Commands

`MenuBarExtra` 组件可用于创建填充 macOS 菜单栏 [附加](https://developer.apple.com/design/human-interface-guidelines/components/system-experiences/the-menu-bar#menu-bar-commands) 部分的命令。

## 起步

如果您还没有扩展，请按照 [入门](https://developers.raycast.com/basics/getting-started) 指南操作，然后返回此页面。现在您的扩展已准备就绪，让我们打开其 `package.json` 文件并向其 `commands` 数组添加一个新条目，确保其 `mode` 属性设置为 `menu-bar`。对于本指南，我们添加以下内容：

```JSON
{
  "name": "github-pull-requests",
  "title": "Pull Requests",
  "subtitle": "GitHub",
  "description": "See your GitHub pull requests at a glance",
  "mode": "menu-bar"
},
```

{% hint style="info" %}
查看 manifest 文件文档中的 [命令属性条目](https://developers.raycast.com/information/manifest#command-properties)，以获取有关每个属性的更多详细信息。
{% endhint %}

在扩展 `src/` 文件夹中创建 `github-pull-requests.tsx` 并添加以下内容：

```typescript
import { MenuBarExtra } from "@raycast/api";

export default function Command() {
  return (
    <MenuBarExtra icon="https://github.githubassets.com/favicons/favicon.png" tooltip="Your Pull Requests">
      <MenuBarExtra.Item title="Seen" />
      <MenuBarExtra.Item
        title="Example Seen Pull Request"
        onAction={() => {
          console.log("seen pull request clicked");
        }}
      />
      <MenuBarExtra.Item title="Unseen" />
      <MenuBarExtra.Item
        title="Example Unseen Pull Request"
        onAction={() => {
          console.log("unseen pull request clicked");
        }}
      />
    </MenuBarExtra>
  );
}
```

如果您的本地服务器正在运行，该命令应该出现在您的根搜索中，并且运行该命令应该会导致 `GitHub` 图标出现在您的菜单栏中。

![GitHub Pull Requests 菜单栏命令](../.gitbook/assets/menu-bar-command.gif)

{% hint style="info" %}
macOS 对于是否显示给定的额外菜单栏拥有最终决定权。如果那里有很多项目，我们刚刚运行的命令可能不会显示。如果是这种情况，请尝试清理菜单栏中的一些空间，你可以关闭一些不需要的项目，或者使用 [HiddenBar](https://github.com/dwarvesf/hidden)、[Bartender](https://www.macbartender.com/) 或类似应用程序隐藏它们。
{% endhint %}

当然，如果我们每次都必须告诉它自行更新，那么我们的拉取请求命令就没有多大用处。要向我们的命令添加后台刷新，我们需要打开之前修改的 `package.json` 文件，并向命令配置对象添加一个 `interval` 键：

```JSON
{
  "name": "github-pull-requests",
  "title": "Pull Requests",
  "subtitle": "GitHub",
  "description": "See your GitHub pull requests at a glance",
  "mode": "menu-bar",
  "interval": "5m"
}
```

您的根搜索应类似于：

![菜单栏命令 - 激活后台刷新](../.gitbook/assets/menu-bar-activate-command.png)

运行一次应该将其激活为：

![菜单栏命令 - 刷新](../.gitbook/assets/menu-bar-refresh.png)

## 生命周期

尽管 `menu-bar` 命令可能会导致项目永久显示在 macOS 菜单栏中，但它们并不是长期存在的进程。相反，与其他命令一样，Raycast 根据需要将它们加载到内存中，执行它们的代码，然后尝试在下一个方便的时间卸载它们。有五个不同的事件可以让菜单栏的项目被放置在菜单栏中，所以让我们逐一介绍一下。

### 从根搜索

与任何其他命令一样，`menu-bar` 命令可以直接从 Raycast 的根搜索运行。最终，菜单栏中会显示一个新项目（如果您有足够的空间并且命令返回 `MenuBarExtra`），或者如果命令返回 `null`，则上一个项目消失。在这种情况下，Raycast 会加载您的命令代码，执行它，等待 `MenuBarExtra` 的 `isLoading` 属性切换为 `false`，然后卸载命令。

{% hint style="danger" %}
如果您的命令返回 `MenuBarExtra`，那么它不能设置 `isLoading` - 在这种情况下，Raycast 将渲染并立即更新命令，或者在执行异步任务（例如 API 调用）时将其设置为 `true`，当异步任务完成后将其立即立即设置为 `false` 。与上面相同，Raycast 将加载命令代码，执行它，等待 `MenuBarExtra` 的 `isLoading` 属性切换为 `false`，然后卸载命令。
{% endhint %}

### 按设定的时间间隔

如果您的 `menu-bar` 命令使用并且激活了 [后台刷新](https://developers.raycast.com/information/lifecycle/background-refresh)，Raycast 将按设定的时间间隔运行该命令。在您的命令中，您可以使用  `environment.launchType` 来检查它是在后台启动还是由用户启动。

{% hint style="info" %}
为了简化测试，配置为在后台运行的命令在开发模式下有一个额外的操作:<img src="../.gitbook/assets/menu-bar-run-in-background.png" alt="Menu Bar Command - Run in Background" data-size="original">
{% endhint %}

### 当用户单击菜单栏中命令的图标/标题时

`view` 命令或 `no-view` 命令的最大区别之一是 `menu-bar` 命令有一个额外的入口：当用户单击菜单栏中的项目时。如果该项目有一个菜单（即 `MenuBarExtra` 提供至少一个子菜单），Raycast 将加载命令代码，执行它并在菜单打开时将其保留在内存中。当菜单关闭时（通过用户单击外部或通过单击 `MenuBarExtra.Item`），命令将被卸载。

### 当 Raycast 重新启动时

这种情况假设您的命令至少运行过一次，导致菜单栏中放置了一个项目。如果是这种情况，退出并再次启动 Raycast 应该会在菜单栏中放置相同的项目。不过，该项目会从 Raycast 的数据库中恢复 - 而不是通过加载和执行命令来恢复。

### 当在首选项中重新启用菜单栏命令时

这种情况应该与 Raycast 重新启动时的工作方式相同。

## 最佳实践

* 充分利用 [缓存 API](https://developers.raycast.com/api-reference/cache) 和我们的 [公共包](https://developers.raycast.com/utilities/getting-started)，以提供快速反馈并确保操作处理程序按预期工作
* 确保命令完成执行后将 `isLoading` 设置为 false
* 避免在 `MenuBarExtra`, `MenuBarExtra.Submenu` 或 `MenuBarExtra.Item` 中设置长标题
* 不要将相同的 `MenuBarExtra.Items` 放在同一级别（`MenuBarExtra` 的直接子项或同一子菜单中），因为它们的 `onAction` 处理程序将无法正确执行

## API 参考

### MenuBarExtra

将一个项目添加到菜单栏，如果其 c`hildren` 属性非空，则可以选择附加菜单。

{% hint style="info" %}
`menu-bar` 命令并不总是需要返回 `MenuBarExtra`。有时从菜单栏中删除某个项目是有意义的，在这种情况下，您可以编写命令逻辑以返回 `null`。
{% endhint %}

#### 例子

```typescript
import { Icon, MenuBarExtra, open } from "@raycast/api";

const data = {
  archivedBookmarks: [{ name: "Google Search", url: "www.google.com" }],
  newBookmarks: [{ name: "Raycast", url: "www.raycast.com" }],
};

export default function Command() {
  return (
    <MenuBarExtra icon={Icon.Bookmark}>
      <MenuBarExtra.Section title="New">
        {data?.newBookmarks.map((bookmark) => (
          <MenuBarExtra.Item key={bookmark.url} title={bookmark.name} onAction={() => open(bookmark.url)} />
        ))}
      </MenuBarExtra.Section>
      <MenuBarExtra.Section title="Archived">
        {data?.archivedBookmarks.map((bookmark) => (
          <MenuBarExtra.Item key={bookmark.url} title={bookmark.name} onAction={() => open(bookmark.url)} />
        ))}
      </MenuBarExtra.Section>
    </MenuBarExtra>
  );
}
```

#### 参数

| 名称        | 描述                                                                                                          | 类型                                                                      | 默认      |
| --------- | ----------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- | ------- |
| children  | `MenuBarExtra.Item`s, `MenuBarExtra.Submenu`s, `MenuBarExtra.Separator` 或两者的混合                              | `React.ReactNode`                                                       | -       |
| icon      | 菜单栏中显示的图标。                                                                                                  | [`Image.ImageLike`](user-interface/icons-and-images.md#image.imagelike) | -       |
| isLoading | 提示 Raycast 不应卸载该命令，因为它仍在执行。如果您设置使用 `isLoading`，则需要确保在正在执行的任务（例如 API 调用）结束时将其设置为 `false`，以便 Raycast 可以卸载该命令。 | `boolean`                                                               | `false` |
| title     | 显示在菜单栏中的字符串。                                                                                                | `string`                                                                | -       |
| tooltip   | 当光标悬停在菜单栏中的项目上时显示的提示。                                                                                       | `string`                                                                | -       |

### MenuBarExtra.Item

[MenuBarExtra](menu-bar-commands.md#menubarextra) 或 [MenuBarExtra.Submenu](menu-bar-commands.md#menubarextra.submenu) 中的一项

#### 例子

{% tabs %}
{% tab title="ItemWithTitle.tsx" %}
仅提供 `title` 属性的项目将被渲染为禁用。使用它来创建章节标题。

```typescript
import { Icon, MenuBarExtra } from "@raycast/api";

export default function Command() {
  return (
    <MenuBarExtra icon={Icon.Bookmark}>
      <MenuBarExtra.Item title="Raycast.com" />
    </MenuBarExtra>
  );
}
```
{% endtab %}

{% tab title="ItemWithTitleAndIcon.tsx" %}
同样，提供 `title` 和 `icon` 的也将表现为禁用。

```typescript
import { Icon, MenuBarExtra } from "@raycast/api";

export default function Command() {
  return (
    <MenuBarExtra icon={Icon.Bookmark}>
      <MenuBarExtra.Item icon="raycast.png" title="Raycast.com" />
    </MenuBarExtra>
  );
}
```
{% endtab %}

{% tab title="ItemWithAction.tsx" %}
与 `title`（以及可选的 `icon`）一起提供 `onAction` 属性的项目不会表现为禁用。当用户单击菜单栏中的此项时，将执行操作处理程序。

```typescript
import { Icon, MenuBarExtra, open } from "@raycast/api";

export default function Command() {
  return (
    <MenuBarExtra icon={Icon.Bookmark}>
      <MenuBarExtra.Item icon="raycast.png" title="Raycast.com" onAction={() => open("https://raycast.com")} />
    </MenuBarExtra>
  );
}
```
{% endtab %}
{% endtabs %}

#### 属性

| 名称                                      | 描述                   | 类型                                                                                               | 默认 |
| --------------------------------------- | -------------------- | ------------------------------------------------------------------------------------------------ | -- |
| title<mark style="color:red;">\*</mark> | 该项显示的主标题。            | `string`                                                                                         | -  |
| icon                                    | 该项的可选图标。             | [`Image.ImageLike`](user-interface/icons-and-images.md#image.imagelike)                          | -  |
| shortcut                                | 用于在其父菜单打开时调用此项的快捷方式。 | [`Keyboard.Shortcut`](keyboard.md#keyboard.shortcut)                                             | -  |
| subtitle                                | 该项显示的副标题。            | `string`                                                                                         | -  |
| tooltip                                 | 当光标悬停在项目上时显示的提示。     | `string`                                                                                         | -  |
| onAction                                | 当用户单击该项时的回调。         | `(event:` [`MenuBarExtra.ActionEvent`](menu-bar-commands.md#menubarextra.actionevent)`) => void` | -  |

### MenuBarExtra.Submenu

`MenuBarExtra.Submenus` 在人们与它交互时展示出来。它们可以分组，但请记住，子菜单会增加界面的复杂性 - 因此请谨慎使用它们！

#### Example

{% tabs %}
{% tab title="Bookmarks.tsx" %}
```typescript
import { Icon, MenuBarExtra, open } from "@raycast/api";

export default function Command() {
  return (
    <MenuBarExtra icon={Icon.Bookmark}>
      <MenuBarExtra.Item icon="raycast.png" title="Raycast.com" onAction={() => open("https://raycast.com")} />
      <MenuBarExtra.Submenu icon="github.png" title="GitHub">
        <MenuBarExtra.Item title="Pull Requests" onAction={() => open("https://github.com/pulls")} />
        <MenuBarExtra.Item title="Issues" onAction={() => open("https://github.com/issues")} />
      </MenuBarExtra.Submenu>
      <MenuBarExtra.Submenu title="Disabled"></MenuBarExtra.Submenu>
    </MenuBarExtra>
  );
}
```
{% endtab %}

{% tab title="DisabledSubmenu.tsx" %}
没有子菜单的子菜单将显示为禁用。

```typescript
import { Icon, MenuBarExtra, open } from "@raycast/api";

export default function Command() {
  return (
    <MenuBarExtra icon={Icon.Bookmark}>
      <MenuBarExtra.Submenu title="Disabled"></MenuBarExtra.Submenu>
    </MenuBarExtra>
  );
}
```
{% endtab %}
{% endtabs %}

#### 参数

| 名称                                      | 描述                                                                             | 类型                                                                      | 默认 |
| --------------------------------------- | ------------------------------------------------------------------------------ | ----------------------------------------------------------------------- | -- |
| title<mark style="color:red;">\*</mark> | 该子菜单显示的主标题。                                                                    | `string`                                                                | -  |
| children                                | `MenuBarExtra.Item`s, `MenuBarExtra.Submenu`s, `MenuBarExtra.Separator` 或两者的混合 | `React.ReactNode`                                                       | -  |
| icon                                    | 该子菜单的可选图标。                                                                     | [`Image.ImageLike`](user-interface/icons-and-images.md#image.imagelike) | -  |

### MenuBarExtra.Section

用于对相关菜单项进行分组的项。它有一个可选的标题，并且在各部分之间自动添加分隔符。

#### 例子

```typescript
import { Icon, MenuBarExtra, open } from "@raycast/api";

const data = {
  archivedBookmarks: [{ name: "Google Search", url: "www.google.com" }],
  newBookmarks: [{ name: "Raycast", url: "www.raycast.com" }],
};

export default function Command() {
  return (
    <MenuBarExtra icon={Icon.Bookmark}>
      <MenuBarExtra.Section title="New">
        {data?.newBookmarks.map((bookmark) => (
          <MenuBarExtra.Item key={bookmark.url} title={bookmark.name} onAction={() => open(bookmark.url)} />
        ))}
      </MenuBarExtra.Section>
      <MenuBarExtra.Section title="Archived">
        {data?.archivedBookmarks.map((bookmark) => (
          <MenuBarExtra.Item key={bookmark.url} title={bookmark.name} onAction={() => open(bookmark.url)} />
        ))}
      </MenuBarExtra.Section>
    </MenuBarExtra>
  );
}
```

#### 属性

| 名称       | 描述         | 类型                | 默认 |
| -------- | ---------- | ----------------- | -- |
| children | 该部分的节点元素。  | `React.ReactNode` | -  |
| title    | 标题显示在该部分上方 | `string`          | -  |

## 类型

### MenuBarExtra.ActionEvent

回调中描述 Action 事件的接口。

#### 属性

| 名称                                     | 描述           | 类型                               |
| -------------------------------------- | ------------ | -------------------------------- |
| type<mark style="color:red;">\*</mark> | action 事件的类型 | `"left-click"` 或 `"right-click"` |

#### 例子

```typescript
import { MenuBarExtra } from "@raycast/api";

export default function Command() {
  return (
    <MenuBarExtra>
      <MenuBarExtra.Item
        title="Log Action Event Type"
        onAction={(event: MenuBarExtra.ActionEvent) => console.log("Action Event Type", event.type)}
      />
    </MenuBarExtra>
  );
}
```
