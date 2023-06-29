# OAuth

## 前提

Raycast 扩展可以使用 OAuth 来授权代表用户访问提供商的资源。由于 Raycast 是一款桌面应用程序，并且扩展被视为“公开”，因此我们仅支持 [PKCE 流程](https://datatracker.ietf.org/doc/html/rfc7636)（代码交换的证明密钥，发音为“pixy”）。此流程是针对无法保守客户端秘密的本机客户端的官方建议。使用 PKCE，客户端动态创建一个秘密，并在代码交换期间再次使用该秘密，确保只有执行初始请求的客户端才能用代码交换访问令牌（“拥有证明”）。

在考虑 OAuth 之前，首先检查您的提供商是否支持 PKCE。您通常可以在提供商的 OAuth 文档中查找 `code_challenge` 和 `code_verifier` 参数找到此内容。 Google、Twitter、GitLab、Spotify、Zoom、Asana 或 Dropbox 等提供商均已支持 PKCE。如果您发现您的提供商尚不支持 PKCE，您通常可以使用其他形式的授权，例如个人访问令牌（可与 Raycast 密码首选项一起使用），或将 OAuth 流“调整”为 PKCE 的开源兼容（在这种情况下，您需要操作自己的后端服务，因此仅建议对于非常高级的用例。）

## OAuth 流程

![](../.gitbook/assets/oauth-overlay-twitter.png)

扩展程序的 OAuth 流程如下所示：

1. 扩展程序启动 OAuth 流程并开始授权
2. Raycast 显示 OAuth 面板（“连接到提供商...”）
3. 用户在网络浏览器中打开提供商的同意页面
4. 用户同意后，提供商重定向回 Raycast
5. Raycast 打开授权完成

当流程完成之后，扩展程序已从提供商处收到访问令牌，并且可以执行 API 调用。该 API 提供安全存储和检索令牌集的功能，以便扩展程序可以检查用户是否已登录以及是否需要刷新过期的访问令牌。 Raycast 还会自动显示注销首选项。

![](../.gitbook/assets/oauth-overlay-twitter-success.png)

## OAuth 应用程序

您首先需要向您的提供商注册一个新的 OAuth 应用程序。这通常在提供商的开发人员导航中完成。注册后，您将收到一个client ID。您还需要配置重定向 URI，请参阅下一节。

注意：请确保选择支持 PKCE 的应用程序类型。一些提供商仍然向您显示客户端密钥，您不需要也不应该在扩展中硬编码，或者仅针对某些类型（例如“桌面”、“本机”或“移动”应用程序类型）支持 PKCE。

## 授权

扩展可以使用 [OAuth.PKCEClient](oauth.md#oauth.pkceclient) 上的方法启动 OAuth 流程并进行授权。

您可以创建一个新客户端并为其配置提供程序名称、图标和描述，这些内容将显示在 OAuth 面板中。您还可以选择不同的重定向方法；根据您选择的方法，您需要在提供商注册的 OAuth 应用程序中将此值配置为重定向 URI。 （请参阅每种方法的 [OAuth.RedirectMethod](oauth.md#oauth.redirectmethod) 文档，以获取支持的重定向 URI 的具体示例。）如果可以选择，请使用 `OAuth.RedirectMethod.Web` 并输入 `https://raycast.com/redirect?packageName=Extension` （无论您是否有添加 `?packageName=Extension` ，这取决于提供商）。

```typescript
import { OAuth } from "@raycast/api";

const client = new OAuth.PKCEClient({
  redirectMethod: OAuth.RedirectMethod.Web,
  providerName: "Twitter",
  providerIcon: "twitter-logo.png",
  description: "Connect your Twitter account…",
});
```

接下来，您使用授权点、客户端 ID 和范围值创建授权请求。当您注册新的 OAuth 应用程序时，您会收到来自提供商文档的所有内容。

返回的 [AuthorizationRequest](oauth.md#oauth.authorizationrequest) 包含代码变更、验证者、状态和重定向 URI 等参数作为标准 OAuth 授权请求。如果需要，您还可以通过 [OAuth.AuthorizationOptions](oauth.md#oauth.authorizationoptions) 自定义授权 URL。

```typescript
const authRequest = await client.authorizationRequest({
  endpoint: "https://twitter.com/i/oauth2/authorize",
  clientId: "YourClientId",
  scope: "tweet.read users.read follows.read",
});
```

要获取令牌交换所需的授权码，请使用上一步中的请求调用 [authorize](oauth.md#oauth.pkceclient-authorize)。此调用显示 Raycast OAuth 面板层，并为用户提供在 Web 浏览器中打开同意页面的选项。授权 promise 在重定向回 Raycast 并进入扩展后变为 resolved ：

```typescript
const { authorizationCode } = await client.authorize(authRequest);
```

{% hint style="info" %}
在开发模式下，确保在测试活动 OAuth 授权和重定向时不要触发自动重新加载（例如通过保存文件）。当您重定向回扩展时，这会导致 OAuth 状态不匹配，因为客户端将在重新加载时重新初始化。
{% endhint %}

现在您已收到授权代码，您可以使用提供商的令牌端点将此代码交换为访问令牌。此令牌交换（以及以下 API 调用）可以使用您首选的 Node HTTP 客户端库来完成。使用  `node-fetch` 获取的示例：

```typescript
async function fetchTokens(authRequest: OAuth.AuthorizationRequest, authCode: string): Promise<OAuth.TokenResponse> {
  const params = new URLSearchParams();
  params.append("client_id", "YourClientId");
  params.append("code", authCode);
  params.append("code_verifier", authRequest.codeVerifier);
  params.append("grant_type", "authorization_code");
  params.append("redirect_uri", authRequest.redirectURI);

  const response = await fetch("https://api.twitter.com/2/oauth2/token", {
    method: "POST",
    body: params,
  });
  if (!response.ok) {
    console.error("fetch tokens error:", await response.text());
    throw new Error(response.statusText);
  }
  return (await response.json()) as OAuth.TokenResponse;
}
```

## Token 存储

PKCE 客户端暴露用于存储、检索和删除 token 的方法。 [TokenSet](oauth.md#oauth.tokenset) 包含访问令牌，通常还包含刷新 token、过期值和当前范围。由于此数据由提供者的 token 端作为标准 OAuth JSON 响应返回，因此您可以直接存储响应 ([OAuth.TokenResponse](oauth.md#oauth.tokenresponse)) 或使用 [OAuth.TokenSetOptions](oauth.md#oauth.tokensetoptions)：

```typescript
await client.setTokens(tokenResponse);
```

存储 token 后，Raycast 将自动显示扩展程序的注销首选项。当用户注销时，token 将被删除。

TokenSet 还允许您在开始授权流程之前检查用户是否已登录：

```typescript
const tokenSet = await client.getTokens();
```

## Token 刷新

由于 access token 通常会过期，因此扩展程序应该提供一种刷新 access token 的方法，否则用户将被注销或看到报错。某些提供商要求您添加离线操作，以便获得刷新 access token。 （例如，Twitter 需要 `Offline.access` 范围，否则它只返回 access token。）基本刷新流程可能如下所示：

```typescript
const tokenSet = await client.getTokens();
if (tokenSet?.accessToken) {
  if (tokenSet.refreshToken && tokenSet.isExpired()) {
    await client.setTokens(await refreshTokens(tokenSet.refreshToken));
  }
  return;
}
// authorize...
```

该代码将在启动授权流程之前运行。它会检查 token 是否存在以查看用户是否已登录，然后检查是否存在刷新 token 以及 token 是否已过期（通过 TokenSet 上的便捷方法 `isExpired()` ）。如果过期，则刷新该 token，并在 token 中进行更新。下面是使用  `node-fetch` 的示例：

```typescript
async function refreshTokens(refreshToken: string): Promise<OAuth.TokenResponse> {
  const params = new URLSearchParams();
  params.append("client_id", "YourClientId");
  params.append("refresh_token", refreshToken);
  params.append("grant_type", "refresh_token");

  const response = await fetch("https://api.twitter.com/2/oauth2/token", {
    method: "POST",
    body: params,
  });
  if (!response.ok) {
    console.error("refresh tokens error:", await response.text());
    throw new Error(response.statusText);
  }

  const tokenResponse = (await response.json()) as OAuth.TokenResponse;
  tokenResponse.refresh_token = tokenResponse.refresh_token ?? refreshToken;
  return tokenResponse;
}
```

## 例子

我们提供了[ Google、Twitter 和 Dropbox 的 OAuth 示例集成](https://github.com/raycast/extensions/tree/main/examples/api-examples)，演示了上面所示的整个流程。

## API 参考

### OAuth.PKCEClient

使用  [OAuth.PKCEClient.Options](oauth.md#oauth.pkceclient.options)  配置 OAuth 叠加层上显示的内容。

#### 签名

```typescript
constructor(options: OAuth.PKCEClient.Options): OAuth.PKCEClient
```

#### 例子

```typescript
import { OAuth } from "@raycast/api";

const client = new OAuth.PKCEClient({
  redirectMethod: OAuth.RedirectMethod.Web,
  providerName: "Twitter",
  providerIcon: "twitter-logo.png",
  description: "Connect your Twitter account…",
});
```

#### 方法

| 方法                                                                                                                      |
| ----------------------------------------------------------------------------------------------------------------------- |
| [`authorizationRequest(options: AuthorizationRequestOptions): Promise`](oauth.md#oauth.pkceclient-authorizationrequest) |
| [`authorize(options: AuthorizationRequest \| AuthorizationOptions): Promise`](oauth.md#oauth.pkceclient-authorize)      |
| [`setTokens(options: TokenSetOptions \| TokenResponse): Promise`](oauth.md#oauth.pkceclient-settokens)                  |
| [`getTokens(): Promise<TokenSet \| undefined>`](oauth.md#oauth.pkceclient-gettokens)                                    |
| [`removeTokens(): Promise`](oauth.md#oauth.pkceclient-removetokens)                                                     |

### OAuth.PKCEClient#authorizationRequest

为提供的授权点、client ID 和 scopes 创建授权请求。在调用  [authorize](oauth.md#oauth.pkceclient-authorize) 之前，您需要先创建授权请求。

PKCE 请求生成的代码变更使用 S256 方法。

#### 签名

```typescript
authorizationRequest(options: AuthorizationRequestOptions): Promise<AuthorizationRequest>;
```

#### 例子

```typescript
const authRequest = await client.authorizationRequest({
  endpoint: "https://twitter.com/i/oauth2/authorize",
  clientId: "YourClientId",
  scope: "tweet.read users.read follows.read",
});
```

#### 参数

<table><thead><tr><th width="122">名称</th><th width="322">类型</th><th width="63">必填</th><th>描述</th></tr></thead><tbody><tr><td>options</td><td><a href="oauth.md#oauth.authorizationrequestoptions"><code>AuthorizationRequestOptions</code></a></td><td>Yes</td><td>用于创建授权请求的选项。</td></tr></tbody></table>

#### 返回

您可以将其用作  [authorize](oauth.md#oauth.pkceclient-authorize) 输入的 AuthorizationRequest  Promise。

### OAuth.PKCEClient#authorize

启动授权并在 Raycast 中显示 OAuth 覆盖。作为参数，您可以直接使用 authorizationRequest 返回的请求，也可以通过从 AuthorizationRequest 中提取参数并通过  AuthorizationOptions 提供您自己的URL来自定义URL。最终该 URL 将用于在 Web 浏览器中打开提供商的授权页面。

#### 签名

```typescript
authorize(options: AuthorizationRequest | AuthorizationOptions): Promise<AuthorizationResponse>;
```

#### 例子

```typescript
const { authorizationCode } = await client.authorize(authRequest);
```

#### 参数

<table><thead><tr><th width="128">名称</th><th width="266">类型</th><th>必填</th><th>描述</th></tr></thead><tbody><tr><td>options</td><td><a href="oauth.md#oauth.authorizationrequest"><code>AuthorizationRequest</code></a> <code>|</code> <a href="oauth.md#oauth.authorizationoptions"><code>AuthorizationOptions</code></a></td><td>Yes</td><td>用于授权的选项。</td></tr></tbody></table>

#### 返回

[AuthorizationResponse](oauth.md#oauth.authorizationresponse) 的 promise，其中包含交换 token 所需的授权代码。当用户从提供商的授权页面重定向回 Raycast 扩展时，promise 变为 resolved。

### OAuth.PKCEClient#setTokens

安全地存储提供者的 TokenSet。从提供商获取 access token 后使用此选项。如果提供者返回标准 OAuth JSON 令牌响应，您可以直接传递 TokenResponse。至少，您需要设置 `accessToken`，通常还需要设置 `refreshToken` 和 `isExpired`。

保存 token 时，Raycast 会自动显示扩展程序的注销首选项。

如果你想使用轻便的 `isExpired()` 方法，则必须配置 `expiresIn` 属性.

#### 签名

```typescript
setTokens(options: TokenSetOptions | TokenResponse): Promise<void>;
```

#### 例子

```typescript
await client.setTokens(tokenResponse);
```

#### 参数

<table><thead><tr><th width="134">名称</th><th>类型</th><th width="160">必填</th><th>描述</th></tr></thead><tbody><tr><td>options</td><td><a href="oauth.md#oauth.tokensetoptions"><code>TokenSetOptions</code></a> <code>|</code> <a href="oauth.md#oauth.tokenresponse"><code>TokenResponse</code></a></td><td>Yes</td><td>用于存储 token 的选项。</td></tr></tbody></table>

#### 返回

当 token 被存储后， promise 变为 resolve。

### OAuth.PKCEClient#getTokens

检索客户端存储的 TokenSet。您可以使用它来初步检查是否应启动授权流程或用户已登录，或者刷新 access token。

#### 签名

```typescript
getTokens(): Promise<TokenSet | undefined>;
```

#### 例子

```typescript
const tokenSet = await client.getTokens();
```

#### 返回

获取 token 后， promise 变为 resolves。

### OAuth.PKCEClient#removeTokens

删除客户端存储的 TokenSet。 Raycast 自动显示删除 token 的注销首选项。仅当您需要在扩展程序中提供额外的注销选项或者您想要因迁移而删除令牌集时，才使用此方法。

#### 签名

```typescript
removeTokens(): Promise<void>;
```

#### 例子

```typescript
await client.removeTokens();
```

#### 返回

当 token 被移除后，promise 变为 resolve。

## 类型

### OAuth.PKCEClient.Options

用于创建新 [PKCEClient](oauth.md#oauth.pkceclient) 的选项。

#### 属性

| 名称                                               | 描述                                                                          | 类型                                                                      |
| ------------------------------------------------ | --------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| providerName<mark style="color:red;">\*</mark>   | 提供者的名称，显示在 OAuth 展示面板中。                                                     | `string`                                                                |
| redirectMethod<mark style="color:red;">\*</mark> | OAuth 流的 重定向方法。确保将其设置为提供程序的正确方法，请参阅 OAuth.RedirectMethod 了解更多信息。            | [`OAuth.RedirectMethod`](oauth.md#oauth.redirectmethod)                 |
| description                                      | 可选描述，显示在 OAuth 展示面板中。您可以使用它来为最终用户自定义消息，例如处理范围更改或其他迁移。如果未配置，Raycast 将显示默认消息。 | `string`                                                                |
| providerIcon                                     | OAuth 展示面板中显示的图标。确保提供至少 64x64 像素的大小。                                        | [`Image.ImageLike`](user-interface/icons-and-images.md#image.imagelike) |
| providerId                                       | 用于将客户端与提供商关联起来的可选 ID。仅当您在扩展中使用多个不同的客户端时才设置此项。                               | `string`                                                                |

### OAuth.RedirectMethod

定义 OAuth 流支持的重定向方法。您可以在 Web 和应用程序方案重定向方法之间进行选择，具体取决于提供商在设置 OAuth 应用程序时的要求。有关需要配置的重定向 URI 的示例，请参阅每种方法的文档。

#### 枚举成员

| 名称     | 值                                                                                                                                                                                                                                                                                                                                         |
| ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Web    | 重定向回 Raycast 网站，然后该网站将打开扩展程序。在 OAuth 应用程序中，配置 `https://raycast.com/redirect?packageName=Extension` （这是所有扩展程序的静态重定向 URL。） 如果提供程序不接受重定向 URL 中的查询参数，您也可以使用 `https://raycast.com/redirect/extension`，然后通过其 `extraParameters` 属性自定义 AuthorizationRequest。例如添加： `extraParameters: { "redirect_uri": "https://raycast.com/redirect/extension" }` |
| App    | 进行基于应用程序方案的重定向，直接打开 Raycast。在 OAuth 应用程序中，配置 `raycast://oauth?package_name=Extension`                                                                                                                                                                                                                                                     |
| AppURI | 用于直接打开 Raycast 的 URI 样式应用程序方案。在 OAuth 应用程序中，配置 `com.raycast:/oauth?package_name=Extension` （请注意单斜线 - 例如，Google 需要这种风格的 OAuth 应用程序，其中 Bundle ID 为 `com.raycast`）                                                                                                                                                                           |

### OAuth.AuthorizationRequestOptions

通过 authorizationRequest 进行授权请求的选项。

| 属性                                         | 描述                                                                                    | 类型                       |
| ------------------------------------------ | ------------------------------------------------------------------------------------- | ------------------------ |
| clientId<mark style="color:red;">\*</mark> | 已配置的 OAuth 应用程序的 client ID。                                                           | `string`                 |
| endpoint<mark style="color:red;">\*</mark> | OAuth 提供程序的授权点的 URL。                                                                  | `string`                 |
| scope<mark style="color:red;">\*</mark>    | 以空格分隔的范围列表，用于标识要代表用户访问的资源。范围通常在浏览器中的提供商同意屏幕上向用户显示。请注意，某些提供商要求在注册的 OAuth 应用程序中配置相同的范围。 | `string`                 |
| extraParameters                            | 授权请求的可选附加参数。请注意，某些提供程序需要额外的参数，例如为了获取长期刷新 token。                                       | `Record<string, string>` |

### OAuth.AuthorizationRequestURLParams

&#x20;[AuthorizationRequest](oauth.md#oauth.authorizationrequest) 的值。 PKCE客户端自动为您生成值并将其返回给 authorizationRequest

| 属性                                              | 描述                        | 类型       |
| ----------------------------------------------- | ------------------------- | -------- |
| codeChallenge<mark style="color:red;">\*</mark> |  PKCE `code_challenge` 的值 | `string` |
| codeVerifier<mark style="color:red;">\*</mark>  | PKCE `code_verifier` 的值   | `string` |
| redirectURI<mark style="color:red;">\*</mark>   | OAuth `redirect_uri` 的值   | `string` |
| state<mark style="color:red;">\*</mark>         | OAuth `state` 的值          | `string` |

### OAuth.AuthorizationRequest

&#x20;[authorizationRequest](oauth.md#oauth.authorizationrequest) 返回的请求。可用作授权的直接输入，或提取参数以在 AuthorizationOptions 中构建自定义 URL。

| 属性                                              | 描述                        | 类型             |
| ----------------------------------------------- | ------------------------- | -------------- |
| codeChallenge<mark style="color:red;">\*</mark> | PKCE `code_challenge` 的值  | `string`       |
| codeVerifier<mark style="color:red;">\*</mark>  | PKCE `code_verifier` 的值   | `string`       |
| redirectURI<mark style="color:red;">\*</mark>   | OAuth `redirect_uri` 的值   | `string`       |
| state<mark style="color:red;">\*</mark>         | OAuth `state` 的值          | `string`       |
| toURL<mark style="color:red;">\*</mark>         | 构造完整的授权 URL               | `() => string` |

#### 方法

| 名称      | 类型             | 描述           |
| ------- | -------------- | ------------ |
| toURL() | `() => string` | 构造完整的授权 URL。 |

### OAuth.AuthorizationOptions

用于自定义 [authorize](oauth.md#oauth.pkceclient-authorize) 的选项。您可以使用 AuthorizationRequest 中的值来构建您自己的 URL。

| 属性                                    | 描述         | 类型       |
| ------------------------------------- | ---------- | -------- |
| url<mark style="color:red;">\*</mark> | 完整的授权 URL。 | `string` |

### OAuth.AuthorizationResponse

&#x20;[authorize](oauth.md#oauth.pkceclient-authorize) 返回的响应，包含提供者重定向后的授权代码。然后，您可以使用提供商的 token 将授权代码交换为 access token。

| 属性                                                  | 描述                 | 类型       |
| --------------------------------------------------- | ------------------ | -------- |
| authorizationCode<mark style="color:red;">\*</mark> | 来自 OAuth 提供商的授权代码。 | `string` |

### OAuth.TokenSet

描述从 OAuth 提供者的 token 响应创建的 TokenSet。 accessToken 是唯一必需的参数，但通常 OAuth 提供程序还会返回刷新 token、过期值和范围。通过 setTokens 安全地存储 token 并通过 getTokens 检索它。

| 属性                                            | 描述                                                                                       | 类型              |
| --------------------------------------------- | ---------------------------------------------------------------------------------------- | --------------- |
| accessToken<mark style="color:red;">\*</mark> | OAuth token 请求返回的访问令牌。                                                                   | `string`        |
| updatedAt<mark style="color:red;">\*</mark>   | 通过 OAuth.PKCEClient.setTokens 存储 token 的日期。                                              | `Date`          |
| isExpired<mark style="color:red;">\*</mark>   | 检查访问 token 是否已过期。由于该方法考虑了几秒的“buffer”，所以它在实际过期时间之前几秒就返回 true。需要设置 `expiresIn` 参数。         | `() => boolean` |
| expiresIn                                     | OAuth token 请求返回的可选过期值（以秒为单位）。                                                           | `number`        |
| idToken                                       | 由身份请求返回的可选 ID 令牌（例如 /me、Open ID Connect）。                                                | `string`        |
| refreshToken                                  | OAuth token 请求返回的可选刷新token。                                                              | `string`        |
| scope                                         | OAuth token 请求返回的可选的以空格分隔的范围列表。您可以使用它来将当前存储的访问范围与扩展在未来版本中可能需要的新访问范围进行比较，然后要求用户使用新范围重新授权。 | `string`        |

#### 方法

| 名称          | 类型              | 描述                                                                                    |
| ----------- | --------------- | ------------------------------------------------------------------------------------- |
| isExpired() | `() => boolean` | 检查 access token 是否已过期。由于该方法考虑了几秒的“buffer”，所以它在实际过期时间之前几秒返回了 true。需要设置 `expiresIn` 参数。 |

### OAuth.TokenSetOptions

通过 setTokens 存储 TokenSet 的选项。

| 属性                                            | 描述                                           | 类型       |
| --------------------------------------------- | -------------------------------------------- | -------- |
| accessToken<mark style="color:red;">\*</mark> | OAuth token 请求返回的 access token。              | `string` |
| expiresIn                                     | OAuth token请求返回的可选过期值（以秒为单位）。                | `number` |
| idToken                                       | 由身份请求返回的可选 ID token（例如 /me、Open ID Connect）。 | `string` |
| refreshToken                                  | OAuth token 请求返回的可选刷新 token。                 | `string` |
| scope                                         | OAuth token 请求返回的可选范围值。                      | `string` |

### OAuth.TokenResponse

定义 OAuth token 请求的标准 JSON 响应。响应可以直接用于通过 setTokens 存储 TokenSet。

| 属性                                              | 描述                                              | 类型       |
| ----------------------------------------------- | ----------------------------------------------- | -------- |
| access\_token<mark style="color:red;">\*</mark> | OAuth token 请求返回的 `access_token` 值。             | `string` |
| expires\_in                                     | OAuth token 请求返回的可选 `expires_in` 值（以秒为单位）。      | `number` |
| id\_token                                       | 身份请求返回的可选 `id_token` 值（例如 /me、Open ID Connect）。 | `string` |
| refresh\_token                                  | OAuth token 请求返回的可选`refresh_token` 值。           | `string` |
| scope                                           | OAuth token 请求返回的可选 `scope`  值。                 | `string` |
