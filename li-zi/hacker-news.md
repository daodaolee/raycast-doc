---
description: 此示例演示如何将 RSS 提要显示为列表。
---

# Hacker News

{% hint style="info" %}
该示例的源代码可以在 [这里](https://github.com/raycast/extensions/tree/main/extensions/hacker-news#readme) 找到。您可以在 [此处](https://www.raycast.com/thomas/hacker-news) 安装它。
{% endhint %}

谁不喜欢早上喝着热咖啡阅读一下 [Hacker News](https://news.ycombinator.com) ？在此示例中，我们会创建一个简单的列表，其中包含首页上的热门新闻。

![示例：阅读 Hacker News 的头版](../.gitbook/assets/example-hacker-news.png)

## 加载热门新闻

首先，让我们了解最新的头条新闻。为此，我们使用一个 [RSS feed](https://hnrss.org):&#x20;

```typescript
import { Action, ActionPanel, List, showToast, Toast } from "@raycast/api";
import { useEffect, useState } from "react";
import Parser from "rss-parser";

const parser = new Parser();

interface State {
  items?: Parser.Item[];
  error?: Error;
}

export default function Command() {
  const [state, setState] = useState<State>({});

  useEffect(() => {
    async function fetchStories() {
      try {
        const feed = await parser.parseURL("https://hnrss.org/frontpage?description=0&count=25");
        setState({ items: feed.items });
      } catch (error) {
        setState({
          error: error instanceof Error ? error : new Error("Something went wrong"),
        });
      }
    }

    fetchStories();
  }, []);

  console.log(state.items); // Prints stories

  return <List isLoading={!state.items && !state.error} />;
}
```

分解一下：

* 我们使用第三方依赖项来解析 RSS 提要。
* 我们将 command state 定义为 TypeScript 接口。
* 在命令挂载后，我们使用 React 的 [`useEffect`](https://reactjs.org/docs/hooks-effect.html) 来解析 RSS feed。
* 我们将头条新闻打印到控制台。
* 只要我们加载新闻，我们就会渲染一个列表并显示 loading 指示器。

## 渲染新闻

现在我们从 Hacker News 获得了数据，我们想要展示这些新闻。为此，我们创建一个新的 React 组件和一些渲染新闻的辅助函数：

```typescript
function StoryListItem(props: { item: Parser.Item; index: number }) {
  const icon = getIcon(props.index + 1);
  const points = getPoints(props.item);
  const comments = getComments(props.item);

  return (
    <List.Item
      icon={icon}
      title={props.item.title ?? "No title"}
      subtitle={props.item.creator}
      accessories={[{ text: `👍 ${points}` }, { text: `💬  ${comments}` }]}
    />
  );
}

const iconToEmojiMap = new Map<number, string>([
  [1, "1️⃣"],
  [2, "2️⃣"],
  [3, "3️⃣"],
  [4, "4️⃣"],
  [5, "5️⃣"],
  [6, "6️⃣"],
  [7, "7️⃣"],
  [8, "8️⃣"],
  [9, "9️⃣"],
  [10, "🔟"],
]);

function getIcon(index: number) {
  return iconToEmojiMap.get(index) ?? "⏺";
}

function getPoints(item: Parser.Item) {
  const matches = item.contentSnippet?.match(/(?<=Points:\s*)(\d+)/g);
  return matches?.[0];
}

function getComments(item: Parser.Item) {
  const matches = item.contentSnippet?.match(/(?<=Comments:\s*)(\d+)/g);
  return matches?.[0];
}
```

为了让列表项看起来更漂亮，我们使用一个简单的数字表情符号作为图标，添加创建者的名字作为副标题，添加 points 和评论作为附件标题。现在我们可以渲染 `<StoryListItem>`:

```typescript
export default function Command() {
  const [state, setState] = useState<State>({});

  // ...

  return (
    <List isLoading={!state.items && !state.error}>
      {state.items?.map((item, index) => (
        <StoryListItem key={item.guid} item={item} index={index} />
      ))}
    </List>
  );
}
```

## 添加操作

当我们在列表中选择一个故事时，我们希望能够在浏览器中打开它并复制它的链接，以便我们可以在 Watercooler Slack 频道中共享它。为此，我们创建一个新的 React 组件：

```typescript
function Actions(props: { item: Parser.Item }) {
  return (
    <ActionPanel title={props.item.title}>
      <ActionPanel.Section>
        {props.item.link && <Action.OpenInBrowser url={props.item.link} />}
        {props.item.guid && <Action.OpenInBrowser url={props.item.guid} title="Open Comments in Browser" />}
      </ActionPanel.Section>
      <ActionPanel.Section>
        {props.item.link && (
          <Action.CopyToClipboard
            content={props.item.link}
            title="Copy Link"
            shortcut={{ modifiers: ["cmd"], key: "." }}
          />
        )}
      </ActionPanel.Section>
    </ActionPanel>
  );
}
```

该组件采用一个故事并使用我们所需的操作渲染一个 [`<ActionPanel>`](../api-reference/user-interface/action-panel.md)  。我们将操作添加到 `<StoryListItem>`:

```typescript
function StoryListItem(props: { item: Parser.Item; index: number }) {
  // ...

  return (
    <List.Item
      icon={icon}
      title={props.item.title ?? "No title"}
      subtitle={props.item.creator}
      accessories={[{ text: `👍 ${points}` }, { text: `💬  ${comments}` }]}
      // Wire up actions
      actions={<Actions item={props.item} />}
    />
  );
}
```

## 错误处理

最后，我们希望适当地处理错误以保证流畅的体验。每当我们的网络请求失败时，我们都会显示一个 toast：

```typescript
export default function Command() {
  const [state, setState] = useState<State>({});

  // ...

  if (state.error) {
    showToast({
      style: Toast.Style.Failure,
      title: "Failed loading stories",
      message: state.error.message,
    });
  }

  // ...
}
```

## 结尾

就是这样，您就有了一个可以阅读 Hacker News 首页的扩展。作为后续步骤，您可以添加另一个命令来显示作业源或添加一个操作来复制 Markdown 格式的链接。
