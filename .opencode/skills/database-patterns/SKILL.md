---
name: database-patterns
description: Database design and query optimization for SQL (PostgreSQL, MySQL, SQLite) and NoSQL (MongoDB, Redis, DynamoDB). Includes schema design, indexing, migrations, and ORM patterns. Use when designing database schemas, optimizing queries, planning migrations, choosing between SQL/NoSQL, or when the user mentions database, schema, SQL, NoSQL, ORM, migration, indexing, or query optimization.
---

# Database Patterns Skill

Design and optimize databases across SQL and NoSQL systems with proven patterns and best practices.

## When to Use
- Designing new database schemas
- Optimizing slow queries
- Planning database migrations
- Choosing between SQL and NoSQL
- Implementing caching strategies
- Setting up database connections and ORMs

## SQL Database Design

### Normalization (3NF)
```sql
-- Users table (entity)
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Posts table (related entity)
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    content TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Indexing Strategy
```sql
-- Single column index
CREATE INDEX idx_users_email ON users(email);

-- Composite index (order matters!)
CREATE INDEX idx_posts_user_created ON posts(user_id, created_at DESC);

-- Partial index (PostgreSQL)
CREATE INDEX idx_active_users ON users(email) WHERE deleted_at IS NULL;
```

### Query Optimization
```sql
-- Use EXPLAIN ANALYZE to check query plans
EXPLAIN ANALYZE
SELECT u.name, COUNT(p.id) as post_count
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
WHERE u.created_at > NOW() - INTERVAL '30 days'
GROUP BY u.id, u.name
HAVING COUNT(p.id) > 5;

-- Optimization tips:
-- 1. Select only needed columns (avoid SELECT *)
-- 2. Use LIMIT with large result sets
-- 3. Ensure JOIN columns are indexed
-- 4. Use EXISTS instead of COUNT for existence checks
```

### Migrations
```sql
-- Migration: add_user_bio.sql
BEGIN;

ALTER TABLE users ADD COLUMN bio TEXT;
CREATE INDEX idx_users_bio_gin ON users USING gin(to_tsvector('english', bio));

COMMIT;
```

## NoSQL Design

### MongoDB (Document Store)
```javascript
// Document structure
{
  _id: ObjectId("..."),
  userId: "user_123",
  name: "John Doe",
  email: "john@example.com",
  tags: ["developer", "python"],
  createdAt: new Date(),
  // Embed related data when accessed together
  posts: [
    { title: "Post 1", createdAt: new Date() }
  ]
}

// Indexing
db.users.createIndex({ email: 1 }, { unique: true });
db.users.createIndex({ tags: 1 });
db.users.createIndex({ createdAt: -1 });
```

### Redis (Key-Value Store)
```bash
# String (simple cache)
SET user:123:name "John Doe"
GET user:123:name
EXPIRE user:123:name 3600  # TTL

# Hash (object storage)
HSET user:123 name "John" email "john@example.com"
HGETALL user:123

# Sorted Set (leaderboard, time-series)
ZADD leaderboard 1000 "user:123"
ZREVRANGE leaderboard 0 9 WITHSCORES  # Top 10
```

### DynamoDB (AWS NoSQL)
```javascript
// Table design with composite key
{
  TableName: "Users",
  KeySchema: [
    { AttributeName: "pk", KeyType: "HASH" },   // Partition key
    { AttributeName: "sk", KeyType: "RANGE" }   // Sort key
  ],
  // Use single-table design for related entities
  // pk: "USER#123", sk: "PROFILE"
  // pk: "USER#123", sk: "POST#456"
}
```

## ORM Patterns

### General Principles
- Use migrations for schema changes (never modify DB directly)
- Lazy load relationships; eager load when you know you need them
- Use transactions for multi-step operations
- Validate at both application and database level

### SQLAlchemy (Python)
```python
from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.orm import relationship

class User(Base):
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)
    email = Column(String(255), unique=True, nullable=False)
    posts = relationship("Post", back_populates="author", cascade="all, delete-orphan")

class Post(Base):
    __tablename__ = 'posts'
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey('users.id'), nullable=False)
    author = relationship("User", back_populates="posts")
```

### TypeORM (TypeScript)
```typescript
@Entity()
class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  email: string;

  @OneToMany(() => Post, post => post.author)
  posts: Post[];
}

@Entity()
class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  userId: number;

  @ManyToOne(() => User, user => user.posts)
  author: User;
}
```

### Prisma (Language-agnostic)
```prisma
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  posts Post[]
}

model Post {
  id       Int     @id @default(autoincrement())
  userId   Int
  author   User    @relation(fields: [userId], references: [id])
}
```

## SQL vs NoSQL Decision Matrix

| Requirement | Choose |
|-------------|--------|
| Complex relationships, ACID transactions | SQL (PostgreSQL, MySQL) |
| Flexible schema, rapid iteration | NoSQL (MongoDB) |
| Simple key-value, caching | Redis |
| Massive scale, single-digit ms latency | DynamoDB, Cassandra |
| Full-text search | Elasticsearch, PostgreSQL with tsvector |
| Analytics, reporting | SQL (PostgreSQL, ClickHouse) |

## Connection Pooling
```python
# Example: SQLAlchemy pool configuration
engine = create_engine(
    'postgresql://user:pass@localhost/db',
    pool_size=10,
    max_overflow=20,
    pool_timeout=30,
    pool_recycle=1800
)
```

## Verification

After implementing database changes:
1. Run migrations in transaction, verify with `--dry-run`
2. Check query performance with `EXPLAIN ANALYZE`
3. Verify indexes are being used (`\d` or `SHOW INDEX`)
4. Test rollback procedures
5. Ensure connection pooling is configured correctly
