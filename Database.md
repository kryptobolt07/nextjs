# Database Integration with Next.js

In this comprehensive guide, we'll integrate both **Prisma ORM** (for PostgreSQL/MySQL) and **Mongoose** (for MongoDB) with your Next.js 15 application, implementing full CRUD operations for both databases.

## Part 1: Prisma ORM with PostgreSQL/MySQL

### What is Prisma?

Prisma is a next-generation ORM that provides type-safe database access with an intuitive data model, automated migrations, and powerful query capabilities. It offers excellent TypeScript support and works seamlessly with Next.js.

### Installing Prisma

Install Prisma CLI as a dev dependency and the Prisma Client as a production dependency:

```bash
npm install prisma --save-dev
npm install @prisma/client
npm install tsx --save-dev
```

### Initializing Prisma

Initialize Prisma in your project:

```bash
npx prisma init
```

This creates:
- `prisma/schema.prisma` - Your database schema
- `.env` - Environment variables file with DATABASE_URL

### Setting Up PostgreSQL Database

For this tutorial, you can use either a local PostgreSQL installation or a cloud service like [Neon](https://neon.tech/), [Supabase](https://supabase.com/), or [Railway](https://railway.app/).

Update your `.env` file:

```env
# PostgreSQL
DATABASE_URL="postgresql://username:password@localhost:5432/mydb?schema=public"

# Or MySQL
# DATABASE_URL="mysql://username:password@localhost:3306/mydb"
```

### Creating Your Schema

Update `prisma/schema.prisma`:

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"  // or "mysql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(uuid())
  name      String
  email     String   @unique
  posts     Post[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Post {
  id        String   @id @default(uuid())
  title     String
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

### Running Migrations

Create and apply your first migration:

```bash
# Create migration
npx prisma migrate dev --name init

# This will:
# 1. Create SQL migration files
# 2. Apply migrations to your database
# 3. Generate Prisma Client
```

### Setting Up Prisma Client (Singleton Pattern)

Next.js's hot-reloading feature can lead to multiple instances of Prisma Client being created, which consumes resources and might cause unexpected behavior. Use the singleton pattern to prevent this:

```typescript
// src/lib/prisma.ts
import { PrismaClient } from '@prisma/client'

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: ['query', 'error', 'warn'],
  })

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma
```

### Seeding Your Database

Prisma has built-in support for seeding your database with initial data:

```typescript
// prisma/seed.ts
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  // Clear existing data
  await prisma.post.deleteMany()
  await prisma.user.deleteMany()

  // Create users with posts
  const alice = await prisma.user.create({
    data: {
      name: 'Alice',
      email: 'alice@example.com',
      posts: {
        create: [
          {
            title: 'Getting Started with Prisma',
            content: 'Prisma makes database access easy...',
            published: true,
          },
          {
            title: 'Next.js and Prisma',
            content: 'Building full-stack apps...',
            published: false,
          },
        ],
      },
    },
  })

  const bob = await prisma.user.create({
    data: {
      name: 'Bob',
      email: 'bob@example.com',
      posts: {
        create: [
          {
            title: 'TypeScript Best Practices',
            content: 'Type safety is important...',
            published: true,
          },
        ],
      },
    },
  })

  console.log({ alice, bob })
}

main()
  .catch((e) => {
    console.error(e)
    process.exit(1)
  })
  .finally(async () => {
    await prisma.$disconnect()
  })
```

Add seed script to `package.json`:

```json
{
  "prisma": {
    "seed": "tsx prisma/seed.ts"
  }
}
```

Run the seed:

```bash
npx prisma db seed
```

### CRUD Operations with Prisma

#### 1. CREATE - Adding New Records

```typescript
// src/app/api/users/route.ts
import { prisma } from '@/lib/prisma'
import { NextRequest } from 'next/server'

export async function POST(request: NextRequest) {
  try {
    const body = await request.json()
    const { name, email } = body

    const user = await prisma.user.create({
      data: {
        name,
        email,
      },
    })

    return Response.json(user, { status: 201 })
  } catch (error: any) {
    return Response.json(
      { error: error.message },
      { status: 500 }
    )
  }
}
```

#### 2. READ - Fetching Records

```typescript
// src/app/api/users/route.ts (continued)
export async function GET(request: NextRequest) {
  try {
    const searchParams = request.nextUrl.searchParams
    const email = searchParams.get('email')

    if (email) {
      // Find specific user
      const user = await prisma.user.findUnique({
        where: { email },
        include: {
          posts: true, // Include related posts
        },
      })

      return Response.json(user)
    }

    // Get all users
    const users = await prisma.user.findMany({
      include: {
        posts: {
          where: {
            published: true,
          },
        },
      },
      orderBy: {
        createdAt: 'desc',
      },
    })

    return Response.json(users)
  } catch (error: any) {
    return Response.json(
      { error: error.message },
      { status: 500 }
    )
  }
}
```

#### 3. UPDATE - Modifying Records

```typescript
// src/app/api/users/[id]/route.ts
import { prisma } from '@/lib/prisma'

export async function PUT(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  try {
    const { id } = await params
    const body = await request.json()
    const { name, email } = body

    const user = await prisma.user.update({
      where: { id },
      data: {
        name,
        email,
      },
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

#### 4. DELETE - Removing Records

```typescript
// src/app/api/users/[id]/route.ts (continued)
export async function DELETE(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  try {
    const { id } = await params

    await prisma.user.delete({
      where: { id },
    })

    return Response.json(
      { message: 'User deleted successfully' },
      { status: 200 }
    )
  } catch (error: any) {
    return Response.json(
      { error: error.message },
      { status: 500 }
    )
  }
}
```

### Advanced Prisma Queries

```typescript
// Complex queries with filtering, pagination, and relations
export async function GET() {
  // Pagination
  const posts = await prisma.post.findMany({
    skip: 0,
    take: 10,
    where: {
      published: true,
      title: {
        contains: 'Prisma',
        mode: 'insensitive',
      },
    },
    include: {
      author: {
        select: {
          name: true,
          email: true,
        },
      },
    },
    orderBy: {
      createdAt: 'desc',
    },
  })

  // Aggregate queries
  const stats = await prisma.post.aggregate({
    _count: true,
    where: {
      published: true,
    },
  })

  // Group by
  const userPostCounts = await prisma.post.groupBy({
    by: ['authorId'],
    _count: {
      id: true,
    },
  })

  return Response.json({ posts, stats, userPostCounts })
}
```

### Using Prisma in Server Components

```tsx
// src/app/users/page.tsx
import { prisma } from '@/lib/prisma'

export default async function UsersPage() {
  const users = await prisma.user.findMany({
    include: {
      posts: true,
    },
  })

  return (
    <div className="container mx-auto p-8">
      <h1 className="text-3xl font-bold mb-6">Users</h1>
      <div className="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
        {users.map((user) => (
          <div key={user.id} className="border rounded-lg p-6">
            <h2 className="text-xl font-semibold">{user.name}</h2>
            <p className="text-gray-600">{user.email}</p>
            <p className="text-sm text-gray-500 mt-2">
              {user.posts.length} posts
            </p>
          </div>
        ))}
      </div>
    </div>
  )
}
```

## Part 2: Mongoose with MongoDB

### What is Mongoose?

Mongoose is an Object Data Modeling (ODM) library for MongoDB and Node.js. It provides schema validation, middleware, and a more structured way to work with MongoDB.

### Installing Mongoose

```bash
npm install mongoose
```

### Setting Up MongoDB

You can use:
1. **MongoDB Atlas** (cloud) - Free tier available at [mongodb.com/cloud/atlas](https://www.mongodb.com/cloud/atlas)
2. **Local MongoDB** installation
3. **Docker**: `docker run -d -p 27017:27017 --name mongodb mongo`

### Creating Database Connection

Create a utility file to manage the connection between your Next.js app and MongoDB, ensuring MongoDB connects efficiently in a serverless environment:

```typescript
// src/lib/mongodb.ts
import mongoose from 'mongoose'

const MONGODB_URI = process.env.MONGODB_URI

if (!MONGODB_URI) {
  throw new Error('Please define the MONGODB_URI environment variable')
}

interface MongooseCache {
  conn: typeof mongoose | null
  promise: Promise<typeof mongoose> | null
}

declare global {
  var mongoose: MongooseCache | undefined
}

let cached: MongooseCache = global.mongoose || {
  conn: null,
  promise: null,
}

if (!global.mongoose) {
  global.mongoose = cached
}

async function dbConnect(): Promise<typeof mongoose> {
  if (cached.conn) {
    return cached.conn
  }

  if (!cached.promise) {
    const opts = {
      bufferCommands: false,
    }

    cached.promise = mongoose.connect(MONGODB_URI!, opts).then((mongoose) => {
      console.log('✅ MongoDB connected')
      return mongoose
    })
  }

  try {
    cached.conn = await cached.promise
  } catch (e) {
    cached.promise = null
    throw e
  }

  return cached.conn
}

export default dbConnect
```

### Environment Variables

Add to `.env.local`:

```env
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/mydb?retryWrites=true&w=majority
```

### Creating Mongoose Models

```typescript
// src/models/Product.ts
import mongoose, { Schema, Document, Model } from 'mongoose'

export interface IProduct extends Document {
  name: string
  price: number
  description?: string
  category: string
  inStock: boolean
  createdAt: Date
  updatedAt: Date
}

const ProductSchema = new Schema<IProduct>(
  {
    name: {
      type: String,
      required: [true, 'Product name is required'],
      trim: true,
      maxlength: [100, 'Name cannot exceed 100 characters'],
    },
    price: {
      type: Number,
      required: [true, 'Price is required'],
      min: [0, 'Price cannot be negative'],
    },
    description: {
      type: String,
      maxlength: [500, 'Description cannot exceed 500 characters'],
    },
    category: {
      type: String,
      required: true,
      enum: ['Electronics', 'Clothing', 'Books', 'Food', 'Other'],
    },
    inStock: {
      type: Boolean,
      default: true,
    },
  },
  {
    timestamps: true,
  }
)

// Prevent model recompilation during hot reload
const Product: Model<IProduct> =
  mongoose.models.Product || mongoose.model<IProduct>('Product', ProductSchema)

export default Product
```

### CRUD Operations with Mongoose

#### 1. CREATE - Adding Products

```typescript
// src/app/api/products/route.ts
import dbConnect from '@/lib/mongodb'
import Product from '@/models/Product'
import { NextRequest } from 'next/server'

export async function POST(request: NextRequest) {
  try {
    await dbConnect()

    const body = await request.json()
    const product = await Product.create(body)

    return Response.json(product, { status: 201 })
  } catch (error: any) {
    return Response.json(
      { error: error.message },
      { status: 400 }
    )
  }
}
```

#### 2. READ - Fetching Products

```typescript
// src/app/api/products/route.ts (continued)
export async function GET(request: NextRequest) {
  try {
    await dbConnect()

    const searchParams = request.nextUrl.searchParams
    const category = searchParams.get('category')
    const inStock = searchParams.get('inStock')

    // Build query
    const query: any = {}
    if (category) query.category = category
    if (inStock !== null) query.inStock = inStock === 'true'

    const products = await Product.find(query)
      .sort({ createdAt: -1 })
      .limit(20)
      .lean() // Returns plain JavaScript objects (faster)

    return Response.json(products)
  } catch (error: any) {
    return Response.json(
      { error: error.message },
      { status: 500 }
    )
  }
}
```

#### 3. UPDATE - Modifying Products

```typescript
// src/app/api/products/[id]/route.ts
import dbConnect from '@/lib/mongodb'
import Product from '@/models/Product'

export async function PUT(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  try {
    await dbConnect()

    const { id } = await params
    const body = await request.json()

    const product = await Product.findByIdAndUpdate(
      id,
      body,
      {
        new: true, // Return updated document
        runValidators: true, // Run schema validators
      }
    )

    if (!product) {
      return Response.json(
        { error: 'Product not found' },
        { status: 404 }
      )
    }

    return Response.json(product)
  } catch (error: any) {
    return Response.json(
      { error: error.message },
      { status: 400 }
    )
  }
}
```

#### 4. DELETE - Removing Products

```typescript
// src/app/api/products/[id]/route.ts (continued)
export async function DELETE(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  try {
    await dbConnect()

    const { id } = await params
    const product = await Product.findByIdAndDelete(id)

    if (!product) {
      return Response.json(
        { error: 'Product not found' },
        { status: 404 }
      )
    }

    return Response.json({
      message: 'Product deleted successfully',
      product,
    })
  } catch (error: any) {
    return Response.json(
      { error: error.message },
      { status: 500 }
    )
  }
}
```

### Advanced Mongoose Queries

```typescript
// Complex queries with Mongoose
export async function GET() {
  await dbConnect()

  // Text search
  const searchResults = await Product.find({
    $text: { $search: 'laptop' },
  })

  // Aggregation
  const stats = await Product.aggregate([
    {
      $group: {
        _id: '$category',
        count: { $sum: 1 },
        avgPrice: { $avg: '$price' },
        totalValue: { $sum: '$price' },
      },
    },
    {
      $sort: { totalValue: -1 },
    },
  ])

  // Pagination
  const page = 1
  const limit = 10
  const products = await Product.find()
    .skip((page - 1) * limit)
    .limit(limit)

  // Count
  const total = await Product.countDocuments()

  return Response.json({ searchResults, stats, products, total })
}
```

### Using Mongoose in Server Components

With Next.js 13+ App Router, you can use Mongoose directly in Server Components:

```tsx
// src/app/products/page.tsx
import dbConnect from '@/lib/mongodb'
import Product from '@/models/Product'

export const runtime = 'nodejs'

export default async function ProductsPage() {
  await dbConnect()

  const products = await Product.find({ inStock: true })
    .sort({ createdAt: -1 })
    .limit(12)
    .lean()

  return (
    <div className="container mx-auto p-8">
      <h1 className="text-3xl font-bold mb-6">Products</h1>
      <div className="grid gap-6 md:grid-cols-3 lg:grid-cols-4">
        {products.map((product: any) => (
          <div key={product._id.toString()} className="border rounded-lg p-4">
            <h2 className="text-lg font-semibold">{product.name}</h2>
            <p className="text-gray-600">${product.price.toFixed(2)}</p>
            <span className="text-sm text-gray-500">{product.category}</span>
          </div>
        ))}
      </div>
    </div>
  )
}
```

## Comparison: Prisma vs Mongoose

| Feature | Prisma | Mongoose |
|---------|--------|----------|
| Database | SQL (PostgreSQL, MySQL, SQLite) | MongoDB |
| Type Safety | Excellent (auto-generated types) | Good (with TypeScript) |
| Migrations | Built-in | Manual |
| Learning Curve | Easy | Moderate |
| Query Builder | Intuitive | Flexible |
| Relations | First-class support | Manual (refs/populate) |

## Best Practices

1. **Always use connection singleton pattern** to prevent connection exhaustion
2. **Use `.lean()` in Mongoose** when you don't need Mongoose document methods
3. **Add proper indexes** for frequently queried fields
4. **Use transactions** for operations that need atomicity
5. **Validate input** before database operations
6. **Handle errors gracefully** and return appropriate status codes
7. **Use environment variables** for sensitive data

## Summary

You now have:

- ✅ **Prisma ORM** set up with PostgreSQL/MySQL
- ✅ **Mongoose** configured with MongoDB
- ✅ Complete **CRUD operations** for both databases
- ✅ Proper **connection management** using singleton pattern
- ✅ **Type-safe** database operations
- ✅ **Advanced querying** capabilities
- ✅ Integration with **Server Components** and **API Routes**

**Next topic**: We'll implement **Authentication and Authorization** using NextAuth.js with Google and GitHub providers, plus role-based access control!

Ready to continue? 🚀
