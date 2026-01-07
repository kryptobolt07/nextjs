# Authentication and Authorization with NextAuth.js v5

In this comprehensive guide, we'll implement a complete authentication system using **NextAuth.js v5** (also known as Auth.js) with Google and GitHub OAuth providers, plus role-based access control (RBAC) for managing user permissions.

## Overview of NextAuth.js v5

NextAuth is transforming into Auth.js. For Next.js 15 we are opting for v5 which is at present in beta/RC, but stable enough for everyone to rely on it. Auth.js version 5 is a major rewrite with stricter OAuth/OIDC spec-compliance and the minimum required Next.js version is now 14.0.

## Part 1: Installing NextAuth.js v5

Install NextAuth v5 beta version:

```bash
npm install next-auth@beta
```

If using Prisma for database storage:

```bash
npm install @auth/prisma-adapter
```

## Part 2: Setting Up OAuth Providers

### Creating GitHub OAuth App

1. Go to [GitHub Developer Settings](https://github.com/settings/developers)
2. Click "New OAuth App"
3. Fill in the details:
   - **Application name**: `next-auth-demo`
   - **Homepage URL**: `http://localhost:3000`
   - **Authorization callback URL**: `http://localhost:3000/api/auth/callback/github`
4. Click "Register application"
5. Generate a new client secret and copy both Client ID and Client Secret

### Creating Google OAuth App

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select existing one
3. Navigate to "APIs & Services" → "Credentials"
4. Click "Create Credentials" → "OAuth client ID"
5. Configure consent screen if needed
6. Application type: "Web application"
7. Add authorized redirect URI: `http://localhost:3000/api/auth/callback/google`
8. Copy Client ID and Client Secret

## Part 3: Environment Variables

Create a .env.local file and add your authentication credentials:

```env
# .env.local
AUTH_SECRET="your-generated-secret-here"
GITHUB_ID="your-github-client-id"
GITHUB_SECRET="your-github-client-secret"
GOOGLE_ID="your-google-client-id"
GOOGLE_SECRET="your-google-client-secret"
```

Generate AUTH_SECRET value by running: `openssl rand -hex 32` in your terminal.

## Part 4: NextAuth Configuration

Create a configuration file in the root of the repository named auth.ts:

```typescript
// auth.ts
import NextAuth from "next-auth"
import GitHub from "next-auth/providers/github"
import Google from "next-auth/providers/google"
import { PrismaAdapter } from "@auth/prisma-adapter"
import { prisma } from "@/lib/prisma"

export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: PrismaAdapter(prisma),
  providers: [
    GitHub({
      clientId: process.env.GITHUB_ID!,
      clientSecret: process.env.GITHUB_SECRET!,
    }),
    Google({
      clientId: process.env.GOOGLE_ID!,
      clientSecret: process.env.GOOGLE_SECRET!,
    }),
  ],
  session: {
    strategy: "jwt", // Use JWT for sessions (required for middleware)
  },
  callbacks: {
    async jwt({ token, user }) {
      // Add role to token on sign in
      if (user) {
        token.role = user.role || "user"
        token.id = user.id
      }
      return token
    },
    async session({ session, token }) {
      // Add role to session
      if (session.user) {
        session.user.role = token.role as string
        session.user.id = token.id as string
      }
      return session
    },
  },
  pages: {
    signIn: '/auth/signin',
    error: '/auth/error',
  },
})
```

## Part 5: API Route Handler

Create an API route in app/api/auth/[...nextauth]/route.ts:

```typescript
// src/app/api/auth/[...nextauth]/route.ts
import { handlers } from "@/auth"

export const { GET, POST } = handlers
```

## Part 6: Updating Prisma Schema for Authentication

Add authentication tables to your Prisma schema:

```prisma
// prisma/schema.prisma
model User {
  id            String    @id @default(cuid())
  name          String?
  email         String    @unique
  emailVerified DateTime?
  image         String?
  role          String    @default("user") // Add role field
  accounts      Account[]
  sessions      Session[]
  posts         Post[]
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
}

model Account {
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String?
  access_token      String?
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String?
  session_state     String?

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@id([provider, providerAccountId])
}

model Session {
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model VerificationToken {
  identifier String
  token      String
  expires    DateTime

  @@id([identifier, token])
}
```

Run migrations:

```bash
npx prisma migrate dev --name add_auth_tables
npx prisma generate
```

## Part 7: TypeScript Type Definitions

Create type definitions to extend NextAuth types:

```typescript
// types/next-auth.d.ts
import { DefaultSession } from "next-auth"

declare module "next-auth" {
  interface Session {
    user: {
      id: string
      role: string
    } & DefaultSession["user"]
  }

  interface User {
    role: string
  }
}

declare module "next-auth/jwt" {
  interface JWT {
    id: string
    role: string
  }
}
```

## Part 8: Creating Sign In Page

```tsx
// src/app/auth/signin/page.tsx
import { signIn } from "@/auth"
import { redirect } from "next/navigation"

export default function SignInPage() {
  return (
    <div className="flex min-h-screen items-center justify-center bg-gray-50">
      <div className="w-full max-w-md space-y-8 rounded-lg bg-white p-8 shadow-lg">
        <div className="text-center">
          <h2 className="text-3xl font-bold">Sign in to your account</h2>
          <p className="mt-2 text-gray-600">Choose your preferred method</p>
        </div>

        <div className="space-y-4">
          <form
            action={async () => {
              "use server"
              await signIn("github", { redirectTo: "/dashboard" })
            }}
          >
            <button
              type="submit"
              className="w-full flex items-center justify-center gap-3 rounded-lg border border-gray-300 bg-white px-4 py-3 text-gray-700 hover:bg-gray-50 transition"
            >
              <svg className="h-5 w-5" fill="currentColor" viewBox="0 0 24 24">
                <path d="M12 0c-6.626 0-12 5.373-12 12 0 5.302 3.438 9.8 8.207 11.387.599.111.793-.261.793-.577v-2.234c-3.338.726-4.033-1.416-4.033-1.416-.546-1.387-1.333-1.756-1.333-1.756-1.089-.745.083-.729.083-.729 1.205.084 1.839 1.237 1.839 1.237 1.07 1.834 2.807 1.304 3.492.997.107-.775.418-1.305.762-1.604-2.665-.305-5.467-1.334-5.467-5.931 0-1.311.469-2.381 1.236-3.221-.124-.303-.535-1.524.117-3.176 0 0 1.008-.322 3.301 1.23.957-.266 1.983-.399 3.003-.404 1.02.005 2.047.138 3.006.404 2.291-1.552 3.297-1.23 3.297-1.23.653 1.653.242 2.874.118 3.176.77.84 1.235 1.911 1.235 3.221 0 4.609-2.807 5.624-5.479 5.921.43.372.823 1.102.823 2.222v3.293c0 .319.192.694.801.576 4.765-1.589 8.199-6.086 8.199-11.386 0-6.627-5.373-12-12-12z"/>
              </svg>
              Continue with GitHub
            </button>
          </form>

          <form
            action={async () => {
              "use server"
              await signIn("google", { redirectTo: "/dashboard" })
            }}
          >
            <button
              type="submit"
              className="w-full flex items-center justify-center gap-3 rounded-lg border border-gray-300 bg-white px-4 py-3 text-gray-700 hover:bg-gray-50 transition"
            >
              <svg className="h-5 w-5" viewBox="0 0 24 24">
                <path fill="#4285F4" d="M22.56 12.25c0-.78-.07-1.53-.2-2.25H12v4.26h5.92c-.26 1.37-1.04 2.53-2.21 3.31v2.77h3.57c2.08-1.92 3.28-4.74 3.28-8.09z"/>
                <path fill="#34A853" d="M12 23c2.97 0 5.46-.98 7.28-2.66l-3.57-2.77c-.98.66-2.23 1.06-3.71 1.06-2.86 0-5.29-1.93-6.16-4.53H2.18v2.84C3.99 20.53 7.7 23 12 23z"/>
                <path fill="#FBBC05" d="M5.84 14.09c-.22-.66-.35-1.36-.35-2.09s.13-1.43.35-2.09V7.07H2.18C1.43 8.55 1 10.22 1 12s.43 3.45 1.18 4.93l2.85-2.22.81-.62z"/>
                <path fill="#EA4335" d="M12 5.38c1.62 0 3.06.56 4.21 1.64l3.15-3.15C17.45 2.09 14.97 1 12 1 7.7 1 3.99 3.47 2.18 7.07l3.66 2.84c.87-2.6 3.3-4.53 6.16-4.53z"/>
              </svg>
              Continue with Google
            </button>
          </form>
        </div>
      </div>
    </div>
  )
}
```

## Part 9: Session Provider for Client Components

```tsx
// src/components/providers/SessionProvider.tsx
"use client"

import { SessionProvider as NextAuthSessionProvider } from "next-auth/react"

export default function SessionProvider({
  children,
}: {
  children: React.ReactNode
}) {
  return <NextAuthSessionProvider>{children}</NextAuthSessionProvider>
}
```

Update your root layout:

```tsx
// src/app/layout.tsx
import SessionProvider from "@/components/providers/SessionProvider"

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>
        <SessionProvider>
          {children}
        </SessionProvider>
      </body>
    </html>
  )
}
```

## Part 10: Using Authentication in Components

### Server Components

```tsx
// src/app/dashboard/page.tsx
import { auth } from "@/auth"
import { redirect } from "next/navigation"

export default async function DashboardPage() {
  const session = await auth()

  if (!session) {
    redirect("/auth/signin")
  }

  return (
    <div className="container mx-auto p-8">
      <h1 className="text-3xl font-bold mb-4">Welcome, {session.user.name}!</h1>
      <p className="text-gray-600">Email: {session.user.email}</p>
      <p className="text-gray-600">Role: {session.user.role}</p>
    </div>
  )
}
```

### Client Components

```tsx
// src/components/UserProfile.tsx
"use client"

import { useSession, signOut } from "next-auth/react"

export default function UserProfile() {
  const { data: session, status } = useSession()

  if (status === "loading") {
    return <div>Loading...</div>
  }

  if (!session) {
    return <div>Not signed in</div>
  }

  return (
    <div className="flex items-center gap-4">
      <img
        src={session.user.image || "/default-avatar.png"}
        alt={session.user.name || "User"}
        className="h-10 w-10 rounded-full"
      />
      <div>
        <p className="font-semibold">{session.user.name}</p>
        <p className="text-sm text-gray-600">{session.user.role}</p>
      </div>
      <button
        onClick={() => signOut({ callbackUrl: "/" })}
        className="ml-auto px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700"
      >
        Sign Out
      </button>
    </div>
  )
}
```

## Part 11: Role-Based Access Control (RBAC)

### Defining User Roles

You can add roles to your authentication by using the profile callback to set a default role if one is not provided, and then use JWT and session callbacks to pass the role through the token to the session:

```typescript
// auth.ts
export const { handlers, auth, signIn, signOut } = NextAuth({
  // ... other config
  callbacks: {
    async jwt({ token, user, trigger, session }) {
      if (user) {
        // Set default role on sign in
        token.role = user.role || "user"
        token.id = user.id
      }
      
      // Handle role updates
      if (trigger === "update" && session?.role) {
        token.role = session.role
      }
      
      return token
    },
    async session({ session, token }) {
      if (session.user) {
        session.user.role = token.role as string
        session.user.id = token.id as string
      }
      return session
    },
  },
})
```

### Middleware for Route Protection

Create middleware to protect routes based on roles:

```typescript
// middleware.ts
import { auth } from "@/auth"
import { NextResponse } from "next/server"

export default auth((req) => {
  const { pathname } = req.nextUrl
  const session = req.auth

  // Public routes
  const publicRoutes = ["/", "/auth/signin", "/auth/error"]
  if (publicRoutes.includes(pathname)) {
    return NextResponse.next()
  }

  // Check if user is authenticated
  if (!session) {
    return NextResponse.redirect(new URL("/auth/signin", req.url))
  }

  // Admin-only routes
  const adminRoutes = ["/admin"]
  if (adminRoutes.some(route => pathname.startsWith(route))) {
    if (session.user.role !== "admin") {
      return NextResponse.redirect(new URL("/403", req.url))
    }
  }

  // Guest cannot access certain routes
  if (session.user.role === "guest") {
    const restrictedForGuests = ["/dashboard/settings", "/dashboard/billing"]
    if (restrictedForGuests.some(route => pathname.startsWith(route))) {
      return NextResponse.redirect(new URL("/403", req.url))
    }
  }

  return NextResponse.next()
})

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
}
```

### Creating a 403 Forbidden Page

```tsx
// src/app/403/page.tsx
import Link from "next/link"

export default function ForbiddenPage() {
  return (
    <div className="flex min-h-screen items-center justify-center bg-gray-50">
      <div className="text-center">
        <h1 className="text-6xl font-bold text-red-600 mb-4">403</h1>
        <h2 className="text-2xl font-semibold mb-2">Access Forbidden</h2>
        <p className="text-gray-600 mb-6">
          You don't have permission to access this page.
        </p>
        <Link
          href="/dashboard"
          className="px-6 py-3 bg-blue-600 text-white rounded-lg hover:bg-blue-700"
        >
          Go to Dashboard
        </Link>
      </div>
    </div>
  )
}
```

### Role-Based Component Rendering

```tsx
// src/components/RoleGate.tsx
"use client"

import { useSession } from "next-auth/react"

interface RoleGateProps {
  children: React.ReactNode
  allowedRoles: string[]
  fallback?: React.ReactNode
}

export default function RoleGate({
  children,
  allowedRoles,
  fallback = <div>You don't have permission to view this content.</div>,
}: RoleGateProps) {
  const { data: session, status } = useSession()

  if (status === "loading") {
    return <div>Loading...</div>
  }

  if (!session || !allowedRoles.includes(session.user.role)) {
    return <>{fallback}</>
  }

  return <>{children}</>
}
```

Usage:

```tsx
// src/app/dashboard/page.tsx
import RoleGate from "@/components/RoleGate"

export default function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      
      <RoleGate allowedRoles={["admin"]}>
        <div className="bg-red-100 p-4 rounded">
          <h2>Admin Only Section</h2>
          <p>This is only visible to administrators.</p>
        </div>
      </RoleGate>

      <RoleGate allowedRoles={["admin", "user"]}>
        <div className="bg-blue-100 p-4 rounded mt-4">
          <h2>User Section</h2>
          <p>This is visible to users and admins.</p>
        </div>
      </RoleGate>
    </div>
  )
}
```

## Part 12: Managing User Roles

### API Route to Update User Role (Admin Only)

```typescript
// src/app/api/users/[id]/role/route.ts
import { auth } from "@/auth"
import { prisma } from "@/lib/prisma"
import { NextRequest } from "next/server"

export async function PUT(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  try {
    const session = await auth()
    const { id } = await params

    // Check if user is admin
    if (!session || session.user.role !== "admin") {
      return Response.json(
        { error: "Unauthorized" },
        { status: 403 }
      )
    }

    const { role } = await request.json()

    // Validate role
    const validRoles = ["admin", "user", "guest"]
    if (!validRoles.includes(role)) {
      return Response.json(
        { error: "Invalid role" },
        { status: 400 }
      )
    }

    const user = await prisma.user.update({
      where: { id },
      data: { role },
    })

    return Response.json(user)
  } catch (error: any) {
    return Response.json(
      { error: error.message },
      { status: 500 }
    )
  }
}
```

### Admin Panel to Manage Users

```tsx
// src/app/admin/users/page.tsx
import { auth } from "@/auth"
import { prisma } from "@/lib/prisma"
import { redirect } from "next/navigation"
import UserRoleManager from "@/components/admin/UserRoleManager"

export default async function AdminUsersPage() {
  const session = await auth()

  if (!session || session.user.role !== "admin") {
    redirect("/403")
  }

  const users = await prisma.user.findMany({
    select: {
      id: true,
      name: true,
      email: true,
      role: true,
      createdAt: true,
    },
    orderBy: {
      createdAt: "desc",
    },
  })

  return (
    <div className="container mx-auto p-8">
      <h1 className="text-3xl font-bold mb-6">User Management</h1>
      <div className="bg-white rounded-lg shadow overflow-hidden">
        <table className="min-w-full divide-y divide-gray-200">
          <thead className="bg-gray-50">
            <tr>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Name
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Email
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Role
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Actions
              </th>
            </tr>
          </thead>
          <tbody className="bg-white divide-y divide-gray-200">
            {users.map((user) => (
              <tr key={user.id}>
                <td className="px-6 py-4 whitespace-nowrap">{user.name}</td>
                <td className="px-6 py-4 whitespace-nowrap">{user.email}</td>
                <td className="px-6 py-4 whitespace-nowrap">
                  <span className={`px-2 py-1 text-xs rounded ${
                    user.role === "admin" ? "bg-red-100 text-red-800" :
                    user.role === "user" ? "bg-blue-100 text-blue-800" :
                    "bg-gray-100 text-gray-800"
                  }`}>
                    {user.role}
                  </span>
                </td>
                <td className="px-6 py-4 whitespace-nowrap">
                  <UserRoleManager userId={user.id} currentRole={user.role} />
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  )
}
```

```tsx
// src/components/admin/UserRoleManager.tsx
"use client"

import { useState } from "react"

export default function UserRoleManager({
  userId,
  currentRole,
}: {
  userId: string
  currentRole: string
}) {
  const [role, setRole] = useState(currentRole)
  const [loading, setLoading] = useState(false)

  const handleRoleChange = async (newRole: string) => {
    setLoading(true)
    try {
      const response = await fetch(`/api/users/${userId}/role`, {
        method: "PUT",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ role: newRole }),
      })

      if (response.ok) {
        setRole(newRole)
        alert("Role updated successfully")
      } else {
        alert("Failed to update role")
      }
    } catch (error) {
      console.error("Error updating role:", error)
      alert("An error occurred")
    } finally {
      setLoading(false)
    }
  }

  return (
    <select
      value={role}
      onChange={(e) => handleRoleChange(e.target.value)}
      di
