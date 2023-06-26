# Command

This set of utilities to work with Raycast commands.

## API Reference

### launchCommand

Launches another command. If the command does not exist, or if it's not enabled, an error will be thrown. If the command is part of another extension, the user will be presented with a permission alert. Use this method if your command needs to open another command based on user interaction, or when an immediate background refresh should be triggered, for example when a command needs to update an associated menu-bar command.

#### Signature

```typescript
export async function launchCommand(options: LaunchOptions): Promise<void>;
```

#### Example

```typescript
import { launchCommand, LaunchType } from "@raycast/api";

export default async function Command() {
  await launchCommand({ name: "list", type: LaunchType.UserInitiated, context: { foo: "bar" } });
}
```

#### Parameters

| Name                                      | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | Type                                        |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------- |
| options<mark style="color:red;">\*</mark> | A parameter object with the properties: `name`: command name as defined in the extension's manifest `type`: [LaunchType.UserInitiated](environment.md#launchtype) or [LaunchType.Background](environment.md#launchtype) `arguments`: optional object for the argument properties and values as defined in the extension's manifest, for example: `{ "argument1": "value1" }` `context`: arbitrary object for custom data that should be passed to the command and accessible as `environment.launchContext`; the object must be JSON serializable (Dates and Buffers supported) | [`LaunchOptions`](command.md#launchoptions) |

#### Return

A Promise that resolves when the command has been launched. (Note that this does not indicate that the launched command has finished executing.)

### updateCommandMetadata

Update the values of properties declared in the manifest of the current command. Note that currently only `subtitle` is supported. Pass `null` to clear the custom subtitle.

{% hint style="info" %}
The actual manifest file is not modified, so the update applies as long as the command remains installed.
{% endhint %}

#### Signature

```typescript
export async function updateCommandMetadata(metadata: { subtitle?: string | null }): Promise<void>;
```

#### Example

```typescript
import { updateCommandMetadata } from "@raycast/api";

async function fetchUnreadNotificationCount() {
  return 10;
}

export default async function Command() {
  const count = await fetchUnreadNotificationCount();
  await updateCommandMetadata({ subtitle: `Unread Notifications: ${count}` });
}
```

#### Return

A Promise that resolves when the command's metadata have been updated.

## Types

### LaunchContext

Represents the passed context object of programmatic command launches.

### LaunchOptions

A parameter object used to decide which command should be launched and what data (arguments, context) it should receive.

#### IntraExtensionLaunchOptions

The options that can be used when launching a command from the same extension.

| Property                               | Description                                                                                                                                                                             | Type                                                                 |
| -------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| name<mark style="color:red;">\*</mark> | command name as defined in the extension's manifest                                                                                                                                     | `string`                                                             |
| type<mark style="color:red;">\*</mark> | [LaunchType.UserInitiated](environment.md#launchtype) or [LaunchType.Background](environment.md#launchtype)                                                                             | [`LaunchType`](environment.md#launchtype)                            |
| arguments                              | optional object for the argument properties and values as defined in the extension's manifest, for example: `{ "argument1": "value1" }`                                                 | [`Arguments`](../zi-liao/lifecycle/arguments.md#arguments) or `null` |
| context                                | arbitrary object for custom data that should be passed to the command and accessible as `environment.launchContext`; the object must be JSON serializable (Dates and Buffers supported) | [`LaunchContext`](command.md#launchcontext) or `null`                |
| fallbackText                           | optional string to send as fallback text to the command                                                                                                                                 | `string` or `null`                                                   |

#### InterExtensionLaunchOptions

The options that can be used when launching a command from a different extension.

| Property                                            | Description                                                                                                                  | Type     |
| --------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- | -------- |
| extensionName<mark style="color:red;">\*</mark>     | when launching command from a different extension, the extension name (as defined in the extension's manifest) is necessary  | `string` |
| ownerOrAuthorName<mark style="color:red;">\*</mark> | when launching command from a different extension, the owner or author (as defined in the extension's manifest) is necessary | `string` |
