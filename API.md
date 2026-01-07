# Data Fetching and API Routes in Next.js

## Overview

Next.js 15 revolutionizes data fetching with a modern approach centered around **Server Components** and **Route Handlers**. By default, layouts and pages are Server Components, which lets you fetch data and render parts of your UI on the server, optionally cache the result, and stream it to the client.

## Part 1: Understanding Rendering Strategies

Before diving into implementation, let's understand the three main rendering approaches:

### 1. **SSR (Server-Side Rendering)**
- Data is fetched on the server for each request
- Fresh data on every page load
- Better for dynamic content that changes frequently
- Good for SEO as content is in the initial HTML

### 2. **SSG (Static Site Generation)**
- Pages are generated at build time
- Served as static HTML
- Fastest performance (served from CDN)
- Best for content that doesn't change often

### 3. **CSR (Client-Side Rendering)**
- Data is fetched in the browser after initial page load
- Uses React hooks like `useEffect`
- Good for user-specific or interactive data
- Not ideal for SEO as content loads after JavaScript executes

## Part 2: Data Fetching in Server Components (Modern Approach)

You can fetch data in Server Components using any asynchronous I/O, such as: The fetch API, an ORM or database, reading from the filesystem using Node.js APIs like fs.

### Basic Server Component Data Fetching

Simply making your function or arrow function component asynchronous allows you to fetch data directly:

```tsx
// src/app/posts/page.tsx
export default async function PostsPage() {
  // Fetch data directly in the component
  const res = await fetch('https://jsonplaceholder.typicode.com/posts')
  const posts = await res.json()
  
  return (
    <div className="container mx-auto p-8">
      <h1 className="text-4xl font-bold mb-8">Blog Posts</h1>
      <div className="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
        {posts.slice(0, 9).map((post: any) => (
          <div key={post.id} className="border rounded-lg p-6 shadow-md hover:shadow-lg transition">
            <h2 className="text-xl font-semibold mb-2">{post.title}</h2>
            <p className="text-gray-600">{post.body}</p>
          </div>
        ))}
      </div>
    </div>
  )
}
```

### Configuring Fetch Behavior

Control caching and revalidation:

```tsx
// Revalidate every 60 seconds (ISR - Incremental Static Regeneration)
export default async function Posts() {
  const res = await fetch('https://api.example.com/posts', {
    next: { revalidate: 60 }
  })
  const posts = await res.json()
  
  return <div>{/* render posts */}</div>
}
```

```tsx
// Force dynamic rendering (no cache)
export default async function Posts() {
  const res = await fetch('https://api.example.com/posts', {
    cache: 'no-store'
  })
  const posts = await res.json()
  
  return <div>{/* render posts */}</div>
}
```

### Loading States with Streaming

You can create a loading.js file in the same folder as your page to stream the entire page while the data is being fetched:

```tsx
// src/app/posts/loading.tsx
export default function Loading() {
  return (
    <div className="container mx-auto p-8">
      <div className="animate-pulse">
        <div className="h-10 bg-gray-200 rounded w-1/4 mb-8"></div>
        <div className="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
          {[...Array(6)].map((_, i) => (
            <div key={i} className="border rounded-lg p-6">
              <div className="h-6 bg-gray-200 rounded mb-4"></div>
              <div className="h-4 bg-gray-200 rounded mb-2"></div>
              <div className="h-4 bg-gray-200 rounded w-3/4"></div>
            </div>
          ))}
        </div>
      </div>
    </div>
  )
}
```

### Error Handling

```tsx
// src/app/posts/error.tsx
'use client'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <div className="container mx-auto p-8 text-center">
      <h2 className="text-2xl font-bold text-red-600 mb-4">Something went wrong!</h2>
      <p className="text-gray-600 mb-4">{error.message}</p>
      <button
        onClick={reset}
        className="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
      >
        Try again
      </button>
    </div>
  )
}
```

## Part 3: Data Fetching in Client Components

When you need interactivity or browser-specific features, use Client Components:

```tsx
// src/components/UserProfile.tsx
'use client'

import { useState, useEffect } from 'react'

export default function UserProfile({ userId }: { userId: number }) {
  const [user, setUser] = useState<any>(null)
  const [loading, setLoading] = useState(true)
  
  useEffect(() => {
    async function fetchUser() {
      try {
        const res = await fetch(`https://jsonplaceholder.typicode.com/users/${userId}`)
        const data = await res.json()
        setUser(data)
      } catch (error) {
        console.error('Failed to fetch user:', error)
      } finally {
        setLoading(false)
      }
    }
    
    fetchUser()
  }, [userId])
  
  if (loading) return <div>Loading user...</div>
  if (!user) return <div>User not found</div>
  
  return (
    <div className="border rounded-lg p-6">
      <h3 className="text-xl font-bold mb-2">{user.name}</h3>
      <p className="text-gray-600">{user.email}</p>
      <p className="text-gray-500">{user.phone}</p>
    </div>
  )
}
```

### Using SWR for Better Client-Side Fetching

Install SWR:

```bash
npm install swr
```

```tsx
// src/components/RealtimeData.tsx
'use client'

import useSWR from 'swr'

const fetcher = (url: string) => fetch(url).then(res => res.json())

export default function RealtimeData() {
  const { data, error, isLoading, mutate } = useSWR(
    'https://api.example.com/data',
    fetcher,
    { refreshInterval: 3000 } // Refresh every 3 seconds
  )
  
  if (error) return <div>Failed to load</div>
  if (isLoading) return <div>Loading...</div>
  
  return (
    <div>
      <h2>{data.title}</h2>
      <button onClick={() => mutate()}>Refresh</button>
    </div>
  )
}
```

## Part 4: API Routes (Route Handlers)

Route Handlers allow you to create custom request handlers for a given route using the Web Request and Response APIs. They are defined in a route.js or route.ts file inside the app directory.

### Creating Your First API Route

Create an api folder inside the app directory. Then, inside the api folder, create a folder for your endpoint, and within that folder, create a file called route.ts:

```tsx
// src/app/api/hello/route.ts
export async function GET(request: Request) {
  return Response.json({ 
    message: 'Hello from Next.js API!',
    timestamp: new Date().toISOString()
  })
}
```

Access it at: `http://localhost:3000/api/hello`

### GET Request with Query Parameters

```tsx
// src/app/api/users/route.ts
import { NextRequest } from 'next/server'

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams
  const page = searchParams.get('page') || '1'
  const limit = searchParams.get('limit') || '10'
  
  // Fetch from external API or database
  const res = await fetch(
    `https://jsonplaceholder.typicode.com/users?_page=${page}&_limit=${limit}`
  )
  const users = await res.json()
  
  return Response.json({
    data: users,
    page: parseInt(page),
    limit: parseInt(limit)
  })
}
```

Usage: `http://localhost:3000/api/users?page=1&limit=5`

### POST Request - Creating Resources

```tsx
// src/app/api/posts/route.ts
export async function POST(request: Request) {
  try {
    const body = await request.json()
    const { title, content, authorId } = body
    
    // Validate input
    if (!title || !content) {
      return Response.json(
        { error: 'Title and content are required' },
        { status: 400 }
      )
    }
    
    // In a real app, save to database
    const newPost = {
      id: Date.now(),
      title,
      content,
      authorId,
      createdAt: new Date().toISOString()
    }
    
    return Response.json(newPost, { status: 201 })
  } catch (error) {
    return Response.json(
      { error: 'Failed to create post' },
      { status: 500 }
    )
  }
}
```

### PUT Request - Updating Resources

```tsx
// src/app/api/posts/[id]/route.ts
export async function PUT(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params
  const body = await request.json()
  
  // Update in database
  const updatedPost = {
    id,
    ...body,
    updatedAt: new Date().toISOString()
  }
  
  return Response.json(updatedPost)
}
```

### DELETE Request - Removing Resources

```tsx
// src/app/api/posts/[id]/route.ts
export async function DELETE(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params
  
  // Delete from database
  // await db.post.delete({ where: { id } })
  
  return Response.json(
    { message: `Post ${id} deleted successfully` },
    { status: 200 }
  )
}
```

## Part 5: Dynamic Routes in API

```tsx
// src/app/api/users/[id]/route.ts
export async function GET(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params
  
  const res = await fetch(`https://jsonplaceholder.typicode.com/users/${id}`)
  
  if (!res.ok) {
    return Response.json(
      { error: 'User not found' },
      { status: 404 }
    )
  }
  
  const user = await res.json()
  return Response.json(user)
}
```

## Part 6: Parallel vs Sequential Data Fetching

With sequential data fetching, requests in a route are dependent on each other. With parallel data fetching, requests in a route are eagerly initiated and will load data at the same time.

### Sequential (Slow - Creates Waterfall)

```tsx
// ❌ BAD: Sequential fetching
export default async function Page() {
  const user = await fetch('/api/user')
  const posts = await fetch(`/api/posts?userId=${user.id}`) // Waits for user
  const comments = await fetch(`/api/comments?postId=${posts[0].id}`) // Waits for posts
  
  // Total time = time1 + time2 + time3
}
```

### Parallel (Fast - All at Once)

```tsx
// ✅ GOOD: Parallel fetching
export default async function Page() {
  const userPromise = fetch('/api/user')
  const postsPromise = fetch('/api/posts')
  const commentsPromise = fetch('/api/comments')
  
  const [user, posts, comments] = await Promise.all([
    userPromise,
    postsPromise,
    commentsPromise
  ])
  
  // Total time = max(time1, time2, time3)
}
```

## Part 7: Complete CRUD Example

Let's build a complete blog API with all CRUD operations:

```tsx
// src/app/api/blog/route.ts
let posts = [
  { id: 1, title: 'First Post', content: 'Hello World', author: 'John' },
  { id: 2, title: 'Second Post', content: 'Next.js is awesome', author: 'Jane' }
]

// GET all posts
export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams
  const author = searchParams.get('author')
  
  let filteredPosts = posts
  if (author) {
    filteredPosts = posts.filter(p => p.author === author)
  }
  
  return Response.json(filteredPosts)
}

// POST - Create new post
export async function POST(request: Request) {
  const body = await request.json()
  
  const newPost = {
    id: posts.length + 1,
    ...body
  }
  
  posts.push(newPost)
  
  return Response.json(newPost, { status: 201 })
}
```

```tsx
// src/app/api/blog/[id]/route.ts
// GET single post
export async function GET(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params
  const post = posts.find(p => p.id === parseInt(id))
  
  if (!post) {
    return Response.json({ error: 'Post not found' }, { status: 404 })
  }
  
  return Response.json(post)
}

// PUT - Update post
export async function PUT(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params
  const body = await request.json()
  
  const index = posts.findIndex(p => p.id === parseInt(id))
  
  if (index === -1) {
    return Response.json({ error: 'Post not found' }, { status: 404 })
  }
  
  posts[index] = { ...posts[index], ...body }
  
  return Response.json(posts[index])
}

// DELETE - Remove post
export async function DELETE(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params
  const index = posts.findIndex(p => p.id === parseInt(id))
  
  if (index === -1) {
    return Response.json({ error: 'Post not found' }, { status: 404 })
  }
  
  const deletedPost = posts.splice(index, 1)[0]
  
  return Response.json({ message: 'Post deleted', post: deletedPost })
}
```

## Summary

You now understand:

- ✅ **SSR, SSG, and CSR** rendering strategies
- ✅ **Server Components** for async data fetching
- ✅ **Client Components** with useEffect and SWR
- ✅ **Route Handlers** for building APIs
- ✅ **GET, POST, PUT, DELETE** HTTP methods
- ✅ **Dynamic routes** with parameters
- ✅ **Parallel vs Sequential** data fetching patterns
- ✅ Complete **CRUD operations**

Server components are perfect for initial data loads since they use fetch with built-in deduplication and caching. Client components make sense when you need interactivity after the initial load.

**Next topic**: We'll integrate databases using **Prisma ORM with PostgreSQL/MySQL** and **Mongoose with MongoDB**!

Ready when you are! 🚀
