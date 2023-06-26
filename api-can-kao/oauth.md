# OAuth

## Prerequisites

A Raycast extension can use OAuth for authorizing access to a provider's resources on the user's behalf. Since Raycast is a desktop app and the extensions are considered "public", we only support the [PKCE flow](https://datatracker.ietf.org/doc/html/rfc7636) (Proof Key for Code Exchange, pronounced “pixy”). This flow is the official recommendation for native clients that cannot keep a client secret. With PKCE, the client dynamically creates a secret and uses the secret again during code exchange, ensuring that only the client that performed the initial request can exchange the code for the access token (”proof of possession”).

Before considering OAuth, first check if your provider supports PKCE. You can usually find this in the provider's OAuth docs by looking for `code_challenge` and `code_verifier` parameters. Providers such as Google, Twitter, GitLab, Spotify, Zoom, Asana or Dropbox are all PKCE-ready. If you find that your provider does not yet support PKCE, you can often use other forms of authorization such as personal access tokens (usable with Raycast password preferences), or open-source PKCE proxies that "adapt" an OAuth flow to be PKCE-compatible (you would need to operate your own backend service in this case, so this is only recommended for very advanced use cases.)

## OAuth Flow

![](../.gitbook/assets/oauth-overlay-twitter.png)

The OAuth flow from an extension looks like this:

1. The extension initiates the OAuth flow and starts authorization
2. Raycast shows the OAuth overlay ("Connect to provider…")
3. The user opens the provider's consent page in the web browser
4. After the user consent, the provider redirects back to Raycast
5. Raycast opens the extension where authorization is completed

When the flow is complete, the extension has received an access token from the provider and can perform API calls. The API provides functions for securely storing and retrieving token sets, so that an extension can check whether the user is already logged in and whether an expired access token needs to be refreshed. Raycast also automatically shows a logout preference.

![](../.gitbook/assets/oauth-overlay-twitter-success.png)

## OAuth App

You first need to register a new OAuth app with your provider. This is usually done in the provider's developer portal. After registering, you will receive a client ID. You also need to configure a redirect URI, see the next section.

Note: Make sure to choose an app type that supports PKCE. Some providers still show you a client secret, which you don't need and should _not_ hardcode in the extension, or support PKCE only for certain types such as "desktop", "native" or "mobile" app types.

## Authorizing

An extension can initiate the OAuth flow and authorize by using the methods on [OAuth.PKCEClient](oauth.md#oauth.pkceclient).

You can create a new client and configure it with a provider name, icon and description that will be shown in the OAuth overlay. You can also choose between different redirect methods; depending on which method you choose, you need to configure this value as redirect URI in your provider's registered OAuth app. (See the [OAuth.RedirectMethod](oauth.md#oauth.redirectmethod) docs for each method to get concrete examples for supported redirect URI.) If you can choose, use `OAuth.RedirectMethod.Web` and enter `https://raycast.com/redirect?packageName=Extension` (whether you have to add the `?packageName=Extension` depends on the provider).

```typescript
import { OAuth } from "@raycast/api";

const client = new OAuth.PKCEClient({
  redirectMethod: OAuth.RedirectMethod.Web,
  providerName: "Twitter",
  providerIcon: "twitter-logo.png",
  description: "Connect your Twitter account…",
});
```

Next you create an authorization request with the authorization endpoint, client ID, and scope values. You receive all values from your provider's docs and when you register a new OAuth app.

The returned [AuthorizationRequest](oauth.md#oauth.authorizationrequest) contains parameters such as the code challenge, verifier, state and redirect URI as standard OAuth authorization request. You can also customize the authorization URL through [OAuth.AuthorizationOptions](oauth.md#oauth.authorizationoptions) if you need to.

```typescript
const authRequest = await client.authorizationRequest({
  endpoint: "https://twitter.com/i/oauth2/authorize",
  clientId: "YourClientId",
  scope: "tweet.read users.read follows.read",
});
```

To get the authorization code needed for the token exchange, you call [authorize](oauth.md#oauth.pkceclient-authorize) with the request from the previous step. This call shows the Raycast OAuth overlay and provides the user with an option to open the consent page in the web browser. The authorize promise is resolved after the redirect back to Raycast and into the extension:

```typescript
const { authorizationCode } = await client.authorize(authRequest);
```

{% hint style="info" %}
When in development mode, make sure not to trigger auto-reloading (e.g. by saving a file) while you're testing an active OAuth authorization and redirect. This would cause an OAuth state mismatch when you're redirected back into the extension since the client would be reinitialized on reload.
{% endhint %}

Now that you have received the authorization code, you can exchange this code for an access token using your provider's token endpoint. This token exchange (and the following API calls) can be done with your preferred Node HTTP client library. Example using `node-fetch`:

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

## Token Storage

The PKCE client exposes methods for storing, retrieving and deleting token sets. A [TokenSet](oauth.md#oauth.tokenset) contains an access token and typically also a refresh token, expires value, and the current scope. Since this data is returned by the provider's token endpoint as standard OAuth JSON response, you can directly store the response ([OAuth.TokenResponse](oauth.md#oauth.tokenresponse)) or alternatively use [OAuth.TokenSetOptions](oauth.md#oauth.tokensetoptions):

```typescript
await client.setTokens(tokenResponse);
```

Once the token set is stored, Raycast will automatically show a logout preference for the extension. When the user logs out, the token set gets removed.

The [TokenSet](oauth.md#oauth.tokenset) also enables you to check whether the user is logged in before starting the authorization flow:

```typescript
const tokenSet = await client.getTokens();
```

## Token Refresh

Since access tokens usually expire, an extension should provide a way to refresh the access token, otherwise users would be logged out or see errors. Some providers require you to add an offline scope so that you get a refresh token. (Twitter, for example, needs the scope `offline.access` or it only returns an access token.) A basic refresh flow could look like this:

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

## Examples

We've provided [OAuth example integrations for Google, Twitter, and Dropbox](https://github.com/raycast/extensions/tree/main/examples/api-examples) that demonstrate the entire flow shown above.

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
