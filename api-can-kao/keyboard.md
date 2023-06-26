# Keyboard

Keyboard API 可帮助您通过键盘快捷键执行操作。快捷键可帮助用户无需触摸鼠标即可使用您的命令。

## 类型

### Keyboard.Shortcut

键盘快捷键由一个或多个修饰键（command、control等）和一个等效键（字符或特殊键）定义。

有关支持的值，请参阅 [KeyModifier](https://developers.raycast.com/api-reference/keyboard#keyboard.keymodifier) 和 [KeyEquivalent](https://developers.raycast.com/api-reference/keyboard#keyboard.keyequivalent)。

#### 例子

```typescript
import { Action, ActionPanel, Detail } from "@raycast/api";

export default function Command() {
  return (
    <Detail
      markdown="Let's play some games 👾"
      actions={
        <ActionPanel title="Game controls">
          <Action title="Up" shortcut={{ modifiers: ["opt"], key: "arrowUp" }} onAction={() => console.log("Go up")} />
          <Action
            title="Down"
            shortcut={{ modifiers: ["opt"], key: "arrowDown" }}
            onAction={() => console.log("Go down")}
          />
          <Action
            title="Left"
            shortcut={{ modifiers: ["opt"], key: "arrowLeft" }}
            onAction={() => console.log("Go left")}
          />
          <Action
            title="Right"
            shortcut={{ modifiers: ["opt"], key: "arrowRight" }}
            onAction={() => console.log("Go right")}
          />
        </ActionPanel>
      }
    />
  );
}
```

#### 属性

| 名称                                          | 描述         | 类型                                                             |
| ------------------------------------------- | ---------- | -------------------------------------------------------------- |
| key<mark style="color:red;">\*</mark>       | 键盘快捷键的键。   | [`Keyboard.KeyEquivalent`](keyboard.md#keyboard.keyequivalent) |
| modifiers<mark style="color:red;">\*</mark> | 键盘快捷键的修饰键。 | [`Keyboard.KeyModifier`](keyboard.md#keyboard.keymodifier)`[]` |

### Keyboard.KeyEquivalent

```typescript
KeyEquivalent: "a" |
  "b" |
  "c" |
  "d" |
  "e" |
  "f" |
  "g" |
  "h" |
  "i" |
  "j" |
  "k" |
  "l" |
  "m" |
  "n" |
  "o" |
  "p" |
  "q" |
  "r" |
  "s" |
  "t" |
  "u" |
  "v" |
  "w" |
  "x" |
  "y" |
  "z" |
  "0" |
  "1" |
  "2" |
  "3" |
  "4" |
  "5" |
  "6" |
  "7" |
  "8" |
  "9" |
  "." |
  "," |
  ";" |
  "=" |
  "+" |
  "-" |
  "[" |
  "]" |
  "{" |
  "}" |
  "«" |
  "»" |
  "(" |
  ")" |
  "/" |
  "\\" |
  "'" |
  "`" |
  "§" |
  "^" |
  "@" |
  "$" |
  "return" |
  "delete" |
  "deleteForward" |
  "tab" |
  "arrowUp" |
  "arrowDown" |
  "arrowLeft" |
  "arrowRight" |
  "pageUp" |
  "pageDown" |
  "home" |
  "end" |
  "space" |
  "escape" |
  "enter" |
  "backspace";
```

[快捷键](https://developers.raycast.com/api-reference/keyboard#keyboard.shortcut) 的等效键

### Keyboard.KeyModifier

```typescript
KeyModifier: "cmd" | "ctrl" | "opt" | "shift";
```

[快捷键](https://developers.raycast.com/api-reference/keyboard#keyboard.shortcut) 修饰符
