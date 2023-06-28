# List

我们的 `List` 组件提供了开箱即用的用户体验：

* 使用内置过滤以获得最佳性能。&#x20;
* 将相关项分组到带有标题和子标题的 section 中。
* 给长久操作项显示 loading 指示器。&#x20;
* 在限制范围内，使用搜索查询代替单个输入查询。

![](../../.gitbook/assets/list.png)

## 搜索栏

搜索栏允许用户与列表项快速交互。默认情况下，如果用户的输入可以与项目的 `title` 或 `keyword`（模糊）匹配，则显示 [List.Items](list.md#list.item) 。

### 自定义过滤

有时，您可能不想依赖 Raycast 的过滤，而是 使用/实现 您自己的过滤。如果是这种情况，您可以将列表的过滤属性设置为 `false`，这时显示的项目将独立于搜索栏的文本。请注意，如果指定了 `onSearchTextChange` 侦听器，过滤也会隐式设置为 `false`。如果您想指定更改侦听器并仍然利用 Raycast 的内置过滤功能，则需要显式将过滤设置为 `true`。

```typescript
import { useEffect, useState } from "react";
import { Action, ActionPanel, List } from "@raycast/api";

const items = ["Augustiner Helles", "Camden Hells", "Leffe Blonde", "Sierra Nevada IPA"];

export default function Command() {
  const [searchText, setSearchText] = useState("");
  const [filteredList, filterList] = useState(items);

  useEffect(() => {
    filterList(items.filter((item) => item.includes(searchText)));
  }, [searchText]);

  return (
    <List
      filtering={false}
      onSearchTextChange={setSearchText}
      navigationTitle="Search Beers"
      searchBarPlaceholder="Search your favorite beer"
    >
      {filteredList.map((item) => (
        <List.Item
          key={item}
          title={item}
          actions={
            <ActionPanel>
              <Action title="Select" onAction={() => console.log(`${item} selected`)} />
            </ActionPanel>
          }
        />
      ))}
    </List>
  );
}
```

### 用编程的方式更新搜索栏

在其他时候，您可能希望扩展程序更新搜索栏的内容，例如，您可以存储用户之前的搜索列表，并在下次访问时允许他们从上次中断的地方“继续”。

为此，您可以使用 `searchText` 属性。

```typescript
import { useEffect, useState } from "react";
import { Action, ActionPanel, List } from "@raycast/api";

const items = ["Augustiner Helles", "Camden Hells", "Leffe Blonde", "Sierra Nevada IPA"];

export default function Command() {
  const [searchText, setSearchText] = useState("");

  return (
    <List
      searchText={searchText}
      onSearchTextChange={setSearchText}
      navigationTitle="Search Beers"
      searchBarPlaceholder="Search your favorite beer"
    >
      {items.map((item) => (
        <List.Item
          key={item}
          title={item}
          actions={
            <ActionPanel>
              <Action title="Select" onAction={() => setSearchText(item)} />
            </ActionPanel>
          }
        />
      ))}
    </List>
  );
}
```

### 下拉菜单

某些扩展可能会为用户提供第二个过滤维度，比如待办事项扩展可能允许用户使用不同的组，阅读报纸扩展可能希望允许快速切换类别等。

这就是 `searchBarAccessory` 属性有用的地方。向其传递一个 [List.Dropdown](list.md#list.dropdown) 组件，它将显示在搜索栏的右侧。通过使用全局快捷键 `⌘` `P` 或单击它来调用它。

## 例子

{% tabs %}
{% tab title="List.tsx" %}
```jsx
import { List } from "@raycast/api";

export default function Command() {
  return (
    <List>
      <List.Item title="Item 1" />
      <List.Item title="Item 2" subtitle="Optional subtitle" />
    </List>
  );
}
```
{% endtab %}

{% tab title="ListWithSections.tsx" %}
```jsx
import { List } from "@raycast/api";

export default function Command() {
  return (
    <List>
      <List.Section title="Section 1">
        <List.Item title="Item 1" />
      </List.Section>
      <List.Section title="Section 2" subtitle="Optional subtitle">
        <List.Item title="Item 1" />
      </List.Section>
    </List>
  );
}
```
{% endtab %}

{% tab title="ListWithActions.tsx" %}
```jsx
import { ActionPanel, Action, List } from "@raycast/api";

export default function Command() {
  return (
    <List>
      <List.Item
        title="Item 1"
        actions={
          <ActionPanel>
            <Action.CopyToClipboard content="👋" />
          </ActionPanel>
        }
      />
    </List>
  );
}
```
{% endtab %}

{% tab title="ListWithDetail.tsx" %}
```jsx
import { useState } from "react";
import { Action, ActionPanel, List } from "@raycast/api";
import { useCachedPromise } from "@raycast/utils";

interface Pokemon {
  name: string;
  height: number;
  weight: number;
  id: string;
  types: string[];
  abilities: Array<{ name: string; isMainSeries: boolean }>;
}

const pokemons: Pokemon[] = [
  {
    name: "bulbasaur",
    height: 7,
    weight: 69,
    id: "001",
    types: ["Grass", "Poison"],
    abilities: [
      { name: "Chlorophyll", isMainSeries: true },
      { name: "Overgrow", isMainSeries: true },
    ],
  },
  {
    name: "ivysaur",
    height: 10,
    weight: 130,
    id: "002",
    types: ["Grass", "Poison"],
    abilities: [
      { name: "Chlorophyll", isMainSeries: true },
      { name: "Overgrow", isMainSeries: true },
    ],
  },
];

export default function Command() {
  const [showingDetail, setShowingDetail] = useState(true);
  const { data, isLoading } = useCachedPromise(() => new Promise<Pokemon[]>((resolve) => resolve(pokemons)));

  return (
    <List isLoading={isLoading} isShowingDetail={showingDetail}>
      {data &&
        data.map((pokemon) => {
          const props: Partial<List.Item.Props> = showingDetail
            ? {
                detail: (
                  <List.Item.Detail
                    markdown={`![Illustration](https://assets.pokemon.com/assets/cms2/img/pokedex/full/${
                      pokemon.id
                    }.png)\n\n${pokemon.types.join(" ")}`}
                  />
                ),
              }
            : { accessories: [{ text: pokemon.types.join(" ") }] };
          return (
            <List.Item
              key={pokemon.id}
              title={pokemon.name}
              subtitle={`#${pokemon.id}`}
              {...props}
              actions={
                <ActionPanel>
                  <Action.OpenInBrowser url={`https://www.pokemon.com/us/pokedex/${pokemon.name}`} />
                  <Action title="Toggle Detail" onAction={() => setShowingDetail(!showingDetail)} />
                </ActionPanel>
              }
            />
          );
        })}
    </List>
  );
}

```
{% endtab %}

{% tab title="ListWithEmptyView.tsx" %}
```typescript
import { useEffect, useState } from "react";
import { List } from "@raycast/api";

export default function CommandWithCustomEmptyView() {
  const [state, setState] = useState({ searchText: "", items: [] });

  useEffect(() => {
    // perform an API call that eventually populates `items`.
  }, [state.searchText]);

  return (
    <List onSearchTextChange={(newValue) => setState((previous) => ({ ...previous, searchText: newValue }))}>
      {state.searchText === "" && state.items.length === 0 ? (
        <List.EmptyView icon={{ source: "https://placekitten.com/500/500" }} title="Type something to get started" />
      ) : (
        state.items.map((item) => <List.Item key={item} title={item} />)
      )}
    </List>
  );
}
```
{% endtab %}
{% endtabs %}

## API 参考

### List

参考 [List.Section](list.md#list.section) 或 [List.Item](list.md#list.item).

该列表通过对列表项的标题和附加关键字建立索引来使用内置过滤。

#### 例子

```typescript
import { List } from "@raycast/api";

export default function Command() {
  return (
    <List navigationTitle="Search Beers" searchBarPlaceholder="Search your favorite beer">
      <List.Item title="Augustiner Helles" />
      <List.Item title="Camden Hells" />
      <List.Item title="Leffe Blonde" />
      <List.Item title="Sierra Nevada IPA" />
    </List>
  );
}
```

#### 参数

<table><thead><tr><th width="175">名称</th><th>描述</th><th>类型</th><th>默认</th></tr></thead><tbody><tr><td>actions</td><td>对 <a href="action-panel.md#actionpanel">ActionPanel</a> 的引用。仅当没有任何子项时才会显示。</td><td><code>React.ReactNode</code></td><td>-</td></tr><tr><td>children</td><td>列出 section 或 item。如果指定了  <a href="list.md#list.item">List.Item</a> 元素，则会自动创建默认部分。</td><td><code>React.ReactNode</code></td><td>-</td></tr><tr><td>filtering</td><td>切换 Raycast 过滤。如果为 <code>true</code>，Raycast 将使用搜索栏中的查询来过滤项目。当为 <code>false</code> 时，扩展程序需要负责过滤。</td><td><code>boolean</code> or <code>{ keepSectionOrder: boolean }</code></td><td>当指定  <code>onSearchTextChange</code> 时为 <code>false</code>，否则为 <code>true</code>。</td></tr><tr><td>isLoading</td><td>是否在搜索栏下方显示或隐藏 loading</td><td><code>boolean</code></td><td><code>false</code></td></tr><tr><td>isShowingDetail</td><td>列表是否应在项目右侧有一个区域以显示有关所选项的其他详细信息。</td><td><code>boolean</code></td><td>-</td></tr><tr><td>navigationTitle</td><td>Raycast 中显示的该视图的主标题</td><td><code>string</code></td><td>Command title</td></tr><tr><td>searchBarAccessory</td><td><a href="list.md#list.dropdown">List.Dropdown</a> 将显示在搜索栏的右侧。</td><td><code>ReactElement&#x3C;</code><a href="list.md#props"><code>List.Dropdown.Props</code></a><code>, string></code></td><td>-</td></tr><tr><td>searchBarPlaceholder</td><td>显示在搜索栏中的占位符文本。</td><td><code>string</code></td><td><code>"Search…"</code></td></tr><tr><td>searchText</td><td>显示在搜索栏中的文本。</td><td><code>string</code></td><td>-</td></tr><tr><td>selectedItemId</td><td>选择具有指定 id 的项。</td><td><code>string</code></td><td>-</td></tr><tr><td>throttle</td><td><code>onSearchTextChange</code> 处理程序是在每次按下键盘时触发还是延迟以限制事件。当自定义过滤逻辑与异步操作（例如网络请求）结合使用时，建议设置为 <code>true</code>。</td><td><code>boolean</code></td><td><code>false</code></td></tr><tr><td>onSearchTextChange</td><td>当搜索栏文本发生变化时触发回调。</td><td><code>(text: string) => void</code></td><td>-</td></tr><tr><td>onSelectionChange</td><td>当列表中的项选择发生变化时触发回调。</td><td><code>(id: string) => void</code></td><td>-</td></tr></tbody></table>

### List.Dropdown

显示在搜索栏右侧的下拉菜单。

#### 例子

```typescript
import { List } from "@raycast/api";

type DrinkType = { id: string; name: string };

function DrinkDropdown(props: { drinkTypes: DrinkType[]; onDrinkTypeChange: (newValue: string) => void }) {
  const { drinkTypes, onDrinkTypeChange } = props;
  return (
    <List.Dropdown
      tooltip="Select Drink Type"
      storeValue={true}
      onChange={(newValue) => {
        onDrinkTypeChange(newValue);
      }}
    >
      <List.Dropdown.Section title="Alcoholic Beverages">
        {drinkTypes.map((drinkType) => (
          <List.Dropdown.Item key={drinkType.id} title={drinkType.name} value={drinkType.id} />
        ))}
      </List.Dropdown.Section>
    </List.Dropdown>
  );
}

export default function Command() {
  const drinkTypes: DrinkType[] = [
    { id: "1", name: "Beer" },
    { id: "2", name: "Wine" },
  ];
  const onDrinkTypeChange = (newValue: string) => {
    console.log(newValue);
  };
  return (
    <List
      navigationTitle="Search Beers"
      searchBarPlaceholder="Search your favorite drink"
      searchBarAccessory={<DrinkDropdown drinkTypes={drinkTypes} onDrinkTypeChange={onDrinkTypeChange} />}
    >
      <List.Item title="Augustiner Helles" />
      <List.Item title="Camden Hells" />
      <List.Item title="Leffe Blonde" />
      <List.Item title="Sierra Nevada IPA" />
    </List>
  );
}
```

#### 参数

<table><thead><tr><th width="168">名称</th><th>描述</th><th>类型</th><th>默认</th></tr></thead><tbody><tr><td>tooltip<mark style="color:red;">*</mark></td><td>鼠标悬停在下拉菜单上时显示提示。</td><td><code>string</code></td><td>-</td></tr><tr><td>children</td><td>下拉部分或 item。如果指定了 <code>Dropdown.Item</code> 元素，则会自动创建默认部分。</td><td><code>React.ReactNode</code></td><td>-</td></tr><tr><td>defaultValue</td><td>下拉列表的默认值。请记住，<code>defaultValue</code> 将在每个组件生命周期配置一次。这意味着如果用户更改该值，则重新渲染时不会配置 <code>defaultValue</code>。</td><td><code>string</code></td><td>-</td></tr><tr><td>filtering</td><td>切换 Raycast 过滤。如果为 <code>true</code>，Raycast 将使用搜索栏中的查询来过滤项目。当为 <code>false</code> 时，扩展程序需要负责过滤。</td><td><code>boolean</code> or <code>{ keepSectionOrder: boolean }</code></td><td>当指定 <code>onSearchTextChange</code> 时为 <code>false</code>，否则为 <code>true</code>。</td></tr><tr><td>id</td><td>下拉列表的 ID。</td><td><code>string</code></td><td>-</td></tr><tr><td>isLoading</td><td>是否应在搜索栏旁边显示或隐藏 loading 指示器</td><td><code>boolean</code></td><td><code>false</code></td></tr><tr><td>placeholder</td><td>显示在下拉搜索字段中的占位符文本。</td><td><code>string</code></td><td><code>"Search…"</code></td></tr><tr><td>storeValue</td><td>下拉列表的值是否应在选择后保留，并在下次渲染下拉列表时恢复。</td><td><code>boolean</code></td><td>-</td></tr><tr><td>throttle</td><td>定义 <code>onSearchTextChange</code> 处理程序是在每次按下键盘时触发还是延迟以限制事件。当将自定义过滤逻辑与异步操作（例如网络请求）结合使用时，建议设置为 <code>true</code>。</td><td><code>boolean</code></td><td><code>false</code></td></tr><tr><td>value</td><td>下拉菜单的当前值。</td><td><code>string</code></td><td>-</td></tr><tr><td>onChange</td><td>当下拉选择发生变化时触发回调。</td><td><code>(newValue: string) => void</code></td><td>-</td></tr><tr><td>onSearchTextChange</td><td>当搜索栏文本发生变化时触发回调。</td><td><code>(text: string) => void</code></td><td>-</td></tr></tbody></table>

### List.Dropdown.Item

[List.Dropdown](list.md#list.dropdown) 中的下拉项

#### 例子

```typescript
import { List } from "@raycast/api";

export default function Command() {
  return (
    <List
      searchBarAccessory={
        <List.Dropdown tooltip="Dropdown With Items">
          <List.Dropdown.Item title="One" value="one" />
          <List.Dropdown.Item title="Two" value="two" />
          <List.Dropdown.Item title="Three" value="three" />
        </List.Dropdown>
      }
    >
      <List.Item title="Item in the Main List" />
    </List>
  );
}
```

#### 参数

<table><thead><tr><th width="145">名称</th><th>描述</th><th>类型</th><th>默认</th></tr></thead><tbody><tr><td>title<mark style="color:red;">*</mark></td><td>显示的标题。</td><td><code>string</code></td><td>-</td></tr><tr><td>value<mark style="color:red;">*</mark></td><td>下拉项的值。确保为每个项目分配每个唯一值。</td><td><code>string</code></td><td>-</td></tr><tr><td>icon</td><td>该项的可选图标。</td><td><a href="icons-and-images.md#image.imagelike"><code>Image.ImageLike</code></a></td><td>-</td></tr><tr><td>keywords</td><td>一个可选属性，用于提供额外的可索引字符串以供搜索。在 Raycast 中过滤项目时，除了标题之外还会搜索关键字。</td><td><code>string[]</code></td><td>The title of its section if any</td></tr></tbody></table>

### List.Dropdown.Section

分离的下拉项组。

使用 section 将相关菜单项分到一个组。

#### 例子

```typescript
import { List } from "@raycast/api";

export default function Command() {
  return (
    <List
      searchBarAccessory={
        <List.Dropdown tooltip="Dropdown With Sections">
          <List.Dropdown.Section title="First Section">
            <List.Dropdown.Item title="One" value="one" />
          </List.Dropdown.Section>
          <List.Dropdown.Section title="Second Section">
            <List.Dropdown.Item title="Two" value="two" />
          </List.Dropdown.Section>
        </List.Dropdown>
      }
    >
      <List.Item title="Item in the Main List" />
    </List>
  );
}
```

#### 参数

<table><thead><tr><th width="142">名称</th><th>描述</th><th>类型</th><th>默认</th></tr></thead><tbody><tr><td>children</td><td>section 项</td><td><code>React.ReactNode</code></td><td>-</td></tr><tr><td>title</td><td>显示在该部分上方的标题</td><td><code>string</code></td><td>-</td></tr></tbody></table>

### List.EmptyView

当没有任何可用项目时显示的视图。

Raycast 提供了一个默认的 `EmptyView`，如果 List 组件没有子组件，或者有子组件，但没有一个与搜索栏中的查询匹配，则将显示该 `EmptyView`。也可以通过与其他 `List.Item` 一起传递空视图来覆盖。

请注意，如果 List 的 `isLoading` 属性为 `true` 并且搜索栏为空，则永远不会显示 `EmptyView`。

![列表为空的插图](../../.gitbook/assets/list-empty-view.png)

#### 例子

```typescript
import { useEffect, useState } from "react";
import { List } from "@raycast/api";

export default function CommandWithCustomEmptyView() {
  const [state, setState] = useState({ searchText: "", items: [] });

  useEffect(() => {
    // perform an API call that eventually populates `items`.
  }, [state.searchText]);

  return (
    <List onSearchTextChange={(newValue) => setState((previous) => ({ ...previous, searchText: newValue }))}>
      {state.searchText === "" && state.items.length === 0 ? (
        <List.EmptyView icon={{ source: "https://placekitten.com/500/500" }} title="Type something to get started" />
      ) : (
        state.items.map((item) => <List.Item key={item} title={item} />)
      )}
    </List>
  );
}
```

#### 参数

| 名称          | 描述                                                | 类型                                                       | 默认 |
| ----------- | ------------------------------------------------- | -------------------------------------------------------- | -- |
| actions     | 对 [ActionPanel](action-panel.md#actionpanel) 的引用。 | `React.ReactNode`                                        | -  |
| description | 空视图的可选描述。                                         | `string`                                                 | -  |
| icon        | 在 EmptyView 中心的图标。                                | [`Image.ImageLike`](icons-and-images.md#image.imagelike) | -  |
| title       | 空视图的主标题。                                          | `string`                                                 | -  |

### List.Item

[List](list.md#list) 中的一项

这是 Raycast 的基本 UI 组件之一。列表项代表单个实体。它可以是 GitHub PR、文件或其他任何内容。您很可能想要对此项目执行操作，因此请让用户清楚该列表项的含义。

#### 例子

```typescript
import { Icon, List } from "@raycast/api";

export default function Command() {
  return (
    <List>
      <List.Item icon={Icon.Star} title="Augustiner Helles" subtitle="0,5 Liter" accessories={[{ text: "Germany" }]} />
    </List>
  );
}
```

#### 参数

<table><thead><tr><th width="160">名称</th><th width="253">描述</th><th>类型</th><th>默认</th></tr></thead><tbody><tr><td>title<mark style="color:red;">*</mark></td><td>显示的主标题，可以选择带有提示。</td><td><code>string</code> 或 <code>{ tooltip: string; value: string }</code></td><td>-</td></tr><tr><td>accessories</td><td>显示在 List.Item 右侧的可选 <a href="list.md#list.item.accessory">List.Item.Accessory</a> 项数组。</td><td><a href="list.md#list.item.accessory"><code>List.Item.Accessory</code></a><code>[]</code></td><td>-</td></tr><tr><td>actions</td><td>将为所选列表项更新的 ActionPanel。</td><td><code>React.ReactNode</code></td><td>-</td></tr><tr><td>detail</td><td>当父列表显示详细信息并且选择该项目时，List.Item.Detail 将在右侧区域呈现。</td><td><code>React.ReactNode</code></td><td>-</td></tr><tr><td>icon</td><td>列表项的可选图标。</td><td><a href="icons-and-images.md#image.imagelike"><code>Image.ImageLike</code></a> 或 <code>{ tooltip: string; value:</code> <a href="icons-and-images.md#image.imagelike"><code>Image.ImageLike</code></a> <code>}</code></td><td>-</td></tr><tr><td>id</td><td>当选择该项时，此字符串将传递到列表的 <code>onSelectionChange</code> 处理。确保为每个项目分配一个唯一的 ID，否则将自动生成 UUID。</td><td><code>string</code></td><td>-</td></tr><tr><td>keywords</td><td>一个可选属性，用于提供额外的可索引字符串以供搜索。当通过搜索栏过滤 Raycast 中的列表时，除了标题之外还会搜索关键字。</td><td><code>string[]</code></td><td>-</td></tr><tr><td>quickLook</td><td>使用 “Quick Look” 预览文件的可选信息。使用 <a href="actions.md#action.togglequicklook">Action.ToggleQuickLook</a> 切换预览。</td><td><code>{ name: string; path: string }</code></td><td>-</td></tr><tr><td>subtitle</td><td>主标题旁边的可选子标题，可以选择带有提示。</td><td><code>string</code> 或 <code>{ tooltip: string; value: string }</code></td><td>-</td></tr></tbody></table>

### List.Item.Detail

在列表右侧的详细信息视图。 显示时，建议不要在 `List.Item` 上显示任何附件，最好将这些附加信息显示在 `List.Item.Detail` 视图中。

![列表详细说明](../../.gitbook/assets/list-detail.png)

#### 例子

```typescript
import { List } from "@raycast/api";

export default function Command() {
  return (
    <List isShowingDetail>
      <List.Item
        title="Pikachu"
        subtitle="Electric"
        detail={
          <List.Item.Detail markdown="![Illustration](https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/25.png)" />
        }
      />
    </List>
  );
}
```

#### 参数

<table><thead><tr><th width="166">名称</th><th width="244">描述</th><th>类型</th><th>默认</th></tr></thead><tbody><tr><td>isLoading</td><td>是否应在详细信息上方显示或隐藏 loading 栏</td><td><code>boolean</code></td><td><code>false</code></td></tr><tr><td>markdown</td><td>当父列表显示详细信息并且选择该项时，要在右侧区域呈现的 <code>CommonMark</code> 字符串。</td><td><code>string</code></td><td>-</td></tr><tr><td>metadata</td><td>要在 <code>List.Item.Detail</code> 底部呈现的 <code>List.Item.Detail.Metadata</code></td><td><code>React.ReactNode</code></td><td>-</td></tr></tbody></table>

### List.Item.Detail.Metadata

在 `List.Item.Detail` 底部的元数据视图。 使用它来显示有关 `List.Item` 内容的附加结构化数据。

#### 例子

{% tabs %}
{% tab title="Metadata + Markdown" %}
![列表详细信息-元数据说明](../../.gitbook/assets/list-detail-metadata-split.png)

```typescript
import { List } from "@raycast/api";

export default function Metadata() {
  const markdown = `
![Illustration](https://assets.pokemon.com/assets/cms2/img/pokedex/full/001.png)
There is a plant seed on its back right from the day this Pokémon is born. The seed slowly grows larger.
`;
  return (
    <List isShowingDetail>
      <List.Item
        title="Bulbasaur"
        detail={
          <List.Item.Detail
            markdown={markdown}
            metadata={
              <List.Item.Detail.Metadata>
                <List.Item.Detail.Metadata.Label title="Types" />
                <List.Item.Detail.Metadata.Label title="Grass" icon="pokemon_types/grass.svg" />
                <List.Item.Detail.Metadata.Separator />
                <List.Item.Detail.Metadata.Label title="Poison" icon="pokemon_types/poison.svg" />
                <List.Item.Detail.Metadata.Separator />
                <List.Item.Detail.Metadata.Label title="Chracteristics" />
                <List.Item.Detail.Metadata.Label title="Height" text="70cm" />
                <List.Item.Detail.Metadata.Separator />
                <List.Item.Detail.Metadata.Label title="Weight" text="6.9 kg" />
                <List.Item.Detail.Metadata.Separator />
                <List.Item.Detail.Metadata.Label title="Abilities" />
                <List.Item.Detail.Metadata.Label title="Chlorophyll" text="Main Series" />
                <List.Item.Detail.Metadata.Separator />
                <List.Item.Detail.Metadata.Label title="Overgrow" text="Main Series" />
                <List.Item.Detail.Metadata.Separator />
              </List.Item.Detail.Metadata>
            }
          />
        }
      />
    </List>
  );
}
```
{% endtab %}

{% tab title="单独的 Metadata" %}
![List Detail-metadata illustration](../../.gitbook/assets/list-detail-metadata-standalone.png)

```typescript
import { List } from "@raycast/api";

export default function Metadata() {
  return (
    <List isShowingDetail>
      <List.Item
        title="Bulbasaur"
        detail={
          <List.Item.Detail
            metadata={
              <List.Item.Detail.Metadata>
                <List.Item.Detail.Metadata.Label title="Types" />
                <List.Item.Detail.Metadata.Label title="Grass" icon="pokemon_types/grass.svg" />
                <List.Item.Detail.Metadata.Separator />
                <List.Item.Detail.Metadata.Label title="Poison" icon="pokemon_types/poison.svg" />
                <List.Item.Detail.Metadata.Separator />
                <List.Item.Detail.Metadata.Label title="Chracteristics" />
                <List.Item.Detail.Metadata.Label title="Height" text="70cm" />
                <List.Item.Detail.Metadata.Separator />
                <List.Item.Detail.Metadata.Label title="Weight" text="6.9 kg" />
                <List.Item.Detail.Metadata.Separator />
                <List.Item.Detail.Metadata.Label title="Abilities" />
                <List.Item.Detail.Metadata.Label title="Chlorophyll" text="Main Series" />
                <List.Item.Detail.Metadata.Separator />
                <List.Item.Detail.Metadata.Label title="Overgrow" text="Main Series" />
                <List.Item.Detail.Metadata.Separator />
              </List.Item.Detail.Metadata>
            }
          />
        }
      />
    </List>
  );
}
```
{% endtab %}
{% endtabs %}

#### 参数

<table><thead><tr><th width="142">名称</th><th width="230">描述</th><th>类型</th><th>默认</th></tr></thead><tbody><tr><td>children<mark style="color:red;">*</mark></td><td>元数据视图的元素。</td><td><code>React.ReactNode</code></td><td>-</td></tr></tbody></table>

### List.Item.Detail.Metadata.Label

标题的右侧可以选择带有图标 和/或 文本。

![List Detail-metadata-label illustration](../../.gitbook/assets/list-detail-metadata-label.png)

#### 例子

```typescript
import { List } from "@raycast/api";

export default function Metadata() {
  return (
    <List isShowingDetail>
      <List.Item
        title="Bulbasaur"
        detail={
          <List.Item.Detail
            metadata={
              <List.Item.Detail.Metadata>
                <List.Item.Detail.Metadata.Label title="Type" icon="pokemon_types/grass.svg" text="Grass" />
              </List.Item.Detail.Metadata>
            }
          />
        }
      />
    </List>
  );
}
```

#### 参数

<table><thead><tr><th width="174">名称</th><th width="269">描述</th><th>类型</th><th>默认</th></tr></thead><tbody><tr><td>title<mark style="color:red;">*</mark></td><td>该项的标题。</td><td><code>string</code></td><td>-</td></tr><tr><td>icon</td><td>该项的图标。</td><td><a href="icons-and-images.md#image.imagelike"><code>Image.ImageLike</code></a></td><td>-</td></tr><tr><td>text</td><td>该项的文本。指定颜色将以提供的颜色显示文本。默认为 <a href="colors.md#color">Color.SecondaryText</a>。</td><td><code>string</code> 或 <code>{ color:</code> <a href="colors.md#color"><code>Color</code></a><code>; value: string }</code></td><td>-</td></tr></tbody></table>

### List.Item.Detail.Metadata.Link

显示链接的项。

![列表详细信息-元数据-链接说明](../../.gitbook/assets/list-detail-metadata-link.png)

#### 例子

```typescript
import { List } from "@raycast/api";

export default function Metadata() {
  return (
    <List isShowingDetail>
      <List.Item
        title="Bulbasaur"
        detail={
          <List.Item.Detail
            metadata={
              <List.Item.Detail.Metadata>
                <List.Item.Detail.Metadata.Link
                  title="Evolution"
                  target="https://www.pokemon.com/us/pokedex/pikachu"
                  text="Raichu"
                />
              </List.Item.Detail.Metadata>
            }
          />
        }
      />
    </List>
  );
}
```

#### 参数

| 名称                                       | 描述        | 类型       | 默认 |
| ---------------------------------------- | --------- | -------- | -- |
| target<mark style="color:red;">\*</mark> | 链接的目标。    | `string` | -  |
| text<mark style="color:red;">\*</mark>   | 项目的文本。    | `string` | -  |
| title<mark style="color:red;">\*</mark>  | 在项目上方的标题。 | `string` | -  |

### List.Item.Detail.Metadata.TagList

单行显示的  [`Tags`](list.md#list.item.detail.metadata.taglist.item)  列表。

![列表详细信息-元数据-标签-列表插图](../../.gitbook/assets/list-detail-metadata-tag-list.png)

#### 例子

```typescript
import { List } from "@raycast/api";

export default function Metadata() {
  return (
    <List isShowingDetail>
      <List.Item
        title="Bulbasaur"
        detail={
          <List.Item.Detail
            metadata={
              <List.Item.Detail.Metadata>
                <List.Item.Detail.Metadata.TagList title="Type">
                  <List.Item.Detail.Metadata.TagList.Item text="Electric" color={"#eed535"} />
                </List.Item.Detail.Metadata.TagList>
              </List.Item.Detail.Metadata>
            }
          />
        }
      />
    </List>
  );
}
```

#### 参数

| 名称                                         | 描述                | 类型                | 默认 |
| ------------------------------------------ | ----------------- | ----------------- | -- |
| children<mark style="color:red;">\*</mark> | TagList 中包含的 tag。 | `React.ReactNode` | -  |
| title<mark style="color:red;">\*</mark>    | 在项目上方的标题。         | `string`          | -  |

### List.Item.Detail.Metadata.TagList.Item

`List.Item.Detail.Metadata.TagList` 中的 tag。

#### 参数

| 名称                                     | 描述                          | 类型                                                       | 默认 |
| -------------------------------------- | --------------------------- | -------------------------------------------------------- | -- |
| text<mark style="color:red;">\*</mark> | tag 的文本。                    | `string`                                                 | -  |
| color                                  | 将文本颜色更改为提供的颜色，并设置相同颜色的透明背景。 | [`Color.ColorLike`](colors.md#color.colorlike)           | -  |
| icon                                   | 标签文本前面的可选图标。                | [`Image.ImageLike`](icons-and-images.md#image.imagelike) | -  |
| onAction                               | 单击该项时触发的回调。                 | `() => void`                                             | -  |

### List.Item.Detail.Metadata.Separator

显示分隔线的元数据项。使用它来分组和直观地分隔元数据项。

![列表详细信息-元数据-分隔符说明](../../.gitbook/assets/list-detail-metadata-separator.png)

#### 例子

```typescript
import { List } from "@raycast/api";

export default function Metadata() {
  return (
    <List isShowingDetail>
      <List.Item
        title="Bulbasaur"
        detail={
          <List.Item.Detail
            metadata={
              <List.Item.Detail.Metadata>
                <List.Item.Detail.Metadata.Label title="Type" icon="pokemon_types/grass.svg" text="Grass" />
                <List.Item.Detail.Metadata.Separator />
                <List.Item.Detail.Metadata.Label title="Type" icon="pokemon_types/poison.svg" text="Poison" />
              </List.Item.Detail.Metadata>
            }
          />
        }
      />
    </List>
  );
}
```

### List.Section

[List.Item](list.md#list.item).

section 是构建列表的好方法。例如，将具有相同状态的 GitHub 问题分组并按优先级排序。这样，用户可以快速访问最相关的内容。

#### 例子

```typescript
import { List } from "@raycast/api";

export default function Command() {
  return (
    <List>
      <List.Section title="Lager">
        <List.Item title="Camden Hells" />
      </List.Section>
      <List.Section title="IPA">
        <List.Item title="Sierra Nevada IPA" />
      </List.Section>
    </List>
  );
}
```

#### 参数

<table><thead><tr><th width="180">名称</th><th>描述</th><th>类型</th><th>默认</th></tr></thead><tbody><tr><td>children</td><td>section 的 <a href="list.md#list.item">List.Item</a> </td><td><code>React.ReactNode</code></td><td>-</td></tr><tr><td>subtitle</td><td>在该 section 标题旁边的可选子标题。</td><td><code>string</code></td><td>-</td></tr><tr><td>title</td><td>该 section 上方的标题。</td><td><code>string</code></td><td>-</td></tr></tbody></table>

## 类型

### List.Item.Accessory

描述 `List.Item` 中的附件视图的接口。

![List.Item 装饰图](../../.gitbook/assets/list-item-accessories.png)

#### 属性

| 名称                                    | 描述                                                                                                                                                     | 类型                                                                                                                                                      |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| tag<mark style="color:red;">\*</mark> |  label 的字符串或日期，可以选择颜色。日期的格式相对于当前时间（例如 `new Date()` 将显示为“`now`”，昨天的日期将显示为“1d”等）。颜色会把文本更改为提供的颜色，并设置相同颜色的透明背景。默认为 [Color.SecondaryText](colors.md#color)。 | `string` 或 `Date` 或 `undefined` 或 `null` 或 `{ color:` [`Color.ColorLike`](colors.md#color.colorlike)`; value: string` 或 `Date` 或 `undefined` 或 `null }` |
| text                                  |  label 的可选文本，可以选择颜色。颜色将文本颜色更改为提供的颜色。默认为 [Color.SecondaryText](colors.md#color).                                                                        | `string` 或 `null` 或 `{ color:` [`Color`](colors.md#color)`; value: string` 或 `undefined` 或 `null }`                                                     |
| date                                  | 将用作 label 的可选日期，可以设置颜色。日期的格式相对于当前时间（例如 `new Date()` 将显示为“`now`”，昨天的日期将显示为“1d”等）。颜色会把文本更改为提供的颜色。 默认为 [Color.SecondaryText](colors.md#color).            | `Date` 或 `null` 或 `color:` [`Color`](colors.md#color)`; value: Date` 或 `undefined` 或 `null }`                                                           |
| icon                                  | 图标的可选 [Image.ImageLike](icons-and-images.md#image.imagelike)。                                                                                          | [`Image.ImageLike`](icons-and-images.md#image.imagelike) 或 `null`                                                                                       |
| tooltip                               | 当附件悬停时显示的提示。                                                                                                                                           | `string` 或 `null`                                                                                                                                       |

#### 例子

```typescript
import { Color, Icon, List } from "@raycast/api";

export default function Command() {
  return (
    <List>
      <List.Item
        title="An Item with Accessories"
        accessories={[
          { text: `An Accessory Text`, icon: Icon.Hammer },
          { text: { value: `A Colored Accessory Text`, color: Color.Orange }, icon: Icon.Hammer },
          { icon: Icon.Person, tooltip: "A person" },
          { text: "Just Do It!" },
          { date: new Date() },
          { tag: new Date() },
          { tag: { value: new Date(), color: Color.Magenta } },
          { tag: { value: "User", color: Color.Magenta }, tooltip: "Tag with tooltip" },
        ]}
      />
    </List>
  );
}
```
