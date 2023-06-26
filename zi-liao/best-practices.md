---
description: 确保扩展程序获得良好用户体验的提示。
---

# 最佳实践

## 一般情况

### 错误处理

网络请求可能会失败，文件权限可能会丢失……更常见的是，错误随处而见。默认情况下，我们处理每个未处理的异常或未解决的 Promise 并把错误展示出来。但是，您应该自行处理命令的 “预期” 错误。您不应该因为出现问题而中断用户流程。例如，如果网络请求失败但您可以读取缓存，那就显示缓存内容。用户可能不需要立即获得新数据。在大多数情况下，最好显示包含错误信息的 Toast。

以下是如何显示错误提示的示例：

```typescript
import { Detail, showToast, Toast } from "@raycast/api";
import { useEffect, useState } from "react";

export default function Command() {
  const [error, setError] = useState<Error>();

  useEffect(() => {
    setTimeout(() => {
      setError(new Error("Booom 💥"));
    }, 1000);
  }, []);

  useEffect(() => {
    if (error) {
      showToast({
        style: Toast.Style.Failure,
        title: "Something went wrong",
        message: error.message,
      });
    }
  }, [error]);

  return <Detail markdown="Example for proper error handling" />;
}
```

### 处理运行时依赖性

理想情况下，您的扩展不依赖于任何运行时依赖项。实际上，有时需要本地安装的应用程序或 CLI 才能执行功能。以下是一些保证良好用户体验的提示：

* 如果命令需要运行时依赖项才能运行（例如需要用户安装的应用程序），请展示有用的消息。
  * 如果您的扩展与应用程序紧密耦合，例如在 Safari 中搜索选项卡或使用 AppleScript 控制 Spotify，代码检查没必要是那么严格，因为用户很可能因为没有本地安装依赖项从而不会安装扩展。
* 如果扩展中只有某些功能需要运行时依赖项，请考虑仅在安装依赖项后才提供此功能。通常，[这是](https://developers.raycast.com/information/terminology#action) 最佳情况，例如在桌面应用程序而不是浏览器中打开 URL。

### 显示 loading 指示器

当命令需要加载大数据集时，最好告知用户这一点。为了让你的命令保持敏捷，尽快渲染 React 组件显得非常重要。

您可以从空列表或静态表单开始，然后加载数据以填充视图。为了让用户了解加载过程，您可以在所有顶级组件上使用 `isLoading` 属性，例如 [`<Detail>`](../api-reference/user-interface/detail.md), [`<Form>`](../api-reference/user-interface/form.md), [`<Grid>`](../api-reference/user-interface/grid.md), 或者 [`<List>`](../api-reference/user-interface/list.md)

以下是在列表中显示 loading 指示器的示例：

```typescript
import { List } from "@raycast/api";
import { useEffect, useState } from "react";

export default function Command() {
  const [items, setItems] = useState<string[]>();

  useEffect(() => {
    setTimeout(() => {
      setItems(["Item 1", "Item 2", "Item 3"]);
    }, 1000);
  }, []);

  return (
    <List isLoading={items === undefined}>
      {items?.map((item, index) => (
        <List.Item key={index} title={item} />
      ))}
    </List>
  );
}
```

***

## 表单

### 使用表单校验

当用户最后输入一些数据时，您可以检查该输入是否是您期望的校验。如果数据格式不正确，您可以在 Form 项上设置 `error` 属性以显示一条消息，解释需要更正的内容，并让他们重试。

![](../.gitbook/assets/form-validation.png)

{% hint style="info" %}
请记住，如果表单有任何错误，就不会触发 [`Action.SubmitForm`](../api-reference/user-interface/actions.md#action.submitform) 回调。
{% endhint %}

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
