# app/todos/actions.ts - Todo 操作 Server Actions

## 文件路径
`app/todos/actions.ts`

## 概述
包含所有 Todo 相关的 Server Actions：创建、更新和删除 Todo 任务。

## 功能说明

### 1. 创建 Todo
```typescript
export async function addTodo(formData: FormData) {
    const supabase = await createClient();
    const text = formData.get("todo") as string | null

    if (!text) {
        throw new Error("Text is required")
    }

    const { data: { user } } = await supabase.auth.getUser()

    if (!user) {
        throw new Error("User is not logged in")
    }

    const { error } = await supabase.from("todos").insert({
        task: text,
        user_id: user.id
    })

    if (error) {
        throw new Error("Error adding task")
    }

    revalidatePath("/todos")
}
```
- 从表单获取任务文本
- 验证用户身份
- 插入新 Todo 到数据库
- 重新验证路径以更新 UI

### 2. 删除 Todo
```typescript
export async function deleteTodo(id: number) {
    const supabase = await createClient();
    const { data: { user } } = await supabase.auth.getUser()

    if (!user) {
        throw new Error("User is not logged in")
    }

    const { error } = await supabase.from("todos").delete().match({
        user_id: user.id,
        id: id
    })

    if (error) {
        throw new Error("Error deleting task")
    }

    revalidatePath("/todos")
}
```
- 接收 Todo ID
- 验证用户身份
- 删除匹配的 Todo（双重验证：user_id 和 id）
- 重新验证路径

### 3. 更新 Todo
```typescript
export async function updateTodo(todo: Todo) {
    const supabase = await createClient();
    const { data: { user } } = await supabase.auth.getUser()

    if (!user) {
        throw new Error("User is not logged in")
    }

    const { error } = await supabase.from("todos").update(todo).match({
        user_id: user.id,
        id: todo.id
    })

    if (error) {
        throw new Error("Error updating task")
    }

    revalidatePath("/todos")
}
```
- 接收完整的 Todo 对象
- 验证用户身份
- 更新匹配的 Todo
- 重新验证路径

## 技术要点

### Server Actions
- 使用 `"use server"` 指令
- 在服务器端执行
- 直接访问数据库

### 认证检查
- 每个操作都验证用户身份
- 确保用户已登录
- 防止未授权操作

### 数据安全
- 双重验证（user_id + id）
- RLS 策略提供额外保护
- 防止用户操作其他用户的数据

### 错误处理
- 抛出错误供客户端处理
- 清晰的错误消息
- 类型安全

## 关键代码解析

### 1. 用户身份验证
```typescript
const { data: { user } } = await supabase.auth.getUser()

if (!user) {
    throw new Error("User is not logged in")
}
```
- 获取当前用户
- 检查用户是否存在
- 未登录时抛出错误

### 2. 数据插入
```typescript
const { error } = await supabase.from("todos").insert({
    task: text,
    user_id: user.id
})
```
- 插入新记录
- 自动设置 `user_id`
- 其他字段使用默认值（`is_complete: false`, `inserted_at: now()`）

### 3. 数据删除（双重验证）
```typescript
const { error } = await supabase.from("todos").delete().match({
    user_id: user.id,
    id: id
})
```
- 匹配 `user_id` 和 `id`
- 确保用户只能删除自己的 Todo
- RLS 策略提供额外保护

### 4. 数据更新
```typescript
const { error } = await supabase.from("todos").update(todo).match({
    user_id: user.id,
    id: todo.id
})
```
- 更新整个 Todo 对象
- 双重验证确保安全
- 可以更新多个字段

### 5. 路径重新验证
```typescript
revalidatePath("/todos")
```
- 清除 `/todos` 路径的缓存
- 触发页面重新渲染
- 显示最新数据

## 使用场景

1. **创建 Todo**
   - 用户提交表单
   - 验证输入
   - 保存到数据库
   - 更新 UI

2. **更新 Todo**
   - 用户切换完成状态
   - 更新数据库
   - 同步 UI

3. **删除 Todo**
   - 用户点击删除按钮
   - 从数据库删除
   - 更新列表

## 数据流

```
用户操作
    ↓
Server Action 调用
    ↓
验证用户身份
    ↓
执行数据库操作
    ↓
检查错误
    ↓
重新验证路径
    ↓
UI 自动更新
```

## 最佳实践

1. **认证检查**
   - 每个操作都验证用户
   - 防止未授权访问
   - 清晰的错误消息

2. **数据验证**
   - 服务器端验证
   - 检查必需字段
   - 验证数据类型

3. **错误处理**
   - 抛出有意义的错误
   - 客户端可以捕获和处理
   - 提供用户反馈

## 相关文件

- `app/todos/page.tsx` - 使用这些 Server Actions
- `components/todo-form.tsx` - 调用 `addTodo`
- `components/todo-item.tsx` - 调用 `updateTodo` 和 `deleteTodo`
- `utils/supabase/server.ts` - Supabase 客户端

## Next.js 15 特性

- ✅ Server Actions（`"use server"`）
- ✅ 异步 `createClient()` API
- ✅ `revalidatePath()` API

## 安全考虑

1. **用户验证**
   - 每个操作都检查用户
   - 防止未授权操作

2. **数据隔离**
   - 双重验证（user_id + id）
   - RLS 策略保护
   - 防止数据泄露

3. **输入验证**
   - 检查必需字段
   - 验证数据类型
   - 防止注入攻击

## 改进建议

1. **添加输入验证**
   ```typescript
   import { z } from 'zod'
   
   const todoSchema = z.object({
     task: z.string().min(4).max(500),
   })
   ```

2. **改进错误处理**
   - 更具体的错误消息
   - 区分不同类型的错误
   - 记录错误日志

3. **添加事务支持**
   - 对于复杂操作
   - 确保数据一致性
   - 回滚机制

