# components/todo-form.tsx - Todo 表单组件

## 文件路径
`components/todo-form.tsx`

## 概述
用于创建新 Todo 任务的表单组件，支持乐观更新，提供即时用户反馈。

## 功能说明

### 1. 组件结构
```typescript
"use client";

export function TodoForm({
  optimisticUpdate,
}: {
  optimisticUpdate: TodoOptimisticUpdate;
}) {
  const formRef = useRef<HTMLFormElement>(null);
  // ...
}
```
- 客户端组件
- 接收乐观更新函数
- 使用 ref 管理表单

### 2. 表单提交处理
```typescript
<form
  ref={formRef}
  className="flex gap-4"
  action={async (data) => {
    const newTodo: Todo = {
      id: -1,
      inserted_at: "",
      user_id: "",
      task: data.get("todo") as string,
      is_complete: false,
    };
    optimisticUpdate({ action: "create", todo: newTodo });
    await addTodo(data);
    formRef.current?.reset();
  }}
>
```
- 异步表单处理
- 创建临时 Todo 对象
- 触发乐观更新
- 调用 Server Action
- 重置表单

### 3. 表单内容组件
```typescript
function FormContent() {
  const { pending } = useFormStatus();
  return (
    <>
      <Textarea
        disabled={pending}
        minLength={4}
        name="todo"
        required
        placeholder="Add a new todo"
      />
      <Button type="submit" size="icon" className="min-w-10" disabled={pending}>
        <Send className="h-5 w-5" />
        <span className="sr-only">Submit Todo</span>
      </Button>
    </>
  );
}
```
- 使用 `useFormStatus` Hook
- 显示加载状态
- 禁用提交时的输入

## 技术要点

### useFormStatus Hook
- React DOM 的 Hook
- 获取表单提交状态
- 自动处理 `pending` 状态
- 必须在表单内部使用

### 乐观更新
- 在 Server Action 执行前更新 UI
- 提供即时反馈
- 改善用户体验

### 表单重置
- 使用 ref 访问表单元素
- 提交成功后重置
- 清除输入内容

## 关键代码解析

### 1. 临时 Todo 对象
```typescript
const newTodo: Todo = {
  id: -1,
  inserted_at: "",
  user_id: "",
  task: data.get("todo") as string,
  is_complete: false,
};
```
- 创建临时 Todo（用于乐观更新）
- 使用负 ID（不会与真实 ID 冲突）
- 服务器会生成真实的 ID

### 2. 乐观更新调用
```typescript
optimisticUpdate({ action: "create", todo: newTodo });
```
- 立即更新 UI
- 新 Todo 出现在列表顶部
- 无需等待服务器响应

### 3. Server Action 调用
```typescript
await addTodo(data);
```
- 异步执行
- 保存到数据库
- 如果失败，`useOptimistic` 会自动回滚

### 4. 表单重置
```typescript
formRef.current?.reset();
```
- 清空输入字段
- 可选链操作符防止错误
- 准备下一次输入

### 5. 表单状态管理
```typescript
const { pending } = useFormStatus();
```
- 获取表单提交状态
- `pending`: 提交进行中时为 `true`
- 用于禁用输入和按钮

## 使用场景

1. **创建新 Todo**
   - 用户输入任务内容
   - 点击提交按钮
   - 立即显示在列表中
   - 后台保存到数据库

2. **表单验证**
   - HTML5 验证（`required`, `minLength`）
   - 最小长度 4 个字符
   - 客户端即时反馈

3. **加载状态**
   - 提交时禁用输入
   - 显示加载指示器（通过按钮状态）
   - 防止重复提交

## 数据流

```
用户输入
    ↓
点击提交
    ↓
创建临时 Todo
    ↓
乐观更新（立即显示）
    ↓
调用 Server Action
    ↓
保存到数据库
    ↓
重新验证路径
    ↓
表单重置
```

## 最佳实践

1. **乐观更新**
   - 提供即时反馈
   - 改善用户体验
   - 处理可能的失败

2. **表单验证**
   - HTML5 验证
   - 服务器端验证（在 Server Action 中）
   - 双重验证确保安全

3. **用户体验**
   - 清晰的占位符
   - 加载状态指示
   - 提交后重置表单

## 相关文件

- `components/todo-list.tsx` - 提供 `optimisticUpdate` 函数
- `app/todos/actions.ts` - `addTodo` Server Action
- `components/ui/textarea.tsx` - 文本输入组件
- `components/ui/button.tsx` - 提交按钮组件

## React 特性

- ✅ `useFormStatus` Hook
- ✅ `useRef` Hook
- ✅ 客户端组件
- ✅ 异步表单处理

## 用户体验优化

1. **即时反馈**
   - 乐观更新立即显示
   - 无需等待服务器

2. **表单状态**
   - 提交时禁用输入
   - 防止重复提交
   - 清晰的视觉反馈

3. **输入体验**
   - 占位符提示
   - 最小长度要求
   - 自动重置

## 改进建议

1. **添加错误处理**
   ```typescript
   try {
     optimisticUpdate({ action: "create", todo: newTodo });
     await addTodo(data);
     formRef.current?.reset();
   } catch (error) {
     // 显示错误消息
   }
   ```

2. **添加字符计数**
   - 显示已输入字符数
   - 最大长度限制
   - 实时反馈

3. **添加键盘快捷键**
   - Enter 提交（Ctrl/Cmd + Enter）
   - Escape 取消
   - 提升效率

