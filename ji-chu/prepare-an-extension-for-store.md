---
description: 了解如何快速通过审核流程
---

# 过审一个扩展

您可以在此处找到需要遵循的要求和指南，以便在您的扩展程序在商店中可用之前通过审核。请仔细阅读，因为这将为您和我们节省时间。本文档不断发展，因此请不时访问它。

## 元数据和配置

* 需要仔细检查的事项 `package.json`
  * 确保您在 `author` 字段中使用 **Raycast** 帐户用户名
  * 确保您在 `license` 字段中使用 `MIT`
  * 确保您使用的是最新的 Raycast API 版本
* 请使用 `npm` 安装依赖项，并在 PR 中包含 `package-lock.json`。在构建和发布扩展时，我们在持续集成 (CI) 环境中使用 `npm`，因此，通过提供 `package-lock.json` 文件，我们确保服务器上的依赖项与本地依赖项的版本匹配。
* Please check the terms of service of third-party services that your extension uses. If your extension doesn't comply with their terms, include a warning in your extension's README. The warning should be similar to:
*   请检查您的扩展使用的第三方服务的服务条款。如果您的扩展程序不符合其条款，请在扩展程序的自述文件中添加警告。警告应该类似于：

    > 警告：此扩展不符合\[服务名称]的服务条款。使用风险自负。
* 在提交扩展以供审核之前，请确保可以在本地使用 `npm run build` **运行发行版构建**。这将执行额外的类型检查并创建优化的构建。在 Raycast 中打开扩展，检查发行版构建是否一切正常。此外，您可以通过运行 `npm run lint` 执行 `linting` 和代码风格检查。 （这些检查稍后也将通过自动 GitHub 检查运行。）

## 扩展和命令命名

* 扩展和命令标题应遵循 [Apple 风格](https://help.apple.com/applestyleguide/#/apsgb744e4a3?sub=apdca93e113f1d64) 指南约定
  * ✅ `Google Workplace`, `Doppler Share Secrets`, `Search in Database`
  * ❌ `Hacker news`, `my issues`
  * 🤔 对于规范地使用小写字母书写的名称和商标，可以使用小写字母。例如 `iOS` , `macOS` , `npm`
* **扩展标题**
  * 它仅在 Store 和首选项中使用
  * 让人们在 Store 中看到它时很容易理解它的作用
    * ✅ `Emoji Search`, `Airport - Discover Testflight Apps`, `Hacker News`
    * ❌ `Converter`, `Images`, `Code Review`, `Utils`
    * 🤔 在某些情况下，您可以向标题添加其他信息，类似于上面的机场示例。仅当它添加上下语境时才可以这样做。
    * 💡 您可以使用更具创意的标题来将您的扩展程序与其他名称相似的扩展程序区分开来。
  * 旨在使用名词而不是动词
    * `Emoji Search` 比 `Search Emoji` 要好
  * 当您的扩展不提供大量命令时，避免使用扩展的通用名称
    * 例如。如果您的扩展程序只能搜索 Notion 中的页面，请将其命名为 `Notion Search` 而不仅仅是 `Notion`。这将帮助用户了解您的扩展。如果您的扩展涵盖了很多功能，那么可以使用像 Notion 这样的通用名称。比如：[Gitlab](https://www.raycast.com/tonka3000/gitlab)。
    * **经验法则：**如果您的扩展只有一个命令，您可能需要将扩展​​命名为与该命令的功能接近的名称。 例如  [Visual Studio Code Recent Projects](https://www.raycast.com/thomas/visual-studio-code) 而不是 `Visual Studio Code`.
* **扩展描述**
  * 用一句话来说，你的扩展程序是做什么的？它会显示在商店的扩展列表中。要保持简短和描述性。查看 Store 中其他已批准的扩展程序的做法以获得灵感。
* **命令标题**
  * 通常使用 `<verb> <noun>` 结构或者仅仅是 `<noun>`
  * 最好的方法是查看 Raycast 中其他命令的命名方式来获取灵感
    * ✅ `Search Recent Projects`, `Translate`, `Open Issues`, `Create Task`
    * ❌ `Recent Projects Search`, `Translation`, `New Task`
  * 避免大费周章
    * ✅ `Search Emoji`, `Create Issue`
    * ❌ `Search an Emoji`, `Create an Issue`
  * 避免只给它一个服务名称，更具体地说明命令的作用
    * ✅ `Search Packages`
    * ❌ `NPM`
* **Command 子标题**
  * 使用子标题为您的命令添加上下语境。通常，它是您集成的应用程序或服务名称。使命令名称更加轻量级，并且无需在命令标题中指定服务名称。
  * 子标题已建立索引，因此您仍然可以使用子标题和标题进行搜索：在上面的示例中，`xcode recent projects` 将返回 `Search Recent Projects`。
  * 不要使用子标题作为命令的描述
    * ❌ `Quickly open Xcode recent projects`
  * 如果子标题不能添加上下语境，请勿使用子标题。通常，这是单命令扩展的情况。
    * There is no need for a subtitle for the `Search Emoji` command since it's self-explanatory
    * `Search Emoji` 命令不需要子标题，因为它是不言自明的
    * **经验法则：**如果您的子标题几乎与命令标题重复，您可能不需要它

![不错的子标题示例](../.gitbook/assets/good-subtitle.png)

## 扩展图标

{% hint style="info" %}
我们制作了一个新的图标生成器工具来简化为您的扩展创建图标的过程。你可以在 [这里](https://icon.ray.so/) 找到它。
{% endhint %}

* 商店中发布的扩展需要是 `png` 格式的 512x512px 图标
* 图标在浅色和深色主题中都应该看起来不错（您可以在 Raycast Preferences → Appearance 中切换主题）
* 如果您有单独的浅色和深色图标，请参阅 `package.json` [mainfest](https://developers.raycast.com/information/manifest#extension-properties) 文档了解如何配置它们
* 使用默认 Raycast 图标的扩展将被拒绝
* 这个 [图标模板](https://www.figma.com/community/file/1030764827259035122/Extensions-Icon-Template) 可以帮助您制作和导出合适的图标
* 确保删除未使用的资源和图标
* 💡 如果您觉得设计图标不适合您，请向 [社区](https://raycast.com/community) 寻求帮助（#extensions 频道）

## 如果需要其他配置，请提供 README

* 如果您的扩展需要额外的设置，例如获取 API 访问令牌、在其他应用程序中启用某些首选项，或者具有重要的用例，请在扩展的根文件夹中提供 README 文件。提供自述文件后，用户将在首选项登录屏幕上看到 “About This Extension” 按钮。
* 支持 README 媒体：将所有链接的媒体文件放入扩展目录内的顶级媒体文件夹中。 （这与扩展程序中运行时所需的资源不同：它们位于资源文件夹内并将捆绑到您的扩展程序中。）

![链接到 README 文件的按钮](../.gitbook/assets/required-preference.png)

## 类别

![扩展详细信息屏幕上显示的类别](../.gitbook/assets/categories-focus.png)

* 所有扩展都应至少以一个类别发布
* 类别区分大小写，应遵循 [标题大小写](https://titlecaseconverter.com/rules/) 约定
* 在 `package.json` [mainfest](https://developers.raycast.com/information/manifest) 文件中添加类别，或在使用 **Create Extension** 命令创建新扩展时选择类别

### 所有类别

| 类别    | 示例                                                                                                                   |
| ----- | -------------------------------------------------------------------------------------------------------------------- |
| 程序    | [Cleanshot X](https://www.raycast.com/Aayush9029/cleanshotx) – 捕获并录制您的屏幕                                             |
| 交流    | [Slack Status](https://www.raycast.com/petr/slack-status) – 快速更改您的 Slack 状态。                                         |
| 数据    | [Random Data Generator](https://www.raycast.com/loris/random) – 使用 Faker 库生成随机数据。                                    |
| 文档    | [Tailwind CSS Documentation](https://www.raycast.com/vimtor/tailwindcss) – 快速搜索 Tailwind CSS 文档并在浏览器中打开。             |
| 设计工具  | [Figma File Search](https://www.raycast.com/michaelschultz/figma-files-raycast-extension) – 列出 Figma 文件，允许您搜索和导航到它们。 |
| 开发者工具 | [Brew](https://www.raycast.com/nhojb/brew) – 搜索并安装 Homebrew 包。                                                       |
| 金融    | [Coinbase Pro](https://www.raycast.com/farisaziz12/coinbase-pro) – 查看您的 Coinbase Pro 投资组合。                           |
| 兴趣    | [8 Ball](https://www.raycast.com/rocksack/8-ball) – 返回类似 8 球的问题答案。                                                   |
| 媒体    | [Unsplash](https://www.raycast.com/eggsy/unsplash) – 在 Unsplash 上搜索图像或集合，下载、复制它们或将它们设置为壁纸，而无需离开 Raycast。             |
| 新闻    | [Hacker News](https://www.raycast.com/thomas/hacker-news) – 阅读 Hacker News 的最新报道。                                    |
| 生产力   | [Todoist](https://www.raycast.com/thomaslombart/todoist) – 检查您的Todoist任务并快速创建新任务。                                    |
| 安全    | [1Password 7](https://www.raycast.com/khasbilegt/1password7) – 从 Raycast 搜索、打开或编辑您的 1Password 7 密码。                  |
| 系统    | [Coffee](https://www.raycast.com/mooxl/coffee) – 阻止 Mac 上的睡眠功能。                                                      |
| 网页    | [Wikipedia](https://www.raycast.com/vimtor/wikipedia) – 搜索维基百科文章并查看它们。                                               |
| 其他    | 如果您认为您的扩展程序不符合上述任何类别，则可以使用。                                                                                          |

## 屏幕截图

![带有屏幕截图元数据的扩展示例](../.gitbook/assets/hn-store.png)

* 屏幕截图显示在扩展详细信息屏幕的元数据中，用户可以在安装之前单击并浏览它们，以更详细地了解您的扩展的功能
* 您最多可以添加六个屏幕截图。我们建议至少添加三个，以便您的扩展程序详细信息屏幕看起来很漂亮。

### 添加屏幕截图

在 Raycast 1.37.0+ 中，我们让您可以轻松地为您的扩展程序拍摄漂亮的像素完美的屏幕截图。

#### 如何使用它?

1. 在高级首选项中设置 Window Capture (快捷键 e.g.: `⌘⇧⌥+M`)
2. 打开命令
3. 按下快捷键，记得勾选 `Save to Metadata`

{% hint style="info" %}
该工具将使用您当前的背景。选择具有良好对比度的背景图像，以便清晰轻松地查看您制作的应用程序和扩展程序。

&#x20;您可以使用 [Raycast 壁纸](https://www.raycast.com/wallpapers) 使您的背景看起来很漂亮
{% endhint %}

### 规格

<table><thead><tr><th width="162">屏幕截图大小</th><th width="194">纵横比</th><th width="212">格式</th><th>深色模式支持</th></tr></thead><tbody><tr><td>2000 x 1250 像素 (横向模式)</td><td>16:10</td><td>PNG</td><td>否</td></tr></tbody></table>

### 要做的和不要做的

* ✅ 选择对比度良好的背景，这样可以清晰轻松地查看您制作的应用程序和扩展程序
* ✅ 选择信息最丰富的命令来展示您的扩展程序的功能 - 专注于为用户提供尽可能多的详细信息
* ❌ 不要为不同的屏幕截图使用多个背景 - 保持一致并在所有屏幕截图中使用相同的背景
* ❌ 不要在屏幕截图中共享敏感数据 - 这些数据将在商店以及 GitHub 上的扩展存储库中可见
* ❌ 避免在不同主题（浅色和深色）中使用屏幕截图，除非是为了演示您的扩展程序的功能

## 版本历史

![应用程序中显示的 CHANGELOG.md 文件](../.gitbook/assets/version-history.png)

* 通过扩展元数据中的 `CHANGELOG.md` 文件，让用户可以更轻松地准确查看扩展的每个版本之间所做的更改
  * 要将版本历史记录添加到您的扩展，请将 `CHANGELOG.md` 文件添加到您的扩展的根文件夹
* 查看带有 [屏幕截图和更改日志文件](https://developers.raycast.com/basics/prepare-an-extension-for-store#adding-screenshots) 的扩展文件结构
* 对于每次更改，请提供有关最新更新的清晰的描述性信息，提供标题作为 h2 标头，后跟日期时间戳 YYYY-MM-DD
  * 确保您的更改标题位于方括号内
  * 用连字符分隔标题和日期 `-` 以及连字符两侧的空格
* 以下是遵循正确格式的变更日志示例

```markdown
# Brew Changelog

## [Added a bunch of new feedback] - 2022-01-17

- Improve reliability of `outdated` command
- Add action to copy formula/cask name
- Add cask name & tap to cask details
- Add Toast action to cancel current action
- Add Toast action to copy error log after failure

## [New Additions] - 2022-12-13

- Add greedy upgrade preference
- Add `upgrade` command

## [Fixes & Bits] - 2021-11-19

- Improve discovery of brew prefix
- Update Cask.installed correctly after installation
- Fix installed state after uninstalling search result
- Fix cache check after installing/uninstalling cask
- Add uninstall action to outdated action panel

## [New Commands] - 2021-11-04

Add support for searching and managing casks

## [Added Brew] - 2021-10-26

Initial version code
```

![raycast.com/store 上的扩展版本历史记录](https://user-images.githubusercontent.com/17166544/159987128-1e9f22a6-506b-4edd-bb40-e121bfdc46f8.png)

{% hint style="info" %}
您可以使用  [保留变更日志](https://keepachangelog.com/en/1.0.0/) 来帮助您正确格式化变更日志
{% endhint %}

## 为现有扩展做出贡献与创建新扩展

* **当您为现有扩展做出贡献而不是创建新扩展时**
  * 您想对已经发布的扩展进行一些小的改进，例如额外的操作、新的偏好、用户体验改进等。通常，这是一个不显着的变化。
  * 您想要添加一个简单的命令来补充现有扩展而不更改扩展标题或描述，例如您想为 Spotify 添加“喜欢当前曲目”命令。当已经有 [Spotify Controls](https://www.raycast.com/thomas/spotify-controls) 扩展时，仅仅为此创建一个全新的扩展是没有意义的。
  * **重要提示：**如果您的更改很重要，那么在投入大量时间之前联系扩展的作者是有意义的。未经作者签字，我们无法合并重要的贡献。
* **当您考虑创建一个新扩展而不是为现有扩展做出贡献时**
  * 对现有扩展的更改将是重大的，并且可能会破坏其他人的工作流程。如果您想继续合作路径，请咨询作者。
  * 您的扩展提供与相同服务的集成，但具有不同的配置，例如一个扩展可以是“GitHub Cloud”，另一个是“GitHub Enterprise”。一个扩展可以是“Spotify Controls”，并且仅使用 AppleScript 来播放/暂停歌曲，而另一个扩展可以通过 API 提供更深入的集成，并需要访问令牌设置。没有理由尝试将所有内容合并在一起，因为这只会使事情变得更加复杂。
* **多个简单扩展与一个大型扩展**
  * 如果您的扩展程序独立工作并为应用商店带来了新内容，则可以创建一个新扩展程序，而不是向现有扩展程序添加命令。例如。一个扩展可能是“GitHub 存储库搜索”，另一个扩展可能是“GitHub 问题搜索”。将与一项服务连接的扩展群合并为一个大型扩展不应成为目标。然而，如果作者决定这样做，将两个扩展合并到一个扩展下也是可以接受的。

## 二进制依赖项和附加配置

* 避免要求用户执行额外的下载，并尝试通过扩展尽可能实现自动化，特别是如果您的目标用户是非开发人员。请参阅 [Speedtest](https://github.com/raycast/extensions/pull/302) 扩展，该扩展在后台下载 CLI，然后在后台使用它。
* 如果您最终在后台下载可执行二进制文件，请确保它是从您无权访问的服务器完成的。否则，我们无法保证您在审核后不会用恶意代码替换二进制文件。例如。从[ install.speedtest.net](http://install.speedtest.net/) 下载 `speedtest-cli` 是可以接受的，但从某些自定义 AWS 服务器执行此操作会导致拒绝。建议建议通过哈希添加额外的完整性检查。
* 不要在来源不可用或不清楚它们是如何构建的地方捆绑不透明的二进制文件。
* 不要在扩展中捆绑大量的二进制依赖项 - 这会导致扩展下载大小增加。
* **与二进制文件交互的示例**
  * ✅ 调用已知的系统二进制文件
  * ✅ 从受信任位置下载或安装的二进制文件，并通过哈希值进行额外的完整性检查（即验证下载的二进制文件是否确实与预期的二进制文件匹配）
  * ✅ 从 npm 包中提取的二进制文件并复制到资产中，并具有可追踪的源代码如何构建二进制文件；注意：我们尚未集成用于复制和比较文件的 CI 操作；同时，请 Raycast 团队的成员为您添加二进制文件
  * ❌ 任何具有不可用源或不清楚构建的二进制文件都已添加到资产文件夹中

## 钥匙串访问

* 出于安全考虑，请求钥匙串访问的扩展将被拒绝。如果您无法解决此限制，请通过 [Slack](https://raycast.com/community) 或通过 `feedback@raycast.com` 与我们联系。

## UI/UX 指南

### 首选项

![打开命令时将显示所需的首选项](../.gitbook/assets/required-preferences-2.png)

* Use the [preferences API](https://developers.raycast.com/api-reference/preferences) to let your users configure your extension or for providing credentials like API tokens
  * When using `required: true`, Raycast will ask the user to set preferences before continuing with an extension. See the example [here](https://github.com/raycast/extensions/blob/main/extensions/gitlab/package.json#L150).
* You should not build separate commands for configuring your extension. If you miss some API to achieve the preferences setup you want, please file a [GitHub issue](https://github.com/raycast/extensions/issues) with a feature request.

### Action Panel

![Raycast Action Panel component](../.gitbook/assets/action-panel.png)

* Actions in the action panel should also follow the **Title Case** naming convention
  * ✅ `Open in Browser`, `Copy to Clipboard`
  * ❌ `Copy url`, `set project`, `Set priority`
* Provide icons for actions if there are other actions with icons in the list
  * Avoid having a list of actions where some have icons and some don't
* Add ellipses `…` for actions that will have a submenu. Don't repeat parent the action name in the submenu
  * ✅ `Set Priority…` and submenu would have `Low`, `Medium`, `High`
  * ❌ `Set Priority` and submenu would have `Set Priority Low`, `Set Priority Medium`, etc

### Navigation

* Use the [Navigation API](https://developers.raycast.com/api-reference/user-interface/navigation) for pushing new screens. This will ensure that a user can navigate within your extension the same way as in the rest of the application.
* Avoid introducing your own navigation stack. Extensions that just replace the view's content when it's expected to push a new screen will be rejected.

### Empty States

* When you update lists with an empty array of elements, the "No results" view will be shown. Avoid introducing your own UI to achieve a similar effect (e.g. showing list item).
  * **Known issue:** Sometimes, there is nothing you can show when the search query is empty, and an extension shows "No results" when you open it (often in search commands). We have plans to provide an API that would improve that experience. In the meantime, you might want to consider introducing some sections that could be helpful in an empty state – e.g. suggestions or recently visited items.
* **Common mistake** – "flickering empty state view" on start
  * If you try rendering an empty list before real data arrives (e.g. from the network or disk), you might see a flickering "No results" view when opening the extension. To prevent this, make sure not to return an empty list of items before you get the data you want to display. In the meantime, you can show the loading indicator. See [this example](https://developers.raycast.com/information/best-practices#show-loading-indicator).

### Navigation Title

* Don't change the `navigationTitle` in the root command - it will be automatically set to the command name. Use `navigationTitle` only in nested screens to provide additional context. See [Slack Status extension](https://github.com/raycast/extensions/blob/020f2232aa5579b5c63b4b3c08d23ad719bce1f8/extensions/slack-status/src/setStatusForm.tsx#L95) as an example of correct usage of the `navigationTitle` property.
* Avoid long titles. If you can't predict how long the navigation title string will be, consider using something else. E.g. in the Jira extension, we use the issue key instead of the issue title to keep it short.
* Avoid updating the navigation title multiple times on one screen depending on some state. If you find yourself doing it, there is a high chance you are misusing it.

### Placeholders in Text Fields

* For a better visual experience, add placeholders in text field and text area components. This includes preferences.
* Don't leave the search bar without a placeholder

### Analytics

* It’s not allowed to include external analytics in extensions. Later on, we will add support to give developers more insights into how their extension is being used.

### Localization / Language

* At the moment, Raycast doesn't support localization and only supports US English. Therefore, please avoid introducing your custom way to localize your extension. If the locale might affect functionality (e.g. using the correct unit of measurement), please use the preferences API.
* Use US English spelling (not British)
