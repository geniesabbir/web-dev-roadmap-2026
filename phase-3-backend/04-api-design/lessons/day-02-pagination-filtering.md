# Day 2: Pagination and Filtering - Managing Large Datasets

## Introduction

As your application grows, returning all records in a single response becomes impractical. Pagination, filtering, and sorting allow clients to request exactly the data they need. Today, you'll learn how to implement these features effectively.

## Learning Objectives

By the end of this lesson, you will be able to:
- Implement offset-based and cursor-based pagination
- Add filtering capabilities to your API
- Enable sorting on multiple fields
- Design field selection for partial responses
- Handle search functionality

---

## Pagination Strategies

### Offset-Based Pagination

The most common approach using `page` and `limit`:

```javascript
// Request
GET /api/products?page=2&limit=10

// Response
{
  "data": [...10 products...],
  "meta": {
    "page": 2,
    "limit": 10,
    "total": 156,
    "totalPages": 16
  },
  "links": {
    "first": "/api/products?page=1&limit=10",
    "prev": "/api/products?page=1&limit=10",
    "next": "/api/products?page=3&limit=10",
    "last": "/api/products?page=16&limit=10"
  }
}
```

### Implementation

```javascript
// middleware/pagination.js
const paginate = (defaultLimit = 10, maxLimit = 100) => {
  return (req, res, next) => {
    let { page = 1, limit = defaultLimit } = req.query;

    page = Math.max(1, parseInt(page));
    limit = Math.min(maxLimit, Math.max(1, parseInt(limit)));

    req.pagination = {
      page,
      limit,
      skip: (page - 1) * limit
    };

    next();
  };
};

// routes/products.js
const express = require('express');
const router = express.Router();
const paginate = require('../middleware/pagination');

router.get('/', paginate(10, 100), async (req, res) => {
  const { page, limit, skip } = req.pagination;

  const [products, total] = await Promise.all([
    prisma.product.findMany({
      skip,
      take: limit,
      orderBy: { createdAt: 'desc' }
    }),
    prisma.product.count()
  ]);

  const totalPages = Math.ceil(total / limit);
  const baseUrl = `${req.protocol}://${req.get('host')}${req.baseUrl}`;

  res.json({
    data: products,
    meta: {
      page,
      limit,
      total,
      totalPages
    },
    links: {
      first: `${baseUrl}?page=1&limit=${limit}`,
      prev: page > 1 ? `${baseUrl}?page=${page - 1}&limit=${limit}` : null,
      next: page < totalPages ? `${baseUrl}?page=${page + 1}&limit=${limit}` : null,
      last: `${baseUrl}?page=${totalPages}&limit=${limit}`
    }
  });
});
```

### Offset Pagination Problems

```javascript
// Problem 1: Data inconsistency with changes
// Page 1 returned items 1-10
// New item inserted
// Page 2 returns items 10-20 (item 10 appears twice!)

// Problem 2: Poor performance on large offsets
// SKIP 10000 LIMIT 10 - database still scans 10000 rows

// Problem 3: Can't handle real-time data
// Social feeds, live updates
```

---

## Cursor-Based Pagination

Uses a pointer (cursor) to the last item instead of page numbers:

```javascript
// Request (first page)
GET /api/posts?limit=10

// Response
{
  "data": [...10 posts...],
  "meta": {
    "hasMore": true,
    "nextCursor": "eyJpZCI6IjEwIn0="  // base64 encoded
  }
}

// Request (next page)
GET /api/posts?limit=10&cursor=eyJpZCI6IjEwIn0=
```

### Implementation

```javascript
// utils/cursor.js
const encodeCursor = (data) => {
  return Buffer.from(JSON.stringify(data)).toString('base64');
};

const decodeCursor = (cursor) => {
  try {
    return JSON.parse(Buffer.from(cursor, 'base64').toString('utf8'));
  } catch {
    return null;
  }
};

// routes/posts.js
router.get('/', async (req, res) => {
  const { limit = 10, cursor } = req.query;
  const take = Math.min(parseInt(limit), 100);

  let where = {};
  let orderBy = { createdAt: 'desc' };

  if (cursor) {
    const decoded = decodeCursor(cursor);
    if (decoded) {
      where = {
        OR: [
          { createdAt: { lt: new Date(decoded.createdAt) } },
          {
            createdAt: new Date(decoded.createdAt),
            id: { lt: decoded.id }
          }
        ]
      };
    }
  }

  const posts = await prisma.post.findMany({
    where,
    orderBy: [
      { createdAt: 'desc' },
      { id: 'desc' }  // Tiebreaker for same timestamps
    ],
    take: take + 1  // Fetch one extra to check if there's more
  });

  const hasMore = posts.length > take;
  const data = hasMore ? posts.slice(0, -1) : posts;

  const lastItem = data[data.length - 1];
  const nextCursor = hasMore && lastItem
    ? encodeCursor({ id: lastItem.id, createdAt: lastItem.createdAt })
    : null;

  res.json({
    data,
    meta: {
      hasMore,
      nextCursor
    }
  });
});
```

### Comparison

| Feature | Offset | Cursor |
|---------|--------|--------|
| Jump to page | ✓ | ✗ |
| Performance | Slow on large offsets | Consistent |
| Data consistency | Possible duplicates | Stable |
| Bidirectional | ✓ | Harder |
| Use case | Traditional lists | Infinite scroll, feeds |

---

## Filtering

Allow clients to filter data based on field values:

```javascript
// Various filter formats

// Equality
GET /api/products?category=electronics
GET /api/products?status=active

// Multiple values
GET /api/products?category=electronics,books

// Comparison operators
GET /api/products?price[gte]=100&price[lte]=500
GET /api/products?createdAt[gt]=2024-01-01

// Boolean
GET /api/products?inStock=true
```

### Implementation

```javascript
// middleware/filter.js
const parseFilters = (allowedFields) => {
  return (req, res, next) => {
    const filters = {};

    for (const field of allowedFields) {
      const value = req.query[field];

      if (value !== undefined) {
        // Check for comparison operators
        if (typeof value === 'object') {
          filters[field] = {};

          if (value.eq) filters[field].equals = value.eq;
          if (value.ne) filters[field].not = value.ne;
          if (value.gt) filters[field].gt = parseValue(value.gt);
          if (value.gte) filters[field].gte = parseValue(value.gte);
          if (value.lt) filters[field].lt = parseValue(value.lt);
          if (value.lte) filters[field].lte = parseValue(value.lte);
          if (value.in) filters[field].in = value.in.split(',');
          if (value.contains) filters[field].contains = value.contains;
        } else {
          // Handle comma-separated values
          if (value.includes(',')) {
            filters[field] = { in: value.split(',') };
          } else if (value === 'true' || value === 'false') {
            filters[field] = value === 'true';
          } else {
            filters[field] = value;
          }
        }
      }
    }

    req.filters = filters;
    next();
  };
};

function parseValue(value) {
  // Try to parse as number
  const num = Number(value);
  if (!isNaN(num)) return num;

  // Try to parse as date
  const date = new Date(value);
  if (!isNaN(date.getTime())) return date;

  return value;
}

module.exports = parseFilters;
```

### Controller Usage

```javascript
// routes/products.js
const parseFilters = require('../middleware/filter');

const allowedFilters = [
  'category',
  'status',
  'inStock',
  'price',
  'createdAt'
];

router.get('/', parseFilters(allowedFilters), paginate(), async (req, res) => {
  const { filters } = req;
  const { page, limit, skip } = req.pagination;

  const where = { ...filters };

  const [products, total] = await Promise.all([
    prisma.product.findMany({
      where,
      skip,
      take: limit
    }),
    prisma.product.count({ where })
  ]);

  res.json({
    data: products,
    meta: { page, limit, total, totalPages: Math.ceil(total / limit) }
  });
});

// Examples:
// GET /api/products?category=electronics
// GET /api/products?price[gte]=100&price[lte]=500
// GET /api/products?inStock=true&category=electronics,books
```

---

## Sorting

Allow clients to sort by one or multiple fields:

```javascript
// Sort formats
GET /api/products?sort=price           // Ascending
GET /api/products?sort=-price          // Descending (prefix with -)
GET /api/products?sort=-createdAt,name // Multiple fields
```

### Implementation

```javascript
// middleware/sort.js
const parseSort = (allowedFields, defaultSort = '-createdAt') => {
  return (req, res, next) => {
    const sortParam = req.query.sort || defaultSort;
    const sortFields = sortParam.split(',');

    const orderBy = [];

    for (const field of sortFields) {
      const isDesc = field.startsWith('-');
      const fieldName = isDesc ? field.substring(1) : field;

      if (allowedFields.includes(fieldName)) {
        orderBy.push({
          [fieldName]: isDesc ? 'desc' : 'asc'
        });
      }
    }

    req.orderBy = orderBy.length > 0 ? orderBy : [{ createdAt: 'desc' }];
    next();
  };
};

module.exports = parseSort;
```

### Usage

```javascript
const parseSort = require('../middleware/sort');

const sortableFields = ['name', 'price', 'createdAt', 'rating'];

router.get('/',
  paginate(),
  parseFilters(allowedFilters),
  parseSort(sortableFields),
  async (req, res) => {
    const products = await prisma.product.findMany({
      where: req.filters,
      orderBy: req.orderBy,
      skip: req.pagination.skip,
      take: req.pagination.limit
    });

    res.json({ data: products });
  }
);
```

---

## Field Selection

Allow clients to request only specific fields:

```javascript
// Request only specific fields
GET /api/users?fields=id,name,email

// Exclude specific fields
GET /api/users?exclude=password,ssn

// Include relations
GET /api/users?include=posts,comments
```

### Implementation

```javascript
// middleware/fieldSelection.js
const parseFields = (allowedFields, defaultFields) => {
  return (req, res, next) => {
    const { fields, exclude, include } = req.query;

    let select = null;

    if (fields) {
      select = {};
      const requestedFields = fields.split(',');

      for (const field of requestedFields) {
        if (allowedFields.includes(field)) {
          select[field] = true;
        }
      }
    } else if (exclude) {
      select = { ...defaultFields };
      const excludedFields = exclude.split(',');

      for (const field of excludedFields) {
        delete select[field];
      }
    }

    // Handle relation includes
    let relationIncludes = {};
    if (include) {
      const relations = include.split(',');
      for (const relation of relations) {
        relationIncludes[relation] = true;
      }
    }

    req.select = select;
    req.include = Object.keys(relationIncludes).length > 0 ? relationIncludes : null;

    next();
  };
};

module.exports = parseFields;
```

### Usage

```javascript
const defaultUserFields = {
  id: true,
  name: true,
  email: true,
  createdAt: true
};

const allowedUserFields = ['id', 'name', 'email', 'avatar', 'bio', 'createdAt'];

router.get('/', parseFields(allowedUserFields, defaultUserFields), async (req, res) => {
  const users = await prisma.user.findMany({
    select: req.select || defaultUserFields,
    include: req.include
  });

  res.json({ data: users });
});
```

---

## Search

Implement full-text search across multiple fields:

```javascript
// Search endpoint
GET /api/products?q=laptop
GET /api/products?search=wireless+keyboard
```

### Basic Search Implementation

```javascript
router.get('/', async (req, res) => {
  const { q, search } = req.query;
  const searchTerm = q || search;

  let where = {};

  if (searchTerm) {
    where = {
      OR: [
        { name: { contains: searchTerm, mode: 'insensitive' } },
        { description: { contains: searchTerm, mode: 'insensitive' } },
        { sku: { contains: searchTerm, mode: 'insensitive' } }
      ]
    };
  }

  const products = await prisma.product.findMany({
    where,
    take: 20
  });

  res.json({ data: products });
});
```

### PostgreSQL Full-Text Search

```javascript
// For better search, use PostgreSQL's full-text search
router.get('/search', async (req, res) => {
  const { q } = req.query;

  if (!q) {
    return res.status(400).json({ error: 'Search query required' });
  }

  // Using raw query for full-text search
  const products = await prisma.$queryRaw`
    SELECT *
    FROM products
    WHERE to_tsvector('english', name || ' ' || description)
          @@ plainto_tsquery('english', ${q})
    ORDER BY ts_rank(
      to_tsvector('english', name || ' ' || description),
      plainto_tsquery('english', ${q})
    ) DESC
    LIMIT 20
  `;

  res.json({ data: products });
});
```

---

## Complete Query Builder

Combine all features into a reusable query builder:

```javascript
// utils/queryBuilder.js
class QueryBuilder {
  constructor(model, allowedFilters, allowedSort, defaultLimit = 10) {
    this.model = model;
    this.allowedFilters = allowedFilters;
    this.allowedSort = allowedSort;
    this.defaultLimit = defaultLimit;
    this.where = {};
    this.orderBy = [];
    this.skip = 0;
    this.take = defaultLimit;
    this.select = null;
    this.include = null;
  }

  filter(query) {
    for (const field of this.allowedFilters) {
      if (query[field] !== undefined) {
        const value = query[field];

        if (typeof value === 'object') {
          this.where[field] = this.parseOperators(value);
        } else if (value.includes(',')) {
          this.where[field] = { in: value.split(',') };
        } else {
          this.where[field] = this.parseValue(value);
        }
      }
    }
    return this;
  }

  parseOperators(obj) {
    const operators = {};
    if (obj.gt) operators.gt = this.parseValue(obj.gt);
    if (obj.gte) operators.gte = this.parseValue(obj.gte);
    if (obj.lt) operators.lt = this.parseValue(obj.lt);
    if (obj.lte) operators.lte = this.parseValue(obj.lte);
    if (obj.contains) operators.contains = obj.contains;
    return operators;
  }

  parseValue(value) {
    if (value === 'true') return true;
    if (value === 'false') return false;
    const num = Number(value);
    if (!isNaN(num)) return num;
    const date = new Date(value);
    if (!isNaN(date.getTime())) return date;
    return value;
  }

  sort(sortParam) {
    if (!sortParam) {
      this.orderBy = [{ createdAt: 'desc' }];
      return this;
    }

    const fields = sortParam.split(',');
    for (const field of fields) {
      const isDesc = field.startsWith('-');
      const fieldName = isDesc ? field.substring(1) : field;

      if (this.allowedSort.includes(fieldName)) {
        this.orderBy.push({ [fieldName]: isDesc ? 'desc' : 'asc' });
      }
    }

    return this;
  }

  paginate(page = 1, limit = this.defaultLimit) {
    this.skip = (Math.max(1, parseInt(page)) - 1) * parseInt(limit);
    this.take = Math.min(100, Math.max(1, parseInt(limit)));
    return this;
  }

  search(searchTerm, searchFields) {
    if (searchTerm) {
      this.where.OR = searchFields.map(field => ({
        [field]: { contains: searchTerm, mode: 'insensitive' }
      }));
    }
    return this;
  }

  fields(fieldsList, allowedFields) {
    if (fieldsList) {
      this.select = {};
      const requested = fieldsList.split(',');
      for (const field of requested) {
        if (allowedFields.includes(field)) {
          this.select[field] = true;
        }
      }
    }
    return this;
  }

  async execute() {
    const [data, total] = await Promise.all([
      this.model.findMany({
        where: this.where,
        orderBy: this.orderBy,
        skip: this.skip,
        take: this.take,
        select: this.select,
        include: this.include
      }),
      this.model.count({ where: this.where })
    ]);

    return {
      data,
      meta: {
        total,
        page: Math.floor(this.skip / this.take) + 1,
        limit: this.take,
        totalPages: Math.ceil(total / this.take)
      }
    };
  }
}

module.exports = QueryBuilder;
```

### Usage

```javascript
const QueryBuilder = require('../utils/queryBuilder');

router.get('/', async (req, res) => {
  const builder = new QueryBuilder(
    prisma.product,
    ['category', 'status', 'price', 'inStock'],
    ['name', 'price', 'createdAt']
  );

  const result = await builder
    .filter(req.query)
    .sort(req.query.sort)
    .paginate(req.query.page, req.query.limit)
    .search(req.query.q, ['name', 'description'])
    .execute();

  res.json(result);
});
```

---

## Response Headers for Pagination

```javascript
// Set pagination info in headers (alternative to body)
router.get('/', async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;

  const [products, total] = await Promise.all([
    prisma.product.findMany({ skip: (page - 1) * limit, take: limit }),
    prisma.product.count()
  ]);

  const totalPages = Math.ceil(total / limit);

  res.set({
    'X-Total-Count': total,
    'X-Total-Pages': totalPages,
    'X-Page': page,
    'X-Per-Page': limit,
    'Link': buildLinkHeader(req, page, totalPages, limit)
  });

  res.json(products);
});

function buildLinkHeader(req, page, totalPages, limit) {
  const base = `${req.protocol}://${req.get('host')}${req.baseUrl}`;
  const links = [];

  links.push(`<${base}?page=1&limit=${limit}>; rel="first"`);

  if (page > 1) {
    links.push(`<${base}?page=${page - 1}&limit=${limit}>; rel="prev"`);
  }

  if (page < totalPages) {
    links.push(`<${base}?page=${page + 1}&limit=${limit}>; rel="next"`);
  }

  links.push(`<${base}?page=${totalPages}&limit=${limit}>; rel="last"`);

  return links.join(', ');
}
```

---

## Practice Exercise

Implement pagination and filtering for an API:

1. **Create a products endpoint** with:
   - Offset pagination (page, limit)
   - Filtering by category, price range, status
   - Sorting by name, price, createdAt
   - Search by name and description

2. **Create a posts endpoint** with:
   - Cursor-based pagination
   - Filter by author, tags
   - Sort by createdAt, likes

3. **Test with various query combinations**:
   ```
   GET /api/products?category=electronics&price[gte]=100&sort=-price&page=1
   GET /api/posts?cursor=abc123&limit=20
   ```

---

## Key Takeaways

1. **Choose pagination type wisely** - Offset for traditional, cursor for feeds
2. **Limit results** - Set max limits to prevent abuse
3. **Allow flexible filtering** - Support ranges, multiple values
4. **Enable sorting** - Multiple fields, asc/desc
5. **Provide metadata** - Total count, page info, links
6. **Validate inputs** - Only allow known fields

---

## What's Next?

Tomorrow, we'll explore **API Versioning** - strategies to evolve your API while maintaining backward compatibility.
