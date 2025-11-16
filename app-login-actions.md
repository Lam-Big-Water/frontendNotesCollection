# app/login/actions.ts - 登录和认证 Server Actions

## 文件路径
`app/login/actions.ts`

## 概述
包含所有用户认证相关的 Server Actions：登录、注册、登出和 OAuth 登录。

## 功能说明

### 1. 邮箱/密码登录
```typescript
export async function emailLogin(formData: FormData) {
    const supabase = await createClient()
    
    const data = {
        email: formData.get('email') as string,
        password: formData.get('password') as string,
    }
    
    const { error } = await supabase.auth.signInWithPassword(data)
    
    if (error) {
        redirect('/login?message=Could not authenticate user')
    }
    
    revalidatePath('/', 'layout')
    redirect('/todos')
}
```
- 接收表单数据
- 使用 Supabase 验证凭据
- 成功：重新验证路径并重定向到 Todo 页面
- 失败：重定向回登录页并显示错误消息

### 2. 用户注册
```typescript
export async function signup(formData: FormData) {
    const supabase = await createClient()
    
    const data = {
        email: formData.get('email') as string,
        password: formData.get('password') as string,
    }
    
    const { error } = await supabase.auth.signUp(data)
    
    if (error) {
        redirect('/login?message=Error signing up')
    }
    
    revalidatePath('/', 'layout')
    redirect('/login')
}
```
- 创建新用户账户
- 成功：重定向到登录页（需要邮箱验证）
- 失败：显示错误消息

### 3. 用户登出
```typescript
export async function signOut() {
    const supabase = await createClient();
    await supabase.auth.signOut();
    redirect('/login')
}
```
- 清除用户会话
- 删除认证 Cookie
- 重定向到登录页

### 4. OAuth 登录
```typescript
export async function oAuthSignIn(provider: Provider) {
    if (!provider) {
        return redirect('/login?message=No provider selected')
    }
    
    const supabase = await createClient();
    const redirectUrl = getURL("/auth/callback")
    const { data, error } = await supabase.auth.signInWithOAuth({
        provider,
        options: {
            redirectTo: redirectUrl,
        }
    })
    
    if (error) {
        redirect('/login?message=Could not authenticate user')
    }
    
    return redirect(data.url)
}
```
- 初始化 OAuth 流程
- 生成 OAuth 授权 URL
- 重定向到 OAuth 提供商
- 回调处理在 `/auth/callback` 路由

## 技术要点

### Server Actions
- 使用 `'use server'` 指令
- 在服务器端执行
- 可以直接访问数据库和服务器资源

### 表单处理
- 接收 `FormData` 对象
- 从表单字段提取数据
- 类型断言（实际应用中应添加验证）

### 重定向和缓存
- 使用 `redirect()` 进行导航
- 使用 `revalidatePath()` 清除缓存
- 确保数据一致性

### Next.js 15 变更
- `createClient()` 现在是异步的
- 必须使用 `await createClient()`

## 关键代码解析

### 1. 表单数据提取
```typescript
const data = {
    email: formData.get('email') as string,
    password: formData.get('password') as string,
}
```
- 从 `FormData` 获取值
- 类型断言（应添加验证）
- 准备 Supabase 认证数据

### 2. 错误处理
```typescript
if (error) {
    redirect('/login?message=Could not authenticate user')
}
```
- 检查 Supabase 错误
- 通过 URL 参数传递错误消息
- 重定向到登录页

### 3. 路径重新验证
```typescript
revalidatePath('/', 'layout')
```
- 清除根路径的缓存
- 更新布局级别的缓存
- 确保用户状态更新

### 4. OAuth 回调 URL
```typescript
const redirectUrl = getURL("/auth/callback")
```
- 生成绝对 URL
- 用于 OAuth 回调
- 支持不同环境（开发/生产）

## 使用场景

1. **用户登录**
   - 表单提交触发 `emailLogin`
   - 验证凭据并创建会话

2. **用户注册**
   - 表单提交触发 `signup`
   - 创建新账户

3. **用户登出**
   - 按钮点击触发 `signOut`
   - 清除会话

4. **OAuth 登录**
   - 按钮点击触发 `oAuthSignIn`
   - 重定向到 OAuth 提供商

## 最佳实践

1. **输入验证**
   - 应添加服务器端验证
   - 检查邮箱格式
   - 验证密码强度

2. **错误处理**
   - 提供友好的错误消息
   - 不暴露敏感信息
   - 记录错误日志

3. **安全性**
   - 使用 HTTPS
   - 保护认证 Cookie
   - 防止 CSRF 攻击

## 相关文件

- `app/login/page.tsx` - 使用这些 Server Actions
- `app/auth/callback/route.ts` - OAuth 回调处理
- `utils/supabase/server.ts` - Supabase 客户端
- `utils/helpers.ts` - `getURL` 辅助函数

## Next.js 15 特性

- ✅ Server Actions（`'use server'`）
- ✅ 异步 `createClient()` API
- ✅ `redirect()` 和 `revalidatePath()` API

## 安全考虑

1. **密码处理**
   - 不在客户端存储密码
   - 使用 Supabase 安全认证
   - 密码哈希由 Supabase 处理

2. **会话管理**
   - 安全的 Cookie 设置
   - 自动会话刷新
   - 防止会话劫持

3. **OAuth 安全**
   - 验证回调 URL
   - 检查状态参数
   - 防止重放攻击

## 改进建议

1. **添加输入验证**
   ```typescript
   import { z } from 'zod'
   
   const loginSchema = z.object({
     email: z.string().email(),
     password: z.string().min(6),
   })
   ```

2. **改进错误处理**
   - 更具体的错误消息
   - 区分不同类型的错误
   - 用户友好的提示

3. **添加日志记录**
   - 记录登录尝试
   - 监控认证失败
   - 安全审计

