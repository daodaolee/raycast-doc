# Action Panel

![](../../.gitbook/assets/action-panel.png)

## API 参考

### ActionPanel

公开用户可以执行的 [actions](https://developers.raycast.com/api-reference/user-interface/actions) 列表。.

通常，项目是上下文感知的，例如，基于所选的列表项目。Actions 带有语义部分，并且可以分配键盘快捷键。

第一和第二动作成为主要和次要动作。他们会自动分配默认的键盘快捷键。在 [List](list.md),、[Grid](grid.md) and [Detail](detail.md) 中，`↵` 表示主要操作，`⌘` `↵` 表示次要操作。在 [Form](https://developers.raycast.com/api-reference/user-interface/form) 中，主要是 `⌘` `↵`，次要是 `⌘` `⇧` `↵`。请记住，虽然您可以为主要和次要操作指定其他快捷方式，但它不会显示在操作面板中。

#### 例子

```typescript
import { ActionPanel, Action, List } from "@raycast/api";

export default function Command() {
  return (
    <List navigationTitle="Open Pull Requests">
      <List.Item
        title="Docs: Update API Reference"
        subtitle="#1"
        actions={
          <ActionPanel title="#1 in raycast/extensions">
            <Action.OpenInBrowser url="https://github.com/raycast/extensions/pull/1" />
            <Action.CopyToClipboard
              title="Copy Pull Request URL"
              content="https://github.com/raycast/extensions/pull/1"
            />
          </ActionPanel>
        }
      />
    </List>
  );
}
```

#### 参数

<table><thead><tr><th width="131">名称</th><th>描述</th><th>类型</th><th>默认</th></tr></thead><tbody><tr><td>children</td><td>不管是 section 还是 action，如果指定了 <a href="https://developers.raycast.com/api-reference/user-interface/actions#action">Action</a> 元素，则会自动创建默认部分。</td><td><a href="action-panel.md#actionpanel.children"><code>ActionPanel.Children</code></a></td><td>-</td></tr><tr><td>title</td><td>标题显示在面板顶部</td><td><code>string</code></td><td>-</td></tr></tbody></table>

### ActionPanel.Section

一组视觉上分开的项。

当 [ActionPanel](https://developers.raycast.com/api-reference/user-interface/action-panel#actionpanel) 包含大量操作时，请使用部分来帮助引导用户执行相关操作。例如，为所有复制操作创建一个 section。

#### 例子

```typescript
import { ActionPanel, Action, List } from "@raycast/api";

export default function Command() {
  return (
    <List navigationTitle="Open Pull Requests">
      <List.Item
        title="Docs: Update API Reference"
        subtitle="#1"
        actions={
          <ActionPanel title="#1 in raycast/extensions">
            <ActionPanel.Section title="Copy">
              <Action.CopyToClipboard title="Copy Pull Request Number" content="#1" />
              <Action.CopyToClipboard
                title="Copy Pull Request URL"
                content="https://github.com/raycast/extensions/pull/1"
              />
              <Action.CopyToClipboard title="Copy Pull Request Title" content="Docs: Update API Reference" />
            </ActionPanel.Section>
            <ActionPanel.Section title="Danger zone">
              <Action title="Close Pull Request" onAction={() => console.log("Close PR #1")} />
            </ActionPanel.Section>
          </ActionPanel>
        }
      />
    </List>
  );
}
```

#### 参数

<table><thead><tr><th width="134">名称</th><th>描述</th><th>类型</th><th>默认值</th></tr></thead><tbody><tr><td>children</td><td>section 里的元素。</td><td><a href="action-panel.md#actionpanel.section.children"><code>ActionPanel.Section.Children</code></a></td><td>-</td></tr><tr><td>title</td><td>标题显示在该部分上方</td><td><code>string</code></td><td>-</td></tr></tbody></table>

### ActionPanel.Submenu

一个非常具体的操作，在选择时将用其子级替换当前的 [ActionPanel](https://developers.raycast.com/api-reference/user-interface/action-panel#actionpanel)。

当操作需要从一系列选项中进行选择时，这非常方便。例如，向 GitHub 拉取请求添加标签或向待办事项添加受让人。

#### 例子

```typescript
import { Action, ActionPanel, Color, Icon, List } from "@raycast/api";

export default function Command() {
  return (
    <List navigationTitle="Open Pull Requests">
      <List.Item
        title="Docs: Update API Reference"
        subtitle="#1"
        actions={
          <ActionPanel title="#1 in raycast/extensions">
            <ActionPanel.Submenu title="Add Label">
              <Action
                icon={{ source: Icon.Circle, tintColor: Color.Red }}
                title="Bug"
                onAction={() => console.log("Add bug label")}
              />
              <Action
                icon={{ source: Icon.Circle, tintColor: Color.Yellow }}
                title="Enhancement"
                onAction={() => console.log("Add enhancement label")}
              />
              <Action
                icon={{ source: Icon.Circle, tintColor: Color.Blue }}
                title="Help Wanted"
                onAction={() => console.log("Add help wanted label")}
              />
            </ActionPanel.Submenu>
          </ActionPanel>
        }
      />
    </List>
  );
}
```

#### 参数

<table><thead><tr><th width="169">名称</th><th>描述</th><th>类型</th><th>默认值</th></tr></thead><tbody><tr><td>title<mark style="color:red;">*</mark></td><td>子菜单显示的标题。</td><td><code>string</code></td><td>-</td></tr><tr><td>autoFocus</td><td>当父 ActionPanel（或 Actionpanel.Submenu）打开时，ActionPanel.Submenu 是否应自动获得焦点。</td><td><code>boolean</code></td><td>-</td></tr><tr><td>children</td><td>子菜单的项。</td><td><a href="action-panel.md#actionpanel.submenu.children"><code>ActionPanel.Submenu.Children</code></a></td><td>-</td></tr><tr><td>filtering</td><td>切换 Raycast 过滤。如果为 <code>true</code>，Raycast 将使用搜索栏中的查询来过滤项目。当为 <code>false</code> 时，扩展程序需要负责过滤。</td><td><code>boolean</code> 或 <code>{ keepSectionOrder: boolean }</code></td><td>当指定 <code>onSearchTextChange</code> 时为 <code>false</code>，否则为 <code>true</code>。</td></tr><tr><td>icon</td><td>显示子菜单的图标。</td><td><a href="icons-and-images.md#image.imagelike"><code>Image.ImageLike</code></a></td><td>-</td></tr><tr><td>isLoading</td><td>是否应在搜索栏旁边显示或隐藏 loading  指示器</td><td><code>boolean</code></td><td><code>false</code></td></tr><tr><td>shortcut</td><td>子菜单的键盘快捷键。</td><td><a href="../keyboard.md#keyboard.shortcut"><code>Keyboard.Shortcut</code></a></td><td>-</td></tr><tr><td>throttle</td><td>定义 <code>onSearchTextChange</code> 处理程序是在每次按下键盘时触发还是延迟以限制事件。当将自定义过滤逻辑与异步操作（例如网络请求）结合使用时，建议设置为 <code>true</code>。</td><td><code>boolean</code></td><td><code>false</code></td></tr><tr><td>onOpen</td><td>打开子菜单时触发的回调。</td><td><code>() => void</code></td><td>-</td></tr><tr><td>onSearchTextChange</td><td>当搜索栏文本发生变化时触发回调。</td><td><code>(text: string) => void</code></td><td>-</td></tr></tbody></table>

## 类型

### ActionPanel.Children

```typescript
ActionPanel.Children: ActionPanel.Section | ActionPanel.Section[] | ActionPanel.Section.Children | null
```

[ActionPanel](https://developers.raycast.com/api-reference/user-interface/action-panel#actionpanel) 组件支持的子组件。

### ActionPanel.Section.Children

```typescript
ActionPanel.Section.Children: Action | Action[] | ReactElement<ActionPanel.Submenu.Props> | ReactElement<ActionPanel.Submenu.Props>[] | null
```

[ActionPanel.Section](https://developers.raycast.com/api-reference/user-interface/action-panel#actionpanel.section) 组件支持的子组件。

### ActionPanel.Submenu.Children

```typescript
ActionPanel.Children: ActionPanel.Section | ActionPanel.Section[] | ActionPanel.Section.Children | null
```

[ActionPanel.Submenu](https://developers.raycast.com/api-reference/user-interface/action-panel#actionpanel.submenu) 组件支持的子组件。
