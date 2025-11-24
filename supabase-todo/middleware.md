# middleware.ts - Next.js 中间件

## 文件路径
`middleware.ts`

## 概述
Next.js 中间件，在请求处理前执行，用于管理用户会话和身份验证。

## 功能说明

### 1. 中间件函数
```typescript
export async function middleware(request: NextRequest) {
    return await updateSession(request)
}
```
- Next.js 中间件标准格式
- 接收 `NextRequest` 对象
- 调用会话更新函数

### 2. 路由匹配配置
```typescript
export const config = {
    matcher: [
        '/((?!_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp)$).*)',
    ],
}
```
- 定义中间件执行的路径模式
- 排除静态文件和 Next.js 内部文件
- 排除图片资源

## 技术要点

### 中间件执行时机
- 在请求到达页面/API 路由之前执行
- 可以修改请求和响应
- 可以重定向请求

### 会话管理
- 自动刷新用户会话
- 更新认证 Cookie
- 确保会话有效性

### 性能考虑
- 只对匹配的路径执行
- 排除静态资源
- 避免不必要的处理

## 关键代码解析

### 1. 路径匹配器
```typescript
matcher: [
    '/((?!_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp)$).*)',
]
```
- 正则表达式匹配所有路径
- 排除 `_next/static`（静态文件）
- 排除 `_next/image`（图片优化）
- 排除 `favicon.ico`
- 排除所有图片文件扩展名

### 2. 会话更新
```typescript
return await updateSession(request)
```
- 调用 Supabase 会话更新函数
- 自动刷新过期的会话
- 更新 Cookie

## 使用场景

1. **会话刷新**
   - 每次请求时检查会话
   - 自动刷新即将过期的会话
   - 保持用户登录状态

2. **认证检查**
   - 在页面渲染前验证用户
   - 可以重定向未授权用户
   - 保护受保护的路由

3. **Cookie 管理**
   - 更新认证 Cookie
   - 确保 Cookie 安全设置
   - 处理 Cookie 过期

## 工作流程

```
用户请求
    ↓
中间件拦截
    ↓
检查路径是否匹配
    ↓
匹配 → 调用 updateSession
    ↓
更新 Supabase 会话
    ↓
刷新 Cookie
    ↓
继续处理请求
```

## 最佳实践

1. **路径匹配优化**
   - 只匹配需要的路径
   - 排除静态资源
   - 减少不必要的执行

2. **会话管理**
   - 自动刷新会话
   - 处理过期情况
   - 更新 Cookie 安全设置

3. **性能考虑**
   - 中间件在 Edge Runtime 运行
   - 保持逻辑简单
   - 避免阻塞请求

## 相关文件

- `utils/supabase/middleware.ts` - Supabase 中间件客户端实现
- `app/login/page.tsx` - 登录页面（受保护）
- `app/todos/page.tsx` - Todo 页面（受保护）

## Next.js 15 特性

- ✅ Edge Runtime 支持
- ✅ 异步中间件
- ✅ 类型安全的请求处理

## 安全考虑

1. **Cookie 安全**
   - 使用 HttpOnly Cookie
   - 设置 Secure 标志（HTTPS）
   - 适当的 SameSite 设置

2. **会话验证**
   - 每次请求验证会话
   - 自动处理过期会话
   - 防止会话劫持

3. **路径保护**
   - 可以添加额外的认证检查
   - 重定向未授权用户
   - 记录访问日志

## 扩展建议

1. **添加认证检查**
   ```typescript
   if (!user && request.nextUrl.pathname.startsWith('/todos')) {
     return NextResponse.redirect(new URL('/login', request.url))
   }
   ```

2. **添加日志记录**
   - 记录中间件执行
   - 监控会话刷新
   - 调试认证问题

3. **性能监控**
   - 测量中间件执行时间
   - 优化慢速路径
   - 缓存策略

