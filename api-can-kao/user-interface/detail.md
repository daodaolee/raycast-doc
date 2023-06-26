# Detail

![](../../.gitbook/assets/detail.png)

## API Reference

### Detail

Renders a markdown ([CommonMark](https://commonmark.org)) string with an optional metadata panel.

Typically used as a standalone view or when navigating from a [List](list.md).

#### Example

{% tabs %}
{% tab title="Render a markdown string" %}
```typescript
import { Detail } from "@raycast/api";

export default function Command() {
  return <Detail markdown="**Hello** _World_!" />;
}
```
{% endtab %}

{% tab title="Render an image from the assets directory" %}
```typescript
import { Detail } from "@raycast/api";

export default function Command() {
  return <Detail markdown={`![Image Title](example.png)`} />;
}
```
{% endtab %}
{% endtabs %}

#### Props

| Prop            | Description                                                                    | Type              | Default       |
| --------------- | ------------------------------------------------------------------------------ | ----------------- | ------------- |
| actions         | A reference to an [ActionPanel](action-panel.md#actionpanel).                  | `React.ReactNode` | -             |
| isLoading       | Indicates whether a loading bar should be shown or hidden below the search bar | `boolean`         | `false`       |
| markdown        | The CommonMark string to be rendered.                                          | `string`          | -             |
| metadata        | The `Detail.Metadata` to be rendered in the right side area                    | `React.ReactNode` | -             |
| navigationTitle | The main title for that view displayed in Raycast                              | `string`          | Command title |

### Detail.Metadata

A Metadata view that will be shown in the right-hand-side of the `Detail`.

Use it to display additional structured data about the main content shown in the `Detail` view.

![Detail-metadata illustration](../../.gitbook/assets/detail-metadata.png)

#### Example

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

#### Props

| Prop                                       | Description                        | Type              | Default |
| ------------------------------------------ | ---------------------------------- | ----------------- | ------- |
| children<mark style="color:red;">\*</mark> | The elements of the Metadata view. | `React.ReactNode` | -       |

### Detail.Metadata.Label

A single value with an optional icon.

![Detail-metadata-label illustration](../../.gitbook/assets/detail-metadata-label.png)

#### Example

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

#### Props

| Prop                                    | Description                                                                                                                                     | Type                                                                 | Default |
| --------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- | ------- |
| title<mark style="color:red;">\*</mark> | The title of the item.                                                                                                                          | `string`                                                             | -       |
| icon                                    | An icon to illustrate the value of the item.                                                                                                    | [`Image.ImageLike`](icons-and-images.md#image.imagelike)             | -       |
| text                                    | The text value of the item. Specifying `color` will display the text in the provided color. Defaults to [Color.SecondaryText](colors.md#color). | `string` or `{ color:` [`Color`](colors.md#color)`; value: string }` | -       |

### Detail.Metadata.Link

An item to display a link.

![Detail-metadata-link illustration](../../.gitbook/assets/detail-metadata-link.png)

#### Example

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

#### Props

| Prop                                     | Description                     | Type     | Default |
| ---------------------------------------- | ------------------------------- | -------- | ------- |
| target<mark style="color:red;">\*</mark> | The target of the link.         | `string` | -       |
| text<mark style="color:red;">\*</mark>   | The text value of the item.     | `string` | -       |
| title<mark style="color:red;">\*</mark>  | The title shown above the item. | `string` | -       |

### Detail.Metadata.TagList

A list of [`Tags`](detail.md#detail.metadata.taglist.item) displayed in a row.

![Detail-metadata-taglist illustration](../../.gitbook/assets/detail-metadata-taglist.png)

#### Example

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

#### Props

| Prop                                       | Description                        | Type              | Default |
| ------------------------------------------ | ---------------------------------- | ----------------- | ------- |
| children<mark style="color:red;">\*</mark> | The tags contained in the TagList. | `React.ReactNode` | -       |
| title<mark style="color:red;">\*</mark>    | The title shown above the item.    | `string`          | -       |

### Detail.Metadata.TagList.Item

A Tag in a `Detail.Metadata.TagList`.

#### Props

| Prop                                   | Description                                                                                         | Type                                                     | Default |
| -------------------------------------- | --------------------------------------------------------------------------------------------------- | -------------------------------------------------------- | ------- |
| text<mark style="color:red;">\*</mark> | The text of the tag.                                                                                | `string`                                                 | -       |
| color                                  | Changes the text color to the provided color and sets a transparent background with the same color. | [`Color.ColorLike`](colors.md#color.colorlike)           | -       |
| icon                                   | An optional icon in front of the text of the tag.                                                   | [`Image.ImageLike`](icons-and-images.md#image.imagelike) | -       |
| onAction                               | Callback that is triggered when the item is clicked.                                                | `() => void`                                             | -       |

### Detail.Metadata.Separator

A metadata item that shows a separator line. Use it for grouping and visually separating metadata items.

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
