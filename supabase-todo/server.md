# utils/supabase/server.ts - Supabase 服务器客户端

## 文件路径
`utils/supabase/server.ts`

## 概述
创建 Supabase 服务器端客户端，用于在 Server Components 和 Server Actions 中访问 Supabase 服务。

## 功能说明

### 1. 客户端创建函数
```typescript
export async function createClient() {
    const cookieStore = await cookies()

    return createServerClient<Database>(
        process.env.NEXT_PUBLIC_SUPABASE_URL!,
        process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
        {
            cookies: {
                // Cookie 处理配置
            },
        }
    )
}
```
- 异步函数（Next.js 15 要求）
- 使用 `@supabase/ssr` 包
- 配置 Cookie 管理
- 返回类型安全的 Supabase 客户端

### 2. Cookie 处理配置
```typescript
cookies: {
    get(name: string) {
        return cookieStore.get(name)?.value
    },
    set(name: string, value: string, options: CookieOptions) {
        try {
            cookieStore.set({ name, value, ...options })
        } catch (error) {
            // 在 Server Component 中调用 set 时忽略错误
        }
    },
    remove(name: string, options: CookieOptions) {
        try {
            cookieStore.set({ name, value: '', ...options })
        } catch (error) {
            // 在 Server Component 中调用 remove 时忽略错误
        }
    },
}
```
- `get`: 读取 Cookie
- `set`: 设置 Cookie（在 Server Component 中可能失败）
- `remove`: 删除 Cookie（通过设置空值）

## 技术要点

### Next.js 15 变更
- `cookies()` API 现在是异步的
- 必须使用 `await cookies()`
- `createClient()` 函数必须是异步的

### Supabase SSR
- 使用 `@supabase/ssr` 包
- 正确处理服务器端 Cookie
- 支持 Server Components 和 Server Actions

### 类型安全
- 使用 Supabase 生成的类型
- TypeScript 类型检查
- 编译时错误检测

## 关键代码解析

### 1. 异步 Cookies API
```typescript
const cookieStore = await cookies()
```
- Next.js 15 的新 API
- 必须使用 `await`
- 返回 Cookie 存储对象

### 2. 环境变量
```typescript
process.env.NEXT_PUBLIC_SUPABASE_URL!,
process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
```
- 使用环境变量配置
- `NEXT_PUBLIC_` 前缀表示客户端可访问
- 非空断言（`!`）表示必须设置

### 3. Cookie 错误处理
```typescript
try {
    cookieStore.set({ name, value, ...options })
} catch (error) {
    // 在 Server Component 中调用 set 时忽略错误
}
```
- Server Component 中不能设置 Cookie
- 错误由中间件处理
- 静默失败不影响功能

### 4. 类型泛型
```typescript
createServerClient<Database>(...)
```
- 使用 Supabase 生成的类型
- 提供类型安全的查询
- 自动补全和类型检查

## 使用场景

1. **Server Components**
   - 在页面组件中获取数据
   - 检查用户认证状态
   - 查询数据库

2. **Server Actions**
   - 处理表单提交
   - 执行数据变更
   - 验证用户权限

3. **API 路由**
   - 处理 API 请求
   - 管理会话
   - 数据操作

## 最佳实践

1. **异步处理**
   - 始终使用 `await`
   - 处理可能的错误
   - 类型安全

2. **Cookie 管理**
   - 理解 Server Component 限制
   - 使用中间件处理 Cookie
   - 错误处理

3. **环境变量**
   - 使用 `.env.local` 文件
   - 不要提交敏感信息
   - 验证环境变量存在

## 相关文件

- `types/supabase.ts` - Supabase 类型定义
- `utils/supabase/middleware.ts` - 中间件客户端
- `middleware.ts` - Next.js 中间件

## Next.js 15 特性

- ✅ 异步 `cookies()` API
- ✅ 异步 `createClient()` 函数
- ✅ Server Components 支持

## 安全考虑

1. **环境变量**
   - 使用 `NEXT_PUBLIC_` 前缀
   - 不要暴露敏感密钥
   - 使用 Anon Key（不是 Service Role Key）

2. **Cookie 安全**
   - 由 Supabase 管理
   - HttpOnly Cookie
   - Secure 标志（HTTPS）

3. **类型安全**
   - 使用生成的类型
   - 编译时检查
   - 减少运行时错误

## 常见问题

1. **为什么需要异步？**
   - Next.js 15 中 `cookies()` 是异步的
   - 必须使用 `await`

2. **Server Component 中设置 Cookie 失败？**
   - 这是预期的行为
   - 使用中间件处理 Cookie
   - 错误可以安全忽略

3. **如何更新类型？**
   - 使用 Supabase CLI 生成
   - 运行 `supabase gen types typescript`
   - 更新 `types/supabase.ts`

