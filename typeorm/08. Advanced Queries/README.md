
## **Advanced Queries**

<details>
<summary>106. How do you implement full-text search in TypeORM?</summary>

### Answer:

**Full-text search** allows searching through text data efficiently, finding matches based on relevance rather than exact string matching. TypeORM supports full-text search through database-specific features and query builders.

---

### **PostgreSQL Full-Text Search:**

PostgreSQL provides powerful full-text search capabilities with `to_tsvector` and `to_tsquery`.

```typescript
import { DataSource, Repository } from 'typeorm';
import { Article } from './entities/Article';

@Entity()
class Article {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column('text')
  content: string;

  @Column()
  author: string;
}

class ArticleService {
  constructor(
    private articleRepo: Repository<Article>,
    private dataSource: DataSource
  ) {}

  // Method 1: Using Raw SQL with Query Builder
  async fullTextSearch(searchTerm: string): Promise<Article[]> {
    return await this.articleRepo
      .createQueryBuilder('article')
      .where(
        `to_tsvector('english', article.title || ' ' || article.content) 
         @@ to_tsquery('english', :searchTerm)`,
        { searchTerm: searchTerm.replace(/\s+/g, ' & ') } // "hello world" -> "hello & world"
      )
      .orderBy(
        `ts_rank(
          to_tsvector('english', article.title || ' ' || article.content),
          to_tsquery('english', :searchTerm)
        )`,
        'DESC'
      )
      .setParameter('searchTerm', searchTerm.replace(/\s+/g, ' & '))
      .getMany();
  }

  // Method 2: Using Raw Query for Complex Searches
  async advancedFullTextSearch(
    searchTerm: string,
    options: { limit?: number; offset?: number } = {}
  ) {
    const { limit = 20, offset = 0 } = options;

    return await this.dataSource.query(
      `
      SELECT 
        id, 
        title, 
        content,
        author,
        ts_rank(
          to_tsvector('english', title || ' ' || content),
          to_tsquery('english', $1)
        ) as relevance
      FROM articles
      WHERE to_tsvector('english', title || ' ' || content) 
            @@ to_tsquery('english', $1)
      ORDER BY relevance DESC
      LIMIT $2 OFFSET $3
      `,
      [searchTerm.replace(/\s+/g, ' & '), limit, offset]
    );
  }

  // Method 3: With Highlighting
  async searchWithHighlight(searchTerm: string): Promise<any[]> {
    return await this.dataSource.query(
      `
      SELECT 
        id,
        title,
        ts_headline('english', content, to_tsquery('english', $1), 
          'MaxWords=50, MinWords=25, ShortWord=3, MaxFragments=3'
        ) as highlighted_content,
        ts_rank(
          to_tsvector('english', title || ' ' || content),
          to_tsquery('english', $1)
        ) as relevance
      FROM articles
      WHERE to_tsvector('english', title || ' ' || content) 
            @@ to_tsquery('english', $1)
      ORDER BY relevance DESC
      LIMIT 10
      `,
      [searchTerm.replace(/\s+/g, ' & ')]
    );
  }
}
```

---

### **MySQL Full-Text Search:**

MySQL uses `MATCH...AGAINST` for full-text searching.

```typescript
class ArticleService {
  constructor(private dataSource: DataSource) {}

  // Requires FULLTEXT index on columns
  async fullTextSearchMySQL(searchTerm: string): Promise<Article[]> {
    return await this.dataSource.query(
      `
      SELECT *, 
        MATCH(title, content) AGAINST(? IN NATURAL LANGUAGE MODE) as relevance
      FROM articles
      WHERE MATCH(title, content) AGAINST(? IN NATURAL LANGUAGE MODE)
      ORDER BY relevance DESC
      LIMIT 20
      `,
      [searchTerm, searchTerm]
    );
  }

  // Boolean mode for more control
  async booleanSearchMySQL(searchTerm: string): Promise<Article[]> {
    // Supports operators: + (must include), - (exclude), * (wildcard)
    return await this.dataSource.query(
      `
      SELECT *
      FROM articles
      WHERE MATCH(title, content) 
            AGAINST(? IN BOOLEAN MODE)
      ORDER BY id DESC
      `,
      [searchTerm] // e.g., "+TypeORM +tutorial -beginner*"
    );
  }
}
```

---

### **Generic Cross-Database Approach:**

For databases without native full-text search, use LIKE with index optimization.

```typescript
class ArticleService {
  async basicTextSearch(searchTerm: string): Promise<Article[]> {
    const terms = searchTerm.toLowerCase().split(/\s+/);

    let query = this.articleRepo.createQueryBuilder('article');

    terms.forEach((term, index) => {
      const condition = `
        LOWER(article.title) LIKE :term${index} OR 
        LOWER(article.content) LIKE :term${index}
      `;
      
      if (index === 0) {
        query = query.where(condition, { [`term${index}`]: `%${term}%` });
      } else {
        query = query.andWhere(condition, { [`term${index}`]: `%${term}%` });
      }
    });

    return await query.take(20).getMany();
  }
}
```

---

### **Production Use Case: Blog Search**

```typescript
class BlogSearchService {
  constructor(private dataSource: DataSource) {}

  async searchArticles(
    query: string,
    filters: {
      author?: string;
      tags?: string[];
      dateFrom?: Date;
      dateTo?: Date;
    } = {}
  ) {
    // PostgreSQL full-text search with filters
    let sql = `
      SELECT 
        a.id,
        a.title,
        a.author,
        a.created_at,
        ts_headline('english', a.content, to_tsquery('english', $1)) as snippet,
        ts_rank(
          to_tsvector('english', a.title || ' ' || a.content),
          to_tsquery('english', $1)
        ) as relevance
      FROM articles a
      WHERE to_tsvector('english', a.title || ' ' || a.content) 
            @@ to_tsquery('english', $1)
    `;

    const params: any[] = [query.replace(/\s+/g, ' & ')];
    let paramIndex = 2;

    // Apply filters
    if (filters.author) {
      sql += ` AND a.author = $${paramIndex}`;
      params.push(filters.author);
      paramIndex++;
    }

    if (filters.tags && filters.tags.length > 0) {
      sql += ` AND a.tags && $${paramIndex}`;
      params.push(filters.tags);
      paramIndex++;
    }

    if (filters.dateFrom) {
      sql += ` AND a.created_at >= $${paramIndex}`;
      params.push(filters.dateFrom);
      paramIndex++;
    }

    if (filters.dateTo) {
      sql += ` AND a.created_at <= $${paramIndex}`;
      params.push(filters.dateTo);
      paramIndex++;
    }

    sql += ' ORDER BY relevance DESC LIMIT 50';

    return await this.dataSource.query(sql, params);
  }
}
```

---

### **Creating Full-Text Indexes:**

```typescript
// Migration to add full-text index (PostgreSQL)
export class AddFullTextIndex1234567890 implements MigrationInterface {
  async up(queryRunner: QueryRunner): Promise<void> {
    // Create GIN index for better performance
    await queryRunner.query(`
      CREATE INDEX articles_fts_idx 
      ON articles 
      USING GIN (to_tsvector('english', title || ' ' || content))
    `);
  }

  async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP INDEX articles_fts_idx`);
  }
}

// MySQL - Add FULLTEXT index
export class AddFullTextIndexMySQL1234567890 implements MigrationInterface {
  async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      ALTER TABLE articles 
      ADD FULLTEXT INDEX ft_title_content (title, content)
    `);
  }

  async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      ALTER TABLE articles 
      DROP INDEX ft_title_content
    `);
  }
}
```

---

### **Best Practices:**

1. ✅ **Create proper indexes** - Essential for performance
2. ✅ **Sanitize search input** - Prevent SQL injection
3. ✅ **Limit results** - Use pagination
4. ✅ **Cache frequent searches** - Improve response times
5. ✅ **Use relevance ranking** - Order by match quality
6. ✅ **Handle special characters** - Escape or remove them

---

### **Interview Tips:**
- Explain database-specific full-text search features (PostgreSQL vs MySQL)
- Discuss the importance of indexing for performance
- Show awareness of relevance ranking and result highlighting
- Mention alternatives like Elasticsearch for complex search requirements

</details>

<details>
<summary>107. How do you perform case-insensitive searches?</summary>

### Answer:

**Case-insensitive searches** allow finding matches regardless of letter casing (e.g., "John" matches "JOHN" or "john"). TypeORM provides several approaches depending on your database.

---

### **Method 1: Using ILike (PostgreSQL)**

PostgreSQL provides the `ILike` operator for case-insensitive pattern matching.

```typescript
import { Repository, ILike } from 'typeorm';
import { User } from './entities/User';

class UserService {
  constructor(private userRepo: Repository<User>) {}

  // Simple case-insensitive search
  async searchByName(name: string): Promise<User[]> {
    return await this.userRepo.find({
      where: {
        name: ILike(`%${name}%`) // Matches "john", "JOHN", "John"
      }
    });
  }

  // Multiple fields
  async searchByEmailOrName(term: string): Promise<User[]> {
    return await this.userRepo.find({
      where: [
        { email: ILike(`%${term}%`) },
        { name: ILike(`%${term}%`) }
      ]
    });
  }

  // Exact match, case-insensitive
  async findByEmailIgnoreCase(email: string): Promise<User | null> {
    return await this.userRepo.findOne({
      where: {
        email: ILike(email) // Exact match, case-insensitive
      }
    });
  }
}
```

---

### **Method 2: Using LOWER() Function (Cross-Database)**

Works with all databases by converting both sides to lowercase.

```typescript
class UserService {
  constructor(private userRepo: Repository<User>) {}

  // Using Query Builder with LOWER()
  async searchCaseInsensitive(searchTerm: string): Promise<User[]> {
    return await this.userRepo
      .createQueryBuilder('user')
      .where('LOWER(user.name) LIKE LOWER(:searchTerm)', {
        searchTerm: `%${searchTerm}%`
      })
      .getMany();
  }

  // Multiple conditions
  async searchUsersByMultipleFields(term: string): Promise<User[]> {
    return await this.userRepo
      .createQueryBuilder('user')
      .where('LOWER(user.name) LIKE LOWER(:term)', { term: `%${term}%` })
      .orWhere('LOWER(user.email) LIKE LOWER(:term)', { term: `%${term}%` })
      .orWhere('LOWER(user.username) LIKE LOWER(:term)', { term: `%${term}%` })
      .getMany();
  }

  // Exact match
  async findByUsernameIgnoreCase(username: string): Promise<User | null> {
    return await this.userRepo
      .createQueryBuilder('user')
      .where('LOWER(user.username) = LOWER(:username)', { username })
      .getOne();
  }
}
```

---

### **Method 3: Using Database Collation**

Set case-insensitive collation at the column level.

```typescript
@Entity()
class User {
  @PrimaryGeneratedColumn()
  id: number;

  // PostgreSQL: Use CITEXT type
  @Column({ type: 'citext' }) // Case-insensitive text
  email: string;

  // MySQL: Use collation
  @Column({ collation: 'utf8mb4_unicode_ci' })
  username: string;

  @Column()
  name: string;
}

// Now regular queries are automatically case-insensitive
class UserService {
  async findByEmail(email: string): Promise<User | null> {
    // This is case-insensitive because email is CITEXT
    return await this.userRepo.findOne({
      where: { email } // "john@EXAMPLE.com" matches "John@example.com"
    });
  }
}
```

---

### **Method 4: Using Raw SQL**

Direct SQL for maximum control.

```typescript
class UserService {
  constructor(private dataSource: DataSource) {}

  async searchUsersRaw(searchTerm: string): Promise<User[]> {
    // PostgreSQL ILIKE
    return await this.dataSource.query(
      `SELECT * FROM users 
       WHERE name ILIKE $1 OR email ILIKE $1`,
      [`%${searchTerm}%`]
    );
  }

  // MySQL (case-insensitive by default with proper collation)
  async searchUsersMySQL(searchTerm: string): Promise<User[]> {
    return await this.dataSource.query(
      `SELECT * FROM users 
       WHERE name LIKE ? OR email LIKE ?`,
      [`%${searchTerm}%`, `%${searchTerm}%`]
    );
  }
}
```

---

### **Production Use Case: User Search with Pagination**

```typescript
class UserSearchService {
  constructor(private userRepo: Repository<User>) {}

  async searchUsers(
    searchTerm: string,
    options: {
      page?: number;
      limit?: number;
      sortBy?: 'name' | 'email' | 'created_at';
      sortOrder?: 'ASC' | 'DESC';
    } = {}
  ) {
    const { 
      page = 1, 
      limit = 20, 
      sortBy = 'name', 
      sortOrder = 'ASC' 
    } = options;

    const query = this.userRepo
      .createQueryBuilder('user')
      .where('LOWER(user.name) LIKE LOWER(:term)', { 
        term: `%${searchTerm}%` 
      })
      .orWhere('LOWER(user.email) LIKE LOWER(:term)', { 
        term: `%${searchTerm}%` 
      });

    // Add sorting
    query.orderBy(`user.${sortBy}`, sortOrder);

    // Add pagination
    query.skip((page - 1) * limit).take(limit);

    // Get results and total count
    const [users, total] = await query.getManyAndCount();

    return {
      data: users,
      meta: {
        total,
        page,
        limit,
        totalPages: Math.ceil(total / limit)
      }
    };
  }
}
```

---

### **Comparison Table:**

| Method | Database Support | Performance | Use Case |
|--------|-----------------|-------------|----------|
| **ILike** | PostgreSQL only | Fast with index | PostgreSQL apps |
| **LOWER()** | All databases | Slower without index | Cross-database |
| **CITEXT/Collation** | DB-specific | Fastest | When defining schema |
| **Raw SQL** | All databases | Varies | Complex queries |

---

### **Performance Optimization:**

```typescript
// Create functional index for LOWER() searches (PostgreSQL)
export class AddCaseInsensitiveIndex implements MigrationInterface {
  async up(queryRunner: QueryRunner): Promise<void> {
    // Index on lowercase values for fast searching
    await queryRunner.query(`
      CREATE INDEX idx_users_name_lower 
      ON users (LOWER(name))
    `);

    await queryRunner.query(`
      CREATE INDEX idx_users_email_lower 
      ON users (LOWER(email))
    `);
  }

  async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP INDEX idx_users_name_lower`);
    await queryRunner.query(`DROP INDEX idx_users_email_lower`);
  }
}
```

---

### **Handling Special Characters:**

```typescript
class SearchService {
  // Sanitize input to prevent SQL injection
  sanitizeSearchTerm(term: string): string {
    // Remove special SQL characters
    return term.replace(/[%_]/g, '\\$&');
  }

  async safeSearch(userInput: string): Promise<User[]> {
    const sanitized = this.sanitizeSearchTerm(userInput);
    
    return await this.userRepo
      .createQueryBuilder('user')
      .where('LOWER(user.name) LIKE LOWER(:term)', {
        term: `%${sanitized}%`
      })
      .getMany();
  }
}
```

---

### **Best Practices:**

1. ✅ **Use ILike for PostgreSQL** - Native and performant
2. ✅ **Create indexes on LOWER()** - Essential for performance
3. ✅ **Consider CITEXT for PostgreSQL** - Best for unique fields like email
4. ✅ **Sanitize user input** - Prevent SQL injection
5. ✅ **Use collation for MySQL** - Set at column definition
6. ✅ **Limit results** - Always paginate search results

---

### **Interview Tips:**
- Explain the difference between Like and ILike
- Discuss performance implications (indexes are crucial)
- Show knowledge of database-specific features (CITEXT, collations)
- Mention cross-database compatibility with LOWER()
- Emphasize input sanitization for security

</details>

<details>
<summary>108. How do you use the Like operator?</summary>

### Answer:

The **Like operator** in TypeORM allows pattern matching in queries, useful for searching partial text matches. It supports SQL wildcards: `%` (any characters) and `_` (single character).

---

### **Basic Like Usage:**

```typescript
import { Repository, Like } from 'typeorm';
import { Product } from './entities/Product';

class ProductService {
  constructor(private productRepo: Repository<Product>) {}

  // Starts with
  async findProductsStartingWith(prefix: string): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        name: Like(`${prefix}%`) // "Laptop%" matches "Laptop Pro", "Laptop Air"
      }
    });
  }

  // Ends with
  async findProductsEndingWith(suffix: string): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        name: Like(`%${suffix}`) // "%Phone" matches "iPhone", "Android Phone"
      }
    });
  }

  // Contains (most common)
  async searchProducts(keyword: string): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        name: Like(`%${keyword}%`) // "%book%" matches "Notebook", "Facebook"
      }
    });
  }

  // Exact length with underscores
  async findProductsByPattern(): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        sku: Like('PRD-____') // Matches "PRD-1234", "PRD-ABCD" (4 chars)
      }
    });
  }
}
```

---

### **Multiple Like Conditions:**

```typescript
class ProductService {
  // OR conditions - matches any
  async searchMultipleFields(keyword: string): Promise<Product[]> {
    return await this.productRepo.find({
      where: [
        { name: Like(`%${keyword}%`) },
        { description: Like(`%${keyword}%`) },
        { category: Like(`%${keyword}%`) }
      ]
    });
  }

  // AND conditions - matches all
  async searchWithMultipleKeywords(
    keyword1: string,
    keyword2: string
  ): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        name: Like(`%${keyword1}%`),
        description: Like(`%${keyword2}%`)
      }
    });
  }
}
```

---

### **Like with Query Builder:**

```typescript
class ProductService {
  constructor(private productRepo: Repository<Product>) {}

  // Basic query builder with Like
  async advancedSearch(filters: {
    name?: string;
    category?: string;
    minPrice?: number;
    maxPrice?: number;
  }): Promise<Product[]> {
    const query = this.productRepo.createQueryBuilder('product');

    // Add Like conditions dynamically
    if (filters.name) {
      query.andWhere('product.name LIKE :name', {
        name: `%${filters.name}%`
      });
    }

    if (filters.category) {
      query.andWhere('product.category LIKE :category', {
        category: `%${filters.category}%`
      });
    }

    // Combine with other operators
    if (filters.minPrice !== undefined) {
      query.andWhere('product.price >= :minPrice', {
        minPrice: filters.minPrice
      });
    }

    if (filters.maxPrice !== undefined) {
      query.andWhere('product.price <= :maxPrice', {
        maxPrice: filters.maxPrice
      });
    }

    return await query.getMany();
  }
}
```

---

### **Like vs ILike (Case Sensitivity):**

```typescript
import { Like, ILike } from 'typeorm';

class UserService {
  constructor(private userRepo: Repository<User>) {}

  // Case-sensitive (SQL LIKE)
  async searchCaseSensitive(name: string): Promise<User[]> {
    return await this.userRepo.find({
      where: {
        name: Like(`%${name}%`) // "John" matches "John" but not "john"
      }
    });
  }

  // Case-insensitive (SQL ILIKE - PostgreSQL only)
  async searchCaseInsensitive(name: string): Promise<User[]> {
    return await this.userRepo.find({
      where: {
        name: ILike(`%${name}%`) // "John" matches "John", "john", "JOHN"
      }
    });
  }

  // Cross-database case-insensitive
  async searchCaseInsensitiveCrossDB(name: string): Promise<User[]> {
    return await this.userRepo
      .createQueryBuilder('user')
      .where('LOWER(user.name) LIKE LOWER(:name)', {
        name: `%${name}%`
      })
      .getMany();
  }
}
```

---

### **Production Use Case: E-commerce Search**

```typescript
class ProductSearchService {
  constructor(private productRepo: Repository<Product>) {}

  async searchProducts(
    searchParams: {
      query?: string;
      category?: string;
      brand?: string;
      tags?: string[];
      minPrice?: number;
      maxPrice?: number;
      inStock?: boolean;
    },
    pagination: {
      page?: number;
      limit?: number;
    } = {}
  ) {
    const { page = 1, limit = 20 } = pagination;
    
    const queryBuilder = this.productRepo
      .createQueryBuilder('product')
      .leftJoinAndSelect('product.reviews', 'reviews');

    // Search query across multiple fields
    if (searchParams.query) {
      queryBuilder.andWhere(
        '(product.name LIKE :query OR product.description LIKE :query OR product.sku LIKE :query)',
        { query: `%${searchParams.query}%` }
      );
    }

    // Category filter
    if (searchParams.category) {
      queryBuilder.andWhere('product.category LIKE :category', {
        category: `%${searchParams.category}%`
      });
    }

    // Brand filter
    if (searchParams.brand) {
      queryBuilder.andWhere('product.brand LIKE :brand', {
        brand: `%${searchParams.brand}%`
      });
    }

    // Tags filter (array contains)
    if (searchParams.tags && searchParams.tags.length > 0) {
      queryBuilder.andWhere('product.tags && :tags', {
        tags: searchParams.tags
      });
    }

    // Price range
    if (searchParams.minPrice !== undefined) {
      queryBuilder.andWhere('product.price >= :minPrice', {
        minPrice: searchParams.minPrice
      });
    }

    if (searchParams.maxPrice !== undefined) {
      queryBuilder.andWhere('product.price <= :maxPrice', {
        maxPrice: searchParams.maxPrice
      });
    }

    // Stock filter
    if (searchParams.inStock !== undefined) {
      if (searchParams.inStock) {
        queryBuilder.andWhere('product.stock > 0');
      } else {
        queryBuilder.andWhere('product.stock = 0');
      }
    }

    // Pagination
    queryBuilder.skip((page - 1) * limit).take(limit);

    // Execute
    const [products, total] = await queryBuilder.getManyAndCount();

    return {
      data: products,
      meta: {
        total,
        page,
        limit,
        totalPages: Math.ceil(total / limit),
        hasMore: page * limit < total
      }
    };
  }
}
```

---

### **Escaping Special Characters:**

```typescript
class SearchService {
  // Escape wildcards in user input
  escapeWildcards(input: string): string {
    return input.replace(/[%_]/g, '\\$&');
  }

  async safeSearch(userInput: string): Promise<Product[]> {
    const escaped = this.escapeWildcards(userInput);
    
    return await this.productRepo.find({
      where: {
        name: Like(`%${escaped}%`)
      }
    });
  }

  // If user wants to search for literal "50%"
  async searchForPercentage(): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        description: Like('%50\\%%') // Escapes the % in "50%"
      }
    });
  }
}
```

---

### **Combining Like with Other Operators:**

```typescript
import { Like, MoreThan, LessThan, Between, In } from 'typeorm';

class ProductService {
  async complexSearch(): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        name: Like('%laptop%'),           // Contains "laptop"
        price: Between(500, 2000),        // Price between $500-$2000
        category: In(['Electronics', 'Computers']), // Specific categories
        stock: MoreThan(0),               // In stock
        brand: Like('Dell%')              // Brand starts with "Dell"
      },
      order: {
        price: 'ASC'
      },
      take: 50
    });
  }
}
```

---

### **Performance Considerations:**

```typescript
// Create indexes for Like queries
export class AddSearchIndexes implements MigrationInterface {
  async up(queryRunner: QueryRunner): Promise<void> {
    // PostgreSQL: Use GIN index for pattern matching
    await queryRunner.query(`
      CREATE INDEX idx_products_name_gin 
      ON products USING gin(name gin_trgm_ops)
    `);

    // Or regular B-tree index (less effective for %keyword% but works)
    await queryRunner.query(`
      CREATE INDEX idx_products_name 
      ON products(name)
    `);

    // For case-insensitive searches
    await queryRunner.query(`
      CREATE INDEX idx_products_name_lower 
      ON products(LOWER(name))
    `);
  }

  async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP INDEX idx_products_name_gin`);
    await queryRunner.query(`DROP INDEX idx_products_name`);
    await queryRunner.query(`DROP INDEX idx_products_name_lower`);
  }
}
```

---

### **Best Practices:**

1. ✅ **Escape user input** - Prevent SQL injection and unexpected matches
2. ✅ **Use indexes** - Essential for performance on large tables
3. ✅ **Prefer ILike for case-insensitive** - When using PostgreSQL
4. ✅ **Limit results** - Always paginate search results
5. ✅ **Avoid leading wildcards** - `%keyword` can't use indexes efficiently
6. ✅ **Cache frequent searches** - Improve response times

---

### **Interview Tips:**
- Explain Like wildcards: `%` (multiple chars) and `_` (single char)
- Discuss the difference between Like and ILike
- Mention performance considerations (indexes, leading wildcards)
- Show how to combine Like with other operators
- Emphasize input sanitization and escaping

</details>

<details>
<summary>109. What are the different operators available (Between, In, IsNull, etc.)?</summary>

### Answer:

TypeORM provides a comprehensive set of **find operators** for building queries. These operators make it easy to construct complex WHERE clauses without writing raw SQL.

---

### **Complete List of Find Operators:**

```typescript
import {
  Equal,
  Not,
  LessThan,
  LessThanOrEqual,
  MoreThan,
  MoreThanOrEqual,
  Between,
  In,
  Any,
  IsNull,
  IsNotNull,
  Like,
  ILike,
  ArrayContains,
  ArrayOverlap,
  ArrayContainedBy,
  Raw
} from 'typeorm';
```

---

### **1. Comparison Operators:**

```typescript
import { Repository, Equal, Not, LessThan, MoreThan, Between } from 'typeorm';
import { Product } from './entities/Product';

class ProductService {
  constructor(private productRepo: Repository<Product>) {}

  // Equal (=) - can be omitted, direct value means Equal
  async findByExactPrice(price: number): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        price: Equal(price)  // Same as: price: price
      }
    });
  }

  // Not (!=)
  async findExcludingCategory(category: string): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        category: Not(category)  // category != 'Electronics'
      }
    });
  }

  // LessThan (<)
  async findCheapProducts(maxPrice: number): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        price: LessThan(maxPrice)  // price < 100
      }
    });
  }

  // LessThanOrEqual (<=)
  async findProductsUpTo(price: number): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        price: LessThanOrEqual(price)  // price <= 100
      }
    });
  }

  // MoreThan (>)
  async findExpensiveProducts(minPrice: number): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        price: MoreThan(minPrice)  // price > 500
      }
    });
  }

  // MoreThanOrEqual (>=)
  async findProductsFrom(price: number): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        price: MoreThanOrEqual(price)  // price >= 500
      }
    });
  }

  // Between (BETWEEN x AND y)
  async findProductsInPriceRange(min: number, max: number): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        price: Between(min, max)  // price BETWEEN 100 AND 500
      }
    });
  }
}
```

---

### **2. List Operators:**

```typescript
class ProductService {
  // In (value IN (...))
  async findProductsByCategories(categories: string[]): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        category: In(categories)  // category IN ('Electronics', 'Books', 'Toys')
      }
    });
  }

  // Not with In (value NOT IN (...))
  async findProductsExcludingCategories(categories: string[]): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        category: Not(In(categories))  // category NOT IN ('Electronics', 'Books')
      }
    });
  }

  // Any (for array columns - PostgreSQL)
  async findProductsByAnyTag(tag: string): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        tags: Any([tag])  // tag = ANY(tags) - checks if tag exists in array
      }
    });
  }
}
```

---

### **3. Null Operators:**

```typescript
class ProductService {
  // IsNull (IS NULL)
  async findProductsWithoutDescription(): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        description: IsNull()  // description IS NULL
      }
    });
  }

  // IsNotNull (IS NOT NULL)
  async findProductsWithDescription(): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        description: IsNotNull()  // description IS NOT NULL
      }
    });
  }

  // Combining with other operators
  async findActiveProductsWithoutDiscount(): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        isActive: true,
        discount: IsNull()
      }
    });
  }
}
```

---

### **4. Pattern Matching Operators:**

```typescript
class ProductService {
  // Like (LIKE - case sensitive)
  async findProductsByNamePattern(pattern: string): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        name: Like(`%${pattern}%`)  // name LIKE '%laptop%'
      }
    });
  }

  // ILike (ILIKE - case insensitive, PostgreSQL only)
  async findProductsCaseInsensitive(pattern: string): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        name: ILike(`%${pattern}%`)  // name ILIKE '%laptop%'
      }
    });
  }

  // Not with Like
  async findProductsNotMatching(pattern: string): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        name: Not(Like(`%${pattern}%`))  // name NOT LIKE '%test%'
      }
    });
  }
}
```

---

### **5. Array Operators (PostgreSQL):**

```typescript
class ProductService {
  // ArrayContains - array column contains all specified values
  async findProductsWithTags(tags: string[]): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        tags: ArrayContains(tags)  // tags @> ['featured', 'sale']
      }
    });
  }

  // ArrayOverlap - array column has any of the specified values
  async findProductsWithAnyTag(tags: string[]): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        tags: ArrayOverlap(tags)  // tags && ['featured', 'sale', 'new']
      }
    });
  }

  // ArrayContainedBy - array column is subset of specified values
  async findProductsContainedBy(tags: string[]): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        tags: ArrayContainedBy(tags)  // tags <@ ['featured', 'sale', 'new']
      }
    });
  }
}
```

---

### **6. Raw Operator (Custom SQL):**

```typescript
class ProductService {
  // Raw - write custom SQL expressions
  async findProductsWithCustomLogic(): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        // Custom SQL in WHERE clause
        price: Raw(alias => `${alias} * 0.9 < 100`)  // price * 0.9 < 100
      }
    });
  }

  // With parameters
  async findWithRawParams(discount: number): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        price: Raw(alias => `${alias} * :discount < 100`, { discount })
      }
    });
  }

  // Complex raw query
  async findByDateRange(): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        createdAt: Raw(alias => `${alias} > NOW() - INTERVAL '7 days'`)
      }
    });
  }
}
```

---

### **Production Use Case: Advanced Product Filtering:**

```typescript
class ProductFilterService {
  constructor(private productRepo: Repository<Product>) {}

  async filterProducts(filters: {
    categories?: string[];
    minPrice?: number;
    maxPrice?: number;
    inStock?: boolean;
    tags?: string[];
    excludeTags?: string[];
    searchTerm?: string;
    hasDiscount?: boolean;
    brands?: string[];
  }): Promise<Product[]> {
    const whereConditions: any = {};

    // In operator for categories
    if (filters.categories && filters.categories.length > 0) {
      whereConditions.category = In(filters.categories);
    }

    // Between for price range
    if (filters.minPrice !== undefined && filters.maxPrice !== undefined) {
      whereConditions.price = Between(filters.minPrice, filters.maxPrice);
    } else if (filters.minPrice !== undefined) {
      whereConditions.price = MoreThanOrEqual(filters.minPrice);
    } else if (filters.maxPrice !== undefined) {
      whereConditions.price = LessThanOrEqual(filters.maxPrice);
    }

    // MoreThan for stock
    if (filters.inStock === true) {
      whereConditions.stock = MoreThan(0);
    } else if (filters.inStock === false) {
      whereConditions.stock = Equal(0);
    }

    // ArrayContains for tags (must have all)
    if (filters.tags && filters.tags.length > 0) {
      whereConditions.tags = ArrayContains(filters.tags);
    }

    // Not with ArrayOverlap for excluded tags
    if (filters.excludeTags && filters.excludeTags.length > 0) {
      whereConditions.tags = Not(ArrayOverlap(filters.excludeTags));
    }

    // ILike for search
    if (filters.searchTerm) {
      whereConditions.name = ILike(`%${filters.searchTerm}%`);
    }

    // IsNull / IsNotNull for discount
    if (filters.hasDiscount === true) {
      whereConditions.discount = IsNotNull();
    } else if (filters.hasDiscount === false) {
      whereConditions.discount = IsNull();
    }

    // In for brands
    if (filters.brands && filters.brands.length > 0) {
      whereConditions.brand = In(filters.brands);
    }

    return await this.productRepo.find({
      where: whereConditions,
      order: { createdAt: 'DESC' },
      take: 100
    });
  }
}
```

---

### **Combining Multiple Operators:**

```typescript
class AdvancedSearchService {
  async complexProductSearch(): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        // Multiple operators combined
        price: Between(50, 500),
        category: In(['Electronics', 'Computers']),
        stock: MoreThan(0),
        discount: IsNotNull(),
        name: ILike('%laptop%'),
        tags: ArrayContains(['featured']),
        isActive: true
      },
      order: {
        price: 'ASC'
      }
    });
  }

  // OR conditions with multiple operators
  async searchWithOR(): Promise<Product[]> {
    return await this.productRepo.find({
      where: [
        { 
          price: LessThan(100), 
          category: 'Books' 
        },
        { 
          discount: IsNotNull(), 
          stock: MoreThan(10) 
        }
      ]
    });
  }
}
```

---

### **Operator Comparison Table:**

| Operator | SQL Equivalent | Example |
|----------|---------------|---------|
| `Equal(x)` | `= x` | `price: Equal(100)` |
| `Not(x)` | `!= x` | `status: Not('deleted')` |
| `LessThan(x)` | `< x` | `price: LessThan(100)` |
| `LessThanOrEqual(x)` | `<= x` | `stock: LessThanOrEqual(10)` |
| `MoreThan(x)` | `> x` | `views: MoreThan(1000)` |
| `MoreThanOrEqual(x)` | `>= x` | `rating: MoreThanOrEqual(4)` |
| `Between(x, y)` | `BETWEEN x AND y` | `price: Between(10, 50)` |
| `In([...])` | `IN (...)` | `status: In(['active', 'pending'])` |
| `IsNull()` | `IS NULL` | `deletedAt: IsNull()` |
| `IsNotNull()` | `IS NOT NULL` | `email: IsNotNull()` |
| `Like(x)` | `LIKE x` | `name: Like('%book%')` |
| `ILike(x)` | `ILIKE x` | `email: ILike('%@gmail.com')` |
| `ArrayContains([...])` | `@>` | `tags: ArrayContains(['new'])` |
| `Raw(sql)` | Custom SQL | `Raw('column > 100')` |

---

### **Best Practices:**

1. ✅ **Use typed operators** - Better IDE support and type safety
2. ✅ **Combine operators** - Build complex queries easily
3. ✅ **Validate input** - Sanitize user input before using in queries
4. ✅ **Use In sparingly** - Large arrays can slow queries
5. ✅ **Prefer operators over Raw** - Raw should be last resort
6. ✅ **Index filtered columns** - Add indexes to improve performance

---

### **Interview Tips:**
- Explain the most common operators: In, Between, Like, IsNull
- Show how to combine multiple operators
- Discuss the Raw operator for custom SQL
- Mention array operators for PostgreSQL
- Emphasize type safety with operators vs raw SQL

</details>

<details>
<summary>110. How do you implement search with multiple conditions?</summary>

### Answer:

**Multiple condition searches** allow filtering data based on various criteria combined with AND/OR logic. TypeORM provides several approaches for building complex multi-condition queries.

---

### **Method 1: Object-Based WHERE (AND Logic):**

All conditions in the same object are combined with AND.

```typescript
import { Repository, Like, MoreThan, In } from 'typeorm';
import { Product } from './entities/Product';

class ProductService {
  constructor(private productRepo: Repository<Product>) {}

  // All conditions must match (AND)
  async searchProducts(filters: {
    minPrice: number;
    category: string;
    inStock: boolean;
  }): Promise<Product[]> {
    return await this.productRepo.find({
      where: {
        price: MoreThan(filters.minPrice),     // AND
        category: filters.category,            // AND
        stock: MoreThan(0)                     // AND
      }
    });
  }
}
```

---

### **Method 2: Array-Based WHERE (OR Logic):**

Each object in the array represents an OR condition.

```typescript
class ProductService {
  // Any condition can match (OR)
  async searchWithOR(): Promise<Product[]> {
    return await this.productRepo.find({
      where: [
        { category: 'Electronics', price: LessThan(1000) },  // OR
        { category: 'Books', inStock: true },                // OR
        { discount: MoreThan(20) }                           // OR
      ]
    });
    // SQL: WHERE (category='Electronics' AND price<1000) 
    //      OR (category='Books' AND inStock=true) 
    //      OR (discount>20)
  }
}
```

---

### **Method 3: Query Builder (Most Flexible):**

```typescript
class ProductSearchService {
  constructor(private productRepo: Repository<Product>) {}

  async advancedSearch(filters: {
    searchTerm?: string;
    categories?: string[];
    minPrice?: number;
    maxPrice?: number;
    brands?: string[];
    inStock?: boolean;
    rating?: number;
  }): Promise<Product[]> {
    const query = this.productRepo.createQueryBuilder('product');

    // Dynamic AND conditions
    if (filters.searchTerm) {
      query.andWhere(
        '(product.name ILIKE :term OR product.description ILIKE :term)',
        { term: `%${filters.searchTerm}%` }
      );
    }

    if (filters.categories && filters.categories.length > 0) {
      query.andWhere('product.category IN (:...categories)', {
        categories: filters.categories
      });
    }

    if (filters.minPrice !== undefined) {
      query.andWhere('product.price >= :minPrice', {
        minPrice: filters.minPrice
      });
    }

    if (filters.maxPrice !== undefined) {
      query.andWhere('product.price <= :maxPrice', {
        maxPrice: filters.maxPrice
      });
    }

    if (filters.brands && filters.brands.length > 0) {
      query.andWhere('product.brand IN (:...brands)', {
        brands: filters.brands
      });
    }

    if (filters.inStock === true) {
      query.andWhere('product.stock > 0');
    }

    if (filters.rating !== undefined) {
      query.andWhere('product.rating >= :rating', {
        rating: filters.rating
      });
    }

    return await query.getMany();
  }
}
```

---

### **Method 4: Complex AND/OR Combinations:**

```typescript
class AdvancedSearchService {
  constructor(private productRepo: Repository<Product>) {}

  async complexSearch(filters: {
    searchTerm?: string;
    priceRange?: { min: number; max: number };
    mustHaveCategories?: string[];
    optionalTags?: string[];
  }): Promise<Product[]> {
    const query = this.productRepo.createQueryBuilder('product');

    // Main search term (AND condition)
    if (filters.searchTerm) {
      query.andWhere(
        new Brackets(qb => {
          qb.where('product.name ILIKE :term', { term: `%${filters.searchTerm}%` })
            .orWhere('product.description ILIKE :term', { term: `%${filters.searchTerm}%` })
            .orWhere('product.sku ILIKE :term', { term: `%${filters.searchTerm}%` });
        })
      );
    }

    // Price range (AND condition)
    if (filters.priceRange) {
      query.andWhere(
        'product.price BETWEEN :minPrice AND :maxPrice',
        {
          minPrice: filters.priceRange.min,
          maxPrice: filters.priceRange.max
        }
      );
    }

    // Must have categories (AND condition)
    if (filters.mustHaveCategories && filters.mustHaveCategories.length > 0) {
      query.andWhere('product.category IN (:...categories)', {
        categories: filters.mustHaveCategories
      });
    }

    // Optional tags (OR condition within AND)
    if (filters.optionalTags && filters.optionalTags.length > 0) {
      query.andWhere(
        new Brackets(qb => {
          filters.optionalTags!.forEach((tag, index) => {
            qb.orWhere(`product.tags @> ARRAY[:tag${index}]`, {
              [`tag${index}`]: tag
            });
          });
        })
      );
    }

    return await query.getMany();
  }
}
```

---

### **Production Use Case: E-commerce Product Filter:**

```typescript
interface ProductFilters {
  // Search
  query?: string;
  
  // Category & Brand
  categories?: string[];
  brands?: string[];
  
  // Price
  minPrice?: number;
  maxPrice?: number;
  
  // Availability
  inStock?: boolean;
  freeShipping?: boolean;
  
  // Ratings & Reviews
  minRating?: number;
  hasReviews?: boolean;
  
  // Features
  tags?: string[];
  color?: string[];
  
  // Sorting & Pagination
  sortBy?: 'price' | 'rating' | 'newest' | 'popular';
  sortOrder?: 'ASC' | 'DESC';
  page?: number;
  limit?: number;
}

class ProductFilterService {
  constructor(private productRepo: Repository<Product>) {}

  async filterProducts(filters: ProductFilters) {
    const {
      page = 1,
      limit = 20,
      sortBy = 'newest',
      sortOrder = 'DESC'
    } = filters;

    const query = this.productRepo
      .createQueryBuilder('product')
      .leftJoinAndSelect('product.reviews', 'reviews')
      .leftJoinAndSelect('product.brand', 'brand');

    // TEXT SEARCH (OR across fields, but AND with other conditions)
    if (filters.query) {
      query.andWhere(
        new Brackets(qb => {
          qb.where('product.name ILIKE :query', { query: `%${filters.query}%` })
            .orWhere('product.description ILIKE :query')
            .orWhere('product.sku ILIKE :query')
            .orWhere('brand.name ILIKE :query');
        })
      );
    }

    // CATEGORY FILTER (IN)
    if (filters.categories && filters.categories.length > 0) {
      query.andWhere('product.category IN (:...categories)', {
        categories: filters.categories
      });
    }

    // BRAND FILTER (IN)
    if (filters.brands && filters.brands.length > 0) {
      query.andWhere('brand.name IN (:...brands)', {
        brands: filters.brands
      });
    }

    // PRICE RANGE
    if (filters.minPrice !== undefined && filters.maxPrice !== undefined) {
      query.andWhere('product.price BETWEEN :minPrice AND :maxPrice', {
        minPrice: filters.minPrice,
        maxPrice: filters.maxPrice
      });
    } else if (filters.minPrice !== undefined) {
      query.andWhere('product.price >= :minPrice', {
        minPrice: filters.minPrice
      });
    } else if (filters.maxPrice !== undefined) {
      query.andWhere('product.price <= :maxPrice', {
        maxPrice: filters.maxPrice
      });
    }

    // IN STOCK
    if (filters.inStock === true) {
      query.andWhere('product.stock > 0');
    }

    // FREE SHIPPING
    if (filters.freeShipping === true) {
      query.andWhere('product.freeShipping = true');
    }

    // MINIMUM RATING
    if (filters.minRating !== undefined) {
      query.andWhere('product.averageRating >= :minRating', {
        minRating: filters.minRating
      });
    }

    // HAS REVIEWS
    if (filters.hasReviews === true) {
      query.andWhere('product.reviewCount > 0');
    }

    // TAGS (must have all specified tags)
    if (filters.tags && filters.tags.length > 0) {
      query.andWhere('product.tags @> :tags', { tags: filters.tags });
    }

    // COLOR (OR condition)
    if (filters.color && filters.color.length > 0) {
      query.andWhere('product.color IN (:...colors)', {
        colors: filters.color
      });
    }

    // SORTING
    const sortMap = {
      price: 'product.price',
      rating: 'product.averageRating',
      newest: 'product.createdAt',
      popular: 'product.salesCount'
    };
    query.orderBy(sortMap[sortBy], sortOrder);

    // PAGINATION
    query.skip((page - 1) * limit).take(limit);

    // EXECUTE
    const [products, total] = await query.getManyAndCount();

    return {
      data: products,
      meta: {
        total,
        page,
        limit,
        totalPages: Math.ceil(total / limit),
        hasMore: page * limit < total
      }
    };
  }
}
```

---

### **Dynamic Filter Builder:**

```typescript
class DynamicFilterService {
  buildWhereConditions(filters: Record<string, any>) {
    const where: any = {};

    // Automatically build conditions from filters
    Object.entries(filters).forEach(([key, value]) => {
      if (value === undefined || value === null) return;

      // Handle different value types
      if (Array.isArray(value)) {
        where[key] = In(value);
      } else if (typeof value === 'object' && 'min' in value && 'max' in value) {
        where[key] = Between(value.min, value.max);
      } else if (typeof value === 'string' && value.includes('%')) {
        where[key] = Like(value);
      } else {
        where[key] = Equal(value);
      }
    });

    return where;
  }

  async search(filters: any): Promise<Product[]> {
    const where = this.buildWhereConditions(filters);
    return await this.productRepo.find({ where });
  }
}
```

---

### **Brackets for Complex Logic:**

```typescript
// (A OR B) AND (C OR D)
async complexLogic(): Promise<Product[]> {
  return await this.productRepo
    .createQueryBuilder('product')
    .where(
      new Brackets(qb => {
        qb.where('product.category = :cat1', { cat1: 'Electronics' })
          .orWhere('product.category = :cat2', { cat2: 'Computers' });
      })
    )
    .andWhere(
      new Brackets(qb => {
        qb.where('product.price < :lowPrice', { lowPrice: 100 })
          .orWhere('product.discount > :highDiscount', { highDiscount: 20 });
      })
    )
    .getMany();
}
```

---

### **Best Practices:**

1. ✅ **Validate all inputs** - Sanitize user-provided filter values
2. ✅ **Use parameterized queries** - Prevent SQL injection
3. ✅ **Index filtered columns** - Improve query performance
4. ✅ **Implement pagination** - Never return unlimited results
5. ✅ **Add default limits** - Protect against large result sets
6. ✅ **Cache frequent searches** - Store common filter combinations
7. ✅ **Log slow queries** - Monitor and optimize performance

---

### **Interview Tips:**
- Explain AND (object) vs OR (array) in find() method
- Show Query Builder for complex dynamic conditions
- Demonstrate Brackets for nested logic
- Discuss pagination and performance with multiple filters
- Mention the importance of indexes on filtered columns

</details>

<details>
<summary>111. How do you perform date range queries?</summary>

### Answer:

**Date range queries** allow filtering records based on date and time values. TypeORM provides several operators and methods for working with dates effectively.

---

### **Method 1: Using Between Operator:**

```typescript
import { Repository, Between, MoreThan, LessThan } from 'typeorm';
import { Order } from './entities/Order';

class OrderService {
  constructor(private orderRepo: Repository<Order>) {}

  // Date range using Between
  async getOrdersByDateRange(startDate: Date, endDate: Date): Promise<Order[]> {
    return await this.orderRepo.find({
      where: {
        createdAt: Between(startDate, endDate)
      },
      order: {
        createdAt: 'DESC'
      }
    });
  }

  // With additional conditions
  async getOrdersByDateRangeAndStatus(
    startDate: Date,
    endDate: Date,
    status: string
  ): Promise<Order[]> {
    return await this.orderRepo.find({
      where: {
        createdAt: Between(startDate, endDate),
        status: status
      }
    });
  }
}
```

---

### **Method 2: Using Comparison Operators:**

```typescript
class OrderService {
  // Orders after a specific date
  async getOrdersAfter(date: Date): Promise<Order[]> {
    return await this.orderRepo.find({
      where: {
        createdAt: MoreThan(date)  // createdAt > date
      }
    });
  }

  // Orders before a specific date
  async getOrdersBefore(date: Date): Promise<Order[]> {
    return await this.orderRepo.find({
      where: {
        createdAt: LessThan(date)  // createdAt < date
      }
    });
  }

  // Combining operators for custom range
  async getOrdersInCustomRange(
    minDate: Date,
    maxDate: Date
  ): Promise<Order[]> {
    return await this.orderRepo.find({
      where: [
        {
          createdAt: MoreThan(minDate),
          updatedAt: LessThan(maxDate)
        }
      ]
    });
  }
}
```

---

### **Method 3: Query Builder for Complex Date Queries:**

```typescript
class OrderService {
  constructor(private orderRepo: Repository<Order>) {}

  // Flexible date range with Query Builder
  async getOrdersWithQueryBuilder(
    startDate?: Date,
    endDate?: Date
  ): Promise<Order[]> {
    const query = this.orderRepo.createQueryBuilder('order');

    if (startDate) {
      query.andWhere('order.createdAt >= :startDate', { startDate });
    }

    if (endDate) {
      query.andWhere('order.createdAt <= :endDate', { endDate });
    }

    return await query.orderBy('order.createdAt', 'DESC').getMany();
  }

  // Date functions in queries
  async getOrdersByMonth(year: number, month: number): Promise<Order[]> {
    return await this.orderRepo
      .createQueryBuilder('order')
      .where('EXTRACT(YEAR FROM order.createdAt) = :year', { year })
      .andWhere('EXTRACT(MONTH FROM order.createdAt) = :month', { month })
      .getMany();
  }

  // Orders from last N days
  async getRecentOrders(days: number): Promise<Order[]> {
    return await this.orderRepo
      .createQueryBuilder('order')
      .where('order.createdAt > NOW() - INTERVAL :days DAY', { days })
      .orderBy('order.createdAt', 'DESC')
      .getMany();
  }
}
```

---

### **Method 4: Using Raw SQL for Database-Specific Features:**

```typescript
class AnalyticsService {
  constructor(private dataSource: DataSource) {}

  // PostgreSQL - Advanced date functions
  async getOrdersByDateRangePostgres(
    startDate: Date,
    endDate: Date
  ): Promise<any[]> {
    return await this.dataSource.query(
      `
      SELECT 
        DATE_TRUNC('day', created_at) as date,
        COUNT(*) as order_count,
        SUM(total) as total_revenue
      FROM orders
      WHERE created_at BETWEEN $1 AND $2
      GROUP BY DATE_TRUNC('day', created_at)
      ORDER BY date DESC
      `,
      [startDate, endDate]
    );
  }

  // MySQL - Date range queries
  async getOrdersByDateRangeMySQL(
    startDate: Date,
    endDate: Date
  ): Promise<any[]> {
    return await this.dataSource.query(
      `
      SELECT 
        DATE(created_at) as date,
        COUNT(*) as order_count,
        SUM(total) as total_revenue
      FROM orders
      WHERE created_at BETWEEN ? AND ?
      GROUP BY DATE(created_at)
      ORDER BY date DESC
      `,
      [startDate, endDate]
    );
  }
}
```

---

### **Production Use Case: Date Range Filters with Helper Functions:**

```typescript
class DateHelper {
  // Get start of day
  static startOfDay(date: Date): Date {
    const result = new Date(date);
    result.setHours(0, 0, 0, 0);
    return result;
  }

  // Get end of day
  static endOfDay(date: Date): Date {
    const result = new Date(date);
    result.setHours(23, 59, 59, 999);
    return result;
  }

  // Get date range for last N days
  static lastNDays(days: number): { start: Date; end: Date } {
    const end = new Date();
    const start = new Date();
    start.setDate(start.getDate() - days);
    return { start, end };
  }

  // Get current month range
  static currentMonth(): { start: Date; end: Date } {
    const now = new Date();
    const start = new Date(now.getFullYear(), now.getMonth(), 1);
    const end = new Date(now.getFullYear(), now.getMonth() + 1, 0, 23, 59, 59, 999);
    return { start, end };
  }

  // Get last month range
  static lastMonth(): { start: Date; end: Date } {
    const now = new Date();
    const start = new Date(now.getFullYear(), now.getMonth() - 1, 1);
    const end = new Date(now.getFullYear(), now.getMonth(), 0, 23, 59, 59, 999);
    return { start, end };
  }

  // Get year to date
  static yearToDate(): { start: Date; end: Date } {
    const now = new Date();
    const start = new Date(now.getFullYear(), 0, 1);
    return { start, end: now };
  }
}

class ReportService {
  constructor(private orderRepo: Repository<Order>) {}

  // Today's orders
  async getTodayOrders(): Promise<Order[]> {
    const today = new Date();
    return await this.orderRepo.find({
      where: {
        createdAt: Between(
          DateHelper.startOfDay(today),
          DateHelper.endOfDay(today)
        )
      }
    });
  }

  // Last 7 days
  async getLastWeekOrders(): Promise<Order[]> {
    const { start, end } = DateHelper.lastNDays(7);
    return await this.orderRepo.find({
      where: {
        createdAt: Between(start, end)
      },
      order: {
        createdAt: 'DESC'
      }
    });
  }

  // Current month
  async getCurrentMonthOrders(): Promise<Order[]> {
    const { start, end } = DateHelper.currentMonth();
    return await this.orderRepo.find({
      where: {
        createdAt: Between(start, end)
      }
    });
  }

  // Year to date
  async getYearToDateOrders(): Promise<Order[]> {
    const { start, end } = DateHelper.yearToDate();
    return await this.orderRepo.find({
      where: {
        createdAt: Between(start, end)
      }
    });
  }

  // Custom date range with validation
  async getOrdersByCustomRange(
    startDate: string,
    endDate: string
  ): Promise<Order[]> {
    const start = new Date(startDate);
    const end = new Date(endDate);

    // Validate dates
    if (isNaN(start.getTime()) || isNaN(end.getTime())) {
      throw new Error('Invalid date format');
    }

    if (start > end) {
      throw new Error('Start date must be before end date');
    }

    // Limit range to prevent performance issues
    const daysDiff = (end.getTime() - start.getTime()) / (1000 * 60 * 60 * 24);
    if (daysDiff > 365) {
      throw new Error('Date range cannot exceed 365 days');
    }

    return await this.orderRepo.find({
      where: {
        createdAt: Between(
          DateHelper.startOfDay(start),
          DateHelper.endOfDay(end)
        )
      },
      order: {
        createdAt: 'DESC'
      }
    });
  }
}
```

---

### **Aggregation by Date Periods:**

```typescript
class AnalyticsService {
  constructor(private orderRepo: Repository<Order>) {}

  // Daily aggregation
  async getDailySales(startDate: Date, endDate: Date): Promise<any[]> {
    return await this.orderRepo
      .createQueryBuilder('order')
      .select("DATE_TRUNC('day', order.createdAt)", 'date')
      .addSelect('COUNT(order.id)', 'count')
      .addSelect('SUM(order.total)', 'revenue')
      .addSelect('AVG(order.total)', 'avgOrderValue')
      .where('order.createdAt BETWEEN :startDate AND :endDate', {
        startDate,
        endDate
      })
      .groupBy('date')
      .orderBy('date', 'DESC')
      .getRawMany();
  }

  // Monthly aggregation
  async getMonthlySales(year: number): Promise<any[]> {
    return await this.orderRepo
      .createQueryBuilder('order')
      .select("TO_CHAR(order.createdAt, 'YYYY-MM')", 'month')
      .addSelect('COUNT(order.id)', 'count')
      .addSelect('SUM(order.total)', 'revenue')
      .where('EXTRACT(YEAR FROM order.createdAt) = :year', { year })
      .groupBy('month')
      .orderBy('month', 'ASC')
      .getRawMany();
  }
}
```

---

### **Timezone Handling:**

```typescript
class DateService {
  // Convert to UTC before querying
  async getOrdersInTimezone(
    startDate: Date,
    endDate: Date,
    timezone: string = 'UTC'
  ): Promise<Order[]> {
    return await this.orderRepo
      .createQueryBuilder('order')
      .where(
        "order.createdAt AT TIME ZONE :timezone BETWEEN :start AND :end",
        {
          timezone,
          start: startDate,
          end: endDate
        }
      )
      .getMany();
  }
}
```

---

### **Best Practices:**

1. ✅ **Use Between for ranges** - Cleaner and more readable
2. ✅ **Include time boundaries** - Use startOfDay/endOfDay helpers
3. ✅ **Validate date inputs** - Check format and logical order
4. ✅ **Limit date ranges** - Prevent performance issues with huge ranges
5. ✅ **Index date columns** - Essential for query performance
6. ✅ **Handle timezones** - Convert to UTC or specify timezone
7. ✅ **Use helper functions** - Standardize common date ranges

---

### **Interview Tips:**
- Explain Between vs MoreThan/LessThan for date ranges
- Discuss timezone considerations in date queries
- Show helper functions for common date ranges (today, last week, etc.)
- Mention performance implications of date range queries
- Demonstrate aggregation by date periods

</details>

<details>
<summary>112. How do you query JSON columns?</summary>

### Answer:

**JSON columns** allow storing structured data in a flexible format. TypeORM provides methods to query JSON/JSONB columns, particularly in PostgreSQL, MySQL 5.7+, and other databases supporting JSON.

---

### **Entity Definition with JSON Columns:**

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // JSON column
  @Column('json')
  metadata: {
    brand?: string;
    color?: string;
    weight?: number;
    dimensions?: {
      width: number;
      height: number;
      depth: number;
    };
  };

  // JSONB column (PostgreSQL - better performance with indexing)
  @Column('jsonb')
  specifications: any;

  // Simple JSON for settings
  @Column('simple-json') // Serializes to string
  settings: {
    notifications: boolean;
    theme: string;
  };
}

@Entity()
class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  @Column('jsonb')
  preferences: {
    language: string;
    timezone: string;
    notifications: {
      email: boolean;
      sms: boolean;
      push: boolean;
    };
  };
}
```

---

### **Method 1: Query JSON Fields (PostgreSQL):**

```typescript
import { Repository } from 'typeorm';

class ProductService {
  constructor(private productRepo: Repository<Product>) {}

  // Access JSON property
  async findByBrand(brand: string): Promise<Product[]> {
    return await this.productRepo
      .createQueryBuilder('product')
      .where("product.metadata->>'brand' = :brand", { brand })
      .getMany();
  }

  // Access nested JSON property
  async findByWidth(width: number): Promise<Product[]> {
    return await this.productRepo
      .createQueryBuilder('product')
      .where("(product.metadata->'dimensions'->>'width')::int = :width", { 
        width 
      })
      .getMany();
  }

  // JSONB contains operator (@>)
  async findByJsonContains(criteria: any): Promise<Product[]> {
    return await this.productRepo
      .createQueryBuilder('product')
      .where('product.specifications @> :criteria', { 
        criteria: JSON.stringify(criteria) 
      })
      .getMany();
  }

  // Check if JSON key exists
  async findProductsWithColor(): Promise<Product[]> {
    return await this.productRepo
      .createQueryBuilder('product')
      .where("product.metadata ? 'color'")
      .getMany();
  }
}
```

---

### **Method 2: Using Raw SQL for Complex JSON Queries:**

```typescript
class ProductService {
  constructor(private dataSource: DataSource) {}

  // Query array within JSON
  async findProductsByTag(tag: string): Promise<Product[]> {
    return await this.dataSource.query(
      `
      SELECT * FROM products
      WHERE specifications->'tags' @> $1
      `,
      [JSON.stringify([tag])]
    );
  }

  // JSON path query
  async findByNestedProperty(value: string): Promise<Product[]> {
    return await this.dataSource.query(
      `
      SELECT * FROM products
      WHERE specifications #>> '{features,processor,brand}' = $1
      `,
      [value]
    );
  }

  // JSON array operations
  async findProductsWithMultipleTags(tags: string[]): Promise<Product[]> {
    return await this.dataSource.query(
      `
      SELECT * FROM products
      WHERE specifications->'tags' ?& $1
      `,
      [tags]
    );
  }
}
```

---

### **Method 3: MySQL JSON Queries:**

```typescript
class ProductServiceMySQL {
  constructor(private productRepo: Repository<Product>) {}

  // MySQL JSON_EXTRACT
  async findByBrandMySQL(brand: string): Promise<Product[]> {
    return await this.productRepo
      .createQueryBuilder('product')
      .where("JSON_EXTRACT(product.metadata, '$.brand') = :brand", { brand })
      .getMany();
  }

  // JSON_CONTAINS
  async findByJsonContainsMySQL(brand: string): Promise<Product[]> {
    return await this.productRepo
      .createQueryBuilder('product')
      .where(
        "JSON_CONTAINS(product.metadata, :json, '$.brand')",
        { json: JSON.stringify(brand) }
      )
      .getMany();
  }

  // JSON_SEARCH
  async searchInJsonMySQL(searchTerm: string): Promise<Product[]> {
    return await this.productRepo
      .createQueryBuilder('product')
      .where(
        "JSON_SEARCH(product.specifications, 'one', :term) IS NOT NULL",
        { term: searchTerm }
      )
      .getMany();
  }
}
```

---

### **Production Use Case: User Preferences Filter:**

```typescript
class UserPreferenceService {
  constructor(private userRepo: Repository<User>) {}

  // Find users with specific notification preference
  async findUsersWithEmailNotifications(): Promise<User[]> {
    return await this.userRepo
      .createQueryBuilder('user')
      .where("user.preferences->'notifications'->>'email' = 'true'")
      .getMany();
  }

  // Find users by timezone
  async findUsersByTimezone(timezone: string): Promise<User[]> {
    return await this.userRepo
      .createQueryBuilder('user')
      .where("user.preferences->>'timezone' = :timezone", { timezone })
      .getMany();
  }

  // Complex JSON query with multiple conditions
  async findUsersWithCriteria(criteria: {
    language?: string;
    emailNotifications?: boolean;
    timezone?: string;
  }): Promise<User[]> {
    const query = this.userRepo.createQueryBuilder('user');

    if (criteria.language) {
      query.andWhere("user.preferences->>'language' = :language", {
        language: criteria.language
      });
    }

    if (criteria.emailNotifications !== undefined) {
      query.andWhere(
        "user.preferences->'notifications'->>'email' = :emailNotif",
        { emailNotif: criteria.emailNotifications.toString() }
      );
    }

    if (criteria.timezone) {
      query.andWhere("user.preferences->>'timezone' = :timezone", {
        timezone: criteria.timezone
      });
    }

    return await query.getMany();
  }

  // Update JSON field
  async updateUserPreference(
    userId: number,
    key: string,
    value: any
  ): Promise<void> {
    await this.userRepo
      .createQueryBuilder()
      .update(User)
      .set({
        preferences: () => `jsonb_set(preferences, '{${key}}', '"${value}"')`
      })
      .where('id = :userId', { userId })
      .execute();
  }
}
```

---

### **JSON Aggregation and Analysis:**

```typescript
class AnalyticsService {
  constructor(private dataSource: DataSource) {}

  // Aggregate JSON data
  async getPreferenceStats(): Promise<any> {
    return await this.dataSource.query(`
      SELECT 
        preferences->>'language' as language,
        COUNT(*) as user_count
      FROM users
      WHERE preferences ? 'language'
      GROUP BY preferences->>'language'
      ORDER BY user_count DESC
    `);
  }

  // Extract and aggregate nested values
  async getNotificationStats(): Promise<any> {
    return await this.dataSource.query(`
      SELECT 
        (preferences->'notifications'->>'email')::boolean as email_enabled,
        (preferences->'notifications'->>'sms')::boolean as sms_enabled,
        COUNT(*) as count
      FROM users
      GROUP BY email_enabled, sms_enabled
    `);
  }
}
```

---

### **JSON Indexing for Performance (PostgreSQL):**

```typescript
// Migration to add JSON indexes
export class AddJsonIndexes implements MigrationInterface {
  async up(queryRunner: QueryRunner): Promise<void> {
    // GIN index on entire JSONB column
    await queryRunner.query(`
      CREATE INDEX idx_product_specifications 
      ON products USING GIN (specifications)
    `);

    // Index on specific JSON path
    await queryRunner.query(`
      CREATE INDEX idx_product_metadata_brand 
      ON products ((metadata->>'brand'))
    `);

    // Partial index for JSON condition
    await queryRunner.query(`
      CREATE INDEX idx_user_email_notifications 
      ON users ((preferences->'notifications'->>'email'))
      WHERE (preferences->'notifications'->>'email')::boolean = true
    `);
  }

  async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP INDEX idx_product_specifications`);
    await queryRunner.query(`DROP INDEX idx_product_metadata_brand`);
    await queryRunner.query(`DROP INDEX idx_user_email_notifications`);
  }
}
```

---

### **JSON Operators (PostgreSQL):**

| Operator | Description | Example |
|----------|-------------|---------|
| `->` | Get JSON object field | `metadata->'brand'` |
| `->>` | Get JSON object field as text | `metadata->>'brand'` |
| `#>` | Get JSON object at path | `metadata#>'{dimensions,width}'` |
| `#>>` | Get JSON object at path as text | `metadata#>>'{dimensions,width}'` |
| `@>` | Contains | `specifications @> '{"color":"red"}'` |
| `<@` | Contained by | `'{"color":"red"}' <@ specifications` |
| `?` | Key exists | `metadata ? 'brand'` |
| `?|` | Any key exists | `metadata ?| array['brand','color']` |
| `?&` | All keys exist | `metadata ?& array['brand','color']` |

---

### **Type-Safe JSON Querying:**

```typescript
// Define JSON schema types
interface ProductMetadata {
  brand: string;
  color: string;
  weight: number;
  dimensions: {
    width: number;
    height: number;
    depth: number;
  };
}

class TypeSafeProductService {
  constructor(private productRepo: Repository<Product>) {}

  // Type-safe JSON query
  async findByMetadata(
    criteria: Partial<ProductMetadata>
  ): Promise<Product[]> {
    const query = this.productRepo.createQueryBuilder('product');

    if (criteria.brand) {
      query.andWhere("product.metadata->>'brand' = :brand", {
        brand: criteria.brand
      });
    }

    if (criteria.color) {
      query.andWhere("product.metadata->>'color' = :color", {
        color: criteria.color
      });
    }

    if (criteria.dimensions?.width) {
      query.andWhere(
        "(product.metadata->'dimensions'->>'width')::int = :width",
        { width: criteria.dimensions.width }
      );
    }

    return await query.getMany();
  }
}
```

---

### **Updating JSON Fields:**

```typescript
class ProductUpdateService {
  constructor(private productRepo: Repository<Product>) {}

  // Update entire JSON column
  async updateMetadata(
    productId: number,
    metadata: ProductMetadata
  ): Promise<void> {
    await this.productRepo.update(productId, { metadata });
  }

  // Update specific JSON field (PostgreSQL)
  async updateJsonField(
    productId: number,
    path: string,
    value: any
  ): Promise<void> {
    await this.productRepo
      .createQueryBuilder()
      .update(Product)
      .set({
        metadata: () => `jsonb_set(metadata, '{${path}}', '"${value}"')`
      })
      .where('id = :productId', { productId })
      .execute();
  }

  // Merge JSON data
  async mergeMetadata(
    productId: number,
    newData: Partial<ProductMetadata>
  ): Promise<void> {
    await this.productRepo
      .createQueryBuilder()
      .update(Product)
      .set({
        metadata: () => `metadata || '${JSON.stringify(newData)}'::jsonb`
      })
      .where('id = :productId', { productId })
      .execute();
  }
}
```

---

### **Best Practices:**

1. ✅ **Use JSONB over JSON (PostgreSQL)** - Better performance with indexing
2. ✅ **Create GIN indexes** - Essential for JSON query performance
3. ✅ **Define TypeScript interfaces** - Type safety for JSON structures
4. ✅ **Validate JSON data** - Ensure data integrity before saving
5. ✅ **Don't overuse JSON** - Use proper columns for frequently queried data
6. ✅ **Document JSON schema** - Make structure clear for team
7. ✅ **Handle null values** - JSON fields can be null

---

### **Interview Tips:**
- Explain difference between JSON and JSONB (PostgreSQL)
- Show JSON operators: `->`, `->>`, `@>`, `?`
- Discuss when to use JSON vs relational columns
- Mention importance of GIN indexes for JSON queries
- Demonstrate both simple and nested JSON queries

</details>

<details>
<summary>113. How do you use raw SQL queries in TypeORM?</summary>

### Answer:

**Raw SQL queries** allow executing custom SQL directly when TypeORM's query builder doesn't support specific database features or when you need fine-grained control over query optimization.

---

### **Method 1: Using DataSource.query():**

```typescript
import { DataSource } from 'typeorm';

class UserService {
  constructor(private dataSource: DataSource) {}

  // Simple SELECT query
  async getUsersRaw(): Promise<any[]> {
    return await this.dataSource.query('SELECT * FROM users WHERE age > $1', [18]);
  }

  // With multiple parameters (PostgreSQL)
  async searchUsers(name: string, minAge: number): Promise<any[]> {
    return await this.dataSource.query(
      'SELECT * FROM users WHERE name ILIKE $1 AND age >= $2',
      [`%${name}%`, minAge]
    );
  }

  // MySQL parameterized query
  async searchUsersMySQL(name: string, minAge: number): Promise<any[]> {
    return await this.dataSource.query(
      'SELECT * FROM users WHERE name LIKE ? AND age >= ?',
      [`%${name}%`, minAge]
    );
  }

  // Complex query with JOINs
  async getUsersWithOrders(): Promise<any[]> {
    return await this.dataSource.query(`
      SELECT 
        u.id,
        u.name,
        u.email,
        COUNT(o.id) as order_count,
        COALESCE(SUM(o.total), 0) as total_spent
      FROM users u
      LEFT JOIN orders o ON u.id = o.user_id
      GROUP BY u.id, u.name, u.email
      HAVING COUNT(o.id) > 0
      ORDER BY total_spent DESC
      LIMIT 10
    `);
  }
}
```

---

### **Method 2: Using EntityManager.query():**

```typescript
import { EntityManager } from 'typeorm';

class OrderService {
  constructor(private entityManager: EntityManager) {}

  // Within transaction
  async processOrderWithRawSQL(orderId: number): Promise<void> {
    await this.entityManager.transaction(async (transactionalManager) => {
      // Raw SQL within transaction
      await transactionalManager.query(
        'UPDATE orders SET status = $1 WHERE id = $2',
        ['PROCESSING', orderId]
      );

      await transactionalManager.query(
        'UPDATE products SET stock = stock - 1 WHERE id IN (SELECT product_id FROM order_items WHERE order_id = $1)',
        [orderId]
      );
    });
  }
}
```

---

### **Method 3: Using Repository.query():**

```typescript
import { Repository } from 'typeorm';
import { User } from './entities/User';

class UserRepository {
  constructor(private userRepo: Repository<User>) {}

  // Execute raw query through repository
  async customQuery(sql: string, parameters?: any[]): Promise<any[]> {
    return await this.userRepo.query(sql, parameters);
  }

  // Database-specific features
  async searchWithFullText(searchTerm: string): Promise<any[]> {
    return await this.userRepo.query(
      `
      SELECT *, 
        ts_rank(to_tsvector('english', name || ' ' || bio), 
                to_tsquery('english', $1)) as rank
      FROM users
      WHERE to_tsvector('english', name || ' ' || bio) 
            @@ to_tsquery('english', $1)
      ORDER BY rank DESC
      LIMIT 20
      `,
      [searchTerm.replace(/\s+/g, ' & ')]
    );
  }
}
```

---

### **Method 4: INSERT, UPDATE, DELETE with Raw SQL:**

```typescript
class RawSQLService {
  constructor(private dataSource: DataSource) {}

  // INSERT
  async insertUser(name: string, email: string): Promise<void> {
    await this.dataSource.query(
      'INSERT INTO users (name, email, created_at) VALUES ($1, $2, NOW())',
      [name, email]
    );
  }

  // INSERT with RETURNING (PostgreSQL)
  async insertUserReturning(name: string, email: string): Promise<any> {
    const result = await this.dataSource.query(
      'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *',
      [name, email]
    );
    return result[0]; // Returns the inserted row
  }

  // Bulk INSERT
  async bulkInsert(users: Array<{ name: string; email: string }>): Promise<void> {
    const values = users.map((_, i) => `($${i * 2 + 1}, $${i * 2 + 2})`).join(',');
    const params = users.flatMap(u => [u.name, u.email]);

    await this.dataSource.query(
      `INSERT INTO users (name, email) VALUES ${values}`,
      params
    );
  }

  // UPDATE
  async updateUser(userId: number, name: string): Promise<void> {
    await this.dataSource.query(
      'UPDATE users SET name = $1, updated_at = NOW() WHERE id = $2',
      [name, userId]
    );
  }

  // DELETE
  async deleteUser(userId: number): Promise<void> {
    await this.dataSource.query(
      'DELETE FROM users WHERE id = $1',
      [userId]
    );
  }

  // Conditional DELETE
  async deleteInactiveUsers(days: number): Promise<number> {
    const result = await this.dataSource.query(
      `DELETE FROM users 
       WHERE last_login_at < NOW() - INTERVAL '${days} days' 
       RETURNING id`
    );
    return result.length; // Number of deleted rows
  }
}
```

---

### **Production Use Case: Complex Analytics Query:**

```typescript
class AnalyticsService {
  constructor(private dataSource: DataSource) {}

  async getDashboardStats(startDate: Date, endDate: Date): Promise<any> {
    return await this.dataSource.query(
      `
      WITH daily_stats AS (
        SELECT 
          DATE_TRUNC('day', created_at) as date,
          COUNT(*) as order_count,
          SUM(total) as revenue,
          AVG(total) as avg_order_value
        FROM orders
        WHERE created_at BETWEEN $1 AND $2
        GROUP BY DATE_TRUNC('day', created_at)
      ),
      user_stats AS (
        SELECT 
          COUNT(DISTINCT user_id) as unique_customers,
          COUNT(*) as total_orders
        FROM orders
        WHERE created_at BETWEEN $1 AND $2
      ),
      product_stats AS (
        SELECT 
          p.name,
          COUNT(oi.id) as times_ordered,
          SUM(oi.quantity) as total_quantity
        FROM order_items oi
        JOIN products p ON oi.product_id = p.id
        JOIN orders o ON oi.order_id = o.id
        WHERE o.created_at BETWEEN $1 AND $2
        GROUP BY p.id, p.name
        ORDER BY times_ordered DESC
        LIMIT 5
      )
      SELECT 
        json_build_object(
          'daily_stats', (SELECT json_agg(daily_stats) FROM daily_stats),
          'user_stats', (SELECT row_to_json(user_stats) FROM user_stats),
          'top_products', (SELECT json_agg(product_stats) FROM product_stats)
        ) as dashboard_data
      `,
      [startDate, endDate]
    );
  }

  // Window functions
  async getRankings(): Promise<any[]> {
    return await this.dataSource.query(`
      SELECT 
        user_id,
        name,
        total_orders,
        total_spent,
        RANK() OVER (ORDER BY total_spent DESC) as spending_rank,
        PERCENT_RANK() OVER (ORDER BY total_spent DESC) as spending_percentile
      FROM (
        SELECT 
          u.id as user_id,
          u.name,
          COUNT(o.id) as total_orders,
          SUM(o.total) as total_spent
        FROM users u
        LEFT JOIN orders o ON u.id = o.user_id
        GROUP BY u.id, u.name
      ) user_totals
      ORDER BY spending_rank
      LIMIT 100
    `);
  }
}
```

---

### **Handling Different Parameter Styles:**

```typescript
class QueryParameterService {
  constructor(private dataSource: DataSource) {}

  // PostgreSQL - Numbered parameters ($1, $2, ...)
  async postgresQuery(): Promise<any[]> {
    return await this.dataSource.query(
      'SELECT * FROM users WHERE name = $1 AND age > $2',
      ['John', 25]
    );
  }

  // MySQL - Question mark placeholders (?, ?, ...)
  async mysqlQuery(): Promise<any[]> {
    return await this.dataSource.query(
      'SELECT * FROM users WHERE name = ? AND age > ?',
      ['John', 25]
    );
  }

  // SQLite - Question mark placeholders
  async sqliteQuery(): Promise<any[]> {
    return await this.dataSource.query(
      'SELECT * FROM users WHERE name = ? AND age > ?',
      ['John', 25]
    );
  }
}
```

---

### **Error Handling with Raw Queries:**

```typescript
class SafeRawQueryService {
  constructor(private dataSource: DataSource) {}

  async executeRawQuery(sql: string, params: any[]): Promise<any> {
    try {
      const result = await this.dataSource.query(sql, params);
      return { success: true, data: result };
    } catch (error) {
      console.error('Raw query failed:', {
        sql,
        params,
        error: error.message
      });

      // Handle specific error types
      if (error.code === '23505') { // Unique violation
        throw new Error('Duplicate entry');
      }

      if (error.code === '23503') { // Foreign key violation
        throw new Error('Referenced record does not exist');
      }

      throw new Error('Database query failed');
    }
  }
}
```

---

### **Combining Raw SQL with Query Builder:**

```typescript
class HybridQueryService {
  constructor(private userRepo: Repository<User>) {}

  // Mix query builder with raw SQL
  async complexHybridQuery(minAge: number): Promise<User[]> {
    return await this.userRepo
      .createQueryBuilder('user')
      .where('user.age > :minAge', { minAge })
      .andWhere(
        // Raw SQL for complex condition
        qb => 
          `EXISTS (
            SELECT 1 FROM orders 
            WHERE orders.user_id = user.id 
            AND orders.total > 1000
          )`
      )
      .getMany();
  }
}
```

---

### **Performance Optimization with Raw SQL:**

```typescript
class OptimizedQueryService {
  constructor(private dataSource: DataSource) {}

  // Use EXPLAIN to analyze query
  async analyzeQuery(sql: string): Promise<any> {
    return await this.dataSource.query(`EXPLAIN ANALYZE ${sql}`);
  }

  // Batch operations
  async batchUpdate(ids: number[], status: string): Promise<void> {
    // More efficient than individual updates
    await this.dataSource.query(
      `UPDATE orders SET status = $1 WHERE id = ANY($2::int[])`,
      [status, ids]
    );
  }

  // UPSERT (PostgreSQL)
  async upsertUser(email: string, name: string): Promise<void> {
    await this.dataSource.query(
      `
      INSERT INTO users (email, name)
      VALUES ($1, $2)
      ON CONFLICT (email)
      DO UPDATE SET name = EXCLUDED.name, updated_at = NOW()
      `,
      [email, name]
    );
  }
}
```

---

### **Best Practices:**

1. ✅ **Use parameterized queries** - Prevent SQL injection
2. ✅ **Never concatenate user input** - Always use parameters
3. ✅ **Handle errors properly** - Wrap in try-catch
4. ✅ **Use raw SQL sparingly** - Prefer query builder when possible
5. ✅ **Document complex queries** - Add comments explaining logic
6. ✅ **Test thoroughly** - Raw SQL bypasses TypeORM validation
7. ✅ **Be database-specific aware** - Different syntax for different DBs

---

### **Interview Tips:**
- Explain when to use raw SQL vs query builder
- Emphasize parameterized queries for security
- Show awareness of database-specific syntax differences
- Discuss performance benefits of raw SQL for complex queries
- Mention importance of testing and error handling

</details>

<details>
<summary>114. What is the difference between query() and manager.query()?</summary>

### Answer:

Both `query()` and `manager.query()` execute raw SQL, but they differ in **context**, **scope**, and **transaction handling**. Understanding these differences is crucial for proper usage in production applications.

---

### **1. DataSource.query() - Global Database Access:**

```typescript
import { DataSource } from 'typeorm';

class UserService {
  constructor(private dataSource: DataSource) {}

  // Executes query on any available connection from pool
  async getAllUsers(): Promise<any[]> {
    return await this.dataSource.query('SELECT * FROM users');
  }

  // Not tied to any transaction
  async createUser(name: string, email: string): Promise<void> {
    await this.dataSource.query(
      'INSERT INTO users (name, email) VALUES ($1, $2)',
      [name, email]
    );
  }
}
```

**Characteristics:**
- ✅ Uses any connection from the pool
- ❌ Not part of a transaction unless explicitly wrapped
- ✅ Good for standalone queries
- ❌ Can't guarantee same connection across multiple calls

---

### **2. EntityManager.query() - Transaction-Aware:**

```typescript
import { EntityManager } from 'typeorm';

class OrderService {
  constructor(private entityManager: EntityManager) {}

  // Uses the manager's connection (may be transactional)
  async processOrder(orderId: number): Promise<void> {
    // If entityManager is transactional, this query is part of transaction
    await this.entityManager.query(
      'UPDATE orders SET status = $1 WHERE id = $2',
      ['PROCESSING', orderId]
    );
  }

  // Within explicit transaction
  async processOrderInTransaction(orderId: number): Promise<void> {
    await this.entityManager.transaction(async (transactionalManager) => {
      // These queries share the same connection and transaction
      await transactionalManager.query(
        'UPDATE orders SET status = $1 WHERE id = $2',
        ['PROCESSING', orderId]
      );

      await transactionalManager.query(
        'UPDATE inventory SET quantity = quantity - 1 WHERE product_id = $1',
        [123]
      );

      // All queries commit together or rollback together
    });
  }
}
```

**Characteristics:**
- ✅ Uses the manager's specific connection
- ✅ Participates in transactions automatically
- ✅ Guarantees same connection for related operations
- ✅ Safe for operations requiring consistency

---

### **Side-by-Side Comparison:**

```typescript
class ComparisonService {
  constructor(
    private dataSource: DataSource,
    private entityManager: EntityManager
  ) {}

  // DataSource.query() - May use different connections
  async withDataSource(): Promise<void> {
    // These might execute on different connections
    await this.dataSource.query('INSERT INTO logs (message) VALUES ($1)', ['Log 1']);
    await this.dataSource.query('INSERT INTO logs (message) VALUES ($1)', ['Log 2']);
    // No guarantee they're in the same transaction
  }

  // EntityManager.query() - Uses same connection
  async withEntityManager(): Promise<void> {
    await this.entityManager.transaction(async (manager) => {
      // These definitely execute on the same connection
      await manager.query('INSERT INTO logs (message) VALUES ($1)', ['Log 1']);
      await manager.query('INSERT INTO logs (message) VALUES ($1)', ['Log 2']);
      // Both are in the same transaction
    });
  }
}
```

---

### **Comparison Table:**

| Feature | DataSource.query() | manager.query() |
|---------|-------------------|-----------------|
| **Connection** | Any from pool | Specific to manager |
| **Transaction** | Not automatically | Yes, if manager is transactional |
| **Use Case** | Standalone queries | Transactional operations |
| **Scope** | Global | Context-specific |
| **Consistency** | Not guaranteed | Guaranteed within transaction |

---

### **Production Example: Money Transfer:**

```typescript
class BankingService {
  constructor(
    private dataSource: DataSource,
    private entityManager: EntityManager
  ) {}

  // ❌ Wrong: Using dataSource.query() for transaction
  async transferMoneyWrong(
    fromAccountId: number,
    toAccountId: number,
    amount: number
  ): Promise<void> {
    try {
      // These may execute on different connections!
      await this.dataSource.query(
        'UPDATE accounts SET balance = balance - $1 WHERE id = $2',
        [amount, fromAccountId]
      );

      // If this fails, the debit above is already committed!
      await this.dataSource.query(
        'UPDATE accounts SET balance = balance + $1 WHERE id = $2',
        [amount, toAccountId]
      );
    } catch (error) {
      // Can't rollback the first query!
      console.error('Transfer failed:', error);
      throw error;
    }
  }

  // ✅ Correct: Using manager.query() within transaction
  async transferMoneyCorrect(
    fromAccountId: number,
    toAccountId: number,
    amount: number
  ): Promise<void> {
    await this.entityManager.transaction(async (manager) => {
      // Both queries on same connection, same transaction
      await manager.query(
        'UPDATE accounts SET balance = balance - $1 WHERE id = $2',
        [amount, fromAccountId]
      );

      await manager.query(
        'UPDATE accounts SET balance = balance + $1 WHERE id = $2',
        [amount, toAccountId]
      );

      // Both commit together or rollback together
    });
  }
}
```

---

### **When to Use Each:**

**Use DataSource.query() when:**
```typescript
// ✅ Simple read operations
async getStatistics(): Promise<any> {
  return await this.dataSource.query(`
    SELECT COUNT(*) as total_users FROM users
  `);
}

// ✅ Independent operations
async logActivity(message: string): Promise<void> {
  await this.dataSource.query(
    'INSERT INTO activity_logs (message) VALUES ($1)',
    [message]
  );
}

// ✅ Migrations
async runMigration(): Promise<void> {
  await this.dataSource.query(`
    ALTER TABLE users ADD COLUMN last_login TIMESTAMP
  `);
}
```

**Use manager.query() when:**
```typescript
// ✅ Multiple related operations
async processOrderTransaction(orderId: number): Promise<void> {
  await this.entityManager.transaction(async (manager) => {
    await manager.query('UPDATE orders SET status = $1 WHERE id = $2', 
      ['PAID', orderId]);
    await manager.query('UPDATE inventory SET stock = stock - 1 WHERE product_id = $1', 
      [123]);
    await manager.query('INSERT INTO order_history (order_id, status) VALUES ($1, $2)', 
      [orderId, 'PAID']);
  });
}

// ✅ Operations requiring consistency
async ensureConsistency(): Promise<void> {
  await this.entityManager.transaction(async (manager) => {
    const result = await manager.query('SELECT balance FROM accounts WHERE id = $1', [1]);
    await manager.query('UPDATE accounts SET balance = $1 WHERE id = $2', 
      [result[0].balance + 100, 1]);
  });
}
```

---

### **Repository.query() - Third Option:**

```typescript
import { Repository } from 'typeorm';
import { User } from './entities/User';

class UserService {
  constructor(private userRepo: Repository<User>) {}

  // Repository.query() - Similar to DataSource.query()
  async customUserQuery(): Promise<any[]> {
    return await this.userRepo.query(
      'SELECT * FROM users WHERE age > $1',
      [18]
    );
  }

  // Uses repository's manager internally
  // Not automatically transactional
}
```

---

### **Practical Testing Example:**

```typescript
describe('Query Methods', () => {
  let dataSource: DataSource;
  let entityManager: EntityManager;

  beforeEach(async () => {
    dataSource = await createConnection();
    entityManager = dataSource.manager;
  });

  it('dataSource.query() - not transactional by default', async () => {
    const initialCount = await dataSource.query('SELECT COUNT(*) FROM users');

    try {
      await dataSource.query('INSERT INTO users (name) VALUES ($1)', ['Test']);
      throw new Error('Simulated error');
    } catch (error) {
      // Insert was already committed!
      const finalCount = await dataSource.query('SELECT COUNT(*) FROM users');
      expect(finalCount[0].count).toBe(initialCount[0].count + 1);
    }
  });

  it('manager.query() in transaction - automatic rollback', async () => {
    const initialCount = await dataSource.query('SELECT COUNT(*) FROM users');

    try {
      await entityManager.transaction(async (manager) => {
        await manager.query('INSERT INTO users (name) VALUES ($1)', ['Test']);
        throw new Error('Simulated error');
      });
    } catch (error) {
      // Insert was rolled back!
      const finalCount = await dataSource.query('SELECT COUNT(*) FROM users');
      expect(finalCount[0].count).toBe(initialCount[0].count);
    }
  });
});
```

---

### **Common Pitfalls:**

```typescript
// ❌ Pitfall 1: Assuming dataSource.query() is transactional
async badExample1() {
  await this.dataSource.query('UPDATE users SET balance = 100');
  throw new Error('Oops'); // Update was already committed!
}

// ❌ Pitfall 2: Mixing dataSource and manager queries
async badExample2() {
  await this.entityManager.transaction(async (manager) => {
    await manager.query('INSERT INTO orders ...');
    // This runs outside the transaction!
    await this.dataSource.query('UPDATE inventory ...');
  });
}

// ✅ Correct: Use manager consistently
async goodExample() {
  await this.entityManager.transaction(async (manager) => {
    await manager.query('INSERT INTO orders ...');
    await manager.query('UPDATE inventory ...');
    // Both in the same transaction
  });
}
```

---

### **Best Practices:**

1. ✅ **Use manager.query() for transactions** - Ensures consistency
2. ✅ **Use dataSource.query() for standalone queries** - Simpler for reads
3. ✅ **Never mix both in transactional code** - Use one consistently
4. ✅ **Understand connection pooling** - dataSource uses pool
5. ✅ **Test transaction behavior** - Verify rollback works correctly

---

### **Interview Tips:**
- Explain the key difference: connection scope and transactions
- Emphasize transaction safety with manager.query()
- Show a money transfer example demonstrating the importance
- Mention that dataSource.query() uses connection pool
- Discuss when to use each method based on requirements

</details>

<details>
<summary>115. How do you map raw query results to entities?</summary>

### Answer:

**Mapping raw query results to entities** allows you to leverage TypeORM's entity features (methods, relations, validation) while using custom SQL for complex queries. TypeORM provides several approaches for this mapping.

---

### **Method 1: Using Repository.query() with Manual Mapping:**

```typescript
import { Repository } from 'typeorm';
import { User } from './entities/User';

@Entity()
class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;

  @Column()
  age: number;

  // Entity methods available after mapping
  getDisplayName(): string {
    return `${this.name} (${this.email})`;
  }
}

class UserService {
  constructor(private userRepo: Repository<User>) {}

  // Manual mapping to entity instances
  async getUsersWithCustomQuery(): Promise<User[]> {
    const rawResults = await this.userRepo.query(
      'SELECT id, name, email, age FROM users WHERE age > $1',
      [18]
    );

    // Map to entity instances
    return rawResults.map(row => {
      const user = new User();
      user.id = row.id;
      user.name = row.name;
      user.email = row.email;
      user.age = row.age;
      return user;
    });
  }

  // Alternative: Use Object.assign
  async getUsersWithObjectAssign(): Promise<User[]> {
    const rawResults = await this.userRepo.query(
      'SELECT * FROM users WHERE age > $1',
      [18]
    );

    return rawResults.map(row => Object.assign(new User(), row));
  }
}
```

---

### **Method 2: Using Query Builder getRawMany() with Mapping:**

```typescript
class ProductService {
  constructor(private productRepo: Repository<Product>) {}

  // Get raw results then map
  async getProductsWithCalculations(): Promise<Product[]> {
    const rawResults = await this.productRepo
      .createQueryBuilder('product')
      .select([
        'product.id',
        'product.name',
        'product.price',
        'product.stock'
      ])
      .addSelect('product.price * 0.9', 'discountedPrice')
      .addSelect('product.stock > 0', 'inStock')
      .where('product.isActive = true')
      .getRawMany();

    // Map to entities
    return rawResults.map(row => {
      const product = new Product();
      product.id = row.product_id;
      product.name = row.product_name;
      product.price = row.product_price;
      product.stock = row.product_stock;
      // Additional calculated fields
      (product as any).discountedPrice = row.discountedPrice;
      (product as any).inStock = row.inStock;
      return product;
    });
  }
}
```

---

### **Method 3: Using getMany() with Raw SQL (Best Approach):**

TypeORM can automatically map when column names match entity properties.

```typescript
class UserService {
  constructor(private dataSource: DataSource) {}

  // Automatic mapping when using entity metadata
  async getUsersAutoMapped(): Promise<User[]> {
    const rawResults = await this.dataSource.query(
      `SELECT id, name, email, age FROM users WHERE age > $1`,
      [18]
    );

    // Create entity instances with repository's create method
    const users = this.dataSource
      .getRepository(User)
      .create(rawResults);

    return users;
  }
}
```

---

### **Method 4: Using Query Builder with Aliases:**

```typescript
class OrderService {
  constructor(private orderRepo: Repository<Order>) {}

  // Complex query with JOINs - automatic entity mapping
  async getOrdersWithDetails(): Promise<Order[]> {
    return await this.orderRepo
      .createQueryBuilder('order')
      .leftJoinAndSelect('order.user', 'user')
      .leftJoinAndSelect('order.items', 'items')
      .leftJoinAndSelect('items.product', 'product')
      .where('order.status = :status', { status: 'COMPLETED' })
      .getMany(); // Automatically maps to Order entities with relations
  }

  // Get raw results and map selectively
  async getOrdersWithCustomMapping(): Promise<Order[]> {
    const results = await this.orderRepo
      .createQueryBuilder('order')
      .leftJoin('order.user', 'user')
      .select([
        'order.id',
        'order.total',
        'order.status',
        'user.name',
        'user.email'
      ])
      .getRawAndEntities();

    // results.entities contains Order entities
    // results.raw contains the raw data
    return results.entities;
  }
}
```

---

### **Method 5: Custom Repository with Mapping Helper:**

```typescript
class CustomUserRepository extends Repository<User> {
  // Reusable mapping helper
  private mapRawToEntity(raw: any): User {
    const user = new User();
    user.id = raw.id;
    user.name = raw.name;
    user.email = raw.email;
    user.age = raw.age;
    user.createdAt = raw.created_at;
    return user;
  }

  async findUsersWithComplexQuery(filters: any): Promise<User[]> {
    const rawResults = await this.query(
      `
      SELECT 
        u.id,
        u.name,
        u.email,
        u.age,
        u.created_at,
        COUNT(o.id) as order_count
      FROM users u
      LEFT JOIN orders o ON u.id = o.user_id
      WHERE u.age > $1
      GROUP BY u.id, u.name, u.email, u.age, u.created_at
      HAVING COUNT(o.id) > $2
      `,
      [filters.minAge, filters.minOrders]
    );

    return rawResults.map(raw => {
      const user = this.mapRawToEntity(raw);
      // Add computed fields
      (user as any).orderCount = parseInt(raw.order_count);
      return user;
    });
  }
}
```

---

### **Production Use Case: Complex Analytics with Entity Mapping:**

```typescript
interface UserWithStats {
  user: User;
  stats: {
    totalOrders: number;
    totalSpent: number;
    avgOrderValue: number;
    lastOrderDate: Date;
  };
}

class AnalyticsService {
  constructor(
    private dataSource: DataSource,
    private userRepo: Repository<User>
  ) {}

  async getUsersWithStatistics(): Promise<UserWithStats[]> {
    const rawResults = await this.dataSource.query(`
      SELECT 
        u.id,
        u.name,
        u.email,
        u.age,
        u.created_at,
        COUNT(o.id) as total_orders,
        COALESCE(SUM(o.total), 0) as total_spent,
        COALESCE(AVG(o.total), 0) as avg_order_value,
        MAX(o.created_at) as last_order_date
      FROM users u
      LEFT JOIN orders o ON u.id = o.user_id
      GROUP BY u.id, u.name, u.email, u.age, u.created_at
      ORDER BY total_spent DESC
      LIMIT 100
    `);

    return rawResults.map(row => {
      // Create user entity
      const user = this.userRepo.create({
        id: row.id,
        name: row.name,
        email: row.email,
        age: row.age,
        createdAt: row.created_at
      });

      return {
        user,
        stats: {
          totalOrders: parseInt(row.total_orders),
          totalSpent: parseFloat(row.total_spent),
          avgOrderValue: parseFloat(row.avg_order_value),
          lastOrderDate: row.last_order_date
        }
      };
    });
  }
}
```

---

### **Handling Relations in Raw Results:**

```typescript
class OrderMappingService {
  constructor(
    private dataSource: DataSource,
    private orderRepo: Repository<Order>,
    private userRepo: Repository<User>
  ) {}

  async getOrdersWithUserMapping(): Promise<Order[]> {
    const rawResults = await this.dataSource.query(`
      SELECT 
        o.id as order_id,
        o.total as order_total,
        o.status as order_status,
        o.created_at as order_created_at,
        u.id as user_id,
        u.name as user_name,
        u.email as user_email
      FROM orders o
      INNER JOIN users u ON o.user_id = u.id
      WHERE o.status = $1
    `, ['COMPLETED']);

    return rawResults.map(row => {
      // Create user entity
      const user = this.userRepo.create({
        id: row.user_id,
        name: row.user_name,
        email: row.user_email
      });

      // Create order entity with relation
      const order = this.orderRepo.create({
        id: row.order_id,
        total: row.order_total,
        status: row.order_status,
        createdAt: row.order_created_at,
        user: user
      });

      return order;
    });
  }
}
```

---

### **Using Class-Transformer for Mapping:**

```typescript
import { plainToInstance } from 'class-transformer';

class TransformService {
  constructor(private dataSource: DataSource) {}

  // Use class-transformer for automatic mapping
  async getUsersWithTransformer(): Promise<User[]> {
    const rawResults = await this.dataSource.query(
      'SELECT * FROM users WHERE age > $1',
      [18]
    );

    // Automatically maps properties and applies decorators
    return plainToInstance(User, rawResults);
  }

  // With validation
  async getUsersWithValidation(): Promise<User[]> {
    const rawResults = await this.dataSource.query(
      'SELECT * FROM users'
    );

    const users = plainToInstance(User, rawResults);

    // Validate each entity
    for (const user of users) {
      const errors = await validate(user);
      if (errors.length > 0) {
        console.error('Validation errors:', errors);
      }
    }

    return users;
  }
}
```

---

### **Mapping with Type Safety:**

```typescript
class TypeSafeMappingService {
  constructor(private userRepo: Repository<User>) {}

  // Type-safe mapping function
  private mapToUser(raw: {
    id: number;
    name: string;
    email: string;
    age: number;
    created_at?: Date;
  }): User {
    return this.userRepo.create({
      id: raw.id,
      name: raw.name,
      email: raw.email,
      age: raw.age,
      createdAt: raw.created_at
    });
  }

  async getUsers(): Promise<User[]> {
    const rawResults = await this.userRepo.query(
      'SELECT id, name, email, age, created_at FROM users'
    );

    return rawResults.map(row => this.mapToUser(row));
  }
}
```

---

### **Handling NULL Values and Defaults:**

```typescript
class SafeMappingService {
  constructor(private productRepo: Repository<Product>) {}

  async getProductsSafely(): Promise<Product[]> {
    const rawResults = await this.productRepo.query(
      'SELECT * FROM products'
    );

    return rawResults.map(row => {
      const product = new Product();
      product.id = row.id;
      product.name = row.name ?? 'Unknown'; // Handle NULL
      product.price = parseFloat(row.price ?? '0'); // Default value
      product.stock = parseInt(row.stock ?? '0');
      product.description = row.description ?? null;
      product.isActive = row.is_active ?? true;
      return product;
    });
  }
}
```

---

### **Batch Mapping for Performance:**

```typescript
class BatchMappingService {
  constructor(
    private userRepo: Repository<User>,
    private dataSource: DataSource
  ) {}

  async getUsersInBatches(batchSize: number = 1000): Promise<User[]> {
    const totalCount = await this.dataSource.query(
      'SELECT COUNT(*) as count FROM users'
    );
    const total = parseInt(totalCount[0].count);

    const allUsers: User[] = [];

    for (let offset = 0; offset < total; offset += batchSize) {
      const rawResults = await this.dataSource.query(
        'SELECT * FROM users LIMIT $1 OFFSET $2',
        [batchSize, offset]
      );

      // Map batch
      const users = this.userRepo.create(rawResults);
      allUsers.push(...users);
    }

    return allUsers;
  }
}
```

---

### **Using getRawAndEntities() for Combined Results:**

```typescript
class CombinedResultsService {
  constructor(private orderRepo: Repository<Order>) {}

  async getOrdersWithCalculations(): Promise<{
    entities: Order[];
    calculations: any[];
  }> {
    const result = await this.orderRepo
      .createQueryBuilder('order')
      .leftJoinAndSelect('order.user', 'user')
      .select([
        'order',
        'user',
      ])
      .addSelect('SUM(order.total)', 'totalRevenue')
      .addSelect('AVG(order.total)', 'avgOrderValue')
      .groupBy('order.id')
      .addGroupBy('user.id')
      .getRawAndEntities();

    return {
      entities: result.entities, // Mapped Order entities
      calculations: result.raw     // Raw data with calculations
    };
  }
}
```

---

### **Best Practices:**

1. ✅ **Use repository.create()** - Proper entity instantiation
2. ✅ **Match column names** - Use aliases if needed
3. ✅ **Handle NULL values** - Provide defaults
4. ✅ **Validate mapped entities** - Ensure data integrity
5. ✅ **Use type-safe mapping** - Define interfaces/types
6. ✅ **Consider class-transformer** - For complex mapping
7. ✅ **Test entity methods** - Ensure they work after mapping
8. ✅ **Document mapping logic** - Make transformation clear

---

### **Common Pitfalls:**

```typescript
// ❌ Wrong: Direct assignment without entity instance
const badUser = rawResult; // Not a User entity!
badUser.getDisplayName(); // ERROR: method doesn't exist

// ✅ Correct: Create proper entity instance
const goodUser = Object.assign(new User(), rawResult);
goodUser.getDisplayName(); // Works!

// ❌ Wrong: Column name mismatch
const raw = { id: 1, user_name: 'John' }; // Snake case
const user = Object.assign(new User(), raw);
console.log(user.name); // undefined

// ✅ Correct: Map column names
const user = new User();
user.id = raw.id;
user.name = raw.user_name; // Explicit mapping
```

---

### **Performance Comparison:**

```typescript
// Method 1: Query Builder (automatic mapping) - Slower but convenient
await this.orderRepo.find({ relations: ['user', 'items'] });

// Method 2: Raw SQL + Manual mapping - Faster for complex queries
const raw = await this.dataSource.query('SELECT ...');
const orders = raw.map(r => this.mapToOrder(r));

// Method 3: Hybrid approach - Best of both worlds
const result = await this.orderRepo
  .createQueryBuilder('order')
  .leftJoinAndSelect('order.user', 'user')
  .getRawAndEntities();
```

---

### **Interview Tips:**
- Explain multiple mapping approaches: manual, repository.create(), class-transformer
- Discuss trade-offs: automatic convenience vs manual control
- Show awareness of NULL handling and type safety
- Mention getRawAndEntities() for combined results
- Emphasize the importance of proper entity instantiation for methods to work

</details>