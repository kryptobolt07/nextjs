## What is Next.js?

Next.js is a powerful React framework developed by Vercel that enables you to build full-stack web applications with modern features like server-side rendering (SSR), static site generation (SSG), API routes, and more. The current stable version is Next.js 15, which now requires Node.js 20.9 or higher and includes React 19 support with improved performance through Turbopack bundling.

## System Requirements

Before starting, ensure your development environment meets these requirements:

- **Node.js**: Version 20.9 or higher
- **Operating Systems**: macOS, Windows (including WSL), or Linux
- **Browser Support**: Chrome 111+, Edge 111+, Firefox 111+, Safari 16.4+

## Installation Steps

### Method 1: Automatic Setup (Recommended)

The quickest way to create a new Next.js application is using `create-next-app`:

```bash
npx create-next-app@latest my-app
```

During installation, you'll be prompted with configuration questions:

```
✔ What is your project named? … my-app
✔ Would you like to use TypeScript? … Yes
✔ Would you like to use ESLint? … Yes
✔ Would you like to use Tailwind CSS? … Yes
✔ Would you like to use `src/` directory? … Yes
✔ Would you like to use App Router? (recommended) … Yes
✔ Would you like to customize the default import alias (@/*)? … No
```

**Recommended selections:**
- **TypeScript**: Yes (for type safety)
- **ESLint**: Yes (for code quality)
- **Tailwind CSS**: Yes (we'll use it for styling)
- **src/ directory**: Yes (for better organization)
- **App Router**: Yes (the modern, recommended approach)
- **Import alias**: No (default @/* is fine)

For a faster setup that accepts all defaults, you can use: `npx create-next-app@latest my-app --yes`

### Method 2: Manual Setup

If you prefer manual installation:

```bash
# Create project directory
mkdir my-app
cd my-app

# Initialize package.json
npm init -y

# Install Next.js and React
npm install next@latest react@latest react-dom@latest

# Install TypeScript dependencies (optional)
npm install --save-dev typescript @types/react @types/node
```

Add these scripts to your `package.json`:

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  }
}
```

## Project Structure

After installation, your Next.js project will have this structure:

```
my-app/
├── src/
│   └── app/
│       ├── layout.tsx       # Root layout (required)
│       ├── page.tsx         # Home page
│       └── globals.css      # Global styles
├── public/                  # Static assets
├── node_modules/
├── package.json
├── tsconfig.json           # TypeScript config
├── next.config.js          # Next.js config
├── tailwind.config.js      # Tailwind config
└── postcss.config.js       # PostCSS config
```

### Key Files Explained

**`src/app/layout.tsx`** - The root layout component that wraps all pages:

```tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  )
}
```

**`src/app/page.tsx`** - Your home page component:

```tsx
export default function Home() {
  return (
    <main>
      <h1>Welcome to Next.js!</h1>
    </main>
  )
}
```

**`public/`** - Store static assets like images, fonts, etc. Files here are served from the root path (`/`).

## Running Your Application

Start the development server:

```bash
npm run dev
```

Your application will be available at `http://localhost:3000`

The development server now uses Turbopack by default for faster builds and hot module replacement.

## App Router vs Pages Router

Next.js offers two routing systems:

**App Router** (Recommended - Modern):
- Uses the `app/` directory
- Server Components by default
- Built-in layouts and templates
- Improved data fetching
- Native support for React Server Components

**Pages Router** (Legacy):
- Uses the `pages/` directory
- All components are Client Components by default
- Simpler mental model for beginners

For this tutorial series, we'll use the **App Router** as it's the recommended approach with more features.

## Understanding File-Based Routing

Next.js uses file-system based routing. The file structure in your `app/` directory determines your routes:

```
app/
├── page.tsx              → /
├── about/
│   └── page.tsx          → /about
├── blog/
│   ├── page.tsx          → /blog
│   └── [slug]/
│       └── page.tsx      → /blog/:slug
└── dashboard/
    ├── layout.tsx        → Shared layout for dashboard/*
    └── page.tsx          → /dashboard
```

## Key Next.js Concepts

**1. Server Components (Default)**
Components in the App Router are Server Components by default, rendered on the server for better performance.

**2. Client Components**
Add `"use client"` at the top of a file to make it a Client Component (needed for interactivity, hooks, browser APIs).

**3. Layouts**
Shared UI that wraps multiple pages. Layouts preserve state and don't re-render on navigation.

**4. Templates**
Similar to layouts but create new instances on navigation.

**5. Loading UI**
Create a `loading.tsx` file to show loading states automatically.

**6. Error Handling**
Create an `error.tsx` file to handle errors in route segments.

## Next Steps

Now that you have Next.js installed and understand the basics, in the next section we'll dive into **Styling and UI Design**, where we'll:
- Configure CSS Modules for component-level styling
- Set up and use Tailwind CSS effectively
- Create a global layout component
- Build responsive designs

Your Next.js project is ready! Feel free to explore the default files, make changes, and see the hot reload in action.

---

**Quick Tips:**
- Press `Ctrl+C` in terminal to stop the dev server
- Changes auto-refresh in the browser
- Check `http://localhost:3000` after starting the dev server
- Use the browser console to debug issues

Let me know when you're ready to move to the next topic: **Styling and UI Design**!
