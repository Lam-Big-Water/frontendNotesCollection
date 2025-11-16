# SupaTodo - Next.js 15 与 Supabase 最佳实践教程项目

## 📋 项目概述

SupaTodo 是一个基于 Next.js 15 和 Supabase 构建的现代化 Todo 应用，展示了如何使用最新的 Web 技术栈构建全栈应用。该项目是学习 Next.js 15 与 Supabase 集成的最佳实践教程。

## 🎯 项目功能

### 核心功能
1. **用户认证系统**
   - 邮箱/密码登录
   - 用户注册
   - OAuth 登录（GitHub）
   - 会话管理和自动刷新
   - 安全的登出功能

2. **Todo 管理**
   - 创建新的 Todo 任务
   - 查看所有 Todo 列表
   - 更新 Todo 状态（完成/未完成）
   - 删除 Todo 任务
   - 乐观更新（Optimistic Updates）提供即时反馈

3. **用户体验优化**
   - 响应式设计
   - 实时 UI 更新
   - 表单状态管理
   - 加载状态提示

## 🛠️ 技术栈

### 前端框架
- **Next.js 15** - React 全栈框架
  - App Router（App 目录路由）
  - Server Components（服务器组件）
  - Server Actions（服务器操作）
  - 中间件（Middleware）

### 后端服务
- **Supabase** - 开源 Firebase 替代方案
  - PostgreSQL 数据库
  - 身份认证服务
  - Row Level Security (RLS)
  - 实时数据同步

### UI 组件库
- **shadcn/ui** - 基于 Radix UI 的组件库
- **Tailwind CSS** - 实用优先的 CSS 框架
- **Radix UI** - 无样式、可访问的 UI 组件
- **Lucide React** - 图标库

### 开发工具
- **TypeScript** - 类型安全的 JavaScript
- **PostCSS** - CSS 后处理器
- **Autoprefixer** - CSS 自动前缀

### React Hooks
- `useOptimistic` - 乐观更新
- `useFormStatus` - 表单状态管理
- `useState` - 本地状态管理

## 📁 项目结构

```
supatodo/
├── app/                    # Next.js App Router 目录
│   ├── auth/              # 认证相关路由
│   │   ├── callback/      # OAuth 回调处理
│   │   └── confirm/       # 邮箱验证处理
│   ├── login/             # 登录页面
│   ├── todos/             # Todo 管理页面
│   ├── layout.tsx         # 根布局
│   ├── page.tsx           # 首页
│   └── globals.css        # 全局样式
├── components/             # React 组件
│   ├── ui/                # shadcn/ui 组件
│   ├── header.tsx         # 页面头部
│   ├── todo-form.tsx      # Todo 表单
│   ├── todo-item.tsx      # Todo 单项
│   └── todo-list.tsx      # Todo 列表
├── utils/                 # 工具函数
│   ├── supabase/          # Supabase 客户端
│   │   ├── server.ts      # 服务器端客户端
│   │   └── middleware.ts   # 中间件客户端
│   └── helpers.ts         # 辅助函数
├── types/                 # TypeScript 类型定义
│   ├── custom.ts          # 自定义类型
│   └── supabase.ts        # Supabase 生成类型
├── lib/                   # 库文件
│   └── utils.ts           # 通用工具函数
├── middleware.ts          # Next.js 中间件
├── next.config.js         # Next.js 配置
├── tailwind.config.ts     # Tailwind 配置
└── tsconfig.json          # TypeScript 配置
```

## 🔄 运行流程

### 1. 应用启动流程

```
用户访问应用
    ↓
Next.js 中间件检查会话
    ↓
未登录 → 重定向到 /login
已登录 → 显示相应页面
```

### 2. 用户认证流程

#### 邮箱/密码登录
```
用户输入邮箱和密码
    ↓
提交表单 → Server Action (emailLogin)
    ↓
Supabase 验证凭据
    ↓
成功 → 设置 Cookie → 重定向到 /todos
失败 → 显示错误信息
```

#### OAuth 登录（GitHub）
```
用户点击 "Login with GitHub"
    ↓
Server Action (oAuthSignIn) → Supabase OAuth
    ↓
重定向到 GitHub 授权页面
    ↓
用户授权 → 回调到 /auth/callback
    ↓
交换授权码 → 设置会话 → 重定向到 /todos
```

### 3. Todo 操作流程

#### 创建 Todo
```
用户输入任务内容
    ↓
提交表单 → 乐观更新（立即显示）
    ↓
Server Action (addTodo) → Supabase 插入数据
    ↓
成功 → 重新验证路径 → 更新 UI
```

#### 更新 Todo
```
用户点击复选框
    ↓
更新本地状态（乐观更新）
    ↓
Server Action (updateTodo) → Supabase 更新数据
    ↓
成功 → 重新验证路径 → 同步状态
```

#### 删除 Todo
```
用户点击删除按钮
    ↓
乐观更新（立即移除）
    ↓
Server Action (deleteTodo) → Supabase 删除数据
    ↓
成功 → 重新验证路径 → 更新列表
```

## 🔐 安全特性

### 1. Row Level Security (RLS)
Supabase 数据库启用了行级安全策略：
- 用户只能查看自己的 Todo
- 用户只能创建、更新、删除自己的 Todo
- 所有操作都通过 `auth.uid()` 验证用户身份

### 2. 会话管理
- 中间件自动刷新用户会话
- Cookie 安全设置
- 服务器端会话验证

### 3. 类型安全
- TypeScript 提供编译时类型检查
- Supabase 生成的类型定义确保数据库操作的类型安全

## 🎓 学习要点

### Next.js 15 特性
1. **Server Actions**
   - 在服务器端执行的数据变更操作
   - 无需创建 API 路由
   - 自动处理表单提交

2. **Server Components**
   - 默认在服务器端渲染
   - 减少客户端 JavaScript 包大小
   - 直接访问服务器资源

3. **异步 Cookies API**
   - Next.js 15 中 `cookies()` 是异步的
   - 需要使用 `await cookies()`

4. **中间件**
   - 在请求处理前执行
   - 用于身份验证和会话管理

### Supabase 集成
1. **服务器端客户端**
   - 使用 `@supabase/ssr` 包
   - 正确处理 Cookie 管理
   - 支持服务器组件和中间件

2. **认证流程**
   - 邮箱/密码认证
   - OAuth 提供商集成
   - 会话刷新机制

3. **数据库操作**
   - 类型安全的查询
   - RLS 策略保护
   - 实时数据同步（可选）

### React 最佳实践
1. **乐观更新**
   - 使用 `useOptimistic` Hook
   - 提供即时用户反馈
   - 处理可能的错误回滚

2. **表单处理**
   - 使用 `useFormStatus` 管理表单状态
   - 服务器操作处理提交
   - 客户端验证和反馈

3. **组件组织**
   - 服务器组件和客户端组件分离
   - 可复用 UI 组件
   - 清晰的组件层次结构

## 🚀 快速开始

### 环境要求
- Node.js 18+ 
- npm/yarn/pnpm
- Supabase 账户

### 安装步骤

1. **克隆项目**
```bash
git clone <repository-url>
cd supatodo
```

2. **安装依赖**
```bash
npm install
# 或
pnpm install
```

3. **配置环境变量**
创建 `.env.local` 文件：
```env
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
```

4. **设置 Supabase 数据库**
在 Supabase 控制台执行 SQL：
```sql
create table todos (
  id bigint generated by default as identity primary key,
  user_id uuid references auth.users not null,
  task text check (char_length(task) > 3),
  is_complete boolean default false,
  inserted_at timestamp with time zone default timezone('utc'::text, now()) not null
);

alter table todos enable row level security;

create policy "Individuals can create todos." on todos for
    insert with check (auth.uid() = user_id);
create policy "Individuals can view their own todos." on todos for
    select using ((select auth.uid()) = user_id);
create policy "Individuals can update their own todos." on todos for
    update using ((select auth.uid()) = user_id);
create policy "Individuals can delete their own todos." on todos for
    delete using ((select auth.uid()) = user_id);
```

5. **运行开发服务器**
```bash
npm run dev
```

访问 http://localhost:3000

## 📚 文档导航

### 核心文件文档
- [app/layout.tsx](./docs/app-layout.md) - 根布局组件
- [app/page.tsx](./docs/app-page.md) - 首页
- [app/login/page.tsx](./docs/app-login-page.md) - 登录页面
- [app/login/actions.ts](./docs/app-login-actions.md) - 登录和认证 Server Actions
- [app/login/oauth-signin.tsx](./docs/components-oauth-signin.md) - OAuth 登录组件
- [app/todos/page.tsx](./docs/app-todos-page.md) - Todo 列表页面
- [app/todos/actions.ts](./docs/app-todos-actions.md) - Todo 操作 Server Actions
- [app/auth/callback/route.ts](./docs/app-auth-callback.md) - OAuth 回调处理
- [app/auth/confirm/route.ts](./docs/app-auth-confirm.md) - 邮箱验证处理
- [middleware.ts](./docs/middleware.md) - Next.js 中间件

### 组件文档
- [components/header.tsx](./docs/components-header.md) - 页面头部组件
- [components/todo-form.tsx](./docs/components-todo-form.md) - Todo 表单组件
- [components/todo-item.tsx](./docs/components-todo-item.md) - Todo 单项组件
- [components/todo-list.tsx](./docs/components-todo-list.md) - Todo 列表组件

### 工具和类型文档
- [utils/supabase/server.ts](./docs/utils-supabase-server.md) - Supabase 服务器客户端
- [utils/supabase/middleware.ts](./docs/utils-supabase-middleware.md) - Supabase 中间件客户端
- [utils/helpers.ts](./docs/utils-helpers.md) - 辅助函数
- [lib/utils.ts](./docs/lib-utils.md) - 通用工具函数
- [types/custom.ts](./docs/types-custom.md) - 自定义类型定义
- [types/supabase.ts](./docs/types-supabase.md) - Supabase 类型定义

### 配置文件文档
- [next.config.js](./docs/next-config.md) - Next.js 配置
- [tailwind.config.ts](./docs/tailwind-config.md) - Tailwind CSS 配置
- [tsconfig.json](./docs/tsconfig.md) - TypeScript 配置

## 🎯 学习路径建议

1. **基础理解**
   - 阅读 `summary.md`（本文档）
   - 了解项目结构和运行流程

2. **认证系统**
   - 学习 `middleware.ts` 和 `utils/supabase/` 目录
   - 理解会话管理和 Cookie 处理

3. **页面和路由**
   - 学习 `app/` 目录下的页面组件
   - 理解 Server Components 和 Server Actions

4. **组件开发**
   - 学习 `components/` 目录下的组件
   - 理解乐观更新和表单处理

5. **类型系统**
   - 学习 `types/` 目录
   - 理解 TypeScript 在项目中的应用

## 📝 注意事项

1. **Next.js 15 变更**
   - `cookies()` API 现在是异步的，必须使用 `await`
   - 所有使用 `cookies()` 的函数都需要是 `async`

2. **Supabase 配置**
   - 确保正确配置 RLS 策略
   - 检查环境变量是否正确设置

3. **开发最佳实践**
   - 使用 TypeScript 类型检查
   - 遵循 React Server Components 最佳实践
   - 合理使用 Server Actions 和 Client Components

## 🔗 相关资源

- [Next.js 15 文档](https://nextjs.org/docs)
- [Supabase 文档](https://supabase.com/docs)
- [shadcn/ui 文档](https://ui.shadcn.com)
- [Tailwind CSS 文档](https://tailwindcss.com/docs)

---

**项目状态**: ✅ 已升级到 Next.js 15，所有代码已更新并测试通过

