---
description: 此示例展示了如何将列表与表单结合使用。
---

# Todo 列表

{% hint style="info" %}
该示例的源代码可以在 [这里](https://github.com/raycast/extensions/tree/main/examples/todo-list#readme) 找到。
{% endhint %}

没有 Todo 列表的示例什么也不是！让我们在 Raycast 中将其合在一起。此示例将展示如何展示列表、导航到表单，也包括创建新元素以及更新列表。

![示例：一个简单的 Todo 列表](../.gitbook/assets/example-todo-list.png)

## 渲染 todo 列表

让我们从一组 todo 项开始，并简单地将它们渲染为 Raycast 中的列表：

```typescript
import { List } from "@raycast/api";
import { useState } from "react";

interface Todo {
  title: string;
  isCompleted: boolean;
}

export default function Command() {
  const [todos, setTodos] = useState<Todo[]>([
    { title: "Write a todo list extension", isCompleted: false },
    { title: "Explain it to others", isCompleted: false },
  ]);

  return (
    <List>
      {todos.map((todo, index) => (
        <List.Item key={index} title={todo.title} />
      ))}
    </List>
  );
}
```

For this we define a TypeScript interface to describe out Todo with a `title` and a `isCompleted` flag that we use later to complete the todo. We use [React's `useState` hook](https://reactjs.org/docs/hooks-state.html) to create a local state of our todos. This allows us to update them later and the list will get re-rendered. Lastly we render a list of all todos.

为此，我们定义了一个 TypeScript 接口来描述带有 `title` 和 `isCompleted` 标志的 Todo，稍后我们将使用该标志来完成 todo。我们使用 React 的 `useState` 钩子来创建 todo 的本地状态。我们可以稍后更新它们并且列表将被重新渲染。最后我们呈现所有tdo的列表。

## Create a todo

A static list of todos isn't that much fun. Let's create new ones with a form. For this, we create a new React component that renders the form:

```typescript
function CreateTodoForm(props: { onCreate: (todo: Todo) => void }) {
  const { pop } = useNavigation();

  function handleSubmit(values: { title: string }) {
    props.onCreate({ title: values.title, isCompleted: false });
    pop();
  }

  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Create Todo" onSubmit={handleSubmit} />
        </ActionPanel>
      }
    >
      <Form.TextField id="title" title="Title" />
    </Form>
  );
}

function CreateTodoAction(props: { onCreate: (todo: Todo) => void }) {
  return (
    <Action.Push
      icon={Icon.Pencil}
      title="Create Todo"
      shortcut={{ modifiers: ["cmd"], key: "n" }}
      target={<CreateTodoForm onCreate={props.onCreate} />}
    />
  );
}
```

The `<CreateTodoForm>` shows a single text field for the title. When the form is submitted, it calls the `onCreate` callback and closes itself.

![Create todo form](../.gitbook/assets/example-create-todo.png)

To use the action, we add it to the `<List>` component. This makes the action available when the list is empty which is exactly what we want to create our first todo.

```typescript
export default function Command() {
  const [todos, setTodos] = useState<Todo[]>([]);

  function handleCreate(todo: Todo) {
    const newTodos = [...todos, todo];
    setTodos(newTodos);
  }

  return (
    <List
      actions={
        <ActionPanel>
          <CreateTodoAction onCreate={handleCreate} />
        </ActionPanel>
      }
    >
      {todos.map((todo, index) => (
        <List.Item key={index} title={todo.title} />
      ))}
    </List>
  );
}
```

## Complete a todo

Now that we can create new todos, we also want to make sure that we can tick off something on our todo list. For this, we create a `<ToggleTodoAction>` that we assign to the `<List.Item>`:

```typescript
export default function Command() {
  const [todos, setTodos] = useState<Todo[]>([]);

  // ...

  function handleToggle(index: number) {
    const newTodos = [...todos];
    newTodos[index].isCompleted = !newTodos[index].isCompleted;
    setTodos(newTodos);
  }

  return (
    <List
      actions={
        <ActionPanel>
          <CreateTodoAction onCreate={handleCreate} />
        </ActionPanel>
      }
    >
      {todos.map((todo, index) => (
        <List.Item
          key={index}
          icon={todo.isCompleted ? Icon.Checkmark : Icon.Circle}
          title={todo.title}
          actions={
            <ActionPanel>
              <ActionPanel.Section>
                <ToggleTodoAction todo={todo} onToggle={() => handleToggle(index)} />
              </ActionPanel.Section>
            </ActionPanel>
          }
        />
      ))}
    </List>
  );
}

function ToggleTodoAction(props: { todo: Todo; onToggle: () => void }) {
  return (
    <Action
      icon={props.todo.isCompleted ? Icon.Circle : Icon.Checkmark}
      title={props.todo.isCompleted ? "Uncomplete Todo" : "Complete Todo"}
      onAction={props.onToggle}
    />
  );
}
```

In this case we added the `<ToggleTodoAction>` to the list item. By doing this we can use the `index` to toggle the appropriate todo. We also added an icon to our todo that reflects the `isCompleted` state.

## Delete a todo

Similar to toggling a todo, we also add the possibility to delete one. You can follow the same steps and create a new `<DeleteTodoAction>` and add it to the `<List.Item>`.

```typescript
export default function Command() {
  const [todos, setTodos] = useState<Todo[]>([]);

  // ...

  function handleDelete(index: number) {
    const newTodos = [...todos];
    newTodos.splice(index, 1);
    setTodos(newTodos);
  }

  return (
    <List
      actions={
        <ActionPanel>
          <CreateTodoAction onCreate={handleCreate} />
        </ActionPanel>
      }
    >
      {todos.map((todo, index) => (
        <List.Item
          key={index}
          icon={todo.isCompleted ? Icon.Checkmark : Icon.Circle}
          title={todo.title}
          actions={
            <ActionPanel>
              <ActionPanel.Section>
                <ToggleTodoAction todo={todo} onToggle={() => handleToggle(index)} />
              </ActionPanel.Section>
              <ActionPanel.Section>
                <CreateTodoAction onCreate={handleCreate} />
                <DeleteTodoAction onDelete={() => handleDelete(index)} />
              </ActionPanel.Section>
            </ActionPanel>
          }
        />
      ))}
    </List>
  );
}

// ...

function DeleteTodoAction(props: { onDelete: () => void }) {
  return (
    <Action
      icon={Icon.Trash}
      title="Delete Todo"
      shortcut={{ modifiers: ["ctrl"], key: "x" }}
      onAction={props.onDelete}
    />
  );
}
```

We also gave the `<DeleteTodoAction>` a keyboard shortcut. This way users can delete todos quicker. Additionally, we also added the `<CreateTodoAction>` to the `<List.Item>`. This makes sure that users can also create new todos when there are some already.

Finally, our command looks like this:

```typescript
import { Action, ActionPanel, Form, Icon, List, useNavigation } from "@raycast/api";
import { useState } from "react";

interface Todo {
  title: string;
  isCompleted: boolean;
}

export default function Command() {
  const [todos, setTodos] = useState<Todo[]>([
    { title: "Write a todo list extension", isCompleted: false },
    { title: "Explain it to others", isCompleted: false },
  ]);

  function handleCreate(todo: Todo) {
    const newTodos = [...todos, todo];
    setTodos(newTodos);
  }

  function handleToggle(index: number) {
    const newTodos = [...todos];
    newTodos[index].isCompleted = !newTodos[index].isCompleted;
    setTodos(newTodos);
  }

  function handleDelete(index: number) {
    const newTodos = [...todos];
    newTodos.splice(index, 1);
    setTodos(newTodos);
  }

  return (
    <List
      actions={
        <ActionPanel>
          <CreateTodoAction onCreate={handleCreate} />
        </ActionPanel>
      }
    >
      {todos.map((todo, index) => (
        <List.Item
          key={index}
          icon={todo.isCompleted ? Icon.Checkmark : Icon.Circle}
          title={todo.title}
          actions={
            <ActionPanel>
              <ActionPanel.Section>
                <ToggleTodoAction todo={todo} onToggle={() => handleToggle(index)} />
              </ActionPanel.Section>
              <ActionPanel.Section>
                <CreateTodoAction onCreate={handleCreate} />
                <DeleteTodoAction onDelete={() => handleDelete(index)} />
              </ActionPanel.Section>
            </ActionPanel>
          }
        />
      ))}
    </List>
  );
}

function CreateTodoForm(props: { onCreate: (todo: Todo) => void }) {
  const { pop } = useNavigation();

  function handleSubmit(values: { title: string }) {
    props.onCreate({ title: values.title, isCompleted: false });
    pop();
  }

  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Create Todo" onSubmit={handleSubmit} />
        </ActionPanel>
      }
    >
      <Form.TextField id="title" title="Title" />
    </Form>
  );
}

function CreateTodoAction(props: { onCreate: (todo: Todo) => void }) {
  return (
    <Action.Push
      icon={Icon.Pencil}
      title="Create Todo"
      shortcut={{ modifiers: ["cmd"], key: "n" }}
      target={<CreateTodoForm onCreate={props.onCreate} />}
    />
  );
}

function ToggleTodoAction(props: { todo: Todo; onToggle: () => void }) {
  return (
    <Action
      icon={props.todo.isCompleted ? Icon.Circle : Icon.Checkmark}
      title={props.todo.isCompleted ? "Uncomplete Todo" : "Complete Todo"}
      onAction={props.onToggle}
    />
  );
}

function DeleteTodoAction(props: { onDelete: () => void }) {
  return (
    <Action
      icon={Icon.Trash}
      title="Delete Todo"
      shortcut={{ modifiers: ["ctrl"], key: "x" }}
      onAction={props.onDelete}
    />
  );
}
```

And that's a wrap. You created a todo list in Raycast, it's that easy. As next steps, you could extract the `<CreateTodoForm>` into a separate command. Then you can create todos also from the root search of Raycast and can even assign a global hotkey to open the form. Also, the todos aren't persisted. If you close the command and reopen it, they are gone. To persist, you can use the [storage](../api-reference/storage.md) or [write it to disc](../api-reference/environment.md#environment).
