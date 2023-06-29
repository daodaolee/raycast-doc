# Form

我们的  `Form`  组件提供了出色的用户体验，可以从用户那里收集一些数据并提交以满足扩展需求。

![](../../.gitbook/assets/example-doppler-share-secrets.png)

## 两种类型：受控与不受控

React 中的项可以是以下两种类型之一：受控或不受控。

不受控的项是两者中较简单的一个。它最接近纯 HTML 输入。 React 将其放在页面上，Raycast 跟踪其余部分。不受控的输入需要更少的代码，但会使执行某些操作变得更加困难。

对于受控，您可以明确控制该项显示的 `value` 。您必须编写代码来通过定义  `onChange`  回调来响应更改，将当前值存储在某处，并将该值传递回要显示的项。这是你的代码中的一种回馈循环。想连接起来这些需要更多的工作，不过它们提供了够多的可控性。

您可以在每个支持的项下查看下面这两种样式。

## 校验

在提交数据之前，确保以正确的格式填写所有必需的表单控件非常重要。

在 Raycast 中，可以通过 API 完全控制验证。为了保持与本机相同的行为，正确的使用方法是验证 `onBlur` 回调中的值，更新条目的错误并使用 `onChange` 回调跟踪更新从而删除错误的值。

![](../../.gitbook/assets/form-validation.png)

{% hint style="info" %}
请记住，表单有任何错误，都不会触发 [`Action.SubmitForm`](actions.md#action.submitform) onSubmit 回调。
{% endhint %}

#### 例子

```typescript
import { Form } from "@raycast/api";
import { useState } from "react";

export default function Command() {
  const [nameError, setNameError] = useState<string | undefined>();
  const [passwordError, setPasswordError] = useState<string | undefined>();

  function dropNameErrorIfNeeded() {
    if (nameError && nameError.length > 0) {
      setNameError(undefined);
    }
  }

  function dropPasswordErrorIfNeeded() {
    if (passwordError && passwordError.length > 0) {
      setPasswordError(undefined);
    }
  }

  return (
    <Form>
      <Form.TextField
        id="nameField"
        title="Full Name"
        placeholder="Enter your name"
        error={nameError}
        onChange={dropNameErrorIfNeeded}
        onBlur={(event) => {
          if (event.target.value?.length == 0) {
            setNameError("The field should't be empty!");
          } else {
            dropNameErrorIfNeeded();
          }
        }}
      />
      <Form.PasswordField
        id="password"
        title="New Password"
        error={passwordError}
        onChange={dropPasswordErrorIfNeeded}
        onBlur={(event) => {
          const value = event.target.value;
          if (value && value.length > 0) {
            if (!validatePassword(value)) {
              setPasswordError("Password should be at least 8 characters!");
            } else {
              dropPasswordErrorIfNeeded();
            }
          } else {
            setPasswordError("The field should't be empty!");
          }
        }}
      />
      <Form.TextArea id="bioTextArea" title="Add Bio" placeholder="Describe who you are" />
      <Form.DatePicker id="birthDate" title="Date of Birth" />
    </Form>
  );
}

function validatePassword(value: string): boolean {
  return value.length >= 8;
}
```

## 草稿

草稿是一种在最终用户退出命令时保留已填写的输入（但尚未提交）的机制。要启用此机制，请在表单上设置 `enableDrafts` 属性，并使用顶级属性 [`draftValues`](../../zi-liao/lifecycle/#launchprops) 填充表单的初始值。

![](../../.gitbook/assets/form-drafts.png)

{% hint style="info" %}
* 尚不支持嵌套在导航中的表单草稿。在这种情况下，您将看到一条警告。
* 草稿不会保留  [`Form.Password`](form.md#form.passwordfield) 的值。
* 一旦  [`Action.SubmitForm`](actions.md#action.submitform) 被触发，草稿就会被丢弃。
* 如果您调用  [`popToRoot()`](../window-and-search-bar.md#poptoroot)，草稿将不会被保留或更新。
{% endhint %}

#### 例子

{% tabs %}
{% tab title="不受控 Form" %}
```typescript
import { Form, ActionPanel, Action, popToRoot, LaunchProps } from "@raycast/api";

interface TodoValues {
  title: string;
  description?: string;
  dueDate?: Date;
}

export default function Command(props: LaunchProps<{ draftValues: TodoValues }>) {
  const { draftValues } = props;

  return (
    <Form
      enableDrafts
      actions={
        <ActionPanel>
          <Action.SubmitForm
            onSubmit={(values: TodoValues) => {
              console.log("onSubmit", values);
              popToRoot();
            }}
          />
        </ActionPanel>
      }
    >
      <Form.TextField id="title" title="Title" defaultValue={draftValues?.title} />
      <Form.TextArea id="description" title="Description" defaultValue={draftValues?.description} />
      <Form.DatePicker id="dueDate" title="Due Date" defaultValue={draftValues?.dueDate} />
    </Form>
  );
}
```
{% endtab %}

{% tab title="受控 Form" %}
```typescript
import { Form, ActionPanel, Action, popToRoot, LaunchProps } from "@raycast/api";
import { useState } from "react";

interface TodoValues {
  title: string;
  description?: string;
  dueDate?: Date;
}

export default function Command(props: LaunchProps<{ draftValues: TodoValues }>) {
  const { draftValues } = props;

  const [title, setTitle] = useState<string>(draftValues?.title || "");
  const [description, setDescription] = useState<string>(draftValues?.description || "");
  const [dueDate, setDueDate] = useState<Date | null>(draftValues?.dueDate || null);

  return (
    <Form
      enableDrafts
      actions={
        <ActionPanel>
          <Action.SubmitForm
            onSubmit={(values: TodoValues) => {
              console.log("onSubmit", values);
              popToRoot();
            }}
          />
        </ActionPanel>
      }
    >
      <Form.TextField id="title" title="Title" value={title} onChange={setTitle} />
      <Form.TextArea id="description" title="Description" value={description} onChange={setDescription} />
      <Form.DatePicker id="dueDate" title="Due Date" value={dueDate} onChange={setDueDate} />
    </Form>
  );
}
```
{% endtab %}
{% endtabs %}

## API 参数

### Form

显示表单项的列表，例如 [Form.TextField](form.md#form.textfield), [Form.Checkbox](form.md#form.checkbox) 或 [Form.Dropdown](form.md#form.dropdown).

#### 参数

| 名称              | 描述                                                | 类型                | 默认            |
| --------------- | ------------------------------------------------- | ----------------- | ------------- |
| actions         | 对 [ActionPanel](action-panel.md#actionpanel) 的引用。 | `React.ReactNode` | -             |
| children        | 表单的 Form.Item 元素。                                 | `React.ReactNode` | -             |
| enableDrafts    | 当用户退出屏幕时是否保留 Form.Items 值。                        | `boolean`         | `false`       |
| isLoading       | 是否应在搜索栏下方显示或隐藏 loading 栏                          | `boolean`         | `false`       |
| navigationTitle | Raycast 中显示的该视图的主标题                               | `string`          | Command title |

### Form.TextField

带有用于输入的文本字段的表单项。

![](../../.gitbook/assets/form-textfield.png)

#### 例子

{% tabs %}
{% tab title="不受控 text field" %}
```typescript
import { ActionPanel, Form, Action } from "@raycast/api";

export default function Command() {
  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit Name" onSubmit={(values) => console.log(values)} />
        </ActionPanel>
      }
    >
      <Form.TextField id="name" defaultValue="Steve" />
    </Form>
  );
}
```
{% endtab %}

{% tab title="受控 text field" %}
```typescript
import { ActionPanel, Form, Action } from "@raycast/api";
import { useState } from "react";

export default function Command() {
  const [name, setName] = useState<string>("");

  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit Name" onSubmit={(values) => console.log(values)} />
        </ActionPanel>
      }
    >
      <Form.TextField id="name" value={name} onChange={setName} />
    </Form>
  );
}
```
{% endtab %}
{% endtabs %}

#### 参数

| 名称                                   | 描述                                                                              | 类型                                   | 默认 |
| ------------------------------------ | ------------------------------------------------------------------------------- | ------------------------------------ | -- |
| id<mark style="color:red;">\*</mark> | 表单项的 ID。确保为每个表单项分配一个唯一的 ID。                                                     | `string`                             | -  |
| autoFocus                            | 表单渲染后项是否应自动获得焦点。                                                                | `boolean`                            | -  |
| defaultValue                         | 项的默认值。请记住，`defaultValue` 将在每个组件生命周期配置一次。这意味着如果用户更改该值，则重新渲染时不会配置 `defaultValue`。 | `string`                             | -  |
| error                                | 显示表单项验证问题的可选错误消息。如果存在 `error` ，表单项将以红色边框突出显示，并在右侧显示错误消息。                        | `string`                             | -  |
| info                                 | 用于描述表单项的可选信息消息。它显示在条目的右侧，并带有一个信息图标。当图标悬停时，将显示信息消息。                              | `string`                             | -  |
| placeholder                          | 文本字段中显示的占位符文本。                                                                  | `string`                             | -  |
| storeValue                           | 提交后是否应保留项的值，并在下次渲染表单时恢复。                                                        | `boolean`                            | -  |
| title                                | 显示在项的左侧标题。                                                                      | `string`                             | -  |
| value                                | 当前项的值                                                                           | `string`                             | -  |
| onBlur                               | 当项失去焦点时将触发的回调。                                                                  | `(event: FormEvent<string>) => void` | -  |
| onChange                             | 当项的 `value` 发生变化时会触发的回调。                                                        | `(newValue: string) => void`         | -  |
| onFocus                              | 当项获得焦点时，应该调用将触发的回调。                                                             | `(event: FormEvent<string>) => void` | -  |

#### 方法（命令式 API）

| 名称    | 签名           | 描述                                  |
| ----- | ------------ | ----------------------------------- |
| focus | `() => void` | 聚焦某项                                |
| reset | `() => void` | 将表单项重置为其初始值，或 `defaultValue`（如果指定）。 |

### Form.PasswordField

具有用于输入密码的安全文本字段的表单项，其中输入的字符必须保密。

![](../../.gitbook/assets/form-password.png)

#### 例子

{% tabs %}
{% tab title="不受控 password field" %}
```typescript
import { ActionPanel, Form, Action } from "@raycast/api";

export default function Command() {
  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit Password" onSubmit={(values) => console.log(values)} />
        </ActionPanel>
      }
    >
      <Form.PasswordField id="password" title="Enter Password" />
    </Form>
  );
}
```
{% endtab %}

{% tab title="受控 password field" %}
```typescript
import { ActionPanel, Form, Action } from "@raycast/api";
import { useState } from "react";

export default function Command() {
  const [password, setPassword] = useState<string>("");

  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit Password" onSubmit={(values) => console.log(values)} />
        </ActionPanel>
      }
    >
      <Form.PasswordField id="password" value={password} onChange={setPassword} />
    </Form>
  );
}
```
{% endtab %}
{% endtabs %}

#### 参数

| 名称                                   | 描述                                                                               | 类型                                   | 默认 |
| ------------------------------------ | -------------------------------------------------------------------------------- | ------------------------------------ | -- |
| id<mark style="color:red;">\*</mark> | 表单项的 ID。确保为每个表单项分配一个唯一的 ID。                                                      | `string`                             | -  |
| autoFocus                            | 表单渲染后项是否应自动获得焦点。                                                                 | `boolean`                            | -  |
| defaultValue                         | 条目的默认值。请记住，`defaultValue` 将在每个组件生命周期配置一次。这意味着如果用户更改该值，则重新渲染时不会配置 `defaultValue`。 | `string`                             | -  |
| error                                | 显示表单项验证问题的可选错误消息。如果存在错误，表单项将以红色边框突出显示，并在右侧显示错误消息。                                | `string`                             | -  |
| info                                 | 用于描述表单项的可选信息消息。它显示在项的右侧，并带有一个信息图标。当图标悬停时，将显示信息消息。                                | `string`                             | -  |
| placeholder                          | 密码字段中显示的占位符文本。                                                                   | `string`                             | -  |
| storeValue                           | 提交后是否应保留项的值，并在下次渲染表单时恢复。                                                         | `boolean`                            | -  |
| title                                | 显示在项的左侧标题。                                                                       | `string`                             | -  |
| value                                | 当前项的值                                                                            | `string`                             | -  |
| onBlur                               | 当项失去焦点时将触发的回调。                                                                   | `(event: FormEvent<string>) => void` | -  |
| onChange                             | 当项的值发生变化时会触发的回调。                                                                 | `(newValue: string) => void`         | -  |
| onFocus                              | 当项获得焦点时，应该调用将触发的回调。                                                              | `(event: FormEvent<string>) => void` | -  |

#### 方法（命令式 API）

| 名称    | 签名           | 描述                                  |
| ----- | ------------ | ----------------------------------- |
| focus | `() => void` | 聚焦该项。                               |
| reset | `() => void` | 将表单项重置为其初始值，或 `defaultValue`（如果指定）。 |

### Form.TextArea

带有用于输入的文本区域的表单项。该项支持多行文本输入。

![](../../.gitbook/assets/form-textarea.png)

#### 例子

{% tabs %}
{% tab title="不受控 text area" %}
```typescript
import { ActionPanel, Form, Action } from "@raycast/api";

const DESCRIPTION =
  "We spend too much time staring at loading indicators. The Raycast team is dedicated to make everybody interact faster with their computers.";

export default function Command() {
  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit Description" onSubmit={(values) => console.log(values)} />
        </ActionPanel>
      }
    >
      <Form.TextArea id="description" defaultValue={DESCRIPTION} />
    </Form>
  );
}
```
{% endtab %}

{% tab title="受控 text area" %}
```typescript
import { ActionPanel, Form, Action } from "@raycast/api";
import { useState } from "react";

export default function Command() {
  const [description, setDescription] = useState<string>("");

  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit Description" onSubmit={(values) => console.log(values)} />
        </ActionPanel>
      }
    >
      <Form.TextArea id="description" value={description} onChange={setDescription} />
    </Form>
  );
}
```
{% endtab %}
{% endtabs %}

#### 参数

| 名称                                   | 描述                                                                                                              | 类型                                   | 默认      |
| ------------------------------------ | --------------------------------------------------------------------------------------------------------------- | ------------------------------------ | ------- |
| id<mark style="color:red;">\*</mark> | 表单项的 ID。确保为每个表单项分配一个唯一的 ID。                                                                                     | `string`                             | -       |
| autoFocus                            | 表单渲染后项是否应自动获得焦点。                                                                                                | `boolean`                            | -       |
| defaultValue                         | 条目的默认值。请记住，`defaultValue` 将在每个组件生命周期配置一次。这意味着如果用户更改该值，则重新渲染时不会配置 `defaultValue`。                                | `string`                             | -       |
| enableMarkdown                       | Markdown 是否会在 TextArea 中突出显示。启用后，Markdown 快捷方式开始对 TextArea 起作用（按 `⌘ + B` 将在所选文本周围添加 **粗体**，`⌘ + I` 将使所选文本变为斜体等） | `boolean`                            | `false` |
| error                                | 显示表单项验证问题的可选错误消息。如果存在错误，表单项将以红色边框突出显示，并在右侧显示错误消息。                                                               | `string`                             | -       |
| info                                 | 用于描述表单项的可选信息消息。它显示在项的右侧，并带有一个信息图标。当图标悬停时，将显示信息消息。                                                               | `string`                             | -       |
| placeholder                          | 文本区域中显示的占位符文本。                                                                                                  | `string`                             | -       |
| storeValue                           | 提交后是否应保留项的值，并在下次渲染表单时恢复。                                                                                        | `boolean`                            | -       |
| title                                | 显示在项的左侧标题。                                                                                                      | `string`                             | -       |
| value                                | 当前项的值                                                                                                           | `string`                             | -       |
| onBlur                               | 当项失去焦点时将触发的回调。                                                                                                  | `(event: FormEvent<string>) => void` | -       |
| onChange                             | 当项的 `value` 发生变化时会触发的回调。                                                                                        | `(newValue: string) => void`         | -       |
| onFocus                              | 当项获得焦点时，应该调用将触发的回调。                                                                                             | `(event: FormEvent<string>) => void` | -       |

#### 方法（命令式 API）

| 名称    | 签名           | 描述                                  |
| ----- | ------------ | ----------------------------------- |
| focus | `() => void` | 聚焦某项                                |
| reset | `() => void` | 将表单项重置为其初始值，或 `defaultValue`（如果指定）。 |

### Form.Checkbox

带有复选框的表单项。

![](../../.gitbook/assets/form-checkbox.png)

#### 例子

{% tabs %}
{% tab title="不受控 checkbox" %}
```typescript
import { ActionPanel, Form, Action } from "@raycast/api";

export default function Command() {
  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit Answer" onSubmit={(values) => console.log(values)} />
        </ActionPanel>
      }
    >
      <Form.Checkbox id="answer" label="Are you happy?" defaultValue={true} />
    </Form>
  );
}
```
{% endtab %}

{% tab title="受控 checkbox" %}
```typescript
import { ActionPanel, Form, Action } from "@raycast/api";
import { useState } from "react";

export default function Command() {
  const [checked, setChecked] = useState(true);

  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit Answer" onSubmit={(values) => console.log(values)} />
        </ActionPanel>
      }
    >
      <Form.Checkbox id="answer" label="Do you like orange juice?" value={checked} onChange={setChecked} />
    </Form>
  );
}
```
{% endtab %}
{% endtabs %}

#### 参数

| 名称                                      | 描述                                                                               | 类型                                    | 默认 |
| --------------------------------------- | -------------------------------------------------------------------------------- | ------------------------------------- | -- |
| id<mark style="color:red;">\*</mark>    | 表单项的 ID。确保为每个表单项分配一个唯一的 ID。                                                      | `string`                              | -  |
| label<mark style="color:red;">\*</mark> | 显示在复选框右侧的标签。                                                                     | `string`                              | -  |
| autoFocus                               | 表单渲染后项是否应自动获得焦点。                                                                 | `boolean`                             | -  |
| defaultValue                            | 条目的默认值。请记住，`defaultValue` 将在每个组件生命周期配置一次。这意味着如果用户更改该值，则重新渲染时不会配置 `defaultValue`。 | `boolean`                             | -  |
| error                                   | 显示表单项验证问题的可选错误消息。如果存在错误，表单项将以红色边框突出显示，并在右侧显示错误消息。                                | `string`                              | -  |
| info                                    | 用于描述表单项的可选信息消息。它显示在项的右侧，并带有一个信息图标。当图标悬停时，将显示信息消息。                                | `string`                              | -  |
| storeValue                              | 提交后是否应保留项的值，并在下次渲染表单时恢复。                                                         | `boolean`                             | -  |
| title                                   | 显示在项的左侧标题。                                                                       | `string`                              | -  |
| value                                   | 当前项的值                                                                            | `boolean`                             | -  |
| onBlur                                  | 当项失去焦点时将触发的回调。                                                                   | `(event: FormEvent<boolean>) => void` | -  |
| onChange                                | 当项的 `value` 发生变化时会触发的回调。                                                         | `(newValue: boolean) => void`         | -  |
| onFocus                                 | 当项获得焦点时，应该调用将触发的回调。                                                              | `(event: FormEvent<boolean>) => void` | -  |

#### 方法（命令式 API）

| 名称    | 签名           | 描述                                  |
| ----- | ------------ | ----------------------------------- |
| focus | `() => void` | 聚焦某项                                |
| reset | `() => void` | 将表单项重置为其初始值，或 `defaultValue`（如果指定）。 |

### Form.DatePicker

带有日期选择器的表单项。

![](../../.gitbook/assets/form-datepicker.png)

#### 例子

{% tabs %}
{% tab title="不受控 date picker" %}
```typescript
import { ActionPanel, Form, Action } from "@raycast/api";

export default function Command() {
  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit Form" onSubmit={(values) => console.log(values)} />
        </ActionPanel>
      }
    >
      <Form.DatePicker id="dateOfBirth" title="Date of Birth" defaultValue={new Date(1955, 1, 24)} />
    </Form>
  );
}
```
{% endtab %}

{% tab title="受控 date picker" %}
```typescript
import { ActionPanel, Form, Action } from "@raycast/api";
import { useState } from "react";

export default function Command() {
  const [date, setDate] = useState<Date | null>(null);

  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit Form" onSubmit={(values) => console.log(values)} />
        </ActionPanel>
      }
    >
      <Form.DatePicker id="launchDate" title="Launch Date" value={date} onChange={setDate} />
    </Form>
  );
}
```
{% endtab %}
{% endtabs %}

#### Props

| Prop                                 | Description                                                                      | Type                                                   | Default |
| ------------------------------------ | -------------------------------------------------------------------------------- | ------------------------------------------------------ | ------- |
| id<mark style="color:red;">\*</mark> | 表单项的 ID。确保为每个表单项分配一个唯一的 ID。                                                      | `string`                                               | -       |
| autoFocus                            | 表单渲染后项是否应自动获得焦点。                                                                 | `boolean`                                              | -       |
| defaultValue                         | 条目的默认值。请记住，`defaultValue` 将在每个组件生命周期配置一次。这意味着如果用户更改该值，则重新渲染时不会配置 `defaultValue`。 | `Date`                                                 | -       |
| error                                | 显示表单项验证问题的可选错误消息。如果存在错误，表单项将以红色边框突出显示，并在右侧显示错误消息。                                | `string`                                               | -       |
| info                                 | 用于描述表单项的可选信息消息。它显示在项的右侧，并带有一个信息图标。当图标悬停时，将显示信息消息。                                | `string`                                               | -       |
| max                                  | 允许选择的最长日期（含）。                                                                    | `Date`                                                 | -       |
| min                                  | 允许选择的最短日期（含）。                                                                    | `Date`                                                 | -       |
| storeValue                           | 提交后是否应保留项的值，并在下次渲染表单时恢复。                                                         | `boolean`                                              | -       |
| title                                | 显示在项的左侧标题。                                                                       | `string`                                               | -       |
| type                                 | 可以选择哪些类型的日期组件                                                                    | [`Form.DatePicker.Type`](form.md#form.datepicker.type) | -       |
| value                                | 当前项的值                                                                            | `Date`                                                 | -       |
| onBlur                               | 当项失去焦点时将触发的回调。                                                                   | `(event: FormEvent<Date \| null>) => void`             | -       |
| onChange                             | 当项的 `value` 发生变化时会触发的回调。                                                         | `(newValue: Date \| null) => void`                     | -       |
| onFocus                              | 当项获得焦点时，应该调用将触发的回调。                                                              | `(event: FormEvent<Date \| null>) => void`             | -       |

#### 方法（命令式 API）

| 名称    | 签名           | 描述                                  |
| ----- | ------------ | ----------------------------------- |
| focus | `() => void` | 聚焦某项                                |
| reset | `() => void` | 将表单项重置为其初始值，或 `defaultValue`（如果指定）。 |

### Form.Dropdown

带有下拉菜单的表单项。

![](../../.gitbook/assets/form-dropdown.png)

#### 例子

{% tabs %}
{% tab title="不受控 dropdown" %}
```typescript
import { ActionPanel, Form, Action } from "@raycast/api";

export default function Command() {
  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit Favorite" onSubmit={(values) => console.log(values)} />
        </ActionPanel>
      }
    >
      <Form.Dropdown id="emoji" title="Favorite Emoji" defaultValue="lol">
        <Form.Dropdown.Item value="poop" title="Pile of poop" icon="💩" />
        <Form.Dropdown.Item value="rocket" title="Rocket" icon="🚀" />
        <Form.Dropdown.Item value="lol" title="Rolling on the floor laughing face" icon="🤣" />
      </Form.Dropdown>
    </Form>
  );
}
```
{% endtab %}

{% tab title="受控 dropdown" %}
```typescript
import { ActionPanel, Form, Action } from "@raycast/api";
import { useState } from "react";

export default function Command() {
  const [programmingLanguage, setProgrammingLanguage] = useState<string>("typescript");

  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit Favorite" onSubmit={(values) => console.log(values)} />
        </ActionPanel>
      }
    >
      <Form.Dropdown
        id="dropdown"
        title="Favorite Language"
        value={programmingLanguage}
        onChange={setProgrammingLanguage}
      >
        <Form.Dropdown.Item value="cpp" title="C++" />
        <Form.Dropdown.Item value="javascript" title="JavaScript" />
        <Form.Dropdown.Item value="ruby" title="Ruby" />
        <Form.Dropdown.Item value="python" title="Python" />
        <Form.Dropdown.Item value="swift" title="Swift" />
        <Form.Dropdown.Item value="typescript" title="TypeScript" />
      </Form.Dropdown>
    </Form>
  );
}
```
{% endtab %}
{% endtabs %}

#### 参数

<table><thead><tr><th width="154">名称</th><th>描述</th><th>类型</th><th>默认</th></tr></thead><tbody><tr><td>id<mark style="color:red;">*</mark></td><td>表单项的 ID。确保为每个表单项分配一个唯一的 ID。</td><td><code>string</code></td><td>-</td></tr><tr><td>autoFocus</td><td>表单渲染后项是否应自动获得焦点。</td><td><code>boolean</code></td><td>-</td></tr><tr><td>children</td><td>内容为 Sections 或是item。如果指定了 <a href="form.md#form.dropdown.item">Form.Dropdown.Item</a> 元素，则会自动创建默认部分。</td><td><code>React.ReactNode</code></td><td>-</td></tr><tr><td>defaultValue</td><td>条目的默认值。请记住，<code>defaultValue</code> 将在每个组件生命周期配置一次。这意味着如果用户更改该值，则重新渲染时不会配置 <code>defaultValue</code>。</td><td><code>string</code></td><td>-</td></tr><tr><td>error</td><td>显示表单项验证问题的可选错误消息。如果存在错误，表单项将以红色边框突出显示，并在右侧显示错误消息。</td><td><code>string</code></td><td>-</td></tr><tr><td>filtering</td><td>切换 Raycast filtering。如果为 <code>true</code>，Raycast 将使用搜索栏中的查询来过滤项目。当为 <code>false</code> 时，扩展程序需要负责过滤。</td><td><code>boolean</code> 或 <code>{ keepSectionOrder: boolean }</code></td><td>当指定 <code>onSearchTextChange</code> 时为 <code>false</code>，否则为 <code>true</code>。</td></tr><tr><td>info</td><td>用于描述表单项的可选信息消息。它显示在项的右侧，并带有一个信息图标。当图标悬停时，将显示信息消息。</td><td><code>string</code></td><td>-</td></tr><tr><td>isLoading</td><td>是否应在搜索栏旁边显示或隐藏 loading 指示器</td><td><code>boolean</code></td><td><code>false</code></td></tr><tr><td>placeholder</td><td>显示在下拉搜索字段中的占位符文本。</td><td><code>string</code></td><td><code>"Search…"</code></td></tr><tr><td>storeValue</td><td>提交后是否应保留项的值，并在下次渲染表单时恢复。</td><td><code>boolean</code></td><td>-</td></tr><tr><td>throttle</td><td><code>onSearchTextChange</code> 处理程序是在每次按下键盘时触发还是延迟以限制事件。当将自定义过滤逻辑与异步操作（例如网络请求）结合使用时，建议设置为 <code>true</code>。</td><td><code>boolean</code></td><td><code>false</code></td></tr><tr><td>title</td><td>显示在项的左侧标题。</td><td><code>string</code></td><td>-</td></tr><tr><td>value</td><td>当前项的值</td><td><code>string</code></td><td>-</td></tr><tr><td>onBlur</td><td>当项失去焦点时将触发的回调。</td><td><code>(event: FormEvent&#x3C;string>) => void</code></td><td>-</td></tr><tr><td>onChange</td><td>当项的 <code>value</code> 发生变化时会触发的回调。</td><td><code>(newValue: string) => void</code></td><td>-</td></tr><tr><td>onFocus</td><td>当项获得焦点时调用将触发的回调。</td><td><code>(event: FormEvent&#x3C;string>) => void</code></td><td>-</td></tr><tr><td>onSearchTextChange</td><td>当项获得焦点时，应该调用将触发的回调。</td><td><code>(text: string) => void</code></td><td>-</td></tr></tbody></table>

#### 方法（命令式 API）

| 名称    | 签名           | 描述                                  |
| ----- | ------------ | ----------------------------------- |
| focus | `() => void` | 聚焦某项                                |
| reset | `() => void` | 将表单项重置为其初始值，或 `defaultValue`（如果指定）。 |

### Form.Dropdown.Item

[Form.Dropdown](form.md#form.dropdown) 的项

#### 例子

```typescript
import { Action, ActionPanel, Form, Icon } from "@raycast/api";

export default function Command() {
  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit Icon" onSubmit={(values) => console.log(values)} />
        </ActionPanel>
      }
    >
      <Form.Dropdown id="icon" title="Icon">
        <Form.Dropdown.Item value="circle" title="Cirlce" icon={Icon.Circle} />
      </Form.Dropdown>
    </Form>
  );
}
```

#### 参数

| Prop                                    | Description                                              | Type                                                     | Default                         |
| --------------------------------------- | -------------------------------------------------------- | -------------------------------------------------------- | ------------------------------- |
| title<mark style="color:red;">\*</mark> | 显示的标题。                                                   | `string`                                                 | -                               |
| value<mark style="color:red;">\*</mark> | 下拉项的值。确保为每个项目分配每个唯一值。                                    | `string`                                                 | -                               |
| icon                                    | 该项显示的可选图标。                                               | [`Image.ImageLike`](icons-and-images.md#image.imagelike) | -                               |
| keywords                                | 一个可选属性，用于提供额外的可索引字符串以供搜索。在 Raycast 中过滤项目时，除了标题之外还会搜索关键字。 | `string[]`                                               | The title of its section if any |

### Form.Dropdown.Section

视觉上分离的下拉项组。&#x20;

使用 sections 将相关的菜单项分组在一起。

#### 例子

```typescript
import { ActionPanel, Form, Action } from "@raycast/api";

export default function Command() {
  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit Favorite" onSubmit={(values) => console.log(values)} />
        </ActionPanel>
      }
    >
      <Form.Dropdown id="food" title="Favorite Food">
        <Form.Dropdown.Section title="Fruits">
          <Form.Dropdown.Item value="apple" title="Apple" icon="🍎" />
          <Form.Dropdown.Item value="banana" title="Banana" icon="🍌" />
        </Form.Dropdown.Section>
        <Form.Dropdown.Section title="Vegetables">
          <Form.Dropdown.Item value="broccoli" title="Broccoli" icon="🥦" />
          <Form.Dropdown.Item value="carrot" title="Carrot" icon="🥕" />
        </Form.Dropdown.Section>
      </Form.Dropdown>
    </Form>
  );
}
```

#### 参数

<table><thead><tr><th width="174">名称</th><th>描述</th><th>类型</th><th>默认</th></tr></thead><tbody><tr><td>children</td><td>子项</td><td><code>React.ReactNode</code></td><td>-</td></tr><tr><td>title</td><td>显示在该部分上方的标题</td><td><code>string</code></td><td>-</td></tr></tbody></table>

### Form.TagPicker

带有标签选择器的表单项，允许用户选择多个项目。

![](../../.gitbook/assets/form-tagpicker.png)

#### 例子

{% tabs %}
{% tab title="不受控 tag picker" %}
```typescript
import { ActionPanel, Form, Action } from "@raycast/api";

export default function Command() {
  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit Favorite" onSubmit={(values) => console.log(values)} />
        </ActionPanel>
      }
    >
      <Form.TagPicker id="sports" title="Favorite Sports" defaultValue={["football"]}>
        <Form.TagPicker.Item value="basketball" title="Basketball" icon="🏀" />
        <Form.TagPicker.Item value="football" title="Football" icon="⚽️" />
        <Form.TagPicker.Item value="tennis" title="Tennis" icon="🎾" />
      </Form.TagPicker>
    </Form>
  );
}
```
{% endtab %}

{% tab title="受控 tag picker" %}
```typescript
import { ActionPanel, Form, Action } from "@raycast/api";
import { useState } from "react";

export default function Command() {
  const [countries, setCountries] = useState<string[]>(["ger", "ned", "pol"]);

  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit Countries" onSubmit={(values) => console.log(values)} />
        </ActionPanel>
      }
    >
      <Form.TagPicker id="countries" title="Visited Countries" value={countries} onChange={setCountries}>
        <Form.TagPicker.Item value="ger" title="Germany" icon="🇩🇪" />
        <Form.TagPicker.Item value="ind" title="India" icon="🇮🇳" />
        <Form.TagPicker.Item value="ned" title="Netherlands" icon="🇳🇱" />
        <Form.TagPicker.Item value="nor" title="Norway" icon="🇳🇴" />
        <Form.TagPicker.Item value="pol" title="Poland" icon="🇵🇱" />
        <Form.TagPicker.Item value="rus" title="Russia" icon="🇷🇺" />
        <Form.TagPicker.Item value="sco" title="Scotland" icon="🏴󠁧󠁢󠁳󠁣󠁴󠁿" />
      </Form.TagPicker>
    </Form>
  );
}
```
{% endtab %}
{% endtabs %}

#### 参数

| 名称                                   | 描述                                                                               | 类型                                     | 默认 |
| ------------------------------------ | -------------------------------------------------------------------------------- | -------------------------------------- | -- |
| id<mark style="color:red;">\*</mark> | 表单项的 ID。确保为每个表单项分配一个唯一的 ID。                                                      | `string`                               | -  |
| autoFocus                            | 表单渲染后项是否应自动获得焦点。                                                                 | `boolean`                              | -  |
| children                             | tag 的子项.                                                                         | `React.ReactNode`                      | -  |
| defaultValue                         | 条目的默认值。请记住，`defaultValue` 将在每个组件生命周期配置一次。这意味着如果用户更改该值，则重新渲染时不会配置 `defaultValue`。 | `string[]`                             | -  |
| error                                | 显示表单项验证问题的可选错误消息。如果存在错误，表单项将以红色边框突出显示，并在右侧显示错误消息。                                | `string`                               | -  |
| info                                 | 用于描述表单项的可选信息消息。它显示在项的右侧，并带有一个信息图标。当图标悬停时，将显示信息消息。                                | `string`                               | -  |
| placeholder                          | token 字段中显示的占位符文本。                                                               | `string`                               | -  |
| storeValue                           | 提交后是否应保留项的值，并在下次渲染表单时恢复。                                                         | `boolean`                              | -  |
| title                                | 显示在项的左侧标题。                                                                       | `string`                               | -  |
| value                                | 当前项的值                                                                            | `string[]`                             | -  |
| onBlur                               | 当项失去焦点时将触发的回调。                                                                   | `(event: FormEvent<string[]>) => void` | -  |
| onChange                             | 当项的 `value` 发生变化时会触发的回调。                                                         | `(newValue: string[]) => void`         | -  |
| onFocus                              | 当项获得焦点时，应该调用将触发的回调。                                                              | `(event: FormEvent<string[]>) => void` | -  |

#### 方法（命令式 API）

| 名称    | 签名           | 描述                                  |
| ----- | ------------ | ----------------------------------- |
| focus | `() => void` | 聚焦某项                                |
| reset | `() => void` | 将表单项重置为其初始值，或 `defaultValue`（如果指定）。 |

### Form.TagPicker.Item

[Form.TagPicker](form.md#form.tagpicker) 的子项

#### 例子

```typescript
import { ActionPanel, Color, Form, Icon, Action } from "@raycast/api";

export default function Command() {
  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit Color" onSubmit={(values) => console.log(values)} />
        </ActionPanel>
      }
    >
      <Form.TagPicker id="color" title="Color">
        <Form.TagPicker.Item value="red" title="Red" icon={{ source: Icon.Circle, tintColor: Color.Red }} />
        <Form.TagPicker.Item value="green" title="Green" icon={{ source: Icon.Circle, tintColor: Color.Green }} />
        <Form.TagPicker.Item value="blue" title="Blue" icon={{ source: Icon.Circle, tintColor: Color.Blue }} />
      </Form.TagPicker>
    </Form>
  );
}
```

#### 参数

| 名称                                      | 描述                 | 类型                                                       | 默认 |
| --------------------------------------- | ------------------ | -------------------------------------------------------- | -- |
| title<mark style="color:red;">\*</mark> | 标签的名称.             | `string`                                                 | -  |
| value<mark style="color:red;">\*</mark> | 标签的值。确保为每个项分配唯一的值。 | `string`                                                 | -  |
| icon                                    | 标签中显示的图标。          | [`Image.ImageLike`](icons-and-images.md#image.imagelike) | -  |

### Form.Separator

显示分隔线的表单项。用于对表单项进行分组并在视觉上分隔表单项。

![](../../.gitbook/assets/form-separator.png)

#### 例子

```typescript
import { ActionPanel, Form, Action } from "@raycast/api";

export default function Command() {
  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit Form" onSubmit={(values) => console.log(values)} />
        </ActionPanel>
      }
    >
      <Form.TextField id="textfield" />
      <Form.Separator />
      <Form.TextArea id="textarea" />
    </Form>
  );
}
```

### Form.FilePicker

带有按钮的表单项，用于打开对话框以选择某些文件 和/或 某些目录（取决于其属性）。

{% hint style="info" %}
当用户选择了一些已存在的项目时，当调用 `onSubmit` 回调时，它们可能会被删除或更改。因此，在对其进行操作之前，您应该始终确保这些项目存在！
{% endhint %}

![](../../.gitbook/assets/form-filepicker-multiple.png)

![Single Selection](../../.gitbook/assets/form-filepicker-single.png)

#### 例子

{% tabs %}
{% tab title="不受控 file picker" %}
```typescript
import { ActionPanel, Form, Action } from "@raycast/api";
import fs from "fs";

export default function Command() {
  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm
            title="Submit Name"
            onSubmit={(values: { files: string[] }) => {
              const files = values.files.filter((file: any) => fs.existsSync(file) && fs.lstatSync(file).isFile());
              console.log(files);
            }}
          />
        </ActionPanel>
      }
    >
      <Form.FilePicker id="files" />
    </Form>
  );
}
```
{% endtab %}

{% tab title="单选 selection file picker" %}
```typescript
import { ActionPanel, Form, Action } from "@raycast/api";
import fs from "fs";

export default function Command() {
  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm
            title="Submit Name"
            onSubmit={(values: { files: string[] }) => {
              const file = values.files[0];
              if (!fs.existsSync(file) || !fs.lstatSync(file).isFile()) {
                return false;
              }
              console.log(file);
            }}
          />
        </ActionPanel>
      }
    >
      <Form.FilePicker id="files" allowMultipleSelection={false} />
    </Form>
  );
}
```
{% endtab %}

{% tab title="目录 picker" %}
```typescript
import { ActionPanel, Form, Action } from "@raycast/api";
import fs from "fs";

export default function Command() {
  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm
            title="Submit Name"
            onSubmit={(values: { folders: string[] }) => {
              const folder = values.folders[0];
              if (!fs.existsSync(folder) || fs.lstatSync(folder).isDirectory()) {
                return false;
              }
              console.log(folder);
            }}
          />
        </ActionPanel>
      }
    >
      <Form.FilePicker id="folders" allowMultipleSelection={false} canChooseDirectories canChooseFiles={false} />
    </Form>
  );
}
```
{% endtab %}

{% tab title="受控 file picker" %}
```typescript
import { ActionPanel, Form, Action } from "@raycast/api";
import { useState } from "react";

export default function Command() {
  const [files, setFiles] = useState<string[]>([]);

  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit Name" onSubmit={(values) => console.log(values)} />
        </ActionPanel>
      }
    >
      <Form.FilePicker id="files" value={files} onChange={setFiles} />
    </Form>
  );
}
```
{% endtab %}
{% endtabs %}

#### 参数

| 名称                                   | 描述                                                                               | 类型                                     | 默认      |
| ------------------------------------ | -------------------------------------------------------------------------------- | -------------------------------------- | ------- |
| id<mark style="color:red;">\*</mark> | 表单项的 ID。确保为每个表单项分配一个唯一的 ID。                                                      | `string`                               | -       |
| allowMultipleSelection               | 指示用户是否可以选择多个项或仅选择一项。                                                             | `boolean`                              | `true`  |
| autoFocus                            | 表单渲染后项是否应自动获得焦点。                                                                 | `boolean`                              | -       |
| canChooseDirectories                 | 是否可以选择目录。                                                                        | `boolean`                              | `false` |
| canChooseFiles                       | 是否可以选择文件。                                                                        | `boolean`                              | `true`  |
| defaultValue                         | 条目的默认值。请记住，`defaultValue` 将在每个组件生命周期配置一次。这意味着如果用户更改该值，则重新渲染时不会配置 `defaultValue`。 | `string[]`                             | -       |
| error                                | 显示表单项验证问题的可选错误消息。如果存在错误，表单项将以红色边框突出显示，并在右侧显示错误消息。                                | `string`                               | -       |
| info                                 | 用于描述表单项的可选信息消息。它显示在项的右侧，并带有一个信息图标。当图标悬停时，将显示信息消息。                                | `string`                               | -       |
| showHiddenFiles                      | 文件选择器是否显示通常对用户隐藏的文件。                                                             | `boolean`                              | `false` |
| storeValue                           | 提交后是否应保留项的值，并在下次渲染表单时恢复。                                                         | `boolean`                              | -       |
| title                                | 显示在项的左侧标题。                                                                       | `string`                               | -       |
| value                                | 当前项的值                                                                            | `string[]`                             | -       |
| onBlur                               | 当项失去焦点时将触发的回调。                                                                   | `(event: FormEvent<string[]>) => void` | -       |
| onChange                             | 当项的 `value` 发生变化时会触发的回调。                                                         | `(newValue: string[]) => void`         | -       |
| onFocus                              | 当项获得焦点时，应该调用将触发的回调。                                                              | `(event: FormEvent<string[]>) => void` | -       |

#### 方法（命令式 API）

| 名称    | 签名           | 描述                                  |
| ----- | ------------ | ----------------------------------- |
| focus | `() => void` | 聚焦某项                                |
| reset | `() => void` | 将表单项重置为其初始值，或 `defaultValue`（如果指定）。 |

### Form.Description

带有简单文本标签的表单项。&#x20;

不要使用此组件来显示其他表单字段的验证消息。

![](../../.gitbook/assets/form-description.png)

#### 例子

```typescript
import { ActionPanel, Form, Action } from "@raycast/api";

export default function Command() {
  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit" onSubmit={(values) => console.log(values)} />
        </ActionPanel>
      }
    >
      <Form.Description
        title="Import / Export"
        text="Exporting will back-up your preferences, quicklinks, snippets, floating notes, script-command folder paths, aliases, hotkeys, favorites and other data."
      />
    </Form>
  );
}
```

#### 参数

| 名称                                     | 描述          | 类型       | 默认 |
| -------------------------------------- | ----------- | -------- | -- |
| text<mark style="color:red;">\*</mark> | 显示在中间的文本。   | `string` | -  |
| title                                  | 描述项左侧的显示标题。 | `string` | -  |

## 类型

#### Form.Event

某些 Form.Item 回调（例如 `onFocus` 和 `onBlur`）可以返回可以以不同方式使用的 `Form.Event` 对象。

| 属性                                       | 描述              | 类型                                           |
| ---------------------------------------- | --------------- | -------------------------------------------- |
| target<mark style="color:red;">\*</mark> | 包含与事件相关的目标数据的接口 | `{ id: string; value: any }`                 |
| type<mark style="color:red;">\*</mark>   | 事件类型            | [`Form.Event.Type`](form.md#form.event.type) |

#### 例子

```typescript
import { Form } from "@raycast/api";

export default function Main() {
  return (
    <Form>
      <Form.TextField id="textField" title="Text Field" onBlur={logEvent} onFocus={logEvent} />
      <Form.TextArea id="textArea" title="Text Area" onBlur={logEvent} onFocus={logEvent} />
      <Form.Dropdown id="dropdown" title="Dropdown" onBlur={logEvent} onFocus={logEvent}>
        {[1, 2, 3, 4, 5, 6, 7].map((num) => (
          <Form.Dropdown.Item value={String(num)} title={String(num)} key={num} />
        ))}
      </Form.Dropdown>
      <Form.TagPicker id="tagPicker" title="Tag Picker" onBlur={logEvent} onFocus={logEvent}>
        {[1, 2, 3, 4, 5, 6, 7].map((num) => (
          <Form.TagPicker.Item value={String(num)} title={String(num)} key={num} />
        ))}
      </Form.TagPicker>
    </Form>
  );
}

function logEvent(event: Form.Event<string[] | string>) {
  console.log(`Event '${event.type}' has happened for '${event.target.id}'. Current 'value': '${event.target.value}'`);
}
```

#### Form.Event.Type

不同类型的 [`Form.Event`](form.md#form.event)。可以是 `"focus"` 或 `"blur"`。

### Form.Values

表单项的值。

对于类型安全的表单值，您可以定义自己的接口。使用表单项的 ID 作为属性名称。

#### 例子

```typescript
import { Form, Action, ActionPanel } from "@raycast/api";

interface Values {
  todo: string;
  due?: Date;
}

export default function Command() {
  function handleSubmit(values: Values) {
    console.log(values);
  }

  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit" onSubmit={handleSubmit} />
        </ActionPanel>
      }
    >
      <Form.TextField id="todo" title="Todo" />
      <Form.DatePicker id="due" title="Due Date" />
    </Form>
  );
}
```

#### 属性

<table><thead><tr><th>名称</th><th>类型</th><th width="102">必填</th><th>描述</th></tr></thead><tbody><tr><td>[itemId: string]</td><td><code>any</code></td><td>Yes</td><td>给定项的表单值。</td></tr></tbody></table>

### Form.DatePicker.Type

用户可以使用 `Form.DatePicker` 选择日期组件的类型。

#### 枚举成员

| 名称       | 描述                 |
| -------- | ------------------ |
| DateTime | 除了年、月、日之外，还可以选择时、秒 |
| Date     | 只能选择年、月、日          |

***

## 命令式API

您可以使用 React 的 [useRef](https://reactjs.org/docs/hooks-reference.html#useref) 钩子来创建可以访问本机表单项公开的命令式 API（例如 `.focus()` 或 `.reset()`）的变量。

{% hint style="info" %}
命令式 API 需要 1.33.0 或更高版本的 `@raycast/api` 包。
{% endhint %}

```typescript
import { useRef } from "react";
import { ActionPanel, Form, Action } from "@raycast/api";

interface FormValues {
  nameField: string;
  bioTextArea: string;
  datePicker: string;
}

export default function Command() {
  const textFieldRef = useRef<Form.TextField>(null);
  const textAreaRef = useRef<Form.TextArea>(null);
  const datePickerRef = useRef<Form.DatePicker>(null);
  const passwordFieldRef = useRef<Form.PasswordField>(null);
  const dropdownRef = useRef<Form.Dropdown>(null);
  const tagPickerRef = useRef<Form.TagPicker>(null);
  const firstCheckboxRef = useRef<Form.Checkbox>(null);
  const secondCheckboxRef = useRef<Form.Checkbox>(null);

  async function handleSubmit(values: FormValues) {
    console.log(values);
    datePickerRef.current?.focus();
    dropdownRef.current?.reset();
  }

  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit" onSubmit={handleSubmit} />
          <ActionPanel.Section title="Focus">
            <Action title="Focus TextField" onAction={() => textFieldRef.current?.focus()} />
            <Action title="Focus TextArea" onAction={() => textAreaRef.current?.focus()} />
            <Action title="Focus DatePicker" onAction={() => datePickerRef.current?.focus()} />
            <Action title="Focus PasswordField" onAction={() => passwordFieldRef.current?.focus()} />
            <Action title="Focus Dropdown" onAction={() => dropdownRef.current?.focus()} />
            <Action title="Focus TagPicker" onAction={() => tagPickerRef.current?.focus()} />
            <Action title="Focus First Checkbox" onAction={() => firstCheckboxRef.current?.focus()} />
            <Action title="Focus Second Checkbox" onAction={() => secondCheckboxRef.current?.focus()} />
          </ActionPanel.Section>
          <ActionPanel.Section title="Reset">
            <Action title="Reset TextField" onAction={() => textFieldRef.current?.reset()} />
            <Action title="Reset TextArea" onAction={() => textAreaRef.current?.reset()} />
            <Action title="Reset DatePicker" onAction={() => datePickerRef.current?.reset()} />
            <Action title="Reset PasswordField" onAction={() => passwordFieldRef.current?.reset()} />
            <Action title="Reset Dropdown" onAction={() => dropdownRef.current?.reset()} />
            <Action title="Reset TagPicker" onAction={() => tagPickerRef.current?.reset()} />
            <Action title="Reset First Checkbox" onAction={() => firstCheckboxRef.current?.reset()} />
            <Action title="Reset Second Checkbox" onAction={() => secondCheckboxRef.current?.reset()} />
          </ActionPanel.Section>
        </ActionPanel>
      }
    >
      <Form.TextField id="textField" title="TextField" ref={textFieldRef} />
      <Form.TextArea id="textArea" title="TextArea" ref={textAreaRef} />
      <Form.DatePicker id="datePicker" title="DatePicker" ref={datePickerRef} />
      <Form.PasswordField id="passwordField" title="PasswordField" ref={passwordFieldRef} />
      <Form.Separator />
      <Form.Dropdown
        id="dropdown"
        title="Dropdown"
        defaultValue="first"
        onChange={(newValue) => {
          console.log(newValue);
        }}
        ref={dropdownRef}
      >
        <Form.Dropdown.Item value="first" title="First" />
        <Form.Dropdown.Item value="second" title="Second" />
      </Form.Dropdown>
      <Form.Separator />
      <Form.TagPicker
        id="tagPicker"
        title="TagPicker"
        ref={tagPickerRef}
        onChange={(t) => {
          console.log(t);
        }}
      >
        {["one", "two", "three"].map((tag) => (
          <Form.TagPicker.Item key={tag} value={tag} title={tag} />
        ))}
      </Form.TagPicker>
      <Form.Separator />
      <Form.Checkbox
        id="firstCheckbox"
        title="First Checkbox"
        label="First Checkbox"
        ref={firstCheckboxRef}
        onChange={(checked) => {
          console.log("first checkbox onChange ", checked);
        }}
      />
      <Form.Checkbox
        id="secondCheckbox"
        title="Second Checkbox"
        label="Second Checkbox"
        ref={secondCheckboxRef}
        onChange={(checked) => {
          console.log("second checkbox onChange ", checked);
        }}
      />
      <Form.Separator />
    </Form>
  );
}
```
