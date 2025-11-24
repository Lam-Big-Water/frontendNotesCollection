# utils/supabase/middleware.ts - Supabase 中间件客户端

## 文件路径
`utils/supabase/middleware.ts`

## 概述
创建 Supabase 中间件客户端，用于在 Next.js 中间件中管理用户会话和刷新认证令牌。

## 功能说明

### 1. 会话更新函数
```typescript
export async function updateSession(request: NextRequest) {
    let response = NextResponse.next({
        request: {
            headers: request.headers,
        },
    })
    // ...
}
```
- 接收 `NextRequest` 对象
- 创建 `NextResponse` 对象
- 更新请求和响应

### 2. Supabase 客户端创建
```typescript
const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
        cookies: {
            get(name: string) {
                return request.cookies.get(name)?.value
            },
            set(name: string, value: string, options: CookieOptions) {
                // 更新请求和响应中的 Cookie
            },
            remove(name: string, options: CookieOptions) {
                // 从请求和响应中删除 Cookie
            },
        },
    }
)
```
- 使用中间件特定的 Cookie 处理
- 从请求读取 Cookie
- 更新响应中的 Cookie

### 3. Cookie 设置逻辑
```typescript
set(name: string, value: string, options: CookieOptions) {
    request.cookies.set({
        name,
        value,
        ...options,
    })
    response = NextResponse.next({
        request: {
            headers: request.headers,
        },
    })
    response.cookies.set({
        name,
        value,
        ...options,
    })
}
```
- 更新请求中的 Cookie
- 创建新的响应对象
- 更新响应中的 Cookie

### 4. 会话刷新
```typescript
await supabase.auth.getUser()

return response
```
- 调用 `getUser()` 触发会话检查
- 如果会话过期，自动刷新
- 返回更新后的响应

## 技术要点

### 中间件环境
- 在 Edge Runtime 中运行
- 可以修改请求和响应
- 在页面渲染前执行

### Cookie 管理
- 从请求读取 Cookie
- 更新响应中的 Cookie
- 确保 Cookie 同步

### 会话刷新
- 自动检查会话状态
- 刷新过期的会话
- 更新认证 Cookie

## 关键代码解析

### 1. 响应对象创建
```typescript
let response = NextResponse.next({
    request: {
        headers: request.headers,
    },
})
```
- 创建下一个响应
- 传递请求头
- 可以修改响应

### 2. Cookie 读取
```typescript
get(name: string) {
    return request.cookies.get(name)?.value
}
```
- 从请求中读取 Cookie
- 返回 Cookie 值
- 可选链处理不存在的情况

### 3. Cookie 设置（双重更新）
```typescript
request.cookies.set({ name, value, ...options })
response = NextResponse.next({ ... })
response.cookies.set({ name, value, ...options })
```
- 更新请求 Cookie（用于当前请求）
- 创建新响应
- 更新响应 Cookie（用于后续请求）

### 4. Cookie 删除
```typescript
remove(name: string, options: CookieOptions) {
    request.cookies.set({ name, value: '', ...options })
    response = NextResponse.next({ ... })
    response.cookies.set({ name, value: '', ...options })
}
```
- 通过设置空值删除
- 更新请求和响应
- 确保 Cookie 被清除

### 5. 会话检查
```typescript
await supabase.auth.getUser()
```
- 触发会话验证
- 如果过期，自动刷新
- 更新 Cookie

## 使用场景

1. **会话刷新**
   - 每次请求检查会话
   - 自动刷新过期会话
   - 保持用户登录状态

2. **认证保护**
   - 验证用户身份
   - 保护受保护的路由
   - 重定向未授权用户

3. **Cookie 管理**
   - 更新认证 Cookie
   - 处理 Cookie 过期
   - 确保 Cookie 安全

## 工作流程

```
请求到达
    ↓
中间件拦截
    ↓
创建 Supabase 客户端
    ↓
读取请求 Cookie
    ↓
调用 getUser() 检查会话
    ↓
会话过期？ → 刷新会话
    ↓
更新响应 Cookie
    ↓
继续处理请求
```

## 最佳实践

1. **Cookie 同步**
   - 同时更新请求和响应
   - 确保 Cookie 一致性
   - 处理所有 Cookie 操作

2. **会话管理**
   - 自动刷新会话
   - 处理过期情况
   - 错误处理

3. **性能优化**
   - 只在需要时刷新
   - 避免不必要的操作
   - 保持中间件轻量

## 相关文件

- `middleware.ts` - Next.js 中间件（使用此函数）
- `utils/supabase/server.ts` - 服务器客户端（对比）
- `app/login/page.tsx` - 登录页面

## Next.js 15 特性

- ✅ Edge Runtime 支持
- ✅ 中间件 Cookie API
- ✅ 异步函数支持

## 安全考虑

1. **Cookie 安全**
   - 使用 HttpOnly Cookie
   - 设置 Secure 标志（HTTPS）
   - 适当的 SameSite 设置

2. **会话验证**
   - 每次请求验证
   - 自动刷新过期会话
   - 防止会话劫持

3. **环境变量**
   - 使用 Anon Key
   - 不要暴露 Service Role Key
   - 保护敏感信息

## 与服务器客户端对比

| 特性 | 中间件客户端 | 服务器客户端 |
|------|------------|------------|
| 运行环境 | Edge Runtime | Node.js Runtime |
| Cookie 读取 | 从请求读取 | 从 Cookie Store 读取 |
| Cookie 设置 | 更新请求和响应 | 只能读取（Server Component） |
| 使用场景 | 中间件 | Server Components/Actions |

## 常见问题

1. **为什么需要双重 Cookie 更新？**
   - 请求 Cookie 用于当前请求
   - 响应 Cookie 用于后续请求
   - 确保 Cookie 同步

2. **中间件中的 Cookie 设置总是成功？**
   - 是的，中间件可以设置 Cookie
   - 与 Server Component 不同
   - 这是中间件的优势

3. **如何调试中间件？**
   - 添加日志记录
   - 检查 Cookie 值
   - 验证会话状态

