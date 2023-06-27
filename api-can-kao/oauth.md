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

扩展可以使用 [OAuth.PKCEClient](https://developers.raycast.com/api-reference/oauth#oauth.pkceclient) 上的方法启动 OAuth 流程并进行授权。

您可以创建一个新客户端并为其配置提供程序名称、图标和描述，这些内容将显示在 OAuth 面板中。您还可以选择不同的重定向方法；根据您选择的方法，您需要在提供商注册的 OAuth 应用程序中将此值配置为重定向 URI。 （请参阅每种方法的 [OAuth.RedirectMethod](https://developers.raycast.com/api-reference/oauth#oauth.redirectmethod) 文档，以获取支持的重定向 URI 的具体示例。）如果可以选择，请使用 `OAuth.RedirectMethod.Web` 并输入 `https://raycast.com/redirect?packageName=Extension` （无论您是否有添加 `?packageName=Extension` ，这取决于提供商）。

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

返回的 [AuthorizationRequest](https://developers.raycast.com/api-reference/oauth#oauth.authorizationrequest) 包含代码变更、验证者、状态和重定向 URI 等参数作为标准 OAuth 授权请求。如果需要，您还可以通过 [OAuth.AuthorizationOptions](https://developers.raycast.com/api-reference/oauth#oauth.authorizationoptions) 自定义授权 URL。

```typescript
const authRequest = await client.authorizationRequest({
  endpoint: "https://twitter.com/i/oauth2/authorize",
  clientId: "YourClientId",
  scope: "tweet.read users.read follows.read",
});
```

要获取令牌交换所需的授权码，请使用上一步中的请求调用 [authorize](https://developers.raycast.com/api-reference/oauth#oauth.pkceclient-authorize)。此调用显示 Raycast OAuth 面板层，并为用户提供在 Web 浏览器中打开同意页面的选项。授权 promise 在重定向回 Raycast 并进入扩展后变为 resolved ：

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

PKCE 客户端暴露用于存储、检索和删除 token 的方法。 [TokenSet](https://developers.raycast.com/api-reference/oauth#oauth.tokenset) 包含访问令牌，通常还包含刷新 token、过期值和当前范围。由于此数据由提供者的 token 端作为标准 OAuth JSON 响应返回，因此您可以直接存储响应 ([OAuth.TokenResponse](https://developers.raycast.com/api-reference/oauth#oauth.tokenresponse)) 或使用 [OAuth.TokenSetOptions](https://developers.raycast.com/api-reference/oauth#oauth.tokensetoptions)：

```typescript
await client.setTokens(tokenResponse);
```

存储 token 后，Raycast 将自动显示扩展程序的注销首选项。当用户注销时，token 将被删除。

[TokenSet](https://developers.raycast.com/api-reference/oauth#oauth.tokenset) 还允许您在开始授权流程之前检查用户是否已登录：

```typescript
const tokenSet = await client.getTokens();
```

## Token 刷新

Since access tokens usually expire, an extension should provide a way to refresh the access token, otherwise users would be logged out or see errors. Some providers require you to add an offline scope so that you get a refresh token. (Twitter, for example, needs the scope `offline.access` or it only returns an access token.) A basic refresh flow could look like this:

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

This code would run before starting the authorization flow. It checks the presence of a token set to see whether the user is logged in and then checks whether there is a refresh token and the token set is expired (through the convenience method `isExpired()` on the [TokenSet](oauth.md#oauth.tokenset)). If it is expired, the token is refreshed and updated in the token set. Example using `node-fetch`:

该代码将在启动授权流程之前运行。它会检查 token 是否存在以查看用户是否已登录，然后检查是否存在刷新 token 以及 token 是否已过期（通过 [TokenSet](https://developers.raycast.com/api-reference/oauth#oauth.tokenset) 上的便捷方法 `isExpired()` ）。如果过期，则刷新该 token，并在 token 中进行更新。下面是使用  `node-fetch` 的示例：

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

## API Reference

### OAuth.PKCEClient

Use [OAuth.PKCEClient.Options](oauth.md#oauth.pkceclient.options) to configure what's shown on the OAuth overlay.

#### Signature

```typescript
constructor(options: OAuth.PKCEClient.Options): OAuth.PKCEClient
```

#### Example

```typescript
import { OAuth } from "@raycast/api";

const client = new OAuth.PKCEClient({
  redirectMethod: OAuth.RedirectMethod.Web,
  providerName: "Twitter",
  providerIcon: "twitter-logo.png",
  description: "Connect your Twitter account…",
});
```

#### Methods

| Method                                                                                                                  |
| ----------------------------------------------------------------------------------------------------------------------- |
| [`authorizationRequest(options: AuthorizationRequestOptions): Promise`](oauth.md#oauth.pkceclient-authorizationrequest) |
| [`authorize(options: AuthorizationRequest \| AuthorizationOptions): Promise`](oauth.md#oauth.pkceclient-authorize)      |
| [`setTokens(options: TokenSetOptions \| TokenResponse): Promise`](oauth.md#oauth.pkceclient-settokens)                  |
| [`getTokens(): Promise<TokenSet \| undefined>`](oauth.md#oauth.pkceclient-gettokens)                                    |
| [`removeTokens(): Promise`](oauth.md#oauth.pkceclient-removetokens)                                                     |

### OAuth.PKCEClient#authorizationRequest

Creates an authorization request for the provided authorization endpoint, client ID, and scopes. You need to first create the authorization request before calling [authorize](oauth.md#oauth.pkceclient-authorize).

The generated code challenge for the PKCE request uses the S256 method.

#### Signature

```typescript
authorizationRequest(options: AuthorizationRequestOptions): Promise<AuthorizationRequest>;
```

#### Example

```typescript
const authRequest = await client.authorizationRequest({
  endpoint: "https://twitter.com/i/oauth2/authorize",
  clientId: "YourClientId",
  scope: "tweet.read users.read follows.read",
});
```

#### Parameters

| Name    | Type                                                                        | Required | Description                                           |
| ------- | --------------------------------------------------------------------------- | -------- | ----------------------------------------------------- |
| options | [`AuthorizationRequestOptions`](oauth.md#oauth.authorizationrequestoptions) | Yes      | The options used to create the authorization request. |

#### Return

A promise for an [AuthorizationRequest](oauth.md#oauth.authorizationrequest) that you can use as input for [authorize](oauth.md#oauth.pkceclient-authorize).

### OAuth.PKCEClient#authorize

Starts the authorization and shows the OAuth overlay in Raycast. As parameter you can either directly use the returned request from [authorizationRequest](oauth.md#oauth.authorizationrequest), or customize the URL by extracting parameters from [AuthorizationRequest](oauth.md#oauth.authorizationrequest) and providing your own URL via [AuthorizationOptions](oauth.md#oauth.authorizationoptions). Eventually the URL will be used to open the authorization page of the provider in the web browser.

#### Signature

```typescript
authorize(options: AuthorizationRequest | AuthorizationOptions): Promise<AuthorizationResponse>;
```

#### Example

```typescript
const { authorizationCode } = await client.authorize(authRequest);
```

#### Parameters

| Name    | Type                                                                                                                             | Required | Description                    |
| ------- | -------------------------------------------------------------------------------------------------------------------------------- | -------- | ------------------------------ |
| options | [`AuthorizationRequest`](oauth.md#oauth.authorizationrequest) `\|` [`AuthorizationOptions`](oauth.md#oauth.authorizationoptions) | Yes      | The options used to authorize. |

#### Return

A promise for an [AuthorizationResponse](oauth.md#oauth.authorizationresponse), which contains the authorization code needed for the token exchange. The promise is resolved when the user was redirected back from the provider's authorization page to the Raycast extension.

### OAuth.PKCEClient#setTokens

Securely stores a [TokenSet](oauth.md#oauth.tokenset) for the provider. Use this after fetching the access token from the provider. If the provider returns a a standard OAuth JSON token response, you can directly pass the [TokenResponse](oauth.md#oauth.tokenresponse). At a minimum, you need to set the `accessToken`, and typically you also set `refreshToken` and `isExpired`.

Raycast automatically shows a logout preference for the extension when a token set was saved.

If you want to make use of the convenience `isExpired()` method, the property `expiresIn` must be configured.

#### Signature

```typescript
setTokens(options: TokenSetOptions | TokenResponse): Promise<void>;
```

#### Example

```typescript
await client.setTokens(tokenResponse);
```

#### Parameters

| Name    | Type                                                                                                     | Required | Description                              |
| ------- | -------------------------------------------------------------------------------------------------------- | -------- | ---------------------------------------- |
| options | [`TokenSetOptions`](oauth.md#oauth.tokensetoptions) `\|` [`TokenResponse`](oauth.md#oauth.tokenresponse) | Yes      | The options used to store the token set. |

#### Return

A promise that resolves when the token set has been stored.

### OAuth.PKCEClient#getTokens

Retrieves the stored [TokenSet](oauth.md#oauth.tokenset) for the client. You can use this to initially check whether the authorization flow should be initiated or the user is already logged in and you might have to refresh the access token.

#### Signature

```typescript
getTokens(): Promise<TokenSet | undefined>;
```

#### Example

```typescript
const tokenSet = await client.getTokens();
```

#### Return

A promise that resolves when the token set has been retrieved.

### OAuth.PKCEClient#removeTokens

Removes the stored [TokenSet](oauth.md#oauth.tokenset) for the client. Raycast automatically shows a logout preference that removes the token set. Use this method only if you need to provide an additional logout option in your extension or you want to remove the token set because of a migration.

#### Signature

```typescript
removeTokens(): Promise<void>;
```

#### Example

```typescript
await client.removeTokens();
```

#### Return

A promise that resolves when the token set has been removed.

## Types

### OAuth.PKCEClient.Options

The options for creating a new [PKCEClient](oauth.md#oauth.pkceclient).

#### Properties

| Property                                         | Description                                                                                                                                                                                                                             | Type                                                                    |
| ------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| providerName<mark style="color:red;">\*</mark>   | The name of the provider, displayed in the OAuth overlay.                                                                                                                                                                               | `string`                                                                |
| redirectMethod<mark style="color:red;">\*</mark> | The redirect method for the OAuth flow. Make sure to set this to the correct method for the provider, see [OAuth.RedirectMethod](oauth.md#oauth.redirectmethod) for more information.                                                   | [`OAuth.RedirectMethod`](oauth.md#oauth.redirectmethod)                 |
| description                                      | An optional description, shown in the OAuth overlay. You can use this to customize the message for the end user, for example for handling scope changes or other migrations. Raycast shows a default message if this is not configured. | `string`                                                                |
| providerIcon                                     | An icon displayed in the OAuth overlay. Make sure to provide at least a size of 64x64 pixels.                                                                                                                                           | [`Image.ImageLike`](user-interface/icons-and-images.md#image.imagelike) |
| providerId                                       | An optional ID for associating the client with a provider. Only set this if you use multiple different clients in your extension.                                                                                                       | `string`                                                                |

### OAuth.RedirectMethod

Defines the supported redirect methods for the OAuth flow. You can choose between web and app-scheme redirect methods, depending on what the provider requires when setting up the OAuth app. For examples on what redirect URI you need to configure, see the docs for each method.

#### Enumeration members

| Name   | Value                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Web    | <p>Use this type for a redirect back to the Raycast website, which will then open the extension. In the OAuth app, configure <code>https://raycast.com/redirect?packageName=Extension</code><br>(This is a static redirect URL for all extensions.)<br>If the provider does not accept query parameters in redirect URLs, you can alternatively use <code>https://raycast.com/redirect/extension</code> and then customize the <a href="oauth.md#oauth.authorizationrequest">AuthorizationRequest</a> via its <code>extraParameters</code> property. For example add: <code>extraParameters: { "redirect_uri": "https://raycast.com/redirect/extension" }</code></p> |
| App    | Use this type for an app-scheme based redirect that directly opens Raycast. In the OAuth app, configure `raycast://oauth?package_name=Extension`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| AppURI | <p>Use this type for a URI-style app scheme that directly opens Raycast. In the OAuth app, configure <code>com.raycast:/oauth?package_name=Extension</code><br>(Note the single slash – Google, for example, would require this flavor for an OAuth app where the Bundle ID is <code>com.raycast</code>)</p>                                                                                                                                                                                                                                                                                                                                                         |

### OAuth.AuthorizationRequestOptions

The options for an authorization request via [authorizationRequest](oauth.md#oauth.authorizationrequest).

| Property                                   | Description                                                                                                                                                                                                                                                                            | Type                     |
| ------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------ |
| clientId<mark style="color:red;">\*</mark> | The client ID of the configured OAuth app.                                                                                                                                                                                                                                             | `string`                 |
| endpoint<mark style="color:red;">\*</mark> | The URL to the authorization endpoint for the OAuth provider.                                                                                                                                                                                                                          | `string`                 |
| scope<mark style="color:red;">\*</mark>    | A space-delimited list of scopes for identifying the resources to access on the user's behalf. The scopes are typically shown to the user on the provider's consent screen in the browser. Note that some providers require the same scopes be configured in the registered OAuth app. | `string`                 |
| extraParameters                            | Optional additional parameters for the authorization request. Note that some providers require additional parameters, for example to obtain long-lived refresh tokens.                                                                                                                 | `Record<string, string>` |

### OAuth.AuthorizationRequestURLParams

Values of [AuthorizationRequest](oauth.md#oauth.authorizationrequest). The PKCE client automatically generates the values for you and returns them for [authorizationRequest](oauth.md#oauth.authorizationrequest)

| Property                                        | Description                      | Type     |
| ----------------------------------------------- | -------------------------------- | -------- |
| codeChallenge<mark style="color:red;">\*</mark> | The PKCE `code_challenge` value. | `string` |
| codeVerifier<mark style="color:red;">\*</mark>  | The PKCE `code_verifier` value.  | `string` |
| redirectURI<mark style="color:red;">\*</mark>   | The OAuth `redirect_uri` value.  | `string` |
| state<mark style="color:red;">\*</mark>         | The OAuth `state` value.         | `string` |

### OAuth.AuthorizationRequest

The request returned by [authorizationRequest](oauth.md#oauth.authorizationrequest). Can be used as direct input to [authorize](oauth.md#oauth.pkceclient-authorize), or to extract parameters for constructing a custom URL in [AuthorizationOptions](oauth.md#oauth.authorizationoptions).

| Property                                        | Description                            | Type           |
| ----------------------------------------------- | -------------------------------------- | -------------- |
| codeChallenge<mark style="color:red;">\*</mark> | The PKCE `code_challenge` value.       | `string`       |
| codeVerifier<mark style="color:red;">\*</mark>  | The PKCE `code_verifier` value.        | `string`       |
| redirectURI<mark style="color:red;">\*</mark>   | The OAuth `redirect_uri` value.        | `string`       |
| state<mark style="color:red;">\*</mark>         | The OAuth `state` value.               | `string`       |
| toURL<mark style="color:red;">\*</mark>         | Constructs the full authorization URL. | `() => string` |

#### Methods

| Name    | Type           | Description                            |
| ------- | -------------- | -------------------------------------- |
| toURL() | `() => string` | Constructs the full authorization URL. |

### OAuth.AuthorizationOptions

Options for customizing [authorize](oauth.md#oauth.pkceclient-authorize). You can use values from [AuthorizationRequest](oauth.md#oauth.authorizationrequest) to build your own URL.

| Property                              | Description                 | Type     |
| ------------------------------------- | --------------------------- | -------- |
| url<mark style="color:red;">\*</mark> | The full authorization URL. | `string` |

### OAuth.AuthorizationResponse

The response returned by [authorize](oauth.md#oauth.pkceclient-authorize), containing the authorization code after the provider redirect. You can then exchange the authorization code for an access token using the provider's token endpoint.

| Property                                            | Description                                     | Type     |
| --------------------------------------------------- | ----------------------------------------------- | -------- |
| authorizationCode<mark style="color:red;">\*</mark> | The authorization code from the OAuth provider. | `string` |

### OAuth.TokenSet

Describes the TokenSet created from an OAuth provider's token response. The `accessToken` is the only required parameter but typically OAuth providers also return a refresh token, an expires value, and the scope. Securely store a token set via [setTokens](oauth.md#oauth.pkceclient-settokens) and retrieve it via [getTokens](oauth.md#oauth.pkceclient-gettokens).

| Property                                      | Description                                                                                                                                                                                                                                                                      | Type            |
| --------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------- |
| accessToken<mark style="color:red;">\*</mark> | The access token returned by an OAuth token request.                                                                                                                                                                                                                             | `string`        |
| updatedAt<mark style="color:red;">\*</mark>   | The date when the token set was stored via [OAuth.PKCEClient.setTokens](oauth.md#oauth.pkceclient).                                                                                                                                                                              | `Date`          |
| isExpired<mark style="color:red;">\*</mark>   | A convenience method for checking whether the access token has expired. The method factors in some seconds of "buffer", so it returns true a couple of seconds before the actual expiration time. This requires the `expiresIn` parameter to be set.                             | `() => boolean` |
| expiresIn                                     | An optional expires value (in seconds) returned by an OAuth token request.                                                                                                                                                                                                       | `number`        |
| idToken                                       | An optional id token returned by an identity request (e.g. /me, Open ID Connect).                                                                                                                                                                                                | `string`        |
| refreshToken                                  | An optional refresh token returned by an OAuth token request.                                                                                                                                                                                                                    | `string`        |
| scope                                         | The optional space-delimited list of scopes returned by an OAuth token request. You can use this to compare the currently stored access scopes against new access scopes the extension might require in a future version, and then ask the user to re-authorize with new scopes. | `string`        |

#### Methods

| Name        | Type            | Description                                                                                                                                                                                                                                          |
| ----------- | --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| isExpired() | `() => boolean` | A convenience method for checking whether the access token has expired. The method factors in some seconds of "buffer", so it returns true a couple of seconds before the actual expiration time. This requires the `expiresIn` parameter to be set. |

### OAuth.TokenSetOptions

Options for a [TokenSet](oauth.md#oauth.tokenset) to store via [setTokens](oauth.md#oauth.pkceclient-settokens).

| Property                                      | Description                                                                       | Type     |
| --------------------------------------------- | --------------------------------------------------------------------------------- | -------- |
| accessToken<mark style="color:red;">\*</mark> | The access token returned by an OAuth token request.                              | `string` |
| expiresIn                                     | An optional expires value (in seconds) returned by an OAuth token request.        | `number` |
| idToken                                       | An optional id token returned by an identity request (e.g. /me, Open ID Connect). | `string` |
| refreshToken                                  | An optional refresh token returned by an OAuth token request.                     | `string` |
| scope                                         | The optional scope value returned by an OAuth token request.                      | `string` |

### OAuth.TokenResponse

Defines the standard JSON response for an OAuth token request. The response can be directly used to store a [TokenSet](oauth.md#oauth.tokenset) via [setTokens](oauth.md#oauth.pkceclient-settokens).

| Property                                        | Description                                                                               | Type     |
| ----------------------------------------------- | ----------------------------------------------------------------------------------------- | -------- |
| access\_token<mark style="color:red;">\*</mark> | The `access_token` value returned by an OAuth token request.                              | `string` |
| expires\_in                                     | An optional `expires_in` value (in seconds) returned by an OAuth token request.           | `number` |
| id\_token                                       | An optional `id_token` value returned by an identity request (e.g. /me, Open ID Connect). | `string` |
| refresh\_token                                  | An optional `refresh_token` value returned by an OAuth token request.                     | `string` |
| scope                                           | The optional `scope` value returned by an OAuth token request.                            | `string` |
