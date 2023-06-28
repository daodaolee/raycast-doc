# Grid

当单个项展示图像时，可以用 `Grid` 组件作为 [List](list.md#list) 组件的替代方案。

{% hint style="info" %}
由于它的 API 和 List 很像，所以将视图从 List 更改为 Grid 应该像下面一样简单：

* 确保您使用的 `@raycast/api` 包的版本至少为 1.36.0
* 将 `import { List } from '@raycast/api'` 的导入更新为 `import { Grid } from '@raycast/api'；`
* 从顶级 List 组件中删除 `isShowingDetail` 属性以及所有 [List.Item](list.md#list.item) 的 detail 属性
* 将所有 [List.Item](list.md#list.item) 的 icon 属性改为 `content`
* 删除所有 List.Items 的 `accessories`、`accessoryIcon` 和 `accessoryTitle` 参数； Grid.Item 目前不支持 `accessories`
* 最后，用 Grid 替换所有 List 。
{% endhint %}

![](../../.gitbook/assets/grid.png)

## 搜索栏

The search bar allows users to interact quickly with grid items. By default,  are displayed if the user's input can be (fuzzy) matched to the item's

搜索栏允许用户与 grid 项快速交互。默认情况下，如果用户的输入可以与项目的 `title` 或 `keywords`（模糊）匹配，则显示 [Grid.Items](grid.md#grid.item)。

### 自定义过滤

Sometimes, you may not want to rely on Raycast's filtering, but use/implement your own. If that's the case, you can set the `Grid`'s `filtering` [prop](grid.md#props) to false, and the items displayed will be independent of the search bar's text. Note that `filtering` is also implicitly set to false if an `onSearchTextChange` listener is specified. If you want to specify a change listener and _still_ take advantage of Raycast's built-in filtering, you can explicitly set `filtering` to true.

有时，您可能不想依赖 Raycast 的过滤，而是 使用/实现 您自己的过滤。如果是这种情况，您可以将 Grid 的过滤属性设置为 `false`，并且显示的项将独立于搜索栏。请注意，如果指定了 `onSearchTextChange` 侦听器，过滤也会隐式设置为 `false`。如果您想指定更改侦听器并仍然利用 Raycast 的内置过滤功能，则可以显式将过滤设置为 `true`。

```typescript
import { useEffect, useState } from "react";
import { Grid } from "@raycast/api";

const items = [
  { content: "🙈", keywords: ["see-no-evil", "monkey"] },
  { content: "🥳", keywords: ["partying", "face"] },
];

export default function Command() {
  const [searchText, setSearchText] = useState("");
  const [filteredList, filterList] = useState(items);

  useEffect(() => {
    filterList(items.filter((item) => item.keywords.some((keyword) => keyword.includes(searchText))));
  }, [searchText]);

  return (
    <Grid
      columns={5}
      inset={Grid.Inset.Large}
      filtering={false}
      onSearchTextChange={setSearchText}
      navigationTitle="Search Emoji"
      searchBarPlaceholder="Search your favorite emoji"
    >
      {filteredList.map((item) => (
        <Grid.Item key={item.content} content={item.content} />
      ))}
    </Grid>
  );
}
```

### 用编程的方式更新搜索栏

其他时候，您可能希望扩展程序更新搜索栏的内容，例如，您可以存储用户之前的搜索列表，并在下次访问时允许他们从上次中断的地方“继续”。

为此，您可以使用 `searchText` 属性。

```typescript
import { useState } from "react";
import { Action, ActionPanel, Grid } from "@raycast/api";

const items = [
  { content: "🙈", keywords: ["see-no-evil", "monkey"] },
  { content: "🥳", keywords: ["partying", "face"] },
];

export default function Command() {
  const [searchText, setSearchText] = useState("");

  return (
    <Grid
      searchText={searchText}
      onSearchTextChange={setSearchText}
      navigationTitle="Search Emoji"
      searchBarPlaceholder="Search your favorite emoji"
    >
      {items.map((item) => (
        <Grid.Item
          key={item.content}
          content={item.content}
          actions={
            <ActionPanel>
              <Action title="Select" onAction={() => setSearchText(item.content)} />
            </ActionPanel>
          }
        />
      ))}
    </Grid>
  );
}
```

### 下拉菜单

某些扩展可能会为用户提供第二个过滤维度，例如媒体文件管理扩展可能允许用户仅查看视频或仅图像，图像搜索扩展可能允许切换搜索引擎等。

这就是 `searchBarAccessory` 属性有用的地方。向其传递一个 [Grid.Dropdown](grid.md#grid.dropdown) 组件，它将显示在搜索栏的右侧。通过使用全局快捷键 `⌘` `P` 或单击它来调用它。

## 例子

{% tabs %}
{% tab title="Grid.tsx" %}
```jsx
import { Grid } from "@raycast/api";

export default function Command() {
  return (
    <Grid columns={8} inset={Grid.Inset.Large}>
      <Grid.Item content="🥳" />
      <Grid.Item content="🙈" />
    </Grid>
  );
}
```
{% endtab %}

{% tab title="GridWithSections.tsx" %}
```typescript
import { Grid } from "@raycast/api";

export default function Command() {
  return (
    <Grid>
      <Grid.Section title="Section 1">
        <Grid.Item content="https://placekitten.com/400/400" title="Item 1" />
      </Grid.Section>
      <Grid.Section title="Section 2" subtitle="Optional subtitle">
        <Grid.Item content="https://placekitten.com/400/400" title="Item 1" />
      </Grid.Section>
    </Grid>
  );
}
```
{% endtab %}

{% tab title="GridWithActions.tsx" %}
```typescript
import { ActionPanel, Action, Grid } from "@raycast/api";

export default function Command() {
  return (
    <Grid>
      <Grid.Item
        content="https://placekitten.com/400/400"
        title="Item 1"
        actions={
          <ActionPanel>
            <Action.CopyToClipboard content="👋" />
          </ActionPanel>
        }
      />
    </Grid>
  );
}
```
{% endtab %}

{% tab title="GridWithEmptyView.tsx" %}
```typescript
import { useEffect, useState } from "react";
import { Grid, Image } from "@raycast/api";

export default function CommandWithCustomEmptyView() {
  const [state, setState] = useState<{
    searchText: string;
    items: { content: Image.ImageLike; title: string }[];
  }>({ searchText: "", items: [] });

  useEffect(() => {
    console.log("Running effect after state.searchText changed. Current value:", JSON.stringify(state.searchText));
    // perform an API call that eventually populates `items`.
  }, [state.searchText]);

  return (
    <Grid onSearchTextChange={(newValue) => setState((previous) => ({ ...previous, searchText: newValue }))}>
      {state.searchText === "" && state.items.length === 0 ? (
        <Grid.EmptyView icon={{ source: "https://placekitten.com/500/500" }} title="Type something to get started" />
      ) : (
        state.items.map((item, index) => <Grid.Item key={index} content={item.content} title={item.title} />)
      )}
    </Grid>
  );
}
```
{% endtab %}
{% endtabs %}

## API 参考

### Grid

展示 [Grid.Section](grid.md#grid.section)s 或 [Grid.Item](grid.md#grid.item)s.

grid 通过索引其项的标题和关键字来使用内置过滤。

#### 例子

```typescript
import { Grid } from "@raycast/api";

const items = [
  { content: "🙈", keywords: ["see-no-evil", "monkey"] },
  { content: "🥳", keywords: ["partying", "face"] },
];

export default function Command() {
  return (
    <Grid
      columns={5}
      inset={Grid.Inset.Large}
      navigationTitle="Search Emoji"
      searchBarPlaceholder="Search your favorite emoji"
    >
      {items.map((item) => (
        <Grid.Item key={item.content} content={item.content} keywords={item.keywords} />
      ))}
    </Grid>
  );
}
```

#### 参数

<table><thead><tr><th width="199">名称</th><th>描述</th><th>类型</th><th>默认</th></tr></thead><tbody><tr><td>actions</td><td>对 ActionPanel 的引用。仅当没有任何子项时才会显示。</td><td><code>React.ReactNode</code></td><td>-</td></tr><tr><td>aspectRatio</td><td>Grid.Item 元素的长宽比。默认为 1。</td><td><code>"1"</code> 或 <code>"3/2"</code> 或 <code>"2/3"</code> 或 <code>"4/3"</code> 或 <code>"3/4"</code> 或 <code>"16/9"</code> 或 <code>"9/16"</code></td><td>-</td></tr><tr><td>children</td><td>Grid section  或 item。如果指定了 Grid.Item 元素，则会自动创建默认部分。</td><td><code>React.ReactNode</code></td><td>-</td></tr><tr><td>columns</td><td>Grid section 的列数。最小值为 1，最大值为 8。</td><td><code>number</code></td><td>5</td></tr><tr><td>filtering</td><td>切换 Raycast过滤。如果为 <code>true</code>，Raycast 将使用搜索栏中的查询来过滤项目。当为 <code>false</code> 时，扩展程序需要负责过滤。</td><td><code>boolean</code> 或 <code>{ keepSectionOrder: boolean }</code></td><td>当指定 <code>onSearchTextChange</code> 时为 <code>false</code>，否则为 <code>true</code>。</td></tr><tr><td>fit</td><td>适应 Grid.Item 元素。默认为“contain”</td><td><a href="grid.md#grid.fit"><code>Grid.Fit</code></a></td><td>-</td></tr><tr><td>inset</td><td>Grid.Items 的内容与其边框之间应有多少空间。绝对值取决于 <code>itemSize</code> 属性的值。</td><td><a href="grid.md#grid.inset"><code>Grid.Inset</code></a></td><td>-</td></tr><tr><td>isLoading</td><td>是否应在搜索栏下方显示或隐藏 loading 栏</td><td><code>boolean</code></td><td><code>false</code></td></tr><tr><td>navigationTitle</td><td>该视图的主标题</td><td><code>string</code></td><td>Command title</td></tr><tr><td>searchBarAccessory</td><td><a href="grid.md#grid.dropdown">Grid.Dropdown</a>  将显示在搜索栏的右侧。</td><td><code>ReactElement&#x3C;</code><a href="list.md#props"><code>List.Dropdown.Props</code></a><code>, string></code></td><td>-</td></tr><tr><td>searchBarPlaceholder</td><td>搜索栏中的占位符文本。</td><td><code>string</code></td><td><code>"Search…"</code></td></tr><tr><td>searchText</td><td>搜索栏中的文本。</td><td><code>string</code></td><td>-</td></tr><tr><td>selectedItemId</td><td>有指定 id 的项。</td><td><code>string</code></td><td>-</td></tr><tr><td>throttle</td><td>定义 <code>onSearchTextChange</code> 处理程序是在每次按下键盘时触发还是延迟以限制事件。当将自定义过滤逻辑与异步操作（例如网络请求）结合使用时，建议设置为 <code>true</code>。</td><td><code>boolean</code></td><td><code>false</code></td></tr><tr><td>onSearchTextChange</td><td>当搜索栏文本发生变化时触发回调。</td><td><code>(text: string) => void</code></td><td>-</td></tr><tr><td>onSelectionChange</td><td>当 Grid 中的项目选择发生变化时触发回调。</td><td><code>(id: string) => void</code></td><td>-</td></tr></tbody></table>

### Grid.Dropdown

在搜索栏右侧的下拉菜单。

#### 例子

```typescript
import { Grid, Image } from "@raycast/api";
import { useState } from "react";

const types = [
  { id: 1, name: "Smileys", value: "smileys" },
  { id: 2, name: "Animals & Nature", value: "animals-and-nature" },
];

const items: { [key: string]: { content: Image.ImageLike; keywords: string[] }[] } = {
  smileys: [{ content: "🥳", keywords: ["partying", "face"] }],
  "animals-and-nature": [{ content: "🙈", keywords: ["see-no-evil", "monkey"] }],
};

export default function Command() {
  const [type, setType] = useState<string>("smileys");

  return (
    <Grid
      navigationTitle="Search Beers"
      searchBarPlaceholder="Search your favorite drink"
      searchBarAccessory={
        <Grid.Dropdown tooltip="Select Emoji Category" storeValue={true} onChange={(newValue) => setType(newValue)}>
          <Grid.Dropdown.Section title="Emoji Categories">
            {types.map((type) => (
              <Grid.Dropdown.Item key={type.id} title={type.name} value={type.value} />
            ))}
          </Grid.Dropdown.Section>
        </Grid.Dropdown>
      }
    >
      {(items[type] || []).map((item) => (
        <Grid.Item key={`${item.content}`} content={item.content} keywords={item.keywords} />
      ))}
    </Grid>
  );
}
```

#### 参数

<table><thead><tr><th width="197">名称</th><th>描述</th><th>类型</th><th>默认</th></tr></thead><tbody><tr><td>tooltip<mark style="color:red;">*</mark></td><td>将鼠标悬停在下拉菜单上时显示的提示。</td><td><code>string</code></td><td>-</td></tr><tr><td>children</td><td>下拉 section 或 item。如果指定了 Dropdown.Item 元素，则会自动创建默认部分。</td><td><code>React.ReactNode</code></td><td>-</td></tr><tr><td>defaultValue</td><td>下拉列表的默认值。请记住，<code>defaultValue</code> 将在每个组件生命周期配置一次。这意味着如果用户更改该值，则重新渲染时不会配置 <code>defaultValue</code>。</td><td><code>string</code></td><td>-</td></tr><tr><td>filtering</td><td>切换 Raycast 过滤。如果为 <code>true</code>，Raycast 将使用搜索栏中的查询来过滤项目。当为 <code>false</code> 时，扩展程序需要负责过滤。</td><td><code>boolean</code> 或 <code>{ keepSectionOrder: boolean }</code></td><td>当指定 <code>onSearchTextChange</code> 时为 <code>false</code>，否则为 <code>true</code>。</td></tr><tr><td>id</td><td>下拉列表的 ID。</td><td><code>string</code></td><td>-</td></tr><tr><td>isLoading</td><td>是否在搜索栏旁边显示或隐藏 loading 指示器</td><td><code>boolean</code></td><td><code>false</code></td></tr><tr><td>placeholder</td><td>在下拉搜索字段中的占位符文本。</td><td><code>string</code></td><td><code>"Search…"</code></td></tr><tr><td>storeValue</td><td>下拉列表的值是否应在选择后保留，并在下次渲染下拉列表时恢复。</td><td><code>boolean</code></td><td>-</td></tr><tr><td>throttle</td><td>定义 <code>onSearchTextChange</code> 处理程序是在每次按下键盘时触发还是延迟以限制事件。当将自定义过滤逻辑与异步操作（例如网络请求）结合使用时，建议设置为 <code>true</code>。</td><td><code>boolean</code></td><td><code>false</code></td></tr><tr><td>value</td><td>下拉菜单的当前值。</td><td><code>string</code></td><td>-</td></tr><tr><td>onChange</td><td>当下拉选择发生变化时触发回调。</td><td><code>(newValue: string) => void</code></td><td>-</td></tr><tr><td>onSearchTextChange</td><td>当搜索栏文本发生变化时触发回调。</td><td><code>(text: string) => void</code></td><td>-</td></tr></tbody></table>

### Grid.Dropdown.Item

[Grid.Dropdown](grid.md#grid.dropdown) 中的下拉项

#### 例子

```typescript
import { Grid } from "@raycast/api";

export default function Command() {
  return (
    <Grid
      searchBarAccessory={
        <Grid.Dropdown tooltip="Dropdown With Items">
          <Grid.Dropdown.Item title="One" value="one" />
          <Grid.Dropdown.Item title="Two" value="two" />
          <Grid.Dropdown.Item title="Three" value="three" />
        </Grid.Dropdown>
      }
    >
      <Grid.Item content="https://placekitten.com/400/400" title="Item in the Main Grid" />
    </Grid>
  );
}
```

#### 参数

| 名称                                      | 描述                                                       | 类型                                                       | 默认                              |
| --------------------------------------- | -------------------------------------------------------- | -------------------------------------------------------- | ------------------------------- |
| title<mark style="color:red;">\*</mark> | 该项的标题。                                                   | `string`                                                 | -                               |
| value<mark style="color:red;">\*</mark> | 下拉项的值。确保为每项分配每个唯一值。                                      | `string`                                                 | -                               |
| icon                                    | 该项的可选图标。                                                 | [`Image.ImageLike`](icons-and-images.md#image.imagelike) | -                               |
| keywords                                | 一个可选属性，用于提供额外的可索引字符串以供搜索。在 Raycast 中过滤项目时，除了标题之外还会搜索关键字。 | `string[]`                                               | The title of its section if any |

### Grid.Dropdown.Section

分离的下拉项组。

&#x20;使用 section 将相关菜单项分到一个组。

#### 例子

```typescript
import { Grid } from "@raycast/api";

export default function Command() {
  return (
    <Grid
      searchBarAccessory={
        <Grid.Dropdown tooltip="Dropdown With Sections">
          <Grid.Dropdown.Section title="First Section">
            <Grid.Dropdown.Item title="One" value="one" />
          </Grid.Dropdown.Section>
          <Grid.Dropdown.Section title="Second Section">
            <Grid.Dropdown.Item title="Two" value="two" />
          </Grid.Dropdown.Section>
        </Grid.Dropdown>
      }
    >
      <Grid.Item content="https://placekitten.com/400/400" title="Item in the Main Grid" />
    </Grid>
  );
}
```

#### 参数

| 名称       | 描述              | 类型                | 默认 |
| -------- | --------------- | ----------------- | -- |
| children | 该 section 项的内容。 | `React.ReactNode` | -  |
| title    | 该 section 上方的标题 | `string`          | -  |

### Grid.EmptyView

当没有任何可用项时显示的视图。

Raycast 提供了一个默认的 `EmptyView`，如果 Grid 组件没有子组件，或者有子组件，但没有一个与搜索栏中的查询匹配，则将显示该 `EmptyView`。这也可以通过与其他 `Grid.Items` 一起传递空视图来覆盖。

请注意，如果 Grid 的 `isLoading` 属性为 `true` 并且搜索栏为空，则永远不会显示 `EmptyView`。

![Grid 为空的插图](../../.gitbook/assets/grid-empty-view.png)

#### 例子

```typescript
import { useEffect, useState } from "react";
import { Grid, Image } from "@raycast/api";

export default function CommandWithCustomEmptyView() {
  const [state, setState] = useState<{
    searchText: string;
    items: { content: Image.ImageLike; title: string }[];
  }>({ searchText: "", items: [] });

  useEffect(() => {
    console.log("Running effect after state.searchText changed. Current value:", JSON.stringify(state.searchText));
    // perform an API call that eventually populates `items`.
  }, [state.searchText]);

  return (
    <Grid onSearchTextChange={(newValue) => setState((previous) => ({ ...previous, searchText: newValue }))}>
      {state.searchText === "" && state.items.length === 0 ? (
        <Grid.EmptyView icon={{ source: "https://placekitten.com/500/500" }} title="Type something to get started" />
      ) : (
        state.items.map((item, index) => <Grid.Item key={index} content={item.content} title={item.title} />)
      )}
    </Grid>
  );
}
```

#### 参数

| 名称          | 描述                                                | 类型                                                       | 默认 |
| ----------- | ------------------------------------------------- | -------------------------------------------------------- | -- |
| actions     | 对 [ActionPanel](action-panel.md#actionpanel) 的引用。 | `React.ReactNode`                                        | -  |
| description | 为什么显示空视图的可选描述。                                    | `string`                                                 | -  |
| icon        | 显示在 EmptyView 中心的图标。                              | [`Image.ImageLike`](icons-and-images.md#image.imagelike) | -  |
| title       | 空视图显示的主标题。                                        | `string`                                                 | -  |

### Grid.Item

[Grid](grid.md#grid) 中的一项

This is one of the foundational UI components of Raycast. A grid item represents a single entity. It can be an image, an emoji, a GIF etc. You most likely want to perform actions on this item, so make it clear to the user what this item is about.

这是 Raycast 的基本 UI 组件之一。grid 项代表单个实体。它可以是图像、表情符号、GIF 等。

#### 例子

```typescript
import { Grid } from "@raycast/api";

export default function Command() {
  return (
    <Grid>
      <Grid.Item content="🥳" title="Partying Face" subtitle="Smiley" />
    </Grid>
  );
}
```

#### 参数

| 名称                                        | 描述                                                                                           | 类型                                                                                                                                                                                                                                                                                | 默认 |
| ----------------------------------------- | -------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -- |
| content<mark style="color:red;">\*</mark> | image 或 color，可以有提示 ，代表网格项的内容。                                                               | [`Image.ImageLike`](icons-and-images.md#image.imagelike) 或 `{ color:` [`Color.ColorLike`](colors.md#color.colorlike) `}` 或 `{ tooltip: string; value:` [`Image.ImageLike`](icons-and-images.md#image.imagelike) 或 `{ color:` [`Color.ColorLike`](colors.md#color.colorlike) `} }` | -  |
| actions                                   | 将针对所选 grid 项更新的 ActionPanel。                                                                 | `React.ReactNode`                                                                                                                                                                                                                                                                 | -  |
| id                                        | 该项的 ID。当选择该项时，此字符串将传递到 Grid 的 `onSelectionChange` 处理程序。确保为每项分配一个唯一的 ID，否则将自动生成 UUID。         | `string`                                                                                                                                                                                                                                                                          | -  |
| keywords                                  | 一个可选属性，用于提供额外的可索引字符串以供搜索。当通过搜索栏过滤 Raycast 中的列表时，除了标题之外还会搜索关键字。                               | `string[]`                                                                                                                                                                                                                                                                        | -  |
| quickLook                                 | 使用“Quick Look”预览文件的可选信息。使用 [Action.ToggleQuickLook](actions.md#action.togglequicklook) 切换预览。 | `{ name: string; path: string }`                                                                                                                                                                                                                                                  | -  |
| subtitle                                  | 标题下方的可选子标题。                                                                                  | `string`                                                                                                                                                                                                                                                                          | -  |
| title                                     | 内容下方的可选标题。                                                                                   | `string`                                                                                                                                                                                                                                                                          | -  |

### Grid.Section

一组 [Grid.Item](grid.md#grid.item).

Section 是构建 grid 的好方法。例如，您可以将在同一地点或同一天拍摄的照片分组。这样，用户可以快速访问最相关的内容。

各部分可以指定自己的 `column`、`fit`、`aspectRatio` 和 `inset` 属性，与主 Grid 组件上定义的内容分开。

#### 例子

![](../../.gitbook/assets/grid-styled-sections.png)

{% tabs %}
{% tab title="GridWithSection.tsx" %}
```typescript
import { Grid } from "@raycast/api";

export default function Command() {
  return (
    <Grid>
      <Grid.Section title="Section 1">
        <Grid.Item content="https://placekitten.com/400/400" title="Item 1" />
      </Grid.Section>
      <Grid.Section title="Section 2" subtitle="Optional subtitle">
        <Grid.Item content="https://placekitten.com/400/400" title="Item 1" />
      </Grid.Section>
    </Grid>
  );
}
```
{% endtab %}

{% tab title="GridWithStyledSection.tsx" %}
```typescript
import { Grid, Color } from "@raycast/api";

export default function Command() {
  return (
    <Grid columns={6}>
      <Grid.Section aspectRatio="2/3" title="Movies">
        <Grid.Item content="https://api.lorem.space/image/movie?w=150&h=220" />
        <Grid.Item content="https://api.lorem.space/image/movie?w=150&h=220" />
        <Grid.Item content="https://api.lorem.space/image/movie?w=150&h=220" />
        <Grid.Item content="https://api.lorem.space/image/movie?w=150&h=220" />
        <Grid.Item content="https://api.lorem.space/image/movie?w=150&h=220" />
        <Grid.Item content="https://api.lorem.space/image/movie?w=150&h=220" />
      </Grid.Section>
      <Grid.Section columns={8} title="Colors">
        {Object.entries(Color).map(([key, value]) => (
          <Grid.Item key={key} content={{ color: value }} title={key} />
        ))}
      </Grid.Section>
    </Grid>
  );
}
```
{% endtab %}
{% endtabs %}

#### 参数

| 名称          | 描述                                | Type                                                                | Default |
| ----------- | --------------------------------- | ------------------------------------------------------------------- | ------- |
| aspectRatio | Grid.Item 元素的长宽比。默认为 1。           | `"1"` 或 `"3/2"` 或 `"2/3"` 或 `"4/3"` 或 `"3/4"` 或 `"16/9"` 或 `"9/16"` | -       |
| children    | 该 section 的 Grid.Item 元素。         | `React.ReactNode`                                                   | -       |
| columns     | 该 section 的列数。最小值为 1，最大值为 8。      | `number`                                                            | 5       |
| fit         | 适合 Grid.Item 元素内容的排列。默认为“contain” | [`Grid.Fit`](grid.md#grid.fit)                                      | -       |
| inset       | Grid.Item 元素内容的空间。默认为“none”。      | [`Grid.Inset`](grid.md#grid.inset)                                  | -       |
| subtitle    | 该 section 标题旁边的可选子标题。             | `string`                                                            | -       |
| title       | 该 section 上方的标题。                  | `string`                                                            | -       |

## 类型

### Grid.Inset

表示 grid 项的内容与其边框之间应存在的空间量的枚举。绝对值取决于 Grid 或 Grid.Section 的 `columns` 属性的值。

#### 枚举成员

| 名称     | 描述            |
| ------ | ------------- |
| Small  | Small insets  |
| Medium | Medium insets |
| Large  | Large insets  |

### Grid.ItemSize (deprecated)

表示 Grid 的子 Grid.Items 大小的枚举。

#### 枚举成员

| 名称     | 描述     |
| ------ | ------ |
| Small  | 一行 8 个 |
| Medium | 一行 5 个 |
| Large  | 一行 3 个 |

### Grid.Fit

表示 Grid.Item 的内容应如何排版的枚举。

#### 枚举成员

| 名称      | 描述                                           |
| ------- | -------------------------------------------- |
| Contain | 内容将包含在 grid 单元格内，如果其纵横比与单元格不同，则带有 垂直/水平 滚动条。 |
| Fill    | 内容将按比例缩放，以便填充整个单元格；部分内容最终可能会被裁剪掉。            |

