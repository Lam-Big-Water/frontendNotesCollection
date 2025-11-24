1. params and searchParams

```tsx
// app/products/[id]/page.tsx
export default async function ProductPage({ 
  params, 
  searchParams 
}: { 
  params: Promise<{ id: string }>
  searchParams: Promise<{ sort: string }>
}) {
  const { id } = await params
  const { sort } = await searchParams
  return <div>Product {id}, sorted by {sort}</div>
}
```

For a given URL

 - `params` is a promise that resolves to an object containing the dynamic route parameters (like id)

 - `searchParams` is a promise that resolves to an object containing the query parameters (link filter and sorting)

 - While page.tsx has access to both params and searchParams, layout.tsx only has access to params


2. import { use } from "react";

```tsx
'use client'
import { use } from 'react'

export function ClientComponent({ dataPromise }) {
  const data = use(dataPromise) // 类似 await，但用于客户端组件
  return <div>{data.name}</div>
}
```

- Specifically designed to handle Promises in client components.

- Similar to await, but suitable for client components.

- React will suspend rendering until the Promise is resolved.

3. Templates

```tsx
// app/template.tsx
export default function Template({ children }) {
  return (
    <div className="animate-fade-in">
      {children} {/* 每次导航都会重新创建 */}
    </div>
  )
}
```

Templates are similar to layouts in that they are also UI shared between multiple pages in your app

Whenever a user navigates between routes sharing a template, you get a completely fresh start

- a new template component instance is mounted
- DOM elements are recreated
- state is cleared
- effects are re-synchronized

Create a template by exporting a default React component from a `template.js` or `template.tsx` file

Like layouts, templates need to accept a children prop to render the nested route segments

4. Loading UI

```tsx
// app/products/loading.tsx
export default function Loading() {
  return <div>Loading products...</div>
}
```

This file helps us create loading states that users see while waiting for content to load in a specific route segment

The loading state appear instantly when navigating, letting users know that the application is responsive and actively loading content

 1. It gives users immediate feedback when they navigate somewhere new. This makes your app feel snappy and responsive, and users know their click actually did something.

 2. Next.js keeps shared layouts interactive while new content loads. Users can still use things like navigation menus or sidebars even if the main content isn't ready yet.

5. error.tsx

```tsx
// app/products/error.tsx
'use client'
export default function Error({ error, reset }) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={reset}>Try again</button>
    </div>
  )
}
```

- It automatically wraps route segments and their nested children in a React Error Boundary

- You can create custom error UIs for specific segments using the file-system hierarchy

- It isolates errors to affected segments while keeping the rest of your app functional

- It enables you to attempt to recover from an error without requiring a full page reload

6. Handling errors in nested routes

```tsx
// app/dashboard/error.tsx - 捕获 dashboard 下所有子路由错误
'use client'
export default function Error({ error, reset }) {
  return (
    <div>
      Dashboard error: {error.message}
      <button onClick={reset}>Retry</button>
    </div>
  )
}
```

- Errors always bubble up to find the closest parent error boundary

- An error.tsx file handles errors not just for its own folder, but for all the nested child segments below it too

- By strategically placing error.tsx files at different levels in your route folders, you can control exactly how detailed your error handing gets

- Where you put your error.tsx file makes a huge difference - it determines exactly which parts of your UI get affected when things go wrong

7. Scope of error.tsx capture:

```tsx
// 能捕获：page.tsx, 组件内的错误
// 不能捕获：layout.tsx, template.tsx, loading.tsx 的错误
```

✅ Page components (page.tsx)

✅ Child components within pages

✅ Components within route segments

❌ Layout components (layout.tsx)

❌ Root layout (app/layout.tsx)

❌ Template components (template.tsx)

❌ Loading components (loading.tsx)

8. Handling global errors

```tsx
// app/global-error.tsx
'use client'
export default function GlobalError({ error, reset }) {
  return (
    <html>
      <body>
        <h2>Global Error!</h2>
        <button onClick={reset}>Try again</button>
      </body>
    </html>
  )
}
```

- If an error boundary can't catch errors in the layout.tsx file from the same segment, what about errors in the root layout?

- It doesn't have a parent segment - how do we handle those errors?

- Next.js provides a special file called `global-error.tsx` that goes in your root app directory

- This is your last line of defense when something goes catastrophically wrong at the highest level of you app
 
 - works only in production mode

 - requires html and body tags to be rendered

9. Parallel routes contd.

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
  users,
  revenue,
}: {
  children: React.ReactNode
  users: React.ReactNode
  revenue: React.ReactNode
}) {
  return (
    <div>
      <div>{children}</div>
      <div>{users}</div>
      <div>{revenue}</div>
    </div>
  )
}
```

- What they are: Parallel routing is an advanced routing mechanism that lets us render multiple pages simultaneously within the same layout

- How to set them up:

 - Parallel routes in Next.js are defined using a feature know as "slots"

 - Slots help organize content in a modular way

 - To create a slot, we use the `@folder` naming convention

 - Each defined slot automatically becomes a prop in its corresponding `layout.tsx` file

 1. Parallel routes use cases

 - Dashboards with multiple sections

 - Split-view interfaces

 - Multi-pane layouts

 - Complex admin interfaces

 2. Parallel routes benefits

 - Parallel routes are great for splitting a layout into manageable slots (especially when different teams work on different parts)

 - Independent route handling

 - Sub-navigation

 3. Independent route handling

 - Each slot in your layout, such as users, revenue, and notifications, can handle its own loading and error states

 - This granular control is particularly useful in scenarios where different sections of the page load at varying speeds or encounter unique errors

 4. Sub-navigation in routes

 - Each slot can essentially function as a mini-application, complete with its own navigation and state management

 - Users can interact with each section separately, applying filters, sorting data, or navigating through pages without affecting other parts

10. Unmatched routes

```tsx
// app/@users/default.tsx
export default function DefaultUsers() {
  return <div>Select a user</div>
}
```

- Navigation from the UI: When navigating through the UI (like clicking links), Next.js keeps showing whatever was in the unmatched slots before

- Page reload

 Next.js looks for a `default.tsx` file in each unmatched slot

 This file is critical as it serves as fallback to render content when the framework cannot retrieve a slot's active state from the current URL

 Without the file, you'll get a 404 error

11. Conditional routes

```tsx
// app/dashboard/layout.tsx
import { getUser } from '@/lib/auth'
export default async function DashboardLayout({
  user,
  guest,
}: {
  user: React.ReactNode
  guest: React.ReactNode
}) {
  const isLoggedIn = await getUser()
  return isLoggedIn ? user : guest
}
```

- Imagine you want to show different content based on whether a user is logged in or not

- You might want to display a dashboard for authenticated users but show a login page for those who aren't

- Conditional routes allow us to achieve this while maintaining completely separate code on the same URL

12. Intercepting routes

```tsx
// app/@modal/(.)photos/[id]/page.tsx
export default function PhotoModal({ params }) {
  return (
    <div className="modal">
      <Photo id={params.id} />
    </div>
  )
}
```

Intercepting routes is an advanced routing mechanism that allows you to load a route from another part of your application within the current layout

It's particularly useful when you want to display new content while keeping your user in the same context

Intercepting routes conventions

- (.) to match segments on the same level

- (..) to match segments one level above

- (..)(..) to match segments two levels above

- (...) to match segments from the root app directory

Use Cases:

Clicking on a photo thumbnail displays the enlarged image in a modal instead of navigating to a new page

Maintains the user's current browsing context

Provides a smoother user experience

 
