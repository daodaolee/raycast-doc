# useForm

Hook 提供了一个高级接口来处理表单，更具体地说，处理表单验证。它融合了所有最佳实践，为您的表单提供出色的用户体验。

## 签名

```ts
function useForm<T extends Form.Values>(props: {
  onSubmit: (values: T) => void | boolean | Promise<void | boolean>;
  initialValues?: Partial<T>;
  validation?: {
    [id in keyof T]?: ((value: T[id]) => string | undefined | null) | FormValidation;
  };
}): {
  handleSubmit: (values: T) => void | boolean | Promise<void | boolean>;
  itemProps: {
    [id in keyof T]: Partial<Form.ItemProps<T[id]>> & {
      id: string;
    };
  };
  setValidationError: (id: keyof T, error: ValidationError) => void;
  setValue: <K extends keyof T>(id: K, value: T[K]) => void;
  values: T;
  focus: (id: keyof T) => void;
  reset: (initialValues?: Partial<T>) => void;
};
```

### 参数

* `onSubmit` 是一个回调，当提交表单并且所有验证通过时将调用该回调。

有几个配置项：

* `initialValues` 是首次渲染表单时要设置的初始值。
* `validation` 是表单的验证规则。表单项的验证是一个函数，它将项的当前值作为参数，并且在验证失败时必须返回一个字符串。我们还提供了一些常见情况的简写，请参阅  [FormValidation](useform.md#formvalidation)。

### 返回

返回一个对象，其中包含在表单中提供良好用户体验所需的方法和参数。

* `handleSubmit` 是一个传递给 `<Action.SubmitForm>` 元素的 `onSubmit` 属性的函数。它用一些与验证相关的东西包装了最初的 `onSubmit` 参数。
* `itemProps` 是必须传递给 `<Form.Item>` 元素以处理验证的 props。

它还包含一些用于轻松操作表单数据的附加内容。

* `values` 是表单的当前值。
* `setValue` 是一个可用于以函数式的方式设置特定字段的值的函数。
* `setValidationError` 是一个可用于以函数式的方式设置特定字段的验证的函数。
* `focus` 是一个可用于以函数式的方式聚焦的函数。
* `reset` 是一个可用于重置表单值的函数。或者，您可以指定重置表单时要设置的值。

## 例子

```tsx
import { Action, ActionPanel, Form, showToast, Toast } from "@raycast/api";
import { useForm, FormValidation } from "@raycast/utils";

interface SignUpFormValues {
  firstName: string;
  lastName: string;
  birthday: Date | null;
  password: string;
  number: string;
  hobbies: string[];
}

export default function Main() {
  const { handleSubmit, itemProps } = useForm<SignUpFormValues>({
    onSubmit(values) {
      showToast({
        style: Toast.Style.Success,
        title: "Yay!",
        message: `${values.firstName} ${values.lastName} account created`,
      });
    },
    validation: {
      firstName: FormValidation.Required,
      lastName: FormValidation.Required,
      password: (value) => {
        if (value && value.length < 8) {
          return "Password must be at least 8 symbols";
        } else if (!value) {
          return "The item is required";
        }
      },
      number: (value) => {
        if (value && value !== "2") {
          return "Please select '2'";
        }
      },
    },
  });
  return (
    <Form
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Submit" onSubmit={handleSubmit} />
        </ActionPanel>
      }
    >
      <Form.TextField title="First Name" placeholder="Enter first name" {...itemProps.firstName} />
      <Form.TextField title="Last Name" placeholder="Enter last name" {...itemProps.lastName} />
      <Form.DatePicker title="Date of Birth" {...itemProps.birthday} />
      <Form.PasswordField
        title="Password"
        placeholder="Enter password at least 8 characters long"
        {...itemProps.password}
      />
      <Form.Dropdown title="Your Favorite Number" {...itemProps.number}>
        {[1, 2, 3, 4, 5, 6, 7].map((num) => {
          return <Form.Dropdown.Item value={String(num)} title={String(num)} key={num} />;
        })}
      </Form.Dropdown>
    </Form>
  );
}
```

## 类型

### FormValidation

常见验证案例的简写

#### 枚举成员

| 名称       | 描述          |
| -------- | ----------- |
| Required | 当项的值为空时显示错误 |
