# Styling and UI Design in Next.js

In this section, we'll cover how to use **CSS Modules** for component-level styling and **Tailwind CSS** for responsive design, along with creating a global layout component for consistent UI across your application.

## Overview of Styling Options in Next.js

Next.js provides several ways to style your application, including Tailwind CSS, CSS Modules, Global CSS, External Stylesheets, Sass, and CSS-in-JS. We'll focus on the two most popular and complementary approaches:

1. **Tailwind CSS** - Utility-first CSS framework for rapid UI development
2. **CSS Modules** - Scoped CSS for component-specific custom styling

## Part 1: Setting Up Tailwind CSS

If you selected Tailwind CSS during project initialization, it's already configured. If not, let's set it up manually.

### Installing Tailwind CSS

Install Tailwind CSS with the command: `pnpm add -D tailwindcss @tailwindcss/postcss`

```bash
npm install -D tailwindcss @tailwindcss/postcss
```

### Configuring Tailwind CSS

Add the PostCSS plugin to your `postcss.config.mjs` file:

```javascript
// postcss.config.mjs
export default {
  plugins: {
    '@tailwindcss/postcss': {},
  },
}
```

### Import Tailwind in Global CSS

Import Tailwind in your global CSS file:

```css
/* src/app/globals.css */
@import 'tailwindcss';
```

### Import CSS in Root Layout

Import the CSS file in your root layout:

```tsx
// src/app/layout.tsx
import './globals.css'

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

Now you can start using Tailwind's utility classes throughout your application!

### Testing Tailwind

Update your home page to verify Tailwind is working:

```tsx
// src/app/page.tsx
export default function Home() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-center bg-gradient-to-br from-blue-50 to-indigo-100 p-24">
      <h1 className="text-5xl font-bold text-blue-600 mb-4">
        Welcome to Next.js with Tailwind
      </h1>
      <p className="text-lg text-gray-700 text-center max-w-2xl">
        Build fast, responsive interfaces with utility-first styling
      </p>
    </main>
  )
}
```

## Part 2: Using CSS Modules

CSS Modules are built into Next.js. Files with a `.module.css` extension are automatically processed so that class names become locally scoped, avoiding collisions and letting you reuse class names across files without worrying about conflicts.

### Creating a CSS Module

Let's create a reusable Button component with CSS Modules:

```css
/* src/components/Button.module.css */
.button {
  padding: 0.75rem 1.5rem;
  border-radius: 0.5rem;
  font-weight: 600;
  transition: all 0.2s;
  border: none;
  cursor: pointer;
}

.primary {
  background-color: #2563eb;
  color: white;
}

.primary:hover {
  background-color: #1d4ed8;
  transform: translateY(-2px);
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.secondary {
  background-color: #6b7280;
  color: white;
}

.secondary:hover {
  background-color: #4b5563;
}

.outline {
  background-color: transparent;
  color: #2563eb;
  border: 2px solid #2563eb;
}

.outline:hover {
  background-color: #2563eb;
  color: white;
}
```

### Using CSS Modules in Components

```tsx
// src/components/Button.tsx
import styles from './Button.module.css'

type ButtonProps = {
  children: React.ReactNode
  variant?: 'primary' | 'secondary' | 'outline'
  onClick?: () => void
}

export default function Button({ 
  children, 
  variant = 'primary',
  onClick 
}: ButtonProps) {
  return (
    <button 
      className={`${styles.button} ${styles[variant]}`}
      onClick={onClick}
    >
      {children}
    </button>
  )
}
```

### Combining CSS Modules with Tailwind

You can use both approaches together! The recommended approach is to use global styles for truly global CSS like Tailwind's base styles, Tailwind CSS for component styling, and CSS Modules for custom scoped CSS when needed.

```tsx
// src/components/Card.tsx
import styles from './Card.module.css'

export default function Card({ children }: { children: React.ReactNode }) {
  return (
    <div className={`${styles.card} p-6 rounded-lg shadow-md`}>
      {children}
    </div>
  )
}
```

```css
/* src/components/Card.module.css */
.card {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.card:hover {
  transform: scale(1.02);
  transition: transform 0.3s ease;
}
```

## Part 3: Responsive Design with Tailwind

Tailwind makes responsive design incredibly easy with its mobile-first breakpoint system.

### Breakpoint Reference

```
sm: 640px   - Small devices
md: 768px   - Medium devices  
lg: 1024px  - Large devices
xl: 1280px  - Extra large devices
2xl: 1536px - 2X large devices
```

### Responsive Example

```tsx
// src/components/Hero.tsx
export default function Hero() {
  return (
    <section className="
      px-4 py-8           /* Mobile: small padding */
      md:px-8 md:py-16   /* Tablet: medium padding */
      lg:px-16 lg:py-24  /* Desktop: large padding */
    ">
      <h1 className="
        text-3xl           /* Mobile: 3xl size */
        md:text-5xl        /* Tablet: 5xl size */
        lg:text-6xl        /* Desktop: 6xl size */
        font-bold 
        text-center
      ">
        Responsive Hero Section
      </h1>
      
      <div className="
        grid 
        grid-cols-1        /* Mobile: 1 column */
        md:grid-cols-2     /* Tablet: 2 columns */
        lg:grid-cols-3     /* Desktop: 3 columns */
        gap-6 
        mt-8
      ">
        {/* Grid items */}
      </div>
    </section>
  )
}
```

## Part 4: Creating a Global Layout Component

Let's create a comprehensive global layout with header, footer, and main content area.

### Layout Structure

```tsx
// src/components/layout/Layout.tsx
import Header from './Header'
import Footer from './Footer'

export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex flex-col min-h-screen">
      <Header />
      <main className="flex-grow container mx-auto px-4 py-8 max-w-7xl">
        {children}
      </main>
      <Footer />
    </div>
  )
}
```

### Header Component

```tsx
// src/components/layout/Header.tsx
import Link from 'next/link'

export default function Header() {
  return (
    <header className="bg-white shadow-md sticky top-0 z-50">
      <nav className="container mx-auto px-4 py-4 flex items-center justify-between max-w-7xl">
        <Link href="/" className="text-2xl font-bold text-blue-600">
          MyApp
        </Link>
        
        <ul className="hidden md:flex space-x-8">
          <li>
            <Link href="/" className="text-gray-700 hover:text-blue-600 transition">
              Home
            </Link>
          </li>
          <li>
            <Link href="/about" className="text-gray-700 hover:text-blue-600 transition">
              About
            </Link>
          </li>
          <li>
            <Link href="/blog" className="text-gray-700 hover:text-blue-600 transition">
              Blog
            </Link>
          </li>
          <li>
            <Link href="/contact" className="text-gray-700 hover:text-blue-600 transition">
              Contact
            </Link>
          </li>
        </ul>
        
        {/* Mobile menu button */}
        <button className="md:hidden text-gray-700">
          <svg className="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M4 6h16M4 12h16M4 18h16" />
          </svg>
        </button>
      </nav>
    </header>
  )
}
```

### Footer Component

```tsx
// src/components/layout/Footer.tsx
export default function Footer() {
  return (
    <footer className="bg-gray-800 text-white mt-auto">
      <div className="container mx-auto px-4 py-8 max-w-7xl">
        <div className="grid grid-cols-1 md:grid-cols-3 gap-8">
          <div>
            <h3 className="text-xl font-bold mb-4">About Us</h3>
            <p className="text-gray-400">
              Building amazing web experiences with Next.js and Tailwind CSS.
            </p>
          </div>
          
          <div>
            <h3 className="text-xl font-bold mb-4">Quick Links</h3>
            <ul className="space-y-2">
              <li><a href="/about" className="text-gray-400 hover:text-white transition">About</a></li>
              <li><a href="/blog" className="text-gray-400 hover:text-white transition">Blog</a></li>
              <li><a href="/contact" className="text-gray-400 hover:text-white transition">Contact</a></li>
            </ul>
          </div>
          
          <div>
            <h3 className="text-xl font-bold mb-4">Connect</h3>
            <div className="flex space-x-4">
              <a href="#" className="text-gray-400 hover:text-white transition">Twitter</a>
              <a href="#" className="text-gray-400 hover:text-white transition">GitHub</a>
              <a href="#" className="text-gray-400 hover:text-white transition">LinkedIn</a>
            </div>
          </div>
        </div>
        
        <div className="border-t border-gray-700 mt-8 pt-8 text-center text-gray-400">
          <p>&copy; 2025 MyApp. All rights reserved.</p>
        </div>
      </div>
    </footer>
  )
}
```

### Applying Layout to Your App

Update your root layout to use the Layout component:

```tsx
// src/app/layout.tsx
import './globals.css'
import Layout from '@/components/layout/Layout'

export const metadata = {
  title: 'My Next.js App',
  description: 'Built with Next.js and Tailwind CSS',
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>
        <Layout>{children}</Layout>
      </body>
    </html>
  )
}
```

## Best Practices

1. **Use Tailwind for most styling** - It covers common design patterns efficiently
2. **Use CSS Modules for complex custom styles** - When Tailwind utilities aren't sufficient
3. **Maintain consistent naming** - Use descriptive names for CSS Module classes
4. **Mobile-first approach** - Start with mobile styles, then add larger breakpoints
5. **Reusable components** - Extract common UI patterns into reusable components
6. **Avoid inline styles** - Use Tailwind classes or CSS Modules instead

## CSS Optimization

Next.js automatically chunks and merges stylesheets during production builds, ensuring minimal CSS is loaded for each route. In development, CSS updates apply instantly with Fast Refresh.

## Summary

You now have:
- ✅ Tailwind CSS configured and ready to use
- ✅ CSS Modules for component-specific styling
- ✅ A responsive global layout with header and footer
- ✅ Understanding of responsive design patterns
- ✅ Best practices for combining both approaches

**Next topic**: We'll dive into **Data Fetching and API Routes**, where you'll learn about SSR, SSG, CSR, and how to create your own API endpoints!

Ready to continue? Let me know when you want to move to the next topic!
