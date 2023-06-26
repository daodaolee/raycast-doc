# Toast

When an asynchronous operation is happening or when an error is thrown, it's usually a good idea to keep the user informed about it. Toasts are made for that.

Additionally, Toasts can have some actions associated to the action they are about. For example, you could provide a way to cancel an asynchronous operation, undo an action, or copy the stack trace of an error.

![](../../.gitbook/assets/toast.png)

## API Reference

### showToast

Creates and shows a Toast with the given [options](toast.md#toast.options).

#### Signature

```typescript
async function showToast(options: Toast.Options): Promise<Toast>;
```

#### Example

```typescript
import { showToast, Toast } from "@raycast/api";

export default async function Command() {
  const success = false;

  if (success) {
    await showToast({ title: "Dinner is ready", message: "Pizza margherita" });
  } else {
    await showToast({
      style: Toast.Style.Failure,
      title: "Dinner isn't ready",
      message: "Pizza dropped on the floor",
    });
  }
}
```

When showing an animated Toast, you can later on update it:

```typescript
import { showToast, Toast } from "@raycast/api";
import { setTimeout } from "timers/promises";

export default async function Command() {
  const toast = await showToast({
    style: Toast.Style.Animated,
    title: "Uploading image",
  });

  try {
    // upload the image
    await setTimeout(1000);

    toast.style = Toast.Style.Success;
    toast.title = "Uploaded image";
  } catch (err) {
    toast.style = Toast.Style.Failure;
    toast.title = "Failed to upload image";
    if (err instanceof Error) {
      toast.message = err.message;
    }
  }
}
```

#### Parameters

| Name                                      | Description                         | Type                                      |
| ----------------------------------------- | ----------------------------------- | ----------------------------------------- |
| options<mark style="color:red;">\*</mark> | The options to customize the Toast. | [`Alert.Options`](alert.md#alert.options) |

#### Return

A Promise that resolves with the shown Toast. The Toast can be used to change or hide it.

## Types

### Toast

A Toast with a certain style, title, and message.

Use [showToast](toast.md#showtoast) to create and show a Toast.

#### Properties

| Property                                          | Description                                                                                                        | Type                                                        |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------- |
| message<mark style="color:red;">\*</mark>         | An additional message for the Toast. Useful to show more information, e.g. an identifier of a newly created asset. | `string`                                                    |
| primaryAction<mark style="color:red;">\*</mark>   | The primary Action the user can take when hovering on the Toast.                                                   | [`Alert.ActionOptions`](alert.md#alert.actionoptions)       |
| secondaryAction<mark style="color:red;">\*</mark> | The secondary Action the user can take when hovering on the Toast.                                                 | [`Alert.ActionOptions`](alert.md#alert.actionoptions)       |
| style<mark style="color:red;">\*</mark>           | The style of a Toast.                                                                                              | [`Action.Style`](../user-interface/actions.md#action.style) |
| title<mark style="color:red;">\*</mark>           | The title of a Toast. Displayed on the top.                                                                        | `string`                                                    |

#### Methods

| Name | Type                  | Description      |
| ---- | --------------------- | ---------------- |
| hide | `() => Promise<void>` | Hides the Toast. |
| show | `() => Promise<void>` | Shows the Toast. |

### Toast.Options

The options to create a [Toast](toast.md#toast).

#### Example

```typescript
import { showToast, Toast } from "@raycast/api";

export default async function Command() {
  const options: Toast.Options = {
    style: Toast.Style.Success,
    title: "Finished cooking",
    message: "Delicious pasta for lunch",
    primaryAction: {
      title: "Do something",
      onAction: (toast) => {
        console.log("The toast action has been triggered");
        toast.hide();
      },
    },
  };
  await showToast(options);
}
```

#### Properties

| Property                                | Description                                                                                                        | Type                                                        |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------- |
| title<mark style="color:red;">\*</mark> | The title of a Toast. Displayed on the top.                                                                        | `string`                                                    |
| message                                 | An additional message for the Toast. Useful to show more information, e.g. an identifier of a newly created asset. | `string`                                                    |
| primaryAction                           | The primary Action the user can take when hovering on the Toast.                                                   | [`Alert.ActionOptions`](alert.md#alert.actionoptions)       |
| secondaryAction                         | The secondary Action the user can take when hovering on the Toast.                                                 | [`Alert.ActionOptions`](alert.md#alert.actionoptions)       |
| style                                   | The style of a Toast.                                                                                              | [`Action.Style`](../user-interface/actions.md#action.style) |

### Toast.Style

Defines the visual style of the Toast.

Use [Toast.Style.Success](toast.md#toast.style) for confirmations and [Toast.Style.Failure](toast.md#toast.style) for displaying errors. Use [Toast.Style.Animated](toast.md#toast.style) when your Toast should be shown until a process is completed. You can hide it later by using [Toast.hide](toast.md#toast) or update the properties of an existing Toast.

#### Enumeration members

| Name     | Value                                         |
| -------- | --------------------------------------------- |
| Animated | ![](../../.gitbook/assets/toast-animated.png) |
| Success  | ![](../../.gitbook/assets/toast-success.png)  |
| Failure  | ![](../../.gitbook/assets/toast-failure.png)  |

### Toast.ActionOptions

The options to create a [Toast](toast.md#toast) Action.

#### Properties

| Property                                   | Description                                     | Type                                                    |
| ------------------------------------------ | ----------------------------------------------- | ------------------------------------------------------- |
| title<mark style="color:red;">\*</mark>    | The title of the action.                        | `string`                                                |
| onAction<mark style="color:red;">\*</mark> | A callback called when the action is triggered. | `(toast:` [`Toast`](toast.md#toast)`) => void`          |
| shortcut                                   | The keyboard shortcut for the action.           | [`Keyboard.Shortcut`](../keyboard.md#keyboard.shortcut) |
