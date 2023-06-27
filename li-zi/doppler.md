---
description: 此示例使用简单的表单来收集数据。
---

# Doppler 共享 Secrets

{% hint style="info" %}
该示例的完整源代码可以在 [这里](https://github.com/raycast/extensions/tree/main/extensions/doppler-share-secrets#readme) 找到。您可以在 [此处](https://www.raycast.com/thomas/doppler-share-secrets) 安装扩展。
{% endhint %}

在此示例中，我们使用表单来收集用户的输入。为了让它变得有趣，我们使用 [Doppler](http://share.doppler.com/)，这是一项可以轻松安全地共享 API 密钥或密码等敏感信息的服务。

![示例：与 Doppler 安全共享秘密](../.gitbook/assets/example-doppler-share-secrets.png)

该扩展有一个命令。该命令是一个简单的表单，其中包含一个用于秘密的文本字段、一个用于查看后到期的下拉列表以及用于在最多天数后备用到期的第二个下拉列表。

## 添加表单项

首先，我们要渲染静态表单。为此，我们添加所有提到的表单项：

```typescript
import { Action, ActionPanel, Clipboard, Form, Icon, showToast, Toast } from "@raycast/api";
import got from "got";

export default function Command() {
  return (
    <Form>
      <Form.TextArea id="secret" title="Secret" placeholder="Enter sensitive data to securely share…" />
      <Form.Dropdown id="expireViews" title="Expire After Views" storeValue>
        <Form.Dropdown.Item value="1" title="1 View" />
        <Form.Dropdown.Item value="2" title="2 Views" />
        <Form.Dropdown.Item value="3" title="3 Views" />
        <Form.Dropdown.Item value="5" title="5 Views" />
        <Form.Dropdown.Item value="10" title="10 Views" />
        <Form.Dropdown.Item value="20" title="20 Views" />
        <Form.Dropdown.Item value="50" title="50 Views" />
        <Form.Dropdown.Item value="-1" title="Unlimited Views" />
      </Form.Dropdown>
      <Form.Dropdown id="expireDays" title="Expire After Days" storeValue>
        <Form.Dropdown.Item value="1" title="1 Day" />
        <Form.Dropdown.Item value="2" title="2 Days" />
        <Form.Dropdown.Item value="3" title="3 Days" />
        <Form.Dropdown.Item value="7" title="1 Week" />
        <Form.Dropdown.Item value="14" title="2 Weeks" />
        <Form.Dropdown.Item value="30" title="1 Month" />
        <Form.Dropdown.Item value="90" title="3 Months" />
      </Form.Dropdown>
    </Form>
  );
}
```

两个下拉列表都将 `storeValue` 设置为 true。当再次打开命令时，会恢复上次选择的值。当您的用户经常选择相同的选项时，此选项非常方便。在这种情况下，我们假设用户希望保留过期设置。

## 提交表单值

现在我们有了表单，我们想要收集插入的值，将它们发送到 Doppler 并复制允许我们安全共享信息的 URL。为此，我们创建一个新操作：

```tsx
function ShareSecretAction() {
  async function handleSubmit(values: { secret: string; expireViews: number; expireDays: number }) {
    if (!values.secret) {
      showToast({
        style: Toast.Style.Failure,
        title: "Secret is required",
      });
      return;
    }

    const toast = await showToast({
      style: Toast.Style.Animated,
      title: "Sharing secret",
    });

    try {
      const { body } = await got.post("https://api.doppler.com/v1/share/secrets/plain", {
        json: {
          secret: values.secret,
          expire_views: values.expireViews,
          expire_days: values.expireDays,
        },
        responseType: "json",
      });

      await Clipboard.copy((body as any).authenticated_url);

      toast.style = Feedback.Toast.Style.Success;
      toast.title = "Shared secret";
      toast.message = "Copied link to clipboard";
    } catch (error) {
      toast.style = Feedback.Toast.Style.Failure;
      toast.title = "Failed sharing secret";
      toast.message = String(error);
    }
  }

  return <Action.SubmitForm icon={Icon.Upload} title="Share Secret" onSubmit={handleSubmit} />;
}
```

让我们来分解一下：

* `<ShareSecretAction>` 返回一个返回一个 [`<Action.SubmitForm>`](../api-can-kao/user-interface/actions.md#action.submitform).
* `handleSubmit()` 函数在表单及其值提交时被调用。
  * 首先我们检查用户是否输入了密码。如果没有，我们提示一个 toast。
  * 然后，我们显示一个 toast，暗示正在进行网络调用以共享 Secret。
  * 我们使用表单值调用  [Doppler's API](https://docs.doppler.com/reference/share-secret)&#x20;
    * 如果网络响应成功，我们会将经过身份验证的 URL 复制到剪贴板并显示成功消息。
    * 如果网络响应失败，我们会显示失败消息以及有关失败的其他信息。

## 连起来

最后一步是将  `<ShareSecretAction>`  添加到表单中：

```typescript
import { Action, ActionPanel, Clipboard, Form, Icon, showToast, Toast } from "@raycast/api";
import got from "got";

export default function Command() {
  return (
    <Form
      actions={
        <ActionPanel>
          <ShareSecretAction />
        </ActionPanel>
      }
    >
      <Form.TextArea id="secret" title="Secret" placeholder="Enter sensitive data to securely share…" />
      <Form.Dropdown id="expireViews" title="Expire After Views" storeValue>
        <Form.Dropdown.Item value="1" title="1 View" />
        <Form.Dropdown.Item value="2" title="2 Views" />
        <Form.Dropdown.Item value="3" title="3 Views" />
        <Form.Dropdown.Item value="5" title="5 Views" />
        <Form.Dropdown.Item value="10" title="10 Views" />
        <Form.Dropdown.Item value="20" title="20 Views" />
        <Form.Dropdown.Item value="50" title="50 Views" />
        <Form.Dropdown.Item value="-1" title="Unlimited Views" />
      </Form.Dropdown>
      <Form.Dropdown id="expireDays" title="Expire After Days" storeValue>
        <Form.Dropdown.Item value="1" title="1 Day" />
        <Form.Dropdown.Item value="2" title="2 Days" />
        <Form.Dropdown.Item value="3" title="3 Days" />
        <Form.Dropdown.Item value="7" title="1 Week" />
        <Form.Dropdown.Item value="14" title="2 Weeks" />
        <Form.Dropdown.Item value="30" title="1 Month" />
        <Form.Dropdown.Item value="90" title="3 Months" />
      </Form.Dropdown>
    </Form>
  );
}

function ShareSecretAction() {
  async function handleSubmit(values: { secret: string; expireViews: number; expireDays: number }) {
    if (!values.secret) {
      showToast({
        style: Toast.Style.Failure,
        title: "Secret is required",
      });
      return;
    }

    const toast = await showToast({
      style: Toast.Style.Animated,
      title: "Sharing secret",
    });

    try {
      const { body } = await got.post("https://api.doppler.com/v1/share/secrets/plain", {
        json: {
          secret: values.secret,
          expire_views: values.expireViews,
          expire_days: values.expireDays,
        },
        responseType: "json",
      });

      await Clipboard.copy((body as any).authenticated_url);

      toast.style = Toast.Style.Success;
      toast.title = "Shared secret";
      toast.message = "Copied link to clipboard";
    } catch (error) {
      toast.style = Toast.Style.Failure;
      toast.title = "Failed sharing secret";
      toast.message = String(error);
    }
  }

  return <Action.SubmitForm icon={Icon.Upload} title="Share Secret" onSubmit={handleSubmit} />;
}
```

结束啦。一个简单的表单，用于输入 secret 并获取您可以与其他人共享的 URL，该 URL 将根据您的偏好 “自我销毁”。后续步骤，您可以使用 `<PasteAction>` 将链接直接粘贴到最前面的应用程序，或者添加另一个清除表单的操作，然后让您创建另一个可共享链接。
