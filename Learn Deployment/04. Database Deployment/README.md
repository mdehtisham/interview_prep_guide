# Database Deployment

## Table of Contents
- [What is Database Deployment?](#what-is-database-deployment)
- [Database Hosting Options](#database-hosting-options)
- [Database Platforms](#database-platforms)
- [Database Configuration](#database-configuration)
- [Database Migrations](#database-migrations)
- [Backup and Recovery](#backup-and-recovery)
- [Security Best Practices](#security-best-practices)
- [Performance Optimization](#performance-optimization)
- [Common Issues and Solutions](#common-issues-and-solutions)
- [Interview Questions](#interview-questions)

---

## What is Database Deployment?

Database deployment involves setting up, configuring, and maintaining a database system in a production environment that your backend application can access securely and reliably.

### Database Architecture After Deployment:
```
Backend Application(s) → Database Connection Pool → Database Server
                                                          ↓
                                                    Data Storage
                                                    Backups
                                                    Read Replicas
```

### Key Considerations:
- **Persistence**: Data must survive application restarts
- **Security**: Protect sensitive data
- **Performance**: Fast queries, proper indexing
- **Scalability**: Handle growing data and traffic
- **Availability**: Minimize downtime
- **Backup**: Regular backups for disaster recovery

---

## Database Hosting Options

### 1. **Managed Database Services (DBaaS)**

**Pros**:
- Automated backups
- Easy scaling
- Automatic updates
- High availability
- Monitoring included
- No server management

**Cons**:
- Less control
- Can be expensive at scale
- Vendor lock-in

**Use Cases**: Most production applications

---

### 2. **Self-Hosted Database (VPS)**

**Pros**:
- Full control
- Cost-effective for large scale
- Custom configurations
- No vendor lock-in

**Cons**:
- Manual backups
- Manual scaling
- Manual updates
- Complex setup
- Requires DB expertise

**Use Cases**: Specific requirements, cost optimization, compliance

---

### 3. **Serverless Databases**

**Pros**:
- Auto-scaling
- Pay per use
- No capacity planning
- Minimal management

**Cons**:
- Cold start latency
- Limited for complex queries
- Cost can be unpredictable

**Use Cases**: Variable traffic, small to medium apps

---

## Database Platforms

### 1. **PostgreSQL**

#### **Managed Services**:

##### **AWS RDS PostgreSQL**
```bash
# Create database via AWS CLI
aws rds create-db-instance \
  --db-instance-identifier myapp-db \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --engine-version 14.7 \
  --master-username admin \
  --master-user-password MyPassword123! \
  --allocated-storage 20 \
  --backup-retention-period 7 \
  --multi-az \
  --publicly-accessible false
```

**Connection String**:
```
postgresql://admin:password@myapp-db.abc123.us-east-1.rds.amazonaws.com:5432/postgres
```

**Features**:
- Automated backups (up to 35 days)
- Multi-AZ for high availability
- Read replicas
- Automatic minor version upgrades
- CloudWatch monitoring
- Encryption at rest

**Pricing**: Pay for instance hours, storage, backups

---

##### **Heroku Postgres**
```bash
# Add to Heroku app
heroku addons:create heroku-postgresql:hobby-dev

# Get connection string
heroku config:get DATABASE_URL

# Access database
heroku pg:psql

# Backup
heroku pg:backups:capture
heroku pg:backups:download
```

**Tiers**:
- Hobby Dev: Free, 10K rows, no backups
- Hobby Basic: $9/mo, 10M rows, backups
- Standard: $50+/mo, 64GB RAM, HA

---

##### **Supabase**
Free tier: 500MB database, 2GB file storage

```javascript
// Connection from Node.js
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.SUPABASE_URL,
  process.env.SUPABASE_ANON_KEY
);

// Query
const { data, error } = await supabase
  .from('users')
  .select('*')
  .eq('active', true);
```

**Features**:
- Real-time subscriptions
- Built-in authentication
- Storage for files
- Auto-generated REST API
- Row-level security

---

##### **Neon (Serverless PostgreSQL)**
```bash
# Connection string
postgresql://user:pass@ep-cool-name-123456.us-east-2.aws.neon.tech/neondb?sslmode=require
```

**Features**:
- Serverless (auto-pause when idle)
- Instant branching (copy database for development)
- Free tier: 3GB storage, 1 project
- Pay per use

---

##### **PlanetScale (Serverless MySQL)**
```bash
# Connection
mysql://user:pass@aws.connect.psdb.cloud/mydb?ssl={"rejectUnauthorized":true}
```

**Features**:
- Git-like branching for databases
- Non-blocking schema changes
- Automatic backups
- Global replication
- Free tier: 5GB storage, 1 billion row reads/month

---

##### **ElephantSQL (Managed PostgreSQL)**
```bash
# Connection string provided after signup
postgresql://user:pass@raja.db.elephantsql.com/dbname
```

**Tiers**:
- Tiny Turtle: Free, 20MB
- Pretty Panda: $5/mo, 2GB
- Higher tiers available

---

#### **Self-Hosted PostgreSQL on VPS**

**Installation**:
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install postgresql postgresql-contrib

# Start service
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Access PostgreSQL
sudo -u postgres psql
```

**Create Database and User**:
```sql
-- Create database
CREATE DATABASE myapp;

-- Create user
CREATE USER myappuser WITH ENCRYPTED PASSWORD 'secure_password';

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE myapp TO myappuser;

-- Connect to database
\c myapp

-- Grant schema privileges
GRANT ALL ON SCHEMA public TO myappuser;
```

**Configure Remote Access** (`/etc/postgresql/14/main/postgresql.conf`):
```conf
listen_addresses = '*'
```

**Configure Authentication** (`/etc/postgresql/14/main/pg_hba.conf`):
```conf
# Allow remote connections with password
host    all             all             0.0.0.0/0               md5
```

**Restart PostgreSQL**:
```bash
sudo systemctl restart postgresql
```

**Firewall**:
```bash
# Allow PostgreSQL port
sudo ufw allow 5432/tcp
```

---

### 2. **MySQL/MariaDB**

#### **Managed Services**:

##### **AWS RDS MySQL**
Similar to RDS PostgreSQL, just change engine:
```bash
aws rds create-db-instance \
  --engine mysql \
  --engine-version 8.0.32 \
  ...
```

##### **PlanetScale** (Recommended for MySQL)
See PostgreSQL section above.

#### **Self-Hosted MySQL**:
```bash
# Install
sudo apt install mysql-server

# Secure installation
sudo mysql_secure_installation

# Access
sudo mysql

# Create database and user
CREATE DATABASE myapp;
CREATE USER 'myappuser'@'%' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON myapp.* TO 'myappuser'@'%';
FLUSH PRIVILEGES;
```

**Configure Remote Access** (`/etc/mysql/mysql.conf.d/mysqld.cnf`):
```conf
bind-address = 0.0.0.0
```

---

### 3. **MongoDB**

#### **MongoDB Atlas (Managed)**

**Setup**:
1. Create account at mongodb.com/atlas
2. Create cluster (Free tier: 512MB)
3. Create database user
4. Whitelist IP addresses (0.0.0.0/0 for any)
5. Get connection string

**Connection String**:
```
mongodb+srv://username:password@cluster0.abc123.mongodb.net/myapp?retryWrites=true&w=majority
```

**Connect from Node.js**:
```javascript
const mongoose = require('mongoose');

mongoose.connect(process.env.MONGODB_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
})
.then(() => console.log('MongoDB connected'))
.catch(err => console.error('MongoDB connection error:', err));
```

**Features**:
- Free tier available
- Automatic backups
- Built-in monitoring
- Global clusters
- Atlas Search
- Triggers and functions

---

#### **Self-Hosted MongoDB**:
```bash
# Install
sudo apt install mongodb

# Or install latest version
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
sudo apt update
sudo apt install -y mongodb-org

# Start
sudo systemctl start mongod
sudo systemctl enable mongod

# Access
mongosh

# Create user
use admin
db.createUser({
  user: "admin",
  pwd: "secure_password",
  roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
})

use myapp
db.createUser({
  user: "myappuser",
  pwd: "secure_password",
  roles: [ { role: "readWrite", db: "myapp" } ]
})
```

**Enable Authentication** (`/etc/mongod.conf`):
```yaml
security:
  authorization: enabled

net:
  bindIp: 0.0.0.0
```

---

### 4. **Redis (Cache/Session Store)**

#### **Managed Redis**:

##### **Redis Cloud (Redis Labs)**
- Free tier: 30MB
- Automatic backups
- High availability

##### **AWS ElastiCache Redis**
```bash
aws elasticache create-cache-cluster \
  --cache-cluster-id myapp-redis \
  --engine redis \
  --cache-node-type cache.t3.micro \
  --num-cache-nodes 1
```

##### **Heroku Redis**
```bash
heroku addons:create heroku-redis:hobby-dev
```

##### **Upstash (Serverless Redis)**
```bash
# REST API connection
curl https://us1-tops-123456.upstash.io/set/key/value \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Features**:
- Serverless pricing
- Global replication
- REST API
- Free tier: 10K commands/day

---

#### **Self-Hosted Redis**:
```bash
# Install
sudo apt install redis-server

# Configure (/etc/redis/redis.conf)
bind 0.0.0.0
requirepass your_redis_password
maxmemory 256mb
maxmemory-policy allkeys-lru

# Restart
sudo systemctl restart redis
```

**Connect from Node.js**:
```javascript
const redis = require('redis');
const client = redis.createClient({
  url: 'redis://:password@host:6379'
});

await client.connect();
await client.set('key', 'value');
const value = await client.get('key');
```

---

### 5. **SQLite (Development/Small Apps)**

**Not recommended for production** with multiple servers, but acceptable for:
- Single server apps
- Small traffic
- Embedded applications

```javascript
// Node.js with better-sqlite3
const Database = require('better-sqlite3');
const db = new Database('/var/data/myapp.db');

// Create table
db.exec(`
  CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL
  )
`);

// Query
const users = db.prepare('SELECT * FROM users').all();
```

**Deployment**:
- Store .db file in persistent volume
- Regular backups to S3
- Use for read-heavy workloads

---

## Database Configuration

### 1. **Connection Pooling**

**PostgreSQL/MySQL with Node.js**:
```javascript
// pg (PostgreSQL)
const { Pool } = require('pg');

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: process.env.NODE_ENV === 'production' ? {
    rejectUnauthorized: false
  } : false,
  max: 20, // maximum pool size
  min: 5,  // minimum pool size
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

// Usage
app.get('/users', async (req, res) => {
  const result = await pool.query('SELECT * FROM users');
  res.json(result.rows);
});
```

**TypeORM**:
```typescript
// data-source.ts
export const AppDataSource = new DataSource({
  type: 'postgres',
  url: process.env.DATABASE_URL,
  ssl: { rejectUnauthorized: false },
  entities: ['dist/**/*.entity.js'],
  synchronize: false,
  logging: false,
  
  // Connection pool settings
  extra: {
    max: 20,
    min: 5,
    idleTimeoutMillis: 30000,
  }
});
```

**Sequelize**:
```javascript
const sequelize = new Sequelize(process.env.DATABASE_URL, {
  dialect: 'postgres',
  ssl: true,
  dialectOptions: {
    ssl: {
      require: true,
      rejectUnauthorized: false
    }
  },
  pool: {
    max: 20,
    min: 5,
    acquire: 30000,
    idle: 10000
  }
});
```

### 2. **SSL/TLS Configuration**

Most production databases require SSL:

```javascript
// PostgreSQL
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: {
    rejectUnauthorized: false, // Allow self-signed certificates
    // Or provide CA certificate
    ca: fs.readFileSync('/path/to/ca-certificate.crt').toString(),
  }
});
```

```javascript
// MongoDB
mongoose.connect(process.env.MONGODB_URI, {
  ssl: true,
  sslValidate: true,
  sslCA: fs.readFileSync('/path/to/ca.pem'),
});
```

### 3. **Read/Write Splitting**

For read replicas:

```javascript
// TypeORM with replication
export const AppDataSource = new DataSource({
  type: 'postgres',
  replication: {
    master: {
      host: 'master.db.com',
      port: 5432,
      username: 'user',
      password: 'pass',
      database: 'myapp'
    },
    slaves: [
      {
        host: 'replica1.db.com',
        port: 5432,
        username: 'user',
        password: 'pass',
        database: 'myapp'
      },
      {
        host: 'replica2.db.com',
        port: 5432,
        username: 'user',
        password: 'pass',
        database: 'myapp'
      }
    ]
  }
});

// Writes go to master, reads from slaves
```

---

## Database Migrations

### 1. **TypeORM Migrations**

**Create Migration**:
```bash
npm run typeorm migration:create src/migrations/CreateUserTable
```

**Generate from Entity Changes**:
```bash
npm run typeorm migration:generate src/migrations/UpdateUserTable -d src/data-source.ts
```

**Migration File**:
```typescript
// src/migrations/1234567890-CreateUserTable.ts
import { MigrationInterface, QueryRunner } from "typeorm";

export class CreateUserTable1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      CREATE TABLE "users" (
        "id" SERIAL PRIMARY KEY,
        "email" VARCHAR(255) NOT NULL UNIQUE,
        "name" VARCHAR(255) NOT NULL,
        "created_at" TIMESTAMP DEFAULT NOW()
      )
    `);
    
    await queryRunner.query(`
      CREATE INDEX "IDX_USER_EMAIL" ON "users" ("email")
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP TABLE "users"`);
  }
}
```

**Run Migrations**:
```bash
# Development
npm run typeorm migration:run -- -d src/data-source.ts

# Production (Heroku)
heroku run npm run migration:run

# Production (VPS)
ssh user@server "cd /var/www/myapp && npm run migration:run"

# Docker
docker exec -it myapp npm run migration:run
```

**Revert Migration**:
```bash
npm run typeorm migration:revert -- -d src/data-source.ts
```

**package.json scripts**:
```json
{
  "scripts": {
    "typeorm": "typeorm-ts-node-commonjs",
    "migration:generate": "npm run typeorm -- migration:generate",
    "migration:create": "npm run typeorm -- migration:create",
    "migration:run": "npm run typeorm -- migration:run -d dist/data-source.js",
    "migration:revert": "npm run typeorm -- migration:revert -d dist/data-source.js"
  }
}
```

---

### 2. **Sequelize Migrations**

**Initialize**:
```bash
npx sequelize-cli init
```

**Create Migration**:
```bash
npx sequelize-cli migration:generate --name create-users-table
```

**Migration File**:
```javascript
// migrations/20231201-create-users-table.js
'use strict';

module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.createTable('users', {
      id: {
        type: Sequelize.INTEGER,
        primaryKey: true,
        autoIncrement: true
      },
      email: {
        type: Sequelize.STRING,
        allowNull: false,
        unique: true
      },
      name: {
        type: Sequelize.STRING,
        allowNull: false
      },
      createdAt: {
        type: Sequelize.DATE,
        defaultValue: Sequelize.NOW
      },
      updatedAt: {
        type: Sequelize.DATE,
        defaultValue: Sequelize.NOW
      }
    });
    
    await queryInterface.addIndex('users', ['email']);
  },

  down: async (queryInterface, Sequelize) => {
    await queryInterface.dropTable('users');
  }
};
```

**Run Migrations**:
```bash
npx sequelize-cli db:migrate

# Revert
npx sequelize-cli db:migrate:undo
```

---

### 3. **Prisma Migrations**

**Schema** (`prisma/schema.prisma`):
```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String
  createdAt DateTime @default(now())
  
  @@index([email])
}
```

**Create Migration**:
```bash
npx prisma migrate dev --name create_users_table
```

**Deploy to Production**:
```bash
npx prisma migrate deploy
```

---

### 4. **MongoDB Migrations**

**Using migrate-mongo**:

```bash
npm install migrate-mongo

# Initialize
npx migrate-mongo init

# Create migration
npx migrate-mongo create add-email-index
```

**Migration File**:
```javascript
// migrations/20231201-add-email-index.js
module.exports = {
  async up(db, client) {
    await db.collection('users').createIndex({ email: 1 }, { unique: true });
  },

  async down(db, client) {
    await db.collection('users').dropIndex('email_1');
  }
};
```

**Run**:
```bash
npx migrate-mongo up
```

---

### 5. **Migration Best Practices**

1. **Always Backward Compatible**: New code should work with old schema
2. **Test in Staging First**: Never run untested migrations in production
3. **Backup Before Migration**: Always have a way to recover
4. **Idempotent Migrations**: Should be safe to run multiple times
5. **Small Incremental Changes**: Easier to debug and rollback
6. **Never Edit Old Migrations**: Create new ones instead
7. **Document Breaking Changes**: Communicate with team

**Deployment Order**:
```
1. Deploy backend code (backward compatible with old schema)
2. Run database migration
3. Verify everything works
4. (Optional) Deploy code that requires new schema
```

---

## Backup and Recovery

### 1. **Automated Backups**

#### **Managed Services** (Automatic):
- **AWS RDS**: Daily automated backups (up to 35 days retention)
- **Heroku Postgres**: Continuous protection on paid plans
- **MongoDB Atlas**: Continuous backups with point-in-time recovery
- **PlanetScale**: Automatic backups

#### **Manual Backups**:

**PostgreSQL**:
```bash
# Backup
pg_dump -h hostname -U username -d database_name > backup.sql

# Or with compression
pg_dump -h hostname -U username -d database_name | gzip > backup.sql.gz

# Backup from Docker
docker exec -t postgres pg_dump -U username database_name > backup.sql

# Restore
psql -h hostname -U username -d database_name < backup.sql
```

**MySQL**:
```bash
# Backup
mysqldump -h hostname -u username -p database_name > backup.sql

# Restore
mysql -h hostname -u username -p database_name < backup.sql
```

**MongoDB**:
```bash
# Backup
mongodump --uri="mongodb://username:password@host:27017/database_name" --out=/backup/

# Restore
mongorestore --uri="mongodb://username:password@host:27017/database_name" /backup/database_name/
```

---

### 2. **Automated Backup Script**

```bash
#!/bin/bash
# backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups"
DATABASE_NAME="myapp"

# PostgreSQL backup
pg_dump -h $DB_HOST -U $DB_USER -d $DATABASE_NAME | gzip > $BACKUP_DIR/backup_$DATE.sql.gz

# Upload to S3
aws s3 cp $BACKUP_DIR/backup_$DATE.sql.gz s3://my-backups/database/

# Delete local backup older than 7 days
find $BACKUP_DIR -type f -mtime +7 -delete

# Delete S3 backups older than 30 days
aws s3 ls s3://my-backups/database/ | while read -r line; do
  createDate=$(echo $line | awk {'print $1" "$2'})
  createDate=$(date -d "$createDate" +%s)
  olderThan=$(date --date "30 days ago" +%s)
  if [[ $createDate -lt $olderThan ]]; then
    fileName=$(echo $line | awk {'print $4'})
    aws s3 rm s3://my-backups/database/$fileName
  fi
done
```

**Schedule with Cron**:
```bash
# Edit crontab
crontab -e

# Add daily backup at 2 AM
0 2 * * * /path/to/backup.sh
```

---

### 3. **Point-in-Time Recovery**

**AWS RDS**:
```bash
# Restore to specific time
aws rds restore-db-instance-to-point-in-time \
  --source-db-instance-identifier myapp-db \
  --target-db-instance-identifier myapp-db-restored \
  --restore-time 2023-12-01T10:00:00Z
```

**MongoDB Atlas**:
- Click "Continuous Backup"
- Select date and time
- Restore to new cluster or download backup

---

### 4. **Disaster Recovery Plan**

1. **Regular Backups**: Automated daily backups
2. **Offsite Storage**: Store backups in different region/cloud
3. **Test Restores**: Regularly test backup restoration
4. **Documentation**: Document recovery procedures
5. **RPO (Recovery Point Objective)**: How much data loss is acceptable? (e.g., 1 hour)
6. **RTO (Recovery Time Objective)**: How quickly must system recover? (e.g., 30 minutes)

---

## Security Best Practices

### 1. **Network Security**

**Firewall Rules**:
```bash
# Only allow backend servers to access database
# AWS Security Group example:
# Inbound: Port 5432 from Backend Security Group
# Outbound: All traffic
```

**Private Network**:
- Deploy database in private subnet (no public IP)
- Use VPN or bastion host for admin access
- Backend accesses database via private network

**IP Whitelisting**:
```bash
# PostgreSQL pg_hba.conf
host    all    all    10.0.1.0/24    md5  # Only backend subnet
```

---

### 2. **Authentication and Authorization**

**Strong Passwords**:
```bash
# Generate secure password
openssl rand -base64 32
```

**Least Privilege**:
```sql
-- Create read-only user
CREATE USER readonly WITH PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE myapp TO readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;

-- Create app user with limited permissions
CREATE USER appuser WITH PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE myapp TO appuser;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO appuser;
```

**IAM Authentication** (AWS RDS):
```javascript
const AWS = require('aws-sdk');
const rds = new AWS.RDS.Signer();

const token = rds.getAuthToken({
  hostname: 'mydb.abc123.us-east-1.rds.amazonaws.com',
  port: 5432,
  region: 'us-east-1',
  username: 'iam_user'
});

const pool = new Pool({
  host: 'mydb.abc123.us-east-1.rds.amazonaws.com',
  port: 5432,
  user: 'iam_user',
  password: token,
  database: 'myapp',
  ssl: 'require'
});
```

---

### 3. **Encryption**

**At Rest**:
- Enable encryption when creating database
- AWS RDS: Encryption option available
- MongoDB Atlas: Enabled by default
- Disk encryption for self-hosted

**In Transit** (SSL/TLS):
```javascript
// Enforce SSL connection
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: {
    rejectUnauthorized: true, // Verify certificate
    ca: fs.readFileSync('/path/to/ca-cert.pem').toString()
  }
});
```

**Column-Level Encryption**:
```javascript
// Encrypt sensitive data before storing
const crypto = require('crypto');

function encrypt(text) {
  const cipher = crypto.createCipher('aes-256-cbc', process.env.ENCRYPTION_KEY);
  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  return encrypted;
}

function decrypt(encrypted) {
  const decipher = crypto.createDecipher('aes-256-cbc', process.env.ENCRYPTION_KEY);
  let decrypted = decipher.update(encrypted, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  return decrypted;
}

// Store encrypted
await db.query('INSERT INTO users (ssn) VALUES ($1)', [encrypt(ssn)]);

// Retrieve and decrypt
const result = await db.query('SELECT ssn FROM users WHERE id = $1', [userId]);
const decryptedSSN = decrypt(result.rows[0].ssn);
```

---

### 4. **SQL Injection Prevention**

**Always Use Parameterized Queries**:

```javascript
// BAD - SQL Injection vulnerable
const userId = req.params.id;
const query = `SELECT * FROM users WHERE id = ${userId}`;
db.query(query); // User could send: 1 OR 1=1

// GOOD - Parameterized query
const userId = req.params.id;
db.query('SELECT * FROM users WHERE id = $1', [userId]);
```

**ORM Protection**:
```javascript
// TypeORM (automatically parameterized)
const user = await userRepository.findOne({ where: { id: userId } });

// Sequelize
const user = await User.findOne({ where: { id: userId } });
```

---

### 5. **Monitoring and Auditing**

**Enable Query Logging**:
```sql
-- PostgreSQL
ALTER SYSTEM SET log_statement = 'all';
SELECT pg_reload_conf();
```

**Audit Failed Login Attempts**:
```sql
-- PostgreSQL log_connections
ALTER SYSTEM SET log_connections = on;
```

**Monitor Slow Queries**:
```sql
-- PostgreSQL
ALTER SYSTEM SET log_min_duration_statement = 1000; -- Log queries > 1s
```

---

## Performance Optimization

### 1. **Indexing**

**Create Indexes**:
```sql
-- Single column index
CREATE INDEX idx_users_email ON users(email);

-- Composite index
CREATE INDEX idx_posts_user_date ON posts(user_id, created_at);

-- Unique index
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- Partial index (PostgreSQL)
CREATE INDEX idx_active_users ON users(email) WHERE active = true;
```

**Find Missing Indexes** (PostgreSQL):
```sql
SELECT
  schemaname,
  tablename,
  seq_scan,
  seq_tup_read,
  idx_scan,
  seq_tup_read / seq_scan AS avg_seq_tup_read
FROM pg_stat_user_tables
WHERE seq_scan > 0
ORDER BY seq_tup_read DESC
LIMIT 25;
```

---

### 2. **Query Optimization**

**Use EXPLAIN ANALYZE**:
```sql
EXPLAIN ANALYZE
SELECT u.name, p.title
FROM users u
JOIN posts p ON p.user_id = u.id
WHERE u.active = true
ORDER BY p.created_at DESC
LIMIT 10;
```

**Avoid N+1 Queries**:
```javascript
// BAD - N+1 queries
const users = await User.findAll();
for (const user of users) {
  user.posts = await Post.findAll({ where: { userId: user.id } });
}

// GOOD - Single query with join
const users = await User.findAll({
  include: [{ model: Post }]
});
```

---

### 3. **Connection Pooling**

```javascript
// Properly sized pool
const pool = new Pool({
  max: 20, // Max connections
  min: 5,  // Min connections
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

// Monitor pool
pool.on('error', (err) => {
  console.error('Unexpected error on idle client', err);
});

pool.on('connect', () => {
  console.log('New client connected to pool');
});
```

---

### 4. **Caching**

```javascript
// Redis cache layer
const redis = require('redis');
const client = redis.createClient({ url: process.env.REDIS_URL });

async function getUser(userId) {
  // Try cache first
  const cached = await client.get(`user:${userId}`);
  if (cached) {
    return JSON.parse(cached);
  }
  
  // Not in cache, query database
  const user = await db.query('SELECT * FROM users WHERE id = $1', [userId]);
  
  // Store in cache (expire in 1 hour)
  await client.setEx(`user:${userId}`, 3600, JSON.stringify(user));
  
  return user;
}
```

---

### 5. **Database Tuning**

**PostgreSQL Configuration**:
```conf
# /etc/postgresql/14/main/postgresql.conf

# Memory settings (for 4GB RAM server)
shared_buffers = 1GB
effective_cache_size = 3GB
work_mem = 16MB
maintenance_work_mem = 256MB

# Connection settings
max_connections = 100

# Query tuning
random_page_cost = 1.1  # For SSD
effective_io_concurrency = 200  # For SSD

# Logging
log_min_duration_statement = 1000  # Log slow queries
```

---

## Common Issues and Solutions

### Issue 1: Connection Refused

**Problem**: `Error: connect ECONNREFUSED`

**Solutions**:
- Check database is running: `systemctl status postgresql`
- Verify connection string is correct
- Check firewall allows connection
- Ensure database bound to correct address
- For managed services: Verify IP whitelist

---

### Issue 2: Too Many Connections

**Problem**: `Error: sorry, too many clients already`

**Solutions**:
- Increase max_connections in database config
- Use connection pooling properly
- Close connections when done
- Reduce pool size if multiple app instances
- Use PgBouncer (connection pooler)

```bash
# Install PgBouncer
sudo apt install pgbouncer

# Configure /etc/pgbouncer/pgbouncer.ini
[databases]
myapp = host=localhost dbname=myapp

[pgbouncer]
listen_addr = *
listen_port = 6432
auth_type = md5
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 25

# Restart
sudo systemctl restart pgbouncer

# Connect via PgBouncer
postgresql://user:pass@host:6432/myapp
```

---

### Issue 3: Slow Queries

**Problem**: Application slow, timeout errors

**Solutions**:
- Add indexes on frequently queried columns
- Optimize queries (use EXPLAIN ANALYZE)
- Avoid SELECT *
- Implement caching
- Use database query logs to find slow queries
- Consider read replicas

---

### Issue 4: Database Out of Storage

**Problem**: `ERROR: could not extend file: No space left on device`

**Solutions**:
- Increase storage (AWS RDS: Modify instance)
- Clean up old data
- Archive old records
- Enable auto-scaling storage
- Monitor disk usage

---

### Issue 5: Authentication Failed

**Problem**: `Error: password authentication failed`

**Solutions**:
- Verify credentials in connection string
- Check user exists: `SELECT * FROM pg_user;`
- Verify user has permissions: `GRANT ALL PRIVILEGES...`
- Check pg_hba.conf allows connection method
- For managed services: Verify security group rules

---

### Issue 6: SSL/TLS Certificate Errors

**Problem**: `Error: unable to verify the first certificate`

**Solutions**:
```javascript
// Option 1: Disable certificate validation (not recommended)
const pool = new Pool({
  ssl: { rejectUnauthorized: false }
});

// Option 2: Provide CA certificate
const pool = new Pool({
  ssl: {
    ca: fs.readFileSync('/path/to/ca-cert.pem').toString()
  }
});

// Option 3: Use sslmode parameter
const connectionString = process.env.DATABASE_URL + '?sslmode=require';
```

---

## Interview Questions

### Basic Level:

**Q1: What is the difference between managed and self-hosted databases?**
**A:** 
- **Managed**: Provider handles backups, updates, scaling (e.g., AWS RDS, MongoDB Atlas). Easy but less control.
- **Self-hosted**: You manage everything on your own server. More control but more complexity.

**Q2: What is a connection pool and why is it important?**
**A:** Connection pool reuses database connections instead of creating new ones for each request. Important because:
- Creating connections is expensive
- Limited number of connections available
- Improves performance
- Prevents "too many connections" errors

**Q3: What is database migration?**
**A:** Migration is a version-controlled way to modify database schema (add tables, columns, indexes). It ensures all environments (dev, staging, prod) have the same schema structure.

**Q4: Why should you never commit database passwords to Git?**
**A:** Security risk:
- Anyone with repository access can see credentials
- Public repositories expose to entire world
- Hard to rotate credentials after exposure
**Solution**: Use environment variables

**Q5: What is SQL injection and how to prevent it?**
**A:** SQL injection is when attacker inserts malicious SQL into queries:
```javascript
// Vulnerable
const query = `SELECT * FROM users WHERE id = ${req.params.id}`;

// Safe - use parameterized queries
db.query('SELECT * FROM users WHERE id = $1', [req.params.id]);
```

### Intermediate Level:

**Q6: How do you connect backend to deployed database?**
**A:** 
1. Get connection string from database provider
2. Store in environment variable: `DATABASE_URL=postgresql://user:pass@host:5432/db`
3. Configure connection with SSL:
```javascript
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: { rejectUnauthorized: false }
});
```
4. Use connection pool
5. Test connection with health check

**Q7: What is the difference between horizontal and vertical scaling for databases?**
**A:**
- **Vertical (Scale Up)**: Add more CPU/RAM to server. Easier but limited.
- **Horizontal (Scale Out)**: Add read replicas. Write to master, read from replicas. Better for read-heavy workloads.

**Q8: Explain read replicas.**
**A:** Read replicas are copies of database that handle read queries:
- Master handles writes
- Replicas handle reads
- Async replication from master to replicas
- Reduces load on master
- Improves read performance
**Use case**: Applications with more reads than writes

**Q9: How do you handle database backups in production?**
**A:** 
- **Managed**: Enable automated backups (daily, retention period)
- **Self-hosted**: Use cron jobs with pg_dump/mongodump
- Store backups offsite (S3, different region)
- Test backup restoration regularly
- Document recovery procedures

**Q10: What is database indexing and when should you use it?**
**A:** Index is a data structure that speeds up queries:
```sql
CREATE INDEX idx_users_email ON users(email);
```
**When to use**:
- Columns frequently in WHERE clauses
- Foreign keys
- Columns used in JOIN
- Columns used for sorting

**Drawbacks**: Slower writes, more storage

### Advanced Level:

**Q11: Explain the complete flow from frontend to database after deployment.**
**A:**
```
1. User action in browser (https://myapp.com)
2. Frontend sends API request (fetch('https://api.myapp.com/users'))
3. DNS resolves api.myapp.com to load balancer IP
4. Load balancer routes to one of backend servers
5. Backend server receives request
6. Express route handler processes request
7. Gets connection from connection pool
8. Executes parameterized SQL query
9. Database processes query using indexes
10. Returns results to backend
11. Backend transforms data, sends JSON response
12. Frontend receives and displays data
```

**Q12: How would you design a database architecture for high availability?**
**A:**
```
Primary Database (Master)
    ↓ (Synchronous replication)
Standby Database (Failover)
    ↓ (Asynchronous replication)
Read Replica 1, 2, 3

Load Balancer:
- Writes → Primary
- Reads → Read Replicas (round-robin)

Failover:
- If primary fails, automatic failover to standby
- Standby becomes new primary
- Old primary becomes standby when recovered

Multi-Region:
- Deploy in multiple AWS regions
- Global database for cross-region replication
```

**Q13: How do you handle database schema changes without downtime?**
**A:** Multi-step backward-compatible migrations:

**Example: Rename column**
```
Step 1: Add new column
ALTER TABLE users ADD COLUMN new_name VARCHAR(255);

Step 2: Deploy code that writes to both columns
UPDATE users SET new_name = old_name;

Step 3: Backfill data
UPDATE users SET new_name = old_name WHERE new_name IS NULL;

Step 4: Deploy code that only uses new column

Step 5: Drop old column
ALTER TABLE users DROP COLUMN old_name;
```

**Q14: Explain database connection pooling in detail.**
**A:** Connection pool maintains a pool of reusable database connections:

**How it works**:
1. Application starts, creates pool (min connections)
2. Request needs database: Gets connection from pool
3. Uses connection to execute query
4. Returns connection to pool (doesn't close)
5. Next request reuses same connection

**Configuration**:
```javascript
{
  max: 20,        // Maximum connections
  min: 5,         // Minimum idle connections
  idleTimeoutMillis: 30000,  // Close idle after 30s
  connectionTimeoutMillis: 2000  // Wait 2s for connection
}
```

**Why it matters**:
- Creating connection takes 20-50ms
- Database limits total connections (100-200 typically)
- Multiple app servers share connection limit
- Math: 10 app servers × 20 max connections = 200 total

**Q15: How do you debug database performance issues in production?**
**A:** Systematic approach:
1. **Enable slow query log**:
```sql
ALTER SYSTEM SET log_min_duration_statement = 1000;
```

2. **Find slow queries**:
```sql
SELECT query, mean_exec_time, calls
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;
```

3. **Analyze query plan**:
```sql
EXPLAIN ANALYZE SELECT...;
```

4. **Check for missing indexes**:
Look for sequential scans in EXPLAIN output

5. **Monitor metrics**:
- Connection count
- CPU usage
- Disk I/O
- Cache hit ratio
- Lock waits

6. **Check application**:
- N+1 queries
- Missing connection pooling
- Not closing connections

**Q16: What is eventual consistency and when would you use it?**
**A:** Eventual consistency means replicas may temporarily have different data, but will eventually synchronize.

**Example**: Write to master, read from replica
- Write: Insert user, returns success
- Read immediately: Might not see new user (replication lag)
- Read after few seconds: Sees new user

**Use cases**:
- Social media feeds (slight delay acceptable)
- Product catalogs
- Analytics data

**Not suitable for**:
- Financial transactions
- Inventory counts
- User authentication

**Q17: How would you migrate from one database to another (e.g., MySQL to PostgreSQL)?**
**A:** Step-by-step migration:
1. **Dual-write phase**:
   - Write to both old and new database
   - Read from old database
   - Verify data consistency

2. **Data migration**:
   - Export from MySQL: `mysqldump`
   - Transform schema differences
   - Import to PostgreSQL: `psql`
   - Verify data integrity

3. **Dual-read phase**:
   - Continue dual-write
   - Read from new database
   - Compare results with old database
   - Monitor for issues

4. **Switch fully**:
   - Stop writing to old database
   - Only use new database
   - Keep old database for rollback

5. **Cleanup**:
   - After stability period (weeks)
   - Decommission old database

**Q18: Explain database sharding.**
**A:** Sharding splits database horizontally across multiple servers:

**Example: User database**
```
Shard 1: Users with ID 1-1M
Shard 2: Users with ID 1M-2M
Shard 3: Users with ID 2M-3M
```

**Sharding strategies**:
- **Range-based**: By ID ranges
- **Hash-based**: Hash user ID to determine shard
- **Geographic**: By user location
- **Feature-based**: Users on shard 1, posts on shard 2

**Challenges**:
- Joins across shards difficult
- Rebalancing shards
- Complex application code
- No ACID across shards

**When to shard**:
- Single database can't handle load
- Data too large for one server
- Need geographic distribution

**Q19: What are database transactions and how do they work?**
**A:** Transaction is a set of operations that execute as a single unit:

```javascript
const client = await pool.connect();
try {
  await client.query('BEGIN');
  
  // Deduct from account 1
  await client.query(
    'UPDATE accounts SET balance = balance - $1 WHERE id = $2',
    [100, accountId1]
  );
  
  // Add to account 2
  await client.query(
    'UPDATE accounts SET balance = balance + $1 WHERE id = $2',
    [100, accountId2]
  );
  
  await client.query('COMMIT');
} catch (e) {
  await client.query('ROLLBACK');
  throw e;
} finally {
  client.release();
}
```

**ACID Properties**:
- **Atomicity**: All or nothing
- **Consistency**: Database stays valid
- **Isolation**: Transactions don't interfere
- **Durability**: Changes permanent after commit

**Q20: How do you monitor database health in production?**
**A:** Comprehensive monitoring:

1. **Connection Metrics**:
   - Active connections
   - Max connections reached
   - Connection errors

2. **Performance Metrics**:
   - Query response time
   - Queries per second
   - Slow query count
   - Cache hit ratio

3. **Resource Metrics**:
   - CPU utilization
   - Memory usage
   - Disk I/O
   - Storage capacity

4. **Replication Metrics**:
   - Replication lag
   - Replica health

5. **Error Metrics**:
   - Failed queries
   - Deadlocks
   - Connection timeouts

**Tools**:
- CloudWatch (AWS)
- Datadog
- New Relic
- pganalyze (PostgreSQL)
- MongoDB Atlas monitoring

**Alerts**:
- CPU > 80% for 5 minutes
- Storage > 85%
- Replication lag > 30 seconds
- Failed connections > 10/min

---

## Summary

Database deployment is critical for application reliability and performance. Choosing between managed and self-hosted, implementing proper security, configuring backups, optimizing performance, and monitoring health are all essential aspects. Understanding how the backend connects to the database and how data flows through the system is fundamental to building robust applications.

**Key Takeaways**:
- Managed databases simplify operations but cost more
- Always use connection pooling
- Secure with SSL, firewalls, strong passwords
- Automate backups and test restoration
- Use migrations for schema changes
- Index frequently queried columns
- Monitor performance and errors
- Plan for scaling (read replicas, sharding)
- Never commit credentials to Git
- Test in staging before production migrations
