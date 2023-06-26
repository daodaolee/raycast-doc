# Navigation

## API Reference

### useNavigation

A hook that lets you push and pop view components in the navigation stack.

You most likely won't use this hook too often. To push a new component, use the [Push Action](actions.md#action.push). When a user presses `ESC`, we automatically pop to the previous component.

#### Signature

```typescript
function useNavigation(): Navigation;
```

#### Example

```typescript
import { Action, ActionPanel, Detail, useNavigation } from "@raycast/api";

function Ping() {
  const { push } = useNavigation();

  return (
    <Detail
      markdown="Ping"
      actions={
        <ActionPanel>
          <Action title="Push" onAction={() => push(<Pong />)} />
        </ActionPanel>
      }
    />
  );
}

function Pong() {
  const { pop } = useNavigation();

  return (
    <Detail
      markdown="Pong"
      actions={
        <ActionPanel>
          <Action title="Pop" onAction={pop} />
        </ActionPanel>
      }
    />
  );
}

export default function Command() {
  return <Ping />;
}
```

#### Return

A [Navigation](navigation.md#navigation) object with [Navigation.push](navigation.md#navigation) and [Navigation.pop](navigation.md#navigation) functions. Use the functions to alter the navigation stack.

## Types

### Navigation

Return type of the [useNavigation](navigation.md#usenavigation) hook to perform push and pop actions.

#### Properties

| Property                               | Description                                           | Type                                   |
| -------------------------------------- | ----------------------------------------------------- | -------------------------------------- |
| pop<mark style="color:red;">\*</mark>  | Pop current view component from the navigation stack. | `() => void`                           |
| push<mark style="color:red;">\*</mark> | Push a new view component to the navigation stack.    | `(component: React.ReactNode) => void` |
