# Detail

![](../../.gitbook/assets/detail.png)

## API 参考

### Detail

使用可选的元数据面板渲染 markdown ([CommonMark](https://commonmark.org)) 字符串。

通常用作独立视图或从  [List](list.md) 导航时使用。

#### 例子

{% tabs %}
{% tab title="渲染 Markdown 字符串" %}
```typescript
import { Detail } from "@raycast/api";

export default function Command() {
  return <Detail markdown="**Hello** _World_!" />;
}
```
{% endtab %}

{% tab title="从资源目录渲染图像" %}
```typescript
import { Detail } from "@raycast/api";

export default function Command() {
  return <Detail markdown={`![Image Title](example.png)`} />;
}
```
{% endtab %}
{% endtabs %}

#### 参数

| 名称              | 描述                                                | 类型                | 默认            |
| --------------- | ------------------------------------------------- | ----------------- | ------------- |
| actions         | 对 [ActionPanel](action-panel.md#actionpanel) 的引用。 | `React.ReactNode` | -             |
| isLoading       | 是否应在搜索栏下方显示或隐藏 loading 栏                          | `boolean`         | `false`       |
| markdown        | 要渲染的 CommonMark 字符串。                              | `string`          | -             |
| metadata        | 要在右侧区域渲染的 `Detail.Metadata`                       | `React.ReactNode` | -             |
| navigationTitle | Raycast 中显示的该视图的主标题                               | `string`          | Command title |

### Detail.Metadata

元数据视图将显示在  `Detail` 的右侧。 使用它来显示有关  `Detail` 视图中显示的主要内容的附加结构化数据。

![详细元数据说明](../../.gitbook/assets/detail-metadata.png)

#### 例子

```typescript
import { Detail } from "@raycast/api";

// Define markdown here to prevent unwanted indentation.
const markdown = `
# Pikachu

![](https://assets.pokemon.com/assets/cms2/img/pokedex/full/025.png)

Pikachu that can generate powerful electricity have cheek sacs that are extra soft and super stretchy.
`;

export default function Main() {
  return (
    <Detail
      markdown={markdown}
      navigationTitle="Pikachu"
      metadata={
        <Detail.Metadata>
          <Detail.Metadata.Label title="Height" text={`1' 04"`} />
          <Detail.Metadata.Label title="Weight" text="13.2 lbs" />
          <Detail.Metadata.TagList title="Type">
            <Detail.Metadata.TagList.Item text="Electric" color={"#eed535"} />
          </Detail.Metadata.TagList>
          <Detail.Metadata.Separator />
          <Detail.Metadata.Link title="Evolution" target="https://www.pokemon.com/us/pokedex/pikachu" text="Raichu" />
        </Detail.Metadata>
      }
    />
  );
}
```

#### 参数

<table><thead><tr><th width="160">名称</th><th>描述</th><th>类型</th><th>默认</th></tr></thead><tbody><tr><td>children<mark style="color:red;">*</mark></td><td>元数据视图的元素。</td><td><code>React.ReactNode</code></td><td>-</td></tr></tbody></table>

### Detail.Metadata.Label

带有可选图标的单个值。

![Detail-metadata-label illustration](../../.gitbook/assets/detail-metadata-label.png)

#### 例子

```typescript
import { Detail } from "@raycast/api";

// Define markdown here to prevent unwanted indentation.
const markdown = `
# Pikachu

![](https://assets.pokemon.com/assets/cms2/img/pokedex/full/025.png)

Pikachu that can generate powerful electricity have cheek sacs that are extra soft and super stretchy.
`;

export default function Main() {
  return (
    <Detail
      markdown={markdown}
      navigationTitle="Pikachu"
      metadata={
        <Detail.Metadata>
          <Detail.Metadata.Label title="Height" text={`1' 04"`} icon="weight.svg" />
        </Detail.Metadata>
      }
    />
  );
}
```

#### 参数

<table><thead><tr><th width="122">名称</th><th>描述</th><th width="192">类型</th><th>默认</th></tr></thead><tbody><tr><td>title<mark style="color:red;">*</mark></td><td>项目的标题。</td><td><code>string</code></td><td>-</td></tr><tr><td>icon</td><td>项目的图标。</td><td><a href="icons-and-images.md#image.imagelike"><code>Image.ImageLike</code></a></td><td>-</td></tr><tr><td>text</td><td>指定颜色以显示文本。默认为 <a href="colors.md#color">Color.SecondaryText</a>。</td><td><code>string</code> 或 <code>{ color:</code> <a href="colors.md#color"><code>Color</code></a><code>; value: string }</code></td><td>-</td></tr></tbody></table>

### Detail.Metadata.Link

显示链接的项目。

![Detail-metadata-link illustration](../../.gitbook/assets/detail-metadata-link.png)

#### 例子

```typescript
import { Detail } from "@raycast/api";

// Define markdown here to prevent unwanted indentation.
const markdown = `
# Pikachu

![](https://assets.pokemon.com/assets/cms2/img/pokedex/full/025.png)

Pikachu that can generate powerful electricity have cheek sacs that are extra soft and super stretchy.
`;

export default function Main() {
  return (
    <Detail
      markdown={markdown}
      navigationTitle="Pikachu"
      metadata={
        <Detail.Metadata>
          <Detail.Metadata.Link title="Evolution" target="https://www.pokemon.com/us/pokedex/pikachu" text="Raichu" />
        </Detail.Metadata>
      }
    />
  );
}
```

#### 参数

| 名称                                       | 描述          | 类型       | 默认 |
| ---------------------------------------- | ----------- | -------- | -- |
| target<mark style="color:red;">\*</mark> | 链接的目标。      | `string` | -  |
| text<mark style="color:red;">\*</mark>   | 项目的文本值。     | `string` | -  |
| title<mark style="color:red;">\*</mark>  | 显示在项目上方的标题。 | `string` | -  |

### Detail.Metadata.TagList

一行显示的  [`Tags`](detail.md#detail.metadata.taglist.item) 列表。

![详细元数据标签列表说明](../../.gitbook/assets/detail-metadata-taglist.png)

#### 例子

```typescript
import { Detail } from "@raycast/api";

// Define markdown here to prevent unwanted indentation.
const markdown = `
# Pikachu

![](https://assets.pokemon.com/assets/cms2/img/pokedex/full/025.png)

Pikachu that can generate powerful electricity have cheek sacs that are extra soft and super stretchy.
`;

export default function Main() {
  return (
    <Detail
      markdown={markdown}
      navigationTitle="Pikachu"
      metadata={
        <Detail.Metadata>
          <Detail.Metadata.TagList title="Type">
            <Detail.Metadata.TagList.Item text="Electric" color={"#eed535"} />
          </Detail.Metadata.TagList>
        </Detail.Metadata>
      }
    />
  );
}
```

#### 参数

<table><thead><tr><th width="130">名称</th><th>描述</th><th>类型</th><th>Default</th></tr></thead><tbody><tr><td>children<mark style="color:red;">*</mark></td><td>TagList 中包含的标签。</td><td><code>React.ReactNode</code></td><td>-</td></tr><tr><td>title<mark style="color:red;">*</mark></td><td>显示在项目上方的标题。</td><td><code>string</code></td><td>-</td></tr></tbody></table>

### Detail.Metadata.TagList.Item

`Detail.Metadata.TagList`. 中的 标签。

#### 参数

| 名称                                     | 描述                          | 类型                                                       | 默认 |
| -------------------------------------- | --------------------------- | -------------------------------------------------------- | -- |
| text<mark style="color:red;">\*</mark> | 标签的文本值                      | `string`                                                 | -  |
| color                                  | 将文本颜色更改为提供的颜色，并设置相同颜色的透明背景。 | [`Color.ColorLike`](colors.md#color.colorlike)           | -  |
| icon                                   | 标签文本前面的可选图标。                | [`Image.ImageLike`](icons-and-images.md#image.imagelike) | -  |
| onAction                               | 单击该项目时触发的回调。                | `() => void`                                             | -  |

### Detail.Metadata.Separator

显示分隔线的元数据项。使用它来分组和直观地分离元数据项。

![](../../.gitbook/assets/detail-metadata-separator.png)

```typescript
import { Detail } from "@raycast/api";

// Define markdown here to prevent unwanted indentation.
const markdown = `
# Pikachu

![](https://assets.pokemon.com/assets/cms2/img/pokedex/full/025.png)

Pikachu that can generate powerful electricity have cheek sacs that are extra soft and super stretchy.
`;

export default function Main() {
  return (
    <Detail
      markdown={markdown}
      navigationTitle="Pikachu"
      metadata={
        <Detail.Metadata>
          <Detail.Metadata.Label title="Height" text={`1' 04"`} />
          <Detail.Metadata.Separator />
          <Detail.Metadata.Label title="Weight" text="13.2 lbs" />
        </Detail.Metadata>
      }
    />
  );
}
```
