# Day 10: Capstone Project - E-Commerce API with Full Database Stack

## Introduction

Today we'll build a production-ready e-commerce API that combines everything you've learned about databases. We'll use PostgreSQL with Prisma for the primary database, Redis for caching and sessions, and implement proper data modeling, optimization, and caching strategies.

## Learning Objectives

By the end of this project, you will have:
- Designed a complete e-commerce database schema
- Implemented CRUD operations with Prisma
- Added Redis caching for performance
- Built session management with Redis
- Created efficient queries with proper indexing
- Implemented common e-commerce patterns

---

## Project Overview

```
E-Commerce API
├── Database: PostgreSQL (via Prisma)
│   ├── Users & Authentication
│   ├── Products & Categories
│   ├── Shopping Cart
│   ├── Orders & Order Items
│   └── Reviews
├── Caching: Redis
│   ├── Product catalog caching
│   ├── Session management
│   ├── Cart data
│   └── Rate limiting
└── Features
    ├── User registration/login
    ├── Product browsing with filters
    ├── Shopping cart management
    ├── Order processing
    └── Product reviews
```

---

## Project Setup

### Initialize Project

```bash
mkdir ecommerce-api && cd ecommerce-api
npm init -y

# Dependencies
npm install express prisma @prisma/client ioredis bcryptjs jsonwebtoken
npm install express-validator cors helmet morgan compression
npm install dotenv uuid

# Dev dependencies
npm install -D typescript ts-node @types/node @types/express
npm install -D @types/bcryptjs @types/jsonwebtoken nodemon

# Initialize TypeScript and Prisma
npx tsc --init
npx prisma init
```

### Project Structure

```
ecommerce-api/
├── prisma/
│   ├── schema.prisma
│   ├── seed.ts
│   └── migrations/
├── src/
│   ├── config/
│   │   ├── index.ts
│   │   └── database.ts
│   ├── lib/
│   │   ├── prisma.ts
│   │   └── redis.ts
│   ├── middleware/
│   │   ├── auth.ts
│   │   ├── cache.ts
│   │   ├── rateLimit.ts
│   │   └── errorHandler.ts
│   ├── modules/
│   │   ├── auth/
│   │   ├── users/
│   │   ├── products/
│   │   ├── cart/
│   │   ├── orders/
│   │   └── reviews/
│   ├── services/
│   │   └── cache.service.ts
│   ├── utils/
│   │   ├── response.ts
│   │   └── pagination.ts
│   ├── app.ts
│   └── server.ts
├── .env
└── package.json
```

---

## Step 1: Prisma Schema

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ==================== Users ====================

model User {
  id            String    @id @default(cuid())
  email         String    @unique
  password      String
  firstName     String
  lastName      String
  phone         String?
  role          Role      @default(CUSTOMER)
  isActive      Boolean   @default(true)
  emailVerified Boolean   @default(false)

  // Relations
  addresses     Address[]
  cart          Cart?
  orders        Order[]
  reviews       Review[]
  wishlist      WishlistItem[]

  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  @@index([email])
  @@map("users")
}

enum Role {
  CUSTOMER
  ADMIN
  SELLER
}

model Address {
  id          String   @id @default(cuid())
  userId      String
  user        User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  label       String?  // "Home", "Office", etc.
  firstName   String
  lastName    String
  street      String
  city        String
  state       String
  postalCode  String
  country     String
  phone       String?
  isDefault   Boolean  @default(false)

  orders      Order[]

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@index([userId])
  @@map("addresses")
}

// ==================== Products ====================

model Category {
  id          String     @id @default(cuid())
  name        String     @unique
  slug        String     @unique
  description String?
  image       String?
  parentId    String?
  parent      Category?  @relation("CategoryHierarchy", fields: [parentId], references: [id])
  children    Category[] @relation("CategoryHierarchy")
  products    Product[]
  isActive    Boolean    @default(true)

  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt

  @@index([slug])
  @@index([parentId])
  @@map("categories")
}

model Product {
  id          String   @id @default(cuid())
  name        String
  slug        String   @unique
  description String   @db.Text
  price       Decimal  @db.Decimal(10, 2)
  comparePrice Decimal? @db.Decimal(10, 2)
  sku         String   @unique
  barcode     String?

  // Inventory
  stock       Int      @default(0)
  lowStockThreshold Int @default(10)
  trackInventory Boolean @default(true)

  // Status
  status      ProductStatus @default(DRAFT)
  isFeatured  Boolean  @default(false)

  // SEO
  metaTitle   String?
  metaDescription String?

  // Relations
  categoryId  String
  category    Category @relation(fields: [categoryId], references: [id])
  images      ProductImage[]
  variants    ProductVariant[]
  reviews     Review[]
  cartItems   CartItem[]
  orderItems  OrderItem[]
  wishlistItems WishlistItem[]

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@index([slug])
  @@index([categoryId])
  @@index([status])
  @@index([isFeatured])
  @@index([price])
  @@map("products")
}

enum ProductStatus {
  DRAFT
  ACTIVE
  ARCHIVED
}

model ProductImage {
  id        String   @id @default(cuid())
  productId String
  product   Product  @relation(fields: [productId], references: [id], onDelete: Cascade)
  url       String
  alt       String?
  position  Int      @default(0)

  @@index([productId])
  @@map("product_images")
}

model ProductVariant {
  id        String   @id @default(cuid())
  productId String
  product   Product  @relation(fields: [productId], references: [id], onDelete: Cascade)

  name      String   // e.g., "Size", "Color"
  value     String   // e.g., "Large", "Red"
  sku       String?
  price     Decimal? @db.Decimal(10, 2)
  stock     Int      @default(0)

  @@index([productId])
  @@map("product_variants")
}

// ==================== Shopping Cart ====================

model Cart {
  id        String     @id @default(cuid())
  userId    String     @unique
  user      User       @relation(fields: [userId], references: [id], onDelete: Cascade)
  items     CartItem[]

  createdAt DateTime   @default(now())
  updatedAt DateTime   @updatedAt

  @@map("carts")
}

model CartItem {
  id        String   @id @default(cuid())
  cartId    String
  cart      Cart     @relation(fields: [cartId], references: [id], onDelete: Cascade)
  productId String
  product   Product  @relation(fields: [productId], references: [id])
  quantity  Int

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@unique([cartId, productId])
  @@index([cartId])
  @@map("cart_items")
}

// ==================== Orders ====================

model Order {
  id              String      @id @default(cuid())
  orderNumber     String      @unique
  userId          String
  user            User        @relation(fields: [userId], references: [id])

  // Pricing
  subtotal        Decimal     @db.Decimal(10, 2)
  discount        Decimal     @default(0) @db.Decimal(10, 2)
  shipping        Decimal     @default(0) @db.Decimal(10, 2)
  tax             Decimal     @default(0) @db.Decimal(10, 2)
  total           Decimal     @db.Decimal(10, 2)

  // Status
  status          OrderStatus @default(PENDING)
  paymentStatus   PaymentStatus @default(PENDING)
  paymentMethod   String?
  paymentId       String?

  // Shipping
  shippingAddressId String?
  shippingAddress Address?    @relation(fields: [shippingAddressId], references: [id])
  shippingMethod  String?
  trackingNumber  String?

  // Notes
  customerNote    String?
  adminNote       String?

  // Relations
  items           OrderItem[]

  createdAt       DateTime    @default(now())
  updatedAt       DateTime    @updatedAt

  @@index([userId])
  @@index([status])
  @@index([orderNumber])
  @@map("orders")
}

enum OrderStatus {
  PENDING
  CONFIRMED
  PROCESSING
  SHIPPED
  DELIVERED
  CANCELLED
  REFUNDED
}

enum PaymentStatus {
  PENDING
  PAID
  FAILED
  REFUNDED
}

model OrderItem {
  id          String   @id @default(cuid())
  orderId     String
  order       Order    @relation(fields: [orderId], references: [id], onDelete: Cascade)
  productId   String
  product     Product  @relation(fields: [productId], references: [id])

  quantity    Int
  price       Decimal  @db.Decimal(10, 2)
  total       Decimal  @db.Decimal(10, 2)

  // Snapshot of product at time of order
  productName String
  productSku  String

  @@index([orderId])
  @@map("order_items")
}

// ==================== Reviews ====================

model Review {
  id        String   @id @default(cuid())
  productId String
  product   Product  @relation(fields: [productId], references: [id], onDelete: Cascade)
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  rating    Int      // 1-5
  title     String?
  content   String   @db.Text
  isVerified Boolean @default(false) // Verified purchase
  isApproved Boolean @default(false)

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@unique([productId, userId])
  @@index([productId])
  @@index([userId])
  @@map("reviews")
}

// ==================== Wishlist ====================

model WishlistItem {
  id        String   @id @default(cuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  productId String
  product   Product  @relation(fields: [productId], references: [id], onDelete: Cascade)

  createdAt DateTime @default(now())

  @@unique([userId, productId])
  @@index([userId])
  @@map("wishlist_items")
}
```

---

## Step 2: Configuration

```typescript
// src/config/index.ts
import dotenv from 'dotenv';
dotenv.config();

export const config = {
  env: process.env.NODE_ENV || 'development',
  port: parseInt(process.env.PORT || '3000', 10),

  jwt: {
    secret: process.env.JWT_SECRET || 'your-secret-key',
    expiresIn: process.env.JWT_EXPIRES_IN || '7d',
  },

  redis: {
    host: process.env.REDIS_HOST || 'localhost',
    port: parseInt(process.env.REDIS_PORT || '6379', 10),
    password: process.env.REDIS_PASSWORD,
  },

  cache: {
    productTTL: 300,      // 5 minutes
    categoryTTL: 3600,    // 1 hour
    sessionTTL: 86400,    // 24 hours
  },
};
```

```typescript
// src/lib/prisma.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as { prisma: PrismaClient | undefined };

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
  });

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}

export default prisma;
```

```typescript
// src/lib/redis.ts
import Redis from 'ioredis';
import { config } from '../config';

const redis = new Redis({
  host: config.redis.host,
  port: config.redis.port,
  password: config.redis.password,
  maxRetriesPerRequest: 3,
});

redis.on('connect', () => console.log('Redis connected'));
redis.on('error', (err) => console.error('Redis error:', err));

export default redis;
```

---

## Step 3: Cache Service

```typescript
// src/services/cache.service.ts
import redis from '../lib/redis';

class CacheService {
  async get<T>(key: string): Promise<T | null> {
    const cached = await redis.get(key);
    return cached ? JSON.parse(cached) : null;
  }

  async set(key: string, value: any, ttl: number): Promise<void> {
    await redis.set(key, JSON.stringify(value), 'EX', ttl);
  }

  async del(key: string): Promise<void> {
    await redis.del(key);
  }

  async delPattern(pattern: string): Promise<void> {
    const keys = await redis.keys(pattern);
    if (keys.length > 0) {
      await redis.del(...keys);
    }
  }

  async getOrSet<T>(key: string, fetchFn: () => Promise<T>, ttl: number): Promise<T> {
    const cached = await this.get<T>(key);
    if (cached !== null) return cached;

    const data = await fetchFn();
    if (data !== null && data !== undefined) {
      await this.set(key, data, ttl);
    }
    return data;
  }

  // Product caching helpers
  productKey(id: string): string {
    return `product:${id}`;
  }

  productSlugKey(slug: string): string {
    return `product:slug:${slug}`;
  }

  productsListKey(params: Record<string, any>): string {
    const hash = JSON.stringify(params);
    return `products:list:${Buffer.from(hash).toString('base64')}`;
  }

  categoryKey(id: string): string {
    return `category:${id}`;
  }

  cartKey(userId: string): string {
    return `cart:${userId}`;
  }

  async invalidateProduct(productId: string, slug: string): Promise<void> {
    await Promise.all([
      this.del(this.productKey(productId)),
      this.del(this.productSlugKey(slug)),
      this.delPattern('products:list:*'),
    ]);
  }
}

export const cacheService = new CacheService();
```

---

## Step 4: Product Service

```typescript
// src/modules/products/product.service.ts
import prisma from '../../lib/prisma';
import { cacheService } from '../../services/cache.service';
import { config } from '../../config';
import { Prisma, ProductStatus } from '@prisma/client';

interface FindProductsParams {
  page?: number;
  pageSize?: number;
  category?: string;
  search?: string;
  minPrice?: number;
  maxPrice?: number;
  sortBy?: string;
  sortOrder?: 'asc' | 'desc';
  status?: ProductStatus;
}

export class ProductService {
  async findAll(params: FindProductsParams) {
    const {
      page = 1,
      pageSize = 20,
      category,
      search,
      minPrice,
      maxPrice,
      sortBy = 'createdAt',
      sortOrder = 'desc',
      status = ProductStatus.ACTIVE,
    } = params;

    const cacheKey = cacheService.productsListKey(params);

    return cacheService.getOrSet(
      cacheKey,
      async () => {
        const where: Prisma.ProductWhereInput = {
          status,
          ...(category && {
            category: { slug: category },
          }),
          ...(search && {
            OR: [
              { name: { contains: search, mode: 'insensitive' } },
              { description: { contains: search, mode: 'insensitive' } },
            ],
          }),
          ...(minPrice !== undefined || maxPrice !== undefined) && {
            price: {
              ...(minPrice !== undefined && { gte: minPrice }),
              ...(maxPrice !== undefined && { lte: maxPrice }),
            },
          },
        };

        const orderBy: Prisma.ProductOrderByWithRelationInput = {
          [sortBy]: sortOrder,
        };

        const [products, total] = await Promise.all([
          prisma.product.findMany({
            where,
            orderBy,
            skip: (page - 1) * pageSize,
            take: pageSize,
            include: {
              category: { select: { id: true, name: true, slug: true } },
              images: { orderBy: { position: 'asc' }, take: 1 },
              _count: { select: { reviews: true } },
            },
          }),
          prisma.product.count({ where }),
        ]);

        // Calculate average rating for each product
        const productsWithRating = await Promise.all(
          products.map(async (product) => {
            const avgRating = await prisma.review.aggregate({
              where: { productId: product.id, isApproved: true },
              _avg: { rating: true },
            });
            return {
              ...product,
              avgRating: avgRating._avg.rating || 0,
            };
          })
        );

        return {
          data: productsWithRating,
          meta: {
            total,
            page,
            pageSize,
            totalPages: Math.ceil(total / pageSize),
          },
        };
      },
      config.cache.productTTL
    );
  }

  async findBySlug(slug: string) {
    const cacheKey = cacheService.productSlugKey(slug);

    return cacheService.getOrSet(
      cacheKey,
      async () => {
        const product = await prisma.product.findUnique({
          where: { slug, status: ProductStatus.ACTIVE },
          include: {
            category: true,
            images: { orderBy: { position: 'asc' } },
            variants: true,
            reviews: {
              where: { isApproved: true },
              orderBy: { createdAt: 'desc' },
              take: 10,
              include: {
                user: { select: { firstName: true, lastName: true } },
              },
            },
            _count: { select: { reviews: true } },
          },
        });

        if (!product) return null;

        const avgRating = await prisma.review.aggregate({
          where: { productId: product.id, isApproved: true },
          _avg: { rating: true },
        });

        return {
          ...product,
          avgRating: avgRating._avg.rating || 0,
        };
      },
      config.cache.productTTL
    );
  }

  async create(data: Prisma.ProductCreateInput) {
    const product = await prisma.product.create({
      data,
      include: { category: true, images: true },
    });

    await cacheService.delPattern('products:list:*');
    return product;
  }

  async update(id: string, data: Prisma.ProductUpdateInput) {
    const product = await prisma.product.update({
      where: { id },
      data,
      include: { category: true, images: true },
    });

    await cacheService.invalidateProduct(id, product.slug);
    return product;
  }

  async updateStock(id: string, quantity: number) {
    const product = await prisma.product.update({
      where: { id },
      data: {
        stock: { decrement: quantity },
      },
    });

    // Check low stock
    if (product.stock <= product.lowStockThreshold) {
      // Trigger low stock notification
      console.log(`Low stock alert: ${product.name} (${product.stock} remaining)`);
    }

    await cacheService.invalidateProduct(id, product.slug);
    return product;
  }
}

export const productService = new ProductService();
```

---

## Step 5: Cart Service

```typescript
// src/modules/cart/cart.service.ts
import prisma from '../../lib/prisma';
import redis from '../../lib/redis';
import { cacheService } from '../../services/cache.service';

interface CartItem {
  productId: string;
  quantity: number;
}

export class CartService {
  // Get cart (prefer Redis, fallback to DB)
  async getCart(userId: string) {
    const cacheKey = cacheService.cartKey(userId);

    // Try Redis first
    const cachedCart = await redis.get(cacheKey);
    if (cachedCart) {
      return JSON.parse(cachedCart);
    }

    // Fallback to database
    const cart = await prisma.cart.findUnique({
      where: { userId },
      include: {
        items: {
          include: {
            product: {
              include: {
                images: { take: 1 },
              },
            },
          },
        },
      },
    });

    if (cart) {
      // Cache in Redis
      await this.cacheCart(userId, cart);
    }

    return cart;
  }

  async addToCart(userId: string, productId: string, quantity: number) {
    // Verify product exists and has stock
    const product = await prisma.product.findUnique({
      where: { id: productId },
    });

    if (!product) {
      throw new Error('Product not found');
    }

    if (product.stock < quantity) {
      throw new Error('Insufficient stock');
    }

    // Get or create cart
    let cart = await prisma.cart.findUnique({
      where: { userId },
    });

    if (!cart) {
      cart = await prisma.cart.create({
        data: { userId },
      });
    }

    // Add or update item
    const cartItem = await prisma.cartItem.upsert({
      where: {
        cartId_productId: {
          cartId: cart.id,
          productId,
        },
      },
      update: {
        quantity: { increment: quantity },
      },
      create: {
        cartId: cart.id,
        productId,
        quantity,
      },
    });

    // Invalidate cart cache
    await this.invalidateCartCache(userId);

    return this.getCart(userId);
  }

  async updateCartItem(userId: string, productId: string, quantity: number) {
    const cart = await prisma.cart.findUnique({
      where: { userId },
    });

    if (!cart) {
      throw new Error('Cart not found');
    }

    if (quantity <= 0) {
      await prisma.cartItem.delete({
        where: {
          cartId_productId: {
            cartId: cart.id,
            productId,
          },
        },
      });
    } else {
      await prisma.cartItem.update({
        where: {
          cartId_productId: {
            cartId: cart.id,
            productId,
          },
        },
        data: { quantity },
      });
    }

    await this.invalidateCartCache(userId);
    return this.getCart(userId);
  }

  async removeFromCart(userId: string, productId: string) {
    const cart = await prisma.cart.findUnique({
      where: { userId },
    });

    if (cart) {
      await prisma.cartItem.delete({
        where: {
          cartId_productId: {
            cartId: cart.id,
            productId,
          },
        },
      });

      await this.invalidateCartCache(userId);
    }

    return this.getCart(userId);
  }

  async clearCart(userId: string) {
    const cart = await prisma.cart.findUnique({
      where: { userId },
    });

    if (cart) {
      await prisma.cartItem.deleteMany({
        where: { cartId: cart.id },
      });

      await this.invalidateCartCache(userId);
    }
  }

  async getCartSummary(userId: string) {
    const cart = await this.getCart(userId);

    if (!cart || !cart.items.length) {
      return {
        items: [],
        itemCount: 0,
        subtotal: 0,
      };
    }

    const items = cart.items.map((item: any) => ({
      productId: item.productId,
      name: item.product.name,
      price: item.product.price,
      quantity: item.quantity,
      total: Number(item.product.price) * item.quantity,
      image: item.product.images[0]?.url,
    }));

    const subtotal = items.reduce((sum: number, item: any) => sum + item.total, 0);

    return {
      items,
      itemCount: items.length,
      subtotal,
    };
  }

  private async cacheCart(userId: string, cart: any) {
    const cacheKey = cacheService.cartKey(userId);
    await redis.set(cacheKey, JSON.stringify(cart), 'EX', 3600);
  }

  private async invalidateCartCache(userId: string) {
    const cacheKey = cacheService.cartKey(userId);
    await redis.del(cacheKey);
  }
}

export const cartService = new CartService();
```

---

## Step 6: Order Service

```typescript
// src/modules/orders/order.service.ts
import prisma from '../../lib/prisma';
import { cartService } from '../cart/cart.service';
import { productService } from '../products/product.service';
import { OrderStatus, PaymentStatus, Prisma } from '@prisma/client';
import { v4 as uuidv4 } from 'uuid';

interface CreateOrderInput {
  userId: string;
  shippingAddressId: string;
  paymentMethod: string;
  customerNote?: string;
}

export class OrderService {
  async createOrder(input: CreateOrderInput) {
    const { userId, shippingAddressId, paymentMethod, customerNote } = input;

    // Get cart with items
    const cart = await prisma.cart.findUnique({
      where: { userId },
      include: {
        items: {
          include: { product: true },
        },
      },
    });

    if (!cart || cart.items.length === 0) {
      throw new Error('Cart is empty');
    }

    // Validate stock for all items
    for (const item of cart.items) {
      if (item.product.stock < item.quantity) {
        throw new Error(`Insufficient stock for ${item.product.name}`);
      }
    }

    // Calculate totals
    const subtotal = cart.items.reduce(
      (sum, item) => sum + Number(item.product.price) * item.quantity,
      0
    );
    const shipping = 10; // Fixed shipping for simplicity
    const tax = subtotal * 0.1; // 10% tax
    const total = subtotal + shipping + tax;

    // Create order in transaction
    const order = await prisma.$transaction(async (tx) => {
      // Create order
      const newOrder = await tx.order.create({
        data: {
          orderNumber: `ORD-${Date.now()}-${uuidv4().substring(0, 8).toUpperCase()}`,
          userId,
          subtotal,
          shipping,
          tax,
          total,
          paymentMethod,
          customerNote,
          shippingAddressId,
          status: OrderStatus.PENDING,
          paymentStatus: PaymentStatus.PENDING,
          items: {
            create: cart.items.map((item) => ({
              productId: item.productId,
              quantity: item.quantity,
              price: item.product.price,
              total: Number(item.product.price) * item.quantity,
              productName: item.product.name,
              productSku: item.product.sku,
            })),
          },
        },
        include: {
          items: true,
          shippingAddress: true,
        },
      });

      // Update product stock
      for (const item of cart.items) {
        await tx.product.update({
          where: { id: item.productId },
          data: { stock: { decrement: item.quantity } },
        });
      }

      // Clear cart
      await tx.cartItem.deleteMany({
        where: { cartId: cart.id },
      });

      return newOrder;
    });

    // Clear cart cache
    await cartService.clearCart(userId);

    return order;
  }

  async findUserOrders(userId: string, page = 1, pageSize = 10) {
    const [orders, total] = await Promise.all([
      prisma.order.findMany({
        where: { userId },
        orderBy: { createdAt: 'desc' },
        skip: (page - 1) * pageSize,
        take: pageSize,
        include: {
          items: {
            include: {
              product: {
                include: { images: { take: 1 } },
              },
            },
          },
          shippingAddress: true,
        },
      }),
      prisma.order.count({ where: { userId } }),
    ]);

    return {
      data: orders,
      meta: {
        total,
        page,
        pageSize,
        totalPages: Math.ceil(total / pageSize),
      },
    };
  }

  async findById(id: string, userId?: string) {
    const where: Prisma.OrderWhereUniqueInput = { id };

    const order = await prisma.order.findUnique({
      where,
      include: {
        items: {
          include: {
            product: {
              include: { images: { take: 1 } },
            },
          },
        },
        shippingAddress: true,
        user: {
          select: { id: true, email: true, firstName: true, lastName: true },
        },
      },
    });

    if (!order) {
      throw new Error('Order not found');
    }

    // Check ownership if userId provided
    if (userId && order.userId !== userId) {
      throw new Error('Order not found');
    }

    return order;
  }

  async updateStatus(id: string, status: OrderStatus) {
    return prisma.order.update({
      where: { id },
      data: { status },
    });
  }

  async processPayment(orderId: string, paymentId: string) {
    return prisma.order.update({
      where: { id: orderId },
      data: {
        paymentStatus: PaymentStatus.PAID,
        paymentId,
        status: OrderStatus.CONFIRMED,
      },
    });
  }
}

export const orderService = new OrderService();
```

---

## Step 7: API Routes

```typescript
// src/modules/products/product.routes.ts
import { Router } from 'express';
import { productService } from './product.service';
import { authenticate, authorize } from '../../middleware/auth';
import { cacheMiddleware } from '../../middleware/cache';

const router = Router();

// Public routes
router.get('/', cacheMiddleware({ ttl: 60 }), async (req, res, next) => {
  try {
    const products = await productService.findAll({
      page: Number(req.query.page) || 1,
      pageSize: Number(req.query.pageSize) || 20,
      category: req.query.category as string,
      search: req.query.search as string,
      minPrice: req.query.minPrice ? Number(req.query.minPrice) : undefined,
      maxPrice: req.query.maxPrice ? Number(req.query.maxPrice) : undefined,
      sortBy: req.query.sortBy as string,
      sortOrder: req.query.sortOrder as 'asc' | 'desc',
    });

    res.json(products);
  } catch (error) {
    next(error);
  }
});

router.get('/:slug', cacheMiddleware({ ttl: 300 }), async (req, res, next) => {
  try {
    const product = await productService.findBySlug(req.params.slug);

    if (!product) {
      return res.status(404).json({ error: 'Product not found' });
    }

    res.json(product);
  } catch (error) {
    next(error);
  }
});

// Admin routes
router.post('/', authenticate, authorize('ADMIN'), async (req, res, next) => {
  try {
    const product = await productService.create(req.body);
    res.status(201).json(product);
  } catch (error) {
    next(error);
  }
});

export default router;
```

```typescript
// src/modules/cart/cart.routes.ts
import { Router } from 'express';
import { cartService } from './cart.service';
import { authenticate } from '../../middleware/auth';

const router = Router();

router.use(authenticate);

router.get('/', async (req, res, next) => {
  try {
    const cart = await cartService.getCartSummary(req.user!.id);
    res.json(cart);
  } catch (error) {
    next(error);
  }
});

router.post('/items', async (req, res, next) => {
  try {
    const { productId, quantity } = req.body;
    const cart = await cartService.addToCart(req.user!.id, productId, quantity);
    res.json(cart);
  } catch (error) {
    next(error);
  }
});

router.patch('/items/:productId', async (req, res, next) => {
  try {
    const { quantity } = req.body;
    const cart = await cartService.updateCartItem(req.user!.id, req.params.productId, quantity);
    res.json(cart);
  } catch (error) {
    next(error);
  }
});

router.delete('/items/:productId', async (req, res, next) => {
  try {
    const cart = await cartService.removeFromCart(req.user!.id, req.params.productId);
    res.json(cart);
  } catch (error) {
    next(error);
  }
});

export default router;
```

---

## Step 8: Main Application

```typescript
// src/app.ts
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import morgan from 'morgan';
import compression from 'compression';

import { config } from './config';
import { errorHandler } from './middleware/errorHandler';
import { rateLimiter } from './middleware/rateLimit';

// Routes
import authRoutes from './modules/auth/auth.routes';
import productRoutes from './modules/products/product.routes';
import cartRoutes from './modules/cart/cart.routes';
import orderRoutes from './modules/orders/order.routes';
import categoryRoutes from './modules/categories/category.routes';

const app = express();

// Middleware
app.use(helmet());
app.use(cors());
app.use(compression());
app.use(morgan('combined'));
app.use(express.json());

// Rate limiting
app.use('/api', rateLimiter({ max: 100, windowMs: 60000 }));

// Routes
app.use('/api/auth', authRoutes);
app.use('/api/products', productRoutes);
app.use('/api/categories', categoryRoutes);
app.use('/api/cart', cartRoutes);
app.use('/api/orders', orderRoutes);

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() });
});

// Error handling
app.use(errorHandler);

export default app;
```

---

## API Endpoints Summary

```
Authentication:
POST   /api/auth/register      - Register user
POST   /api/auth/login         - Login user
POST   /api/auth/refresh       - Refresh token

Products:
GET    /api/products           - List products (with filters)
GET    /api/products/:slug     - Get product by slug
POST   /api/products           - Create product (admin)
PATCH  /api/products/:id       - Update product (admin)
DELETE /api/products/:id       - Delete product (admin)

Categories:
GET    /api/categories         - List categories
GET    /api/categories/:slug   - Get category with products

Cart:
GET    /api/cart               - Get cart
POST   /api/cart/items         - Add to cart
PATCH  /api/cart/items/:id     - Update cart item
DELETE /api/cart/items/:id     - Remove from cart

Orders:
GET    /api/orders             - List user orders
GET    /api/orders/:id         - Get order details
POST   /api/orders             - Create order
PATCH  /api/orders/:id/status  - Update order status (admin)

Reviews:
GET    /api/products/:id/reviews  - Get product reviews
POST   /api/products/:id/reviews  - Create review
```

---

## Key Takeaways

1. **Design schema for queries** - Think about how data will be accessed
2. **Use transactions for data integrity** - Orders must be atomic
3. **Cache strategically** - Products and categories are great cache candidates
4. **Invalidate properly** - Clear cache when data changes
5. **Index foreign keys** - Essential for JOIN performance
6. **Separate read/write concerns** - Read from cache, write to database
7. **Handle edge cases** - Stock validation, cart emptiness, etc.

---

## Congratulations!

You've completed the Databases module! You now have:
- Strong SQL and Prisma skills
- Understanding of MongoDB for document storage
- Redis caching expertise
- Experience building production-ready database layers

**Next up: Auth & Security** - JWT, OAuth, and securing your applications!
