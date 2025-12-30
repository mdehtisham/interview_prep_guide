# NestJS Scheduling & Background Jobs - Top Interview Questions

## Scheduling Fundamentals

<details>
<summary><strong>1. What is task scheduling and when do you need it?</strong></summary>

**Answer:**

**Task scheduling** is the ability to execute code automatically at **specific times or intervals** without manual intervention. Instead of users triggering actions, the **system runs tasks in the background** based on time-based triggers (every day at midnight, every 5 minutes, once per hour, on specific dates). NestJS provides `@nestjs/schedule` package with decorators (`@Cron()`, `@Interval()`, `@Timeout()`) to define scheduled tasks that run automatically. Common use cases: **data cleanup** (delete old records daily), **report generation** (monthly reports at midnight), **email notifications** (send reminders hourly), **data synchronization** (sync with external APIs every 15 minutes), **cache warming** (pre-load cache before peak hours), **health checks** (ping services every minute), **backups** (database backup nightly), **billing** (process subscriptions monthly), **token expiration** (invalidate expired tokens hourly). Benefits: **automation** (no manual intervention), **reliability** (runs even when no users active), **consistency** (tasks execute on schedule), **resource optimization** (run heavy tasks during off-peak hours). Alternatives: manually triggering tasks (not scalable), external cron services (additional infrastructure), cloud schedulers (AWS EventBridge, Google Cloud Scheduler - more expensive).

---

### **Task Scheduling Architecture:**

```
TIME-BASED TRIGGERS:
┌─────────────────────────────────────────────────────┐
│  Scheduler (NestJS @nestjs/schedule)                │
├─────────────────────────────────────────────────────┤
│  Cron Jobs         → Execute at specific times      │
│  (0 0 * * *)         (midnight daily)               │
├─────────────────────────────────────────────────────┤
│  Intervals         → Execute repeatedly              │
│  (every 5 minutes)   (fixed time between runs)      │
├─────────────────────────────────────────────────────┤
│  Timeouts          → Execute once after delay       │
│  (in 10 seconds)     (one-time delayed task)        │
└─────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────┐
│  Background Task Execution                          │
│  - Data cleanup                                     │
│  - Report generation                                │
│  - Email notifications                              │
│  - API synchronization                              │
│  - Cache management                                 │
└─────────────────────────────────────────────────────┘

Benefits:
✅ Automated execution (no manual trigger)
✅ Runs 24/7 (even when no users active)
✅ Consistent timing (reliable schedule)
✅ Off-peak processing (heavy tasks at night)
```

---

### **Method 1: Data Cleanup - Delete Old Records**

```typescript
// ========== USE CASE: DATA CLEANUP ==========

// Delete expired sessions, old logs, soft-deleted records
// Runs automatically without manual intervention

// Install @nestjs/schedule
// npm install @nestjs/schedule

// Import ScheduleModule in AppModule
// src/app.module.ts
import { ScheduleModule } from '@nestjs/schedule';

@Module({
  imports: [
    ScheduleModule.forRoot(),  // Enable scheduling
    // ... other modules
  ],
})
export class AppModule {}

// Create scheduled task service
// src/features/cleanup/cleanup.service.ts
import { Injectable, Logger } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';

@Injectable()
export class CleanupService {
  private readonly logger = new Logger(CleanupService.name);

  constructor(
    private readonly sessionsRepository: SessionsRepository,
    private readonly logsRepository: LogsRepository,
  ) {}

  // Run daily at 2 AM to delete old records
  @Cron(CronExpression.EVERY_DAY_AT_2AM)
  async cleanupExpiredSessions() {
    this.logger.log('Starting session cleanup job');

    const now = new Date();
    
    // Delete sessions older than 7 days
    const result = await this.sessionsRepository.delete({
      expiresAt: LessThan(now),
    });

    this.logger.log(`Deleted ${result.affected} expired sessions`);
  }

  // Run weekly on Sunday at 3 AM
  @Cron('0 3 * * 0')  // Minute Hour Day Month DayOfWeek
  async cleanupOldLogs() {
    this.logger.log('Starting logs cleanup job');

    const thirtyDaysAgo = new Date();
    thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);

    // Delete logs older than 30 days
    const result = await this.logsRepository.delete({
      createdAt: LessThan(thirtyDaysAgo),
    });

    this.logger.log(`Deleted ${result.affected} old logs`);
  }
}

// Register service in module
// src/features/cleanup/cleanup.module.ts
@Module({
  providers: [CleanupService],
})
export class CleanupModule {}

// BENEFITS:
// - Runs automatically at 2 AM daily (no manual trigger)
// - Database stays clean (old data removed)
// - Off-peak execution (minimal user impact)
// - Logging for monitoring (track deleted records)
```

---

### **Method 2: Report Generation - Monthly Reports**

```typescript
// ========== USE CASE: REPORT GENERATION ==========

// Generate monthly sales reports automatically
// Email to management team at start of each month

// src/features/reports/reports.service.ts
@Injectable()
export class ReportsService {
  private readonly logger = new Logger(ReportsService.name);

  constructor(
    private readonly ordersRepository: OrdersRepository,
    private readonly emailService: EmailService,
  ) {}

  // Run on 1st day of every month at midnight
  @Cron('0 0 1 * *')  // Minute=0, Hour=0, Day=1, Month=*, DayOfWeek=*
  async generateMonthlySalesReport() {
    this.logger.log('Generating monthly sales report');

    // Calculate previous month
    const now = new Date();
    const firstDayLastMonth = new Date(now.getFullYear(), now.getMonth() - 1, 1);
    const lastDayLastMonth = new Date(now.getFullYear(), now.getMonth(), 0);

    // Query orders from last month
    const orders = await this.ordersRepository.find({
      where: {
        createdAt: Between(firstDayLastMonth, lastDayLastMonth),
      },
    });

    // Calculate metrics
    const totalRevenue = orders.reduce((sum, order) => sum + order.total, 0);
    const averageOrderValue = totalRevenue / orders.length;

    // Generate report
    const report = {
      month: firstDayLastMonth.toLocaleString('default', { month: 'long', year: 'numeric' }),
      totalOrders: orders.length,
      totalRevenue,
      averageOrderValue,
      generatedAt: new Date().toISOString(),
    };

    // Email report to management
    await this.emailService.sendEmail({
      to: 'management@company.com',
      subject: `Monthly Sales Report - ${report.month}`,
      html: this.formatReportHTML(report),
    });

    this.logger.log(`Monthly report sent: ${report.totalOrders} orders, $${report.totalRevenue}`);
  }

  private formatReportHTML(report: any): string {
    return `
      <h1>Monthly Sales Report</h1>
      <h2>${report.month}</h2>
      <ul>
        <li>Total Orders: ${report.totalOrders}</li>
        <li>Total Revenue: $${report.totalRevenue.toFixed(2)}</li>
        <li>Average Order Value: $${report.averageOrderValue.toFixed(2)}</li>
      </ul>
      <p>Generated at: ${report.generatedAt}</p>
    `;
  }
}

// BENEFITS:
// - Automatic generation on 1st of month (consistent schedule)
// - Management gets reports without asking (proactive)
// - Always includes full previous month (accurate timeframe)
// - Logging for audit trail (track report generation)
```

---

### **Method 3: API Synchronization - Sync External Data**

```typescript
// ========== USE CASE: API SYNCHRONIZATION ==========

// Sync product prices from external supplier API every 15 minutes
// Keep inventory updated in real-time

// src/features/sync/sync.service.ts
@Injectable()
export class SyncService {
  private readonly logger = new Logger(SyncService.name);

  constructor(
    private readonly productsRepository: ProductsRepository,
    private readonly httpService: HttpService,
  ) {}

  // Run every 15 minutes
  @Cron('*/15 * * * *')  // Every 15 minutes (0, 15, 30, 45)
  async syncProductPrices() {
    this.logger.log('Starting product price sync');

    try {
      // Fetch latest prices from supplier API
      const { data } = await this.httpService.axiosRef.get(
        'https://supplier-api.com/products/prices',
        {
          headers: { 'API-Key': process.env.SUPPLIER_API_KEY },
          timeout: 10000,  // 10 second timeout
        },
      );

      // Update prices in database
      let updatedCount = 0;
      
      for (const item of data.products) {
        const product = await this.productsRepository.findOne({
          where: { supplierSku: item.sku },
        });

        if (product && product.price !== item.price) {
          await this.productsRepository.update(product.id, {
            price: item.price,
            lastSyncedAt: new Date(),
          });
          updatedCount++;
        }
      }

      this.logger.log(`Price sync complete: ${updatedCount} products updated`);
      
    } catch (error) {
      this.logger.error('Price sync failed', error.stack);
      // Could send alert to Slack/email on failure
    }
  }

  // Run every hour at minute 0
  @Cron(CronExpression.EVERY_HOUR)
  async syncInventoryLevels() {
    this.logger.log('Starting inventory sync');

    try {
      const { data } = await this.httpService.axiosRef.get(
        'https://supplier-api.com/products/inventory',
        {
          headers: { 'API-Key': process.env.SUPPLIER_API_KEY },
        },
      );

      for (const item of data.products) {
        await this.productsRepository.update(
          { supplierSku: item.sku },
          { stock: item.quantity, lastSyncedAt: new Date() },
        );
      }

      this.logger.log(`Inventory sync complete: ${data.products.length} products`);
      
    } catch (error) {
      this.logger.error('Inventory sync failed', error.stack);
    }
  }
}

// BENEFITS:
// - Prices updated every 15 minutes (near real-time)
// - Inventory synced hourly (always current)
// - Error handling with logging (catch API failures)
// - Timeout protection (prevent hanging requests)
```

---

### **Method 4: Email Notifications - Send Reminders**

```typescript
// ========== USE CASE: EMAIL NOTIFICATIONS ==========

// Send reminder emails for upcoming appointments
// Runs hourly to check for appointments in next 24 hours

// src/features/notifications/notifications.service.ts
@Injectable()
export class NotificationsService {
  private readonly logger = new Logger(NotificationsService.name);

  constructor(
    private readonly appointmentsRepository: AppointmentsRepository,
    private readonly emailService: EmailService,
  ) {}

  // Run every hour at minute 0
  @Cron('0 * * * *')  // 0:00, 1:00, 2:00, etc.
  async sendAppointmentReminders() {
    this.logger.log('Checking for upcoming appointments');

    const now = new Date();
    const in24Hours = new Date(now.getTime() + 24 * 60 * 60 * 1000);

    // Find appointments in next 24 hours that haven't been reminded
    const appointments = await this.appointmentsRepository.find({
      where: {
        scheduledAt: Between(now, in24Hours),
        reminderSent: false,
      },
      relations: ['user'],
    });

    let sentCount = 0;

    for (const appointment of appointments) {
      await this.emailService.sendEmail({
        to: appointment.user.email,
        subject: 'Appointment Reminder',
        html: `
          <h2>Appointment Reminder</h2>
          <p>You have an appointment scheduled for:</p>
          <p><strong>${appointment.scheduledAt.toLocaleString()}</strong></p>
          <p>Location: ${appointment.location}</p>
        `,
      });

      // Mark reminder as sent
      await this.appointmentsRepository.update(appointment.id, {
        reminderSent: true,
      });

      sentCount++;
    }

    this.logger.log(`Sent ${sentCount} appointment reminders`);
  }

  // Send weekly digest on Monday at 9 AM
  @Cron('0 9 * * 1')  // Monday at 9:00
  async sendWeeklyDigest() {
    this.logger.log('Sending weekly digest emails');

    // Get all active users
    const users = await this.usersRepository.find({
      where: { isActive: true },
    });

    for (const user of users) {
      // Get user's activity from past week
      const weekAgo = new Date();
      weekAgo.setDate(weekAgo.getDate() - 7);

      const activity = await this.getActivitySummary(user.id, weekAgo);

      await this.emailService.sendEmail({
        to: user.email,
        subject: 'Your Weekly Summary',
        html: this.formatDigestHTML(activity),
      });
    }

    this.logger.log(`Sent weekly digest to ${users.length} users`);
  }
}

// BENEFITS:
// - Reminders sent automatically (users don't forget)
// - Hourly check ensures timely notifications
// - Weekly digest keeps users engaged
// - Mark reminder as sent (prevent duplicates)
```

---

### **Method 5: When You Need Task Scheduling**

```typescript
// ========== WHEN TO USE TASK SCHEDULING ==========

// ✅ USE SCHEDULING WHEN:

// 1. TIME-BASED AUTOMATION
//    - Run tasks at specific times (midnight, hourly, weekly)
//    - No user action triggers the task
//    - Example: Daily backups, monthly reports

// 2. PERIODIC DATA PROCESSING
//    - Process data at regular intervals
//    - Keep data synchronized
//    - Example: Sync with external APIs every 15 minutes

// 3. MAINTENANCE TASKS
//    - Cleanup old data
//    - Optimize database
//    - Example: Delete expired sessions daily

// 4. NOTIFICATIONS & REMINDERS
//    - Send emails/notifications on schedule
//    - Example: Appointment reminders, weekly digests

// 5. OFF-PEAK PROCESSING
//    - Heavy tasks during low-traffic hours
//    - Example: Generate reports at 2 AM

// ❌ DON'T USE SCHEDULING WHEN:

// 1. USER-TRIGGERED ACTIONS
//    - Task should run when user performs action
//    - Use regular API endpoints instead
//    - Example: User uploads file → process immediately

// 2. REAL-TIME PROCESSING
//    - Task must run immediately
//    - Use message queues (Bull) instead
//    - Example: Send order confirmation email (use queue, not cron)

// 3. EVENT-DRIVEN TASKS
//    - Task triggered by external event
//    - Use webhooks/event handlers instead
//    - Example: Payment webhook → fulfill order

// 4. HIGH-FREQUENCY TASKS
//    - Tasks need to run every second
//    - Use Interval() but consider performance
//    - Example: Real-time monitoring (use dedicated service)

// ========== COMPARISON ==========

// Scheduled Jobs (Cron):
// - Time-based triggers (every hour, daily at midnight)
// - Predictable schedule (know exactly when runs)
// - Examples: Cleanup, reports, sync

// Message Queues (Bull):
// - Event-based triggers (user action, webhook)
// - Process immediately or with priority
// - Examples: Email on signup, process uploaded file

// Both can coexist:
// - Cron: Daily report generation
// - Queue: Send individual order confirmation emails
```

**Key Takeaway:** **Task scheduling** is automatic code execution at **specific times or intervals** without manual intervention using NestJS `@nestjs/schedule` package with decorators (`@Cron()` for specific times like midnight daily, `@Interval()` for repeated execution every N milliseconds, `@Timeout()` for one-time delayed execution). **Common use cases**: **data cleanup** (delete expired sessions daily at 2 AM, remove old logs weekly, purge soft-deleted records monthly), **report generation** (monthly sales reports on 1st of month, weekly analytics every Monday, quarterly summaries), **email notifications** (appointment reminders hourly, weekly digests Monday 9 AM, promotional emails at optimal send times), **API synchronization** (sync prices every 15 minutes, update inventory hourly, fetch external data periodically), **cache warming** (pre-load frequently accessed data before peak hours, refresh cached reports nightly), **health checks** (ping services every minute, verify database connectivity every 5 minutes), **backups** (database backup nightly at 3 AM, export data weekly), **billing** (process subscriptions monthly, charge recurring fees, send invoices), **token expiration** (invalidate expired JWT tokens hourly, cleanup refresh tokens daily). **Benefits**: **automation** (no manual intervention needed, runs 24/7 even when no users active), **reliability** (consistent execution on schedule, doesn't depend on user actions), **resource optimization** (run heavy tasks during off-peak hours like 2-4 AM when traffic low, avoid impacting user experience), **consistency** (tasks execute at same time reliably, predictable behavior). **When to use**: time-based automation (specific times/intervals), periodic data processing (keep data fresh), maintenance tasks (cleanup, optimization), off-peak processing (heavy computation at night). **When NOT to use**: user-triggered actions (use API endpoints), real-time processing (use message queues like Bull), event-driven tasks (use webhooks), tasks need immediate execution (use queues not cron).

</details>

<details>
<summary><strong>2. What is `@nestjs/schedule` package?</strong></summary>

**Answer:**

`@nestjs/schedule` is an **official NestJS package** that provides **declarative task scheduling** using decorators (`@Cron()`, `@Interval()`, `@Timeout()`). It's built on top of **node-cron** library and integrates seamlessly with NestJS dependency injection system. Install with `npm install @nestjs/schedule`, import `ScheduleModule.forRoot()` in AppModule, then use decorators on service methods to define scheduled tasks. The package provides: **cron jobs** (run at specific times using cron syntax `0 0 * * *` for midnight daily), **intervals** (run repeatedly every N milliseconds with `@Interval(5000)`), **timeouts** (run once after delay with `@Timeout(10000)`), **dynamic scheduling** (add/remove jobs at runtime using `SchedulerRegistry`), **named jobs** (reference jobs by name for management). Behind the scenes, it uses **node-cron** for cron expression parsing and **setTimeout/setInterval** for timeouts/intervals. Advantages: **declarative syntax** (use decorators, no manual cron setup), **NestJS integration** (access services via DI, use logging, configuration), **TypeScript support** (type-safe job definitions), **dynamic management** (add/remove jobs programmatically), **testing support** (mock scheduled methods in tests).

---

### **@nestjs/schedule Package Overview:**

```
@nestjs/schedule ARCHITECTURE:
┌─────────────────────────────────────────────────────┐
│  ScheduleModule.forRoot()                           │
│  - Initializes scheduler                            │
│  - Registers SchedulerRegistry                      │
│  - Scans for decorated methods                      │
└────────────────┬────────────────────────────────────┘
                 │
     ┌───────────┴───────────┐
     │                       │
     ▼                       ▼
┌─────────────┐      ┌─────────────┐
│  node-cron  │      │ setTimeout/ │
│  (cron)     │      │ setInterval │
└─────────────┘      └─────────────┘
     │                       │
     ▼                       ▼
┌─────────────┐      ┌─────────────┐
│  @Cron()    │      │ @Interval() │
│  Decorator  │      │ @Timeout()  │
└─────────────┘      └─────────────┘
     │                       │
     └───────────┬───────────┘
                 ▼
┌─────────────────────────────────────┐
│  Your Service Methods               │
│  - Executed automatically           │
│  - Access DI services               │
│  - Use logger, repositories, etc.   │
└─────────────────────────────────────┘

Features:
✅ Declarative decorators (@Cron, @Interval, @Timeout)
✅ NestJS DI integration (inject services)
✅ Dynamic scheduling (SchedulerRegistry)
✅ TypeScript support (type-safe)
✅ Named jobs (reference by name)
```

---

### **Method 1: Installation and Setup**

```typescript
// ========== INSTALLATION ==========

// Step 1: Install package
// npm install @nestjs/schedule

// Step 2: Import ScheduleModule in root module
// src/app.module.ts
import { Module } from '@nestjs/common';
import { ScheduleModule } from '@nestjs/schedule';

@Module({
  imports: [
    // Enable scheduling globally
    ScheduleModule.forRoot(),
    
    // Other modules
    TypeOrmModule.forRoot(/* ... */),
    ConfigModule.forRoot(),
    
    // Feature modules with scheduled tasks
    TasksModule,
    CleanupModule,
    ReportsModule,
  ],
})
export class AppModule {}

// ScheduleModule.forRoot() does:
// 1. Initialize scheduler
// 2. Register SchedulerRegistry (for dynamic scheduling)
// 3. Scan all modules for @Cron/@Interval/@Timeout decorators
// 4. Register and start scheduled tasks

// Step 3: Create service with scheduled methods
// src/features/tasks/tasks.service.ts
import { Injectable, Logger } from '@nestjs/common';
import { Cron, Interval, Timeout } from '@nestjs/schedule';

@Injectable()
export class TasksService {
  private readonly logger = new Logger(TasksService.name);

  // Cron job: Run at specific times
  @Cron('0 0 * * *')  // Midnight daily
  handleCron() {
    this.logger.log('Cron job executed');
  }

  // Interval: Run every N milliseconds
  @Interval(10000)  // Every 10 seconds
  handleInterval() {
    this.logger.log('Interval executed');
  }

  // Timeout: Run once after delay
  @Timeout(5000)  // After 5 seconds
  handleTimeout() {
    this.logger.log('Timeout executed');
  }
}

// Step 4: Register service in module
// src/features/tasks/tasks.module.ts
@Module({
  providers: [TasksService],
})
export class TasksModule {}

// That's it! Scheduled tasks start automatically when app starts
```

---

### **Method 2: Using @Cron() Decorator**

```typescript
// ========== @CRON() DECORATOR ==========

// Execute at specific times using cron expressions
import { Cron, CronExpression } from '@nestjs/schedule';

@Injectable()
export class ScheduledTasksService {
  private readonly logger = new Logger(ScheduledTasksService.name);

  // Method 1: Using cron syntax string
  @Cron('0 0 * * *')  // Midnight every day
  runAtMidnight() {
    this.logger.log('Running at midnight');
  }

  // Method 2: Using CronExpression enum (recommended)
  @Cron(CronExpression.EVERY_5_MINUTES)
  runEveryFiveMinutes() {
    this.logger.log('Running every 5 minutes');
  }

  // Method 3: Using cron expression with options
  @Cron('0 9 * * 1-5', {
    name: 'weekdayMorning',  // Name for dynamic management
    timeZone: 'America/New_York',  // Timezone
  })
  runWeekdayMornings() {
    this.logger.log('Running weekday mornings at 9 AM EST');
  }

  // Common CronExpression constants:
  // - EVERY_SECOND: '* * * * * *'
  // - EVERY_5_SECONDS: '*/5 * * * * *'
  // - EVERY_10_SECONDS: '*/10 * * * * *'
  // - EVERY_30_SECONDS: '*/30 * * * * *'
  // - EVERY_MINUTE: '0 * * * * *'
  // - EVERY_5_MINUTES: '0 */5 * * * *'
  // - EVERY_10_MINUTES: '0 */10 * * * *'
  // - EVERY_30_MINUTES: '0 */30 * * * *'
  // - EVERY_HOUR: '0 0 * * * *'
  // - EVERY_DAY_AT_MIDNIGHT: '0 0 0 * * *'
  // - EVERY_DAY_AT_NOON: '0 0 12 * * *'
  // - EVERY_WEEK: '0 0 0 * * 0'
  // - EVERY_MONTH: '0 0 0 1 * *'
  // - EVERY_YEAR: '0 0 0 1 1 *'
}

// Cron syntax: second minute hour day month dayOfWeek
// Example: '30 15 10 * * 1-5'
//           │  │  │  │  │  └─ Monday to Friday (1-5)
//           │  │  │  │  └──── Every month (*)
//           │  │  │  └─────── Every day (*)
//           │  │  └────────── 10 AM (10)
//           │  └───────────── 15 minutes (15)
//           └──────────────── 30 seconds (30)
// Result: 10:15:30 AM every weekday
```

---

### **Method 3: Using @Interval() and @Timeout() Decorators**

```typescript
// ========== @INTERVAL() DECORATOR ==========

// Execute repeatedly at fixed intervals
@Injectable()
export class IntervalTasksService {
  private readonly logger = new Logger(IntervalTasksService.name);

  // Run every 10 seconds
  @Interval(10000)  // 10000 milliseconds = 10 seconds
  handleInterval() {
    this.logger.log('Interval task executed');
    // Task logic here
  }

  // Named interval for dynamic management
  @Interval('healthCheck', 30000)  // Every 30 seconds
  healthCheck() {
    this.logger.log('Health check running');
    // Ping database, external services, etc.
  }

  // WARNING: Interval starts immediately when app starts
  // First execution: 0 seconds (on startup)
  // Second execution: 10 seconds later
  // Third execution: 20 seconds later
  // ...
}

// ========== @TIMEOUT() DECORATOR ==========

// Execute once after a delay
@Injectable()
export class TimeoutTasksService {
  private readonly logger = new Logger(TimeoutTasksService.name);

  // Run once, 5 seconds after app starts
  @Timeout(5000)  // 5000 milliseconds = 5 seconds
  handleTimeout() {
    this.logger.log('Timeout task executed once');
    // One-time initialization, cache warming, etc.
  }

  // Named timeout
  @Timeout('initCache', 10000)
  initializeCache() {
    this.logger.log('Initializing cache');
    // Pre-load frequently accessed data
  }
}

// DIFFERENCE:
// @Interval() → Runs repeatedly (every N ms)
// @Timeout() → Runs once (after N ms delay)
```

---

### **Method 4: Dynamic Scheduling with SchedulerRegistry**

```typescript
// ========== DYNAMIC SCHEDULING ==========

// Add/remove jobs at runtime (not defined with decorators)
import { Injectable, Logger } from '@nestjs/common';
import { SchedulerRegistry } from '@nestjs/schedule';
import { CronJob } from 'cron';

@Injectable()
export class DynamicSchedulerService {
  private readonly logger = new Logger(DynamicSchedulerService.name);

  constructor(private readonly schedulerRegistry: SchedulerRegistry) {}

  // Add cron job dynamically
  addCronJob(name: string, cronExpression: string) {
    const job = new CronJob(cronExpression, () => {
      this.logger.log(`Cron job ${name} executed`);
      // Job logic here
    });

    this.schedulerRegistry.addCronJob(name, job);
    job.start();

    this.logger.log(`Cron job ${name} added: ${cronExpression}`);
  }

  // Delete cron job
  deleteCronJob(name: string) {
    this.schedulerRegistry.deleteCronJob(name);
    this.logger.log(`Cron job ${name} deleted`);
  }

  // Get cron job
  getCronJob(name: string): CronJob {
    return this.schedulerRegistry.getCronJob(name);
  }

  // Add interval dynamically
  addInterval(name: string, milliseconds: number) {
    const callback = () => {
      this.logger.log(`Interval ${name} executed`);
    };

    const interval = setInterval(callback, milliseconds);
    this.schedulerRegistry.addInterval(name, interval);

    this.logger.log(`Interval ${name} added: every ${milliseconds}ms`);
  }

  // Delete interval
  deleteInterval(name: string) {
    this.schedulerRegistry.deleteInterval(name);
    this.logger.log(`Interval ${name} deleted`);
  }

  // Add timeout dynamically
  addTimeout(name: string, milliseconds: number) {
    const callback = () => {
      this.logger.log(`Timeout ${name} executed`);
    };

    const timeout = setTimeout(callback, milliseconds);
    this.schedulerRegistry.addTimeout(name, timeout);

    this.logger.log(`Timeout ${name} added: in ${milliseconds}ms`);
  }

  // Delete timeout
  deleteTimeout(name: string) {
    this.schedulerRegistry.deleteTimeout(name);
    this.logger.log(`Timeout ${name} deleted`);
  }
}

// Usage in controller
@Controller('scheduler')
export class SchedulerController {
  constructor(private readonly dynamicScheduler: DynamicSchedulerService) {}

  @Post('jobs')
  addJob(@Body() dto: { name: string; cronExpression: string }) {
    this.dynamicScheduler.addCronJob(dto.name, dto.cronExpression);
    return { message: 'Job added' };
  }

  @Delete('jobs/:name')
  deleteJob(@Param('name') name: string) {
    this.dynamicScheduler.deleteCronJob(name);
    return { message: 'Job deleted' };
  }
}

// BENEFITS:
// - Add jobs based on user configuration
// - Remove jobs when no longer needed
// - Modify job schedule at runtime
```

---

### **Method 5: Integration with NestJS DI**

```typescript
// ========== DEPENDENCY INJECTION ==========

// Scheduled methods have full access to NestJS DI system
@Injectable()
export class OrdersSchedulerService {
  private readonly logger = new Logger(OrdersSchedulerService.name);

  // Inject any services
  constructor(
    private readonly ordersRepository: OrdersRepository,
    private readonly emailService: EmailService,
    private readonly configService: ConfigService,
    @InjectQueue('notifications') private readonly notificationQueue: Queue,
  ) {}

  @Cron(CronExpression.EVERY_HOUR)
  async processExpiredOrders() {
    this.logger.log('Processing expired orders');

    // Access configuration
    const expirationHours = this.configService.get<number>('ORDER_EXPIRATION_HOURS');

    // Query repository
    const cutoffTime = new Date();
    cutoffTime.setHours(cutoffTime.getHours() - expirationHours);

    const expiredOrders = await this.ordersRepository.find({
      where: {
        status: 'pending',
        createdAt: LessThan(cutoffTime),
      },
    });

    // Process each order
    for (const order of expiredOrders) {
      // Update database
      await this.ordersRepository.update(order.id, {
        status: 'expired',
      });

      // Add job to queue
      await this.notificationQueue.add('orderExpired', {
        orderId: order.id,
        userId: order.userId,
      });
    }

    this.logger.log(`Processed ${expiredOrders.length} expired orders`);
  }
}

// BENEFITS:
// - Full DI access (repositories, services, queues)
// - Use ConfigService for configuration
// - Access Logger for monitoring
// - Inject HttpService for API calls
// - Use any NestJS feature (guards, interceptors not applicable but services yes)
```

**Key Takeaway:** `@nestjs/schedule` is **official NestJS package** for **declarative task scheduling** using decorators built on **node-cron** library with seamless NestJS integration. **Installation**: `npm install @nestjs/schedule`, import `ScheduleModule.forRoot()` in AppModule (scans for decorated methods and initializes scheduler), use decorators on service methods (`@Cron()`, `@Interval()`, `@Timeout()`). **Features**: **cron jobs** with `@Cron('0 0 * * *')` or `@Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT)` for specific times using cron syntax (second minute hour day month dayOfWeek), **intervals** with `@Interval(10000)` for repeated execution every N milliseconds (runs immediately on startup then every interval), **timeouts** with `@Timeout(5000)` for one-time delayed execution (runs once after N milliseconds), **named jobs** with `@Cron('0 0 * * *', { name: 'dailyBackup' })` for dynamic management, **timezone support** with `@Cron('0 9 * * *', { timeZone: 'America/New_York' })` for specific timezone execution. **Dynamic scheduling**: inject `SchedulerRegistry` to add/remove/get jobs at runtime (schedulerRegistry.addCronJob(name, new CronJob()), schedulerRegistry.deleteCronJob(name), schedulerRegistry.addInterval/deleteInterval, schedulerRegistry.addTimeout/deleteTimeout), useful for user-configured schedules or conditional job creation. **NestJS integration**: scheduled methods have full DI access (inject repositories, services, ConfigService, Logger, HttpService, queues), use any NestJS feature in job logic, methods are class methods so can be private/public/async, TypeScript type-safe. **Behind the scenes**: uses **node-cron** for cron expression parsing and job scheduling, uses native **setTimeout/setInterval** for timeouts/intervals, ScheduleModule scans modules on startup for decorated methods and registers them. **Advantages**: **declarative syntax** (decorators not manual cron setup), **type-safe** (TypeScript support), **testable** (mock scheduled methods in tests), **centralized** (all schedules in code not external cron files), **dynamic** (add/remove at runtime), **integrated** (use NestJS DI, logging, config).

</details>

<details>
<summary><strong>3. What types of scheduled tasks can you create (cron, intervals, timeouts)?</strong></summary>

**Answer:**

`@nestjs/schedule` provides **three types of scheduled tasks**: **Cron jobs** (`@Cron()`) execute at specific times using cron expressions (`0 0 * * *` for midnight daily, `*/15 * * * *` for every 15 minutes), **Intervals** (`@Interval()`) execute repeatedly at fixed time intervals in milliseconds (`@Interval(5000)` every 5 seconds, runs immediately on startup then every interval), **Timeouts** (`@Timeout()`) execute once after a delay in milliseconds (`@Timeout(10000)` runs once 10 seconds after startup, one-time delayed execution). **Cron jobs** are for **calendar-based scheduling** (specific times/dates like daily at 2 AM, weekdays at 9 AM, first of month), **Intervals** are for **fixed-frequency tasks** (health checks every 30 seconds, cache refresh every 5 minutes, metrics collection every minute), **Timeouts** are for **one-time initialization** (warm cache on startup, delayed startup tasks, one-off scheduled tasks). All three support **named tasks** for dynamic management via SchedulerRegistry, have full NestJS DI access (inject services, repositories, etc.), and can be added/removed dynamically at runtime.

---

### **Three Types of Scheduled Tasks:**

```
TYPE 1: CRON JOBS (@Cron)
┌─────────────────────────────────────────────────────┐
│  Calendar-Based Scheduling                          │
│  ─────────────────────────────                      │
│  Trigger: Specific times/dates                      │
│  Syntax: Cron expression (0 0 * * *)                │
│                                                      │
│  Examples:                                           │
│  • @Cron('0 0 * * *')     → Midnight daily          │
│  • @Cron('0 9 * * 1-5')   → 9 AM weekdays           │
│  • @Cron('0 0 1 * *')     → 1st of month            │
│  • @Cron('*/15 * * * *')  → Every 15 minutes        │
│                                                      │
│  Use When:                                           │
│  ✅ Need specific time (midnight, 9 AM, etc.)       │
│  ✅ Calendar-based (daily, weekly, monthly)         │
│  ✅ Human-readable schedule ("every Monday")        │
└─────────────────────────────────────────────────────┘

TYPE 2: INTERVALS (@Interval)
┌─────────────────────────────────────────────────────┐
│  Fixed-Frequency Execution                          │
│  ─────────────────────────                          │
│  Trigger: Every N milliseconds                      │
│  Syntax: Milliseconds (10000 = 10 seconds)          │
│                                                      │
│  Examples:                                           │
│  • @Interval(5000)        → Every 5 seconds         │
│  • @Interval(60000)       → Every 1 minute          │
│  • @Interval(300000)      → Every 5 minutes         │
│                                                      │
│  Behavior:                                           │
│  • First run: Immediately on startup                │
│  • Subsequent runs: Every N milliseconds            │
│  • Continues until app stops                        │
│                                                      │
│  Use When:                                           │
│  ✅ Fixed interval (every 30 seconds)               │
│  ✅ Health checks, polling, monitoring              │
│  ✅ Don't care about specific time                  │
└─────────────────────────────────────────────────────┘

TYPE 3: TIMEOUTS (@Timeout)
┌─────────────────────────────────────────────────────┐
│  One-Time Delayed Execution                         │
│  ───────────────────────────                        │
│  Trigger: Once after N milliseconds                 │
│  Syntax: Milliseconds (5000 = 5 seconds)            │
│                                                      │
│  Examples:                                           │
│  • @Timeout(5000)         → After 5 seconds         │
│  • @Timeout(60000)        → After 1 minute          │
│                                                      │
│  Behavior:                                           │
│  • Runs ONCE after delay from startup               │
│  • Never runs again                                  │
│                                                      │
│  Use When:                                           │
│  ✅ Initialization tasks (cache warming)            │
│  ✅ Delayed startup actions                         │
│  ✅ One-time scheduled task                         │
└─────────────────────────────────────────────────────┘
```

---

### **Method 1: Cron Jobs - Calendar-Based Scheduling**

```typescript
// ========== CRON JOBS: SPECIFIC TIMES/DATES ==========

import { Injectable, Logger } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';

@Injectable()
export class CronTasksService {
  private readonly logger = new Logger(CronTasksService.name);

  // Method 1: Using cron expression string
  @Cron('0 0 * * *')  // Midnight every day (00:00:00)
  runAtMidnight() {
    this.logger.log('Running at midnight');
    // Cleanup old data, generate daily reports, etc.
  }

  // Method 2: Using CronExpression enum (recommended)
  @Cron(CronExpression.EVERY_DAY_AT_2AM)
  runAt2AM() {
    this.logger.log('Running at 2 AM');
    // Database backup, heavy processing during off-peak hours
  }

  // Method 3: Complex schedule - Weekdays at 9 AM
  @Cron('0 9 * * 1-5')  // Monday-Friday at 09:00:00
  runWeekdayMornings() {
    this.logger.log('Running weekday mornings');
    // Send daily standup reminders, morning reports
  }

  // Method 4: First day of month
  @Cron('0 0 1 * *')  // 1st day of every month at midnight
  runMonthly() {
    this.logger.log('Running monthly');
    // Generate monthly reports, process subscriptions
  }

  // Method 5: Every 15 minutes
  @Cron('*/15 * * * *')  // 0, 15, 30, 45 minutes of every hour
  runEvery15Minutes() {
    this.logger.log('Running every 15 minutes');
    // Sync with external API, refresh cache
  }

  // Method 6: With options (timezone, name)
  @Cron('0 12 * * *', {
    name: 'lunchReminder',  // Name for dynamic management
    timeZone: 'America/New_York',  // EST/EDT timezone
  })
  sendLunchReminder() {
    this.logger.log('Sending lunch reminder (12 PM EST)');
  }
}

// WHEN TO USE CRON:
// ✅ Need specific time (2 AM, 9 AM, midnight)
// ✅ Calendar-based (daily, weekly, monthly, weekdays)
// ✅ Human-readable schedule ("every Monday at 9 AM")
// ✅ Business hours (9 AM - 5 PM weekdays)
// ✅ Off-peak processing (2-4 AM when traffic low)
```

---

### **Method 2: Intervals - Fixed-Frequency Execution**

```typescript
// ========== INTERVALS: EVERY N MILLISECONDS ==========

@Injectable()
export class IntervalTasksService {
  private readonly logger = new Logger(IntervalTasksService.name);

  // Method 1: Every 10 seconds
  @Interval(10000)  // 10000ms = 10 seconds
  checkHealth() {
    this.logger.log('Health check running');
    // Ping database, external services, etc.
  }

  // Method 2: Every 30 seconds with name
  @Interval('metricsCollection', 30000)  // 30 seconds
  collectMetrics() {
    this.logger.log('Collecting metrics');
    // Gather CPU, memory, request count, etc.
  }

  // Method 3: Every 5 minutes
  @Interval(5 * 60 * 1000)  // 5 minutes = 300000ms
  refreshCache() {
    this.logger.log('Refreshing cache');
    // Update frequently accessed data in cache
  }

  // Method 4: Every minute
  @Interval(60000)  // 1 minute
  monitorQueue() {
    this.logger.log('Monitoring queue depth');
    // Check job queue, alert if too many pending jobs
  }
}

// EXECUTION TIMELINE:
// App starts at 00:00:00
// First run:  00:00:00 (immediately on startup)
// Second run: 00:00:10 (10 seconds later)
// Third run:  00:00:20 (10 seconds later)
// Fourth run: 00:00:30 (10 seconds later)
// ...

// WHEN TO USE INTERVALS:
// ✅ Fixed frequency (every 30 seconds, every 5 minutes)
// ✅ Health checks, polling, monitoring
// ✅ Don't care about specific time (just regular checks)
// ✅ Continuous monitoring tasks
// ❌ Don't use for specific times (use Cron instead)
```

---

### **Method 3: Timeouts - One-Time Delayed Execution**

```typescript
// ========== TIMEOUTS: RUN ONCE AFTER DELAY ==========

@Injectable()
export class TimeoutTasksService {
  private readonly logger = new Logger(TimeoutTasksService.name);

  // Method 1: Run once, 5 seconds after app starts
  @Timeout(5000)  // 5000ms = 5 seconds
  initializeCache() {
    this.logger.log('Initializing cache');
    // Pre-load frequently accessed data into cache
    // Runs once on startup, never again
  }

  // Method 2: Named timeout
  @Timeout('warmup', 10000)  // 10 seconds
  warmupConnections() {
    this.logger.log('Warming up database connections');
    // Establish connection pool, test connections
  }

  // Method 3: Delayed initialization
  @Timeout(30000)  // 30 seconds
  async syncInitialData() {
    this.logger.log('Performing initial data sync');
    // Wait for app to fully start, then sync data
    // Useful when app needs to be ready before task runs
  }
}

// EXECUTION TIMELINE:
// App starts at 00:00:00
// Timeout scheduled for 00:00:05
// Task runs at 00:00:05
// Task NEVER runs again (one-time only)

// WHEN TO USE TIMEOUTS:
// ✅ One-time initialization (cache warming, connection setup)
// ✅ Delayed startup tasks (wait for app to be ready)
// ✅ One-off scheduled task (run once after delay)
// ❌ Don't use for recurring tasks (use Cron or Interval)
```

---

### **Method 4: Comparison - When to Use Each Type**

```typescript
// ========== DECISION MATRIX ==========

// SCENARIO: Delete expired sessions
// SOLUTION: Cron job
@Cron(CronExpression.EVERY_DAY_AT_2AM)
async cleanupSessions() {
  // Runs daily at 2 AM (off-peak hours)
  // Calendar-based: specific time
}

// SCENARIO: Health check every 30 seconds
// SOLUTION: Interval
@Interval(30000)
async healthCheck() {
  // Runs every 30 seconds continuously
  // Fixed frequency: don't care about specific time
}

// SCENARIO: Pre-load cache on startup
// SOLUTION: Timeout
@Timeout(5000)
async warmCache() {
  // Runs once, 5 seconds after startup
  // One-time: initialization task
}

// SCENARIO: Generate monthly reports
// SOLUTION: Cron job
@Cron('0 0 1 * *')  // 1st of month at midnight
async generateMonthlyReport() {
  // Runs on 1st day of every month
  // Calendar-based: specific date
}

// SCENARIO: Monitor queue depth
// SOLUTION: Interval
@Interval(60000)  // Every minute
async monitorQueue() {
  // Runs every minute continuously
  // Fixed frequency: regular monitoring
}

// SCENARIO: Initial database seed
// SOLUTION: Timeout
@Timeout(10000)
async seedDatabase() {
  // Runs once, 10 seconds after startup
  // One-time: initialization task
}
```

---

### **Method 5: Combining Multiple Task Types**

```typescript
// ========== REAL-WORLD EXAMPLE: E-COMMERCE SYSTEM ==========

@Injectable()
export class EcommerceSchedulerService {
  private readonly logger = new Logger(EcommerceSchedulerService.name);

  constructor(
    private readonly ordersRepository: OrdersRepository,
    private readonly cacheService: CacheService,
    private readonly metricsService: MetricsService,
  ) {}

  // CRON: Daily cleanup at 2 AM
  @Cron(CronExpression.EVERY_DAY_AT_2AM)
  async cleanupOldOrders() {
    this.logger.log('Cleaning up old orders');
    const thirtyDaysAgo = new Date();
    thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);
    
    await this.ordersRepository.delete({
      status: 'cancelled',
      createdAt: LessThan(thirtyDaysAgo),
    });
  }

  // CRON: Monthly reports on 1st of month
  @Cron('0 0 1 * *')
  async generateMonthlySalesReport() {
    this.logger.log('Generating monthly sales report');
    // Generate and email report
  }

  // INTERVAL: Check payment status every 5 minutes
  @Interval(5 * 60 * 1000)  // 5 minutes
  async checkPendingPayments() {
    this.logger.log('Checking pending payments');
    // Query payment gateway for status updates
  }

  // INTERVAL: Refresh product cache every minute
  @Interval(60000)  // 1 minute
  async refreshProductCache() {
    this.logger.log('Refreshing product cache');
    await this.cacheService.refreshTopProducts();
  }

  // TIMEOUT: Pre-load cache on startup
  @Timeout(10000)  // After 10 seconds
  async initializeProductCache() {
    this.logger.log('Initializing product cache');
    await this.cacheService.warmCache();
  }

  // TIMEOUT: Collect initial metrics
  @Timeout('initialMetrics', 5000)
  async collectInitialMetrics() {
    this.logger.log('Collecting initial metrics');
    await this.metricsService.recordStartupMetrics();
  }
}

// BENEFITS OF COMBINING:
// - Cron for calendar-based tasks (daily, monthly)
// - Interval for continuous monitoring (every minute, every 5 minutes)
// - Timeout for one-time initialization (cache warming, metrics)
// - All three types work together seamlessly
```

**Key Takeaway:** `@nestjs/schedule` provides **three types of scheduled tasks**: **Cron jobs** with `@Cron()` for **calendar-based scheduling** at specific times/dates using cron expressions (`@Cron('0 0 * * *')` midnight daily, `@Cron('0 9 * * 1-5')` weekdays at 9 AM, `@Cron(CronExpression.EVERY_DAY_AT_2AM)` using enum), best for specific times (2 AM daily, 1st of month, weekday mornings) and business logic tied to calendar (daily cleanup, monthly reports, weekday reminders). **Intervals** with `@Interval(milliseconds)` for **fixed-frequency execution** every N milliseconds (`@Interval(5000)` every 5 seconds, `@Interval(60000)` every minute), runs immediately on startup then every interval continuously, best for health checks (every 30 seconds), monitoring (every minute), polling (every 5 minutes), and tasks not tied to specific time. **Timeouts** with `@Timeout(milliseconds)` for **one-time delayed execution** after N milliseconds from startup (`@Timeout(5000)` runs once after 5 seconds, `@Timeout(10000)` after 10 seconds), runs once never again, best for initialization (cache warming, connection setup), delayed startup tasks (wait for app ready), and one-off scheduled operations. **All three support**: named tasks with `@Cron('expression', { name: 'jobName' })` or `@Interval('jobName', milliseconds)` for dynamic management via SchedulerRegistry, full NestJS DI (inject services, repositories, config, logger), timezone configuration with `timeZone: 'America/New_York'` option, can be added/removed dynamically at runtime. **When to use**: Cron for calendar-based (specific times, dates, business hours), Interval for fixed-frequency (continuous monitoring, polling, health checks), Timeout for one-time initialization (startup tasks, cache warming).

</details>

## Cron Jobs

<details>
<summary><strong>4. What are cron jobs?</strong></summary>

**Answer:**

**Cron jobs** are **scheduled tasks** that execute automatically at **specific times or dates** defined by **cron expressions** (time pattern syntax). The name comes from **Unix cron daemon** (chronos = time in Greek) which has been scheduling tasks since 1970s. A cron expression is a **string with 5-6 fields** representing minute, hour, day, month, and day of week (optionally seconds): `0 0 * * *` means "run at minute 0, hour 0, any day, any month, any day of week" = midnight daily. In NestJS, use `@Cron()` decorator with cron expression or `CronExpression` enum constants. **Benefits**: **human-readable schedules** ("every Monday", "1st of month", "weekdays at 9 AM"), **calendar-aware** (knows weekdays vs weekends, month boundaries, timezone), **precise timing** (runs at exact time specified), **flexible patterns** (every 15 minutes, hourly on weekdays, quarterly, etc.). Common patterns: `0 * * * *` (hourly), `0 0 * * *` (daily midnight), `0 9 * * 1-5` (weekdays 9 AM), `*/15 * * * *` (every 15 minutes), `0 0 1 * *` (1st of month). Used for: scheduled reports, data cleanup, backups, batch processing, sending notifications at specific times.

---

### **Cron Jobs Fundamentals:**

```
CRON EXPRESSION ANATOMY:
┌─────────────────────────────────────────────────────┐
│  STANDARD CRON (5 fields)                           │
│  ────────────────────────                           │
│                                                      │
│    *    *    *    *    *                            │
│    │    │    │    │    │                            │
│    │    │    │    │    └─── Day of Week (0-6)      │
│    │    │    │    │         (0=Sunday, 1=Monday)    │
│    │    │    │    └──────── Month (1-12)            │
│    │    │    └───────────── Day of Month (1-31)     │
│    │    └────────────────── Hour (0-23)             │
│    └─────────────────────── Minute (0-59)           │
│                                                      │
│  EXTENDED CRON (6 fields) - with seconds            │
│  ─────────────────────────────────────              │
│                                                      │
│    *    *    *    *    *    *                       │
│    │    │    │    │    │    │                       │
│    │    │    │    │    │    └─ Day of Week (0-6)   │
│    │    │    │    │    └────── Month (1-12)         │
│    │    │    │    └─────────── Day of Month (1-31)  │
│    │    │    └──────────────── Hour (0-23)          │
│    │    └───────────────────── Minute (0-59)        │
│    └────────────────────────── Second (0-59)        │
└─────────────────────────────────────────────────────┘

SPECIAL CHARACTERS:
─────────────────
*   → Any value ("every")
,   → List ("1,3,5" = 1, 3, and 5)
-   → Range ("1-5" = 1, 2, 3, 4, 5)
/   → Step ("*/15" = every 15 units)

EXAMPLES:
────────
0 0 * * *         → Midnight daily (00:00)
0 9 * * 1-5       → 9 AM weekdays (Mon-Fri)
*/15 * * * *      → Every 15 minutes (0,15,30,45)
0 0 1 * *         → 1st of month at midnight
0 0,12 * * *      → Midnight and noon daily
0 9-17 * * 1-5    → Every hour 9AM-5PM weekdays
```

---

### **Method 1: Basic Cron Job Implementation**

```typescript
// ========== BASIC CRON JOB WITH @Cron() ==========

import { Injectable, Logger } from '@nestjs/common';
import { Cron } from '@nestjs/schedule';

@Injectable()
export class BasicCronService {
  private readonly logger = new Logger(BasicCronService.name);

  // Run at midnight every day (00:00:00)
  @Cron('0 0 * * *')
  handleMidnightJob() {
    this.logger.log('Midnight cron job executing');
    this.logger.log(`Current time: ${new Date().toISOString()}`);
    
    // Task logic here
    // Example: Delete old records, generate reports, etc.
  }
}

// CRON EXPRESSION BREAKDOWN: '0 0 * * *'
// ┌─── Minute: 0 (at minute 0)
// │ ┌─── Hour: 0 (at hour 0 = midnight)
// │ │ ┌─── Day of Month: * (every day)
// │ │ │ ┌─── Month: * (every month)
// │ │ │ │ ┌─── Day of Week: * (every day of week)
// │ │ │ │ │
// 0 0 * * *
//
// Result: Runs at 00:00:00 every day

// EXECUTION TIMELINE:
// Jan 1, 2024 00:00:00 → ✅ Runs
// Jan 1, 2024 00:00:01 → ❌ Doesn't run
// Jan 1, 2024 12:00:00 → ❌ Doesn't run
// Jan 2, 2024 00:00:00 → ✅ Runs
// Jan 3, 2024 00:00:00 → ✅ Runs
// ...
```

---

### **Method 2: Using CronExpression Enum (Recommended)**

```typescript
// ========== CronExpression ENUM ==========

import { Cron, CronExpression } from '@nestjs/schedule';

@Injectable()
export class CronExpressionService {
  private readonly logger = new Logger(CronExpressionService.name);

  // Method 1: Every second (testing only)
  @Cron(CronExpression.EVERY_SECOND)
  everySecond() {
    this.logger.log('Running every second');
  }

  // Method 2: Every 5 seconds
  @Cron(CronExpression.EVERY_5_SECONDS)
  everyFiveSeconds() {
    this.logger.log('Running every 5 seconds');
  }

  // Method 3: Every minute
  @Cron(CronExpression.EVERY_MINUTE)
  everyMinute() {
    this.logger.log('Running every minute');
  }

  // Method 4: Every 5 minutes
  @Cron(CronExpression.EVERY_5_MINUTES)
  everyFiveMinutes() {
    this.logger.log('Running every 5 minutes');
  }

  // Method 5: Every hour
  @Cron(CronExpression.EVERY_HOUR)
  everyHour() {
    this.logger.log('Running every hour');
  }

  // Method 6: Every day at midnight
  @Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT)
  everyDayMidnight() {
    this.logger.log('Running at midnight');
  }

  // Method 7: Every day at 1 AM
  @Cron(CronExpression.EVERY_DAY_AT_1AM)
  everyDayAt1AM() {
    this.logger.log('Running at 1 AM');
  }

  // Method 8: Every day at 2 AM (common for backups)
  @Cron(CronExpression.EVERY_DAY_AT_2AM)
  everyDayAt2AM() {
    this.logger.log('Running at 2 AM');
    // Ideal for database backups, heavy processing
  }

  // Method 9: Every day at noon
  @Cron(CronExpression.EVERY_DAY_AT_NOON)
  everyDayNoon() {
    this.logger.log('Running at noon');
  }

  // Method 10: Every week (Sunday at midnight)
  @Cron(CronExpression.EVERY_WEEK)
  everyWeek() {
    this.logger.log('Running weekly on Sunday');
  }

  // Method 11: Every month (1st at midnight)
  @Cron(CronExpression.EVERY_MONTH)
  everyMonth() {
    this.logger.log('Running monthly on 1st');
  }

  // Method 12: Every year (Jan 1st at midnight)
  @Cron(CronExpression.EVERY_YEAR)
  everyYear() {
    this.logger.log('Running yearly on Jan 1st');
  }
}

// BENEFITS OF CronExpression ENUM:
// ✅ Type-safe (compile-time checking)
// ✅ Self-documenting (clear what schedule means)
// ✅ No syntax errors (predefined valid expressions)
// ✅ IDE autocomplete support
// ✅ Easy to read and maintain
```

---

### **Method 3: Custom Cron Patterns**

```typescript
// ========== CUSTOM CRON PATTERNS ==========

@Injectable()
export class CustomCronService {
  private readonly logger = new Logger(CustomCronService.name);

  // Every 15 minutes (0, 15, 30, 45)
  @Cron('*/15 * * * *')
  every15Minutes() {
    this.logger.log('Running every 15 minutes');
    // Sync with external API, refresh cache
  }

  // Weekdays at 9 AM (Monday-Friday)
  @Cron('0 9 * * 1-5')
  weekdayMornings() {
    this.logger.log('Running weekday mornings at 9 AM');
    // Send daily standup reminders
  }

  // First day of every month at midnight
  @Cron('0 0 1 * *')
  firstOfMonth() {
    this.logger.log('Running on 1st of month');
    // Generate monthly reports, process subscriptions
  }

  // Every hour during business hours (9 AM - 5 PM, weekdays)
  @Cron('0 9-17 * * 1-5')
  businessHours() {
    this.logger.log('Running during business hours');
    // Check support tickets, monitor SLA
  }

  // Every Monday at 8 AM
  @Cron('0 8 * * 1')
  mondayMorning() {
    this.logger.log('Running Monday morning');
    // Send weekly summary email
  }

  // Midnight and noon every day
  @Cron('0 0,12 * * *')
  twiceDaily() {
    this.logger.log('Running twice daily (midnight and noon)');
    // Health check reports
  }

  // Every 30 minutes during peak hours (9 AM - 6 PM)
  @Cron('*/30 9-18 * * *')
  peakHours() {
    this.logger.log('Running every 30 min during peak hours');
    // Scale resources, monitor performance
  }

  // Quarterly: 1st day of Jan, Apr, Jul, Oct at midnight
  @Cron('0 0 1 1,4,7,10 *')
  quarterly() {
    this.logger.log('Running quarterly');
    // Quarterly business reports
  }
}

// CRON SYNTAX TIPS:
// */N  → Every N units (*/15 = every 15)
// 1-5  → Range (Monday to Friday)
// 1,3,5 → Specific values (Monday, Wednesday, Friday)
// 9-17 → Business hours (9 AM to 5 PM)
```

---

### **Method 4: Cron Jobs with Options**

```typescript
// ========== CRON OPTIONS ==========

@Injectable()
export class CronOptionsService {
  private readonly logger = new Logger(CronOptionsService.name);

  // Named cron job (for dynamic management)
  @Cron('0 0 * * *', {
    name: 'dailyBackup',  // Reference this job by name
  })
  namedJob() {
    this.logger.log('Named cron job executing');
  }

  // Timezone-aware cron job
  @Cron('0 9 * * *', {
    timeZone: 'America/New_York',  // EST/EDT
  })
  easternTime() {
    this.logger.log('Running at 9 AM Eastern Time');
    // Runs at 9 AM in New York, regardless of server timezone
  }

  // Multiple timezones for different regions
  @Cron('0 9 * * *', {
    name: 'morningReportPST',
    timeZone: 'America/Los_Angeles',  // PST/PDT
  })
  pacificTime() {
    this.logger.log('Running at 9 AM Pacific Time');
  }

  @Cron('0 9 * * *', {
    name: 'morningReportGMT',
    timeZone: 'Europe/London',  // GMT/BST
  })
  londonTime() {
    this.logger.log('Running at 9 AM London Time');
  }

  // Disabled cron job (won't run)
  @Cron('0 0 * * *', {
    disabled: true,  // Job defined but not active
  })
  disabledJob() {
    this.logger.log('This will not run');
  }
}

// TIMEZONE CONSIDERATIONS:
// - Always specify timezone if job must run at specific local time
// - Without timezone, uses server timezone
// - Useful for: business hours, regional notifications, compliance
// - Common timezones:
//   • America/New_York (EST/EDT)
//   • America/Los_Angeles (PST/PDT)
//   • America/Chicago (CST/CDT)
//   • Europe/London (GMT/BST)
//   • Asia/Tokyo (JST)
//   • UTC (no DST)
```

---

### **Method 5: Real-World Cron Job Examples**

```typescript
// ========== PRODUCTION CRON JOBS ==========

@Injectable()
export class ProductionCronService {
  private readonly logger = new Logger(ProductionCronService.name);

  constructor(
    private readonly ordersRepository: OrdersRepository,
    private readonly backupService: BackupService,
    private readonly emailService: EmailService,
    private readonly reportsService: ReportsService,
  ) {}

  // Database backup at 2 AM daily (off-peak hours)
  @Cron(CronExpression.EVERY_DAY_AT_2AM, {
    name: 'databaseBackup',
  })
  async backupDatabase() {
    this.logger.log('Starting database backup');
    
    try {
      const backupFile = await this.backupService.createBackup();
      this.logger.log(`Backup completed: ${backupFile}`);
    } catch (error) {
      this.logger.error('Backup failed', error.stack);
      // Send alert to ops team
    }
  }

  // Cancel expired orders every hour
  @Cron(CronExpression.EVERY_HOUR, {
    name: 'cancelExpiredOrders',
  })
  async cancelExpiredOrders() {
    this.logger.log('Cancelling expired orders');
    
    const oneHourAgo = new Date();
    oneHourAgo.setHours(oneHourAgo.getHours() - 1);
    
    const expiredOrders = await this.ordersRepository.find({
      where: {
        status: 'pending_payment',
        createdAt: LessThan(oneHourAgo),
      },
    });
    
    for (const order of expiredOrders) {
      await this.ordersRepository.update(order.id, {
        status: 'expired',
      });
      
      // Notify customer
      await this.emailService.sendOrderExpired(order);
    }
    
    this.logger.log(`Cancelled ${expiredOrders.length} expired orders`);
  }

  // Monthly reports on 1st at midnight
  @Cron('0 0 1 * *', {
    name: 'monthlyReport',
    timeZone: 'America/New_York',
  })
  async generateMonthlyReport() {
    this.logger.log('Generating monthly report');
    
    const report = await this.reportsService.generateMonthlySalesReport();
    
    await this.emailService.sendEmail({
      to: 'management@company.com',
      subject: 'Monthly Sales Report',
      attachments: [report],
    });
    
    this.logger.log('Monthly report sent');
  }

  // Send reminder emails every weekday at 9 AM
  @Cron('0 9 * * 1-5', {
    name: 'dailyReminders',
    timeZone: 'America/New_York',
  })
  async sendDailyReminders() {
    this.logger.log('Sending daily reminders');
    
    // Send standup reminders, task notifications, etc.
  }
}

// BEST PRACTICES:
// ✅ Use descriptive names for jobs
// ✅ Add try-catch for error handling
// ✅ Log execution start and end
// ✅ Set appropriate timezone
// ✅ Run heavy tasks during off-peak hours (2-4 AM)
// ✅ Monitor job execution (failures, duration)
```

**Key Takeaway:** **Cron jobs** are **scheduled tasks** executing automatically at **specific times/dates** using **cron expressions** (time pattern syntax from Unix cron daemon). A cron expression has **5-6 fields**: `minute hour day month dayOfWeek` optionally with `second` field (`0 0 * * *` = midnight daily, `0 9 * * 1-5` = weekdays 9 AM, `*/15 * * * *` = every 15 minutes). **Special characters**: `*` for any value (every), `-` for ranges (`1-5` = Monday-Friday), `,` for lists (`1,3,5` = Mon/Wed/Fri), `/` for steps (`*/15` = every 15 units). In NestJS use `@Cron()` decorator with expression string or `CronExpression` enum (recommended for type-safety): `@Cron(CronExpression.EVERY_DAY_AT_2AM)`, `@Cron(CronExpression.EVERY_HOUR)`, `@Cron('0 9 * * 1-5')` for custom patterns. **Options**: `name` for dynamic management (`{ name: 'dailyBackup' }`), `timeZone` for timezone-aware execution (`{ timeZone: 'America/New_York' }` runs at specific local time regardless of server timezone), `disabled` to define but not activate job. **Common patterns**: hourly (`0 * * * *`), daily midnight (`0 0 * * *`), weekdays 9 AM (`0 9 * * 1-5`), every 15 minutes (`*/15 * * * *`), 1st of month (`0 0 1 * *`), business hours (`0 9-17 * * 1-5`), quarterly (`0 0 1 1,4,7,10 *`). **Use cases**: scheduled reports (monthly on 1st), data cleanup (daily at 2 AM), backups (nightly during off-peak), batch processing (hourly), notifications at specific times (weekday mornings). **Benefits**: human-readable schedules, calendar-aware (weekdays/weekends/months), precise timing, flexible patterns, timezone support.

</details>

<details>
<summary><strong>5. How do you create a cron job using `@Cron()` decorator?</strong></summary>

**Answer:**

Create a cron job by adding `@Cron()` decorator above a method in a **service class** that's registered as a provider. Steps: **1) Install** `@nestjs/schedule` (`npm install @nestjs/schedule`), **2) Import** `ScheduleModule.forRoot()` in AppModule, **3) Create service** with `@Injectable()`, **4) Add method** with `@Cron(expression)` decorator passing cron expression or `CronExpression` enum, **5) Register service** in module providers. The decorator accepts: **cron expression string** (`@Cron('0 0 * * *')`), **CronExpression enum** (`@Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT)`), **options object** (`@Cron('0 0 * * *', { name: 'backup', timeZone: 'America/New_York' })`). The method can be **async** (return Promise), has access to **dependency injection** (inject repositories, services via constructor), can use **Logger** for monitoring, and supports **error handling** with try-catch. Options include: `name` (string identifier for dynamic management), `timeZone` (IANA timezone like 'America/New_York'), `disabled` (boolean to define but not activate). The cron job starts automatically when application starts and runs based on schedule until app stops.

---

### **Creating Cron Jobs - Complete Setup:**

```
SETUP FLOW:
┌────────────────────────────────────────────────────┐
│  1. Install Package                                │
│     npm install @nestjs/schedule                   │
└─────────────────┬──────────────────────────────────┘
                  ↓
┌────────────────────────────────────────────────────┐
│  2. Enable Scheduling in AppModule                 │
│     ScheduleModule.forRoot()                       │
└─────────────────┬──────────────────────────────────┘
                  ↓
┌────────────────────────────────────────────────────┐
│  3. Create Service with @Injectable()              │
│     Mark class as injectable provider              │
└─────────────────┬──────────────────────────────────┘
                  ↓
┌────────────────────────────────────────────────────┐
│  4. Add Method with @Cron() Decorator              │
│     @Cron('0 0 * * *')                             │
│     handleCron() { }                               │
└─────────────────┬──────────────────────────────────┘
                  ↓
┌────────────────────────────────────────────────────┐
│  5. Register Service in Module                     │
│     providers: [MyScheduledService]                │
└─────────────────┬──────────────────────────────────┘
                  ↓
┌────────────────────────────────────────────────────┐
│  6. App Starts → Cron Job Runs Automatically       │
│     Scheduler detects @Cron() and registers job    │
└────────────────────────────────────────────────────┘
```

---

### **Method 1: Basic Cron Job Setup**

```typescript
// ========== STEP 1: INSTALL PACKAGE ==========
// Terminal command:
// npm install @nestjs/schedule

// ========== STEP 2: ENABLE SCHEDULING ==========
// src/app.module.ts
import { Module } from '@nestjs/common';
import { ScheduleModule } from '@nestjs/schedule';
import { TasksModule } from './features/tasks/tasks.module';

@Module({
  imports: [
    // Enable scheduling globally
    ScheduleModule.forRoot(),
    
    // Import modules with scheduled tasks
    TasksModule,
  ],
})
export class AppModule {}

// ========== STEP 3: CREATE SERVICE ==========
// src/features/tasks/tasks.service.ts
import { Injectable, Logger } from '@nestjs/common';
import { Cron } from '@nestjs/schedule';

@Injectable()
export class TasksService {
  private readonly logger = new Logger(TasksService.name);

  // ========== STEP 4: ADD @Cron() DECORATOR ==========
  @Cron('0 0 * * *')  // Midnight every day
  handleDailyCron() {
    this.logger.log('Daily cron job executed');
    // Your task logic here
  }
}

// ========== STEP 5: REGISTER SERVICE ==========
// src/features/tasks/tasks.module.ts
import { Module } from '@nestjs/common';
import { TasksService } from './tasks.service';

@Module({
  providers: [TasksService],  // Register as provider
})
export class TasksModule {}

// That's it! The cron job will run automatically at midnight daily
```

---

### **Method 2: Using CronExpression Enum (Recommended)**

```typescript
// ========== CronExpression ENUM FOR TYPE SAFETY ==========

import { Injectable, Logger } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';

@Injectable()
export class ScheduledTasksService {
  private readonly logger = new Logger(ScheduledTasksService.name);

  // Method 1: Every minute (for testing)
  @Cron(CronExpression.EVERY_MINUTE)
  everyMinute() {
    this.logger.log('Runs every minute');
  }

  // Method 2: Every 5 minutes
  @Cron(CronExpression.EVERY_5_MINUTES)
  everyFiveMinutes() {
    this.logger.log('Runs every 5 minutes');
  }

  // Method 3: Every hour
  @Cron(CronExpression.EVERY_HOUR)
  everyHour() {
    this.logger.log('Runs every hour');
  }

  // Method 4: Daily at midnight
  @Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT)
  dailyMidnight() {
    this.logger.log('Runs at midnight daily');
  }

  // Method 5: Daily at 2 AM
  @Cron(CronExpression.EVERY_DAY_AT_2AM)
  dailyAt2AM() {
    this.logger.log('Runs at 2 AM daily');
  }

  // Method 6: Weekly (Sunday at midnight)
  @Cron(CronExpression.EVERY_WEEK)
  weekly() {
    this.logger.log('Runs weekly on Sunday');
  }

  // Method 7: Monthly (1st at midnight)
  @Cron(CronExpression.EVERY_MONTH)
  monthly() {
    this.logger.log('Runs monthly on 1st');
  }
}

// BENEFITS OF CronExpression ENUM:
// ✅ Type-safe (compile-time checking)
// ✅ Autocomplete in IDE
// ✅ Self-documenting code
// ✅ No syntax errors
// ✅ Easy to maintain
```

---

### **Method 3: Custom Cron Expressions**

```typescript
// ========== CUSTOM CRON PATTERNS ==========

@Injectable()
export class CustomCronService {
  private readonly logger = new Logger(CustomCronService.name);

  // Every 15 minutes (0, 15, 30, 45)
  @Cron('*/15 * * * *')
  every15Minutes() {
    this.logger.log('Runs every 15 minutes');
  }

  // Weekdays at 9 AM (Monday-Friday)
  @Cron('0 9 * * 1-5')
  weekdayMornings() {
    this.logger.log('Runs weekdays at 9 AM');
  }

  // Business hours: every hour from 9 AM to 5 PM on weekdays
  @Cron('0 9-17 * * 1-5')
  businessHours() {
    this.logger.log('Runs every hour during business hours');
  }

  // First day of every month at midnight
  @Cron('0 0 1 * *')
  firstOfMonth() {
    this.logger.log('Runs on 1st of month');
  }

  // Every Monday at 8 AM
  @Cron('0 8 * * 1')
  mondayMorning() {
    this.logger.log('Runs Monday morning');
  }

  // Twice daily: midnight and noon
  @Cron('0 0,12 * * *')
  twiceDaily() {
    this.logger.log('Runs at midnight and noon');
  }

  // Every 30 minutes during peak hours (9 AM - 6 PM)
  @Cron('*/30 9-18 * * *')
  peakHours() {
    this.logger.log('Runs every 30 min during peak hours');
  }

  // Quarterly: Jan 1, Apr 1, Jul 1, Oct 1 at midnight
  @Cron('0 0 1 1,4,7,10 *')
  quarterly() {
    this.logger.log('Runs quarterly');
  }
}
```

---

### **Method 4: Cron Jobs with Options**

```typescript
// ========== CRON OPTIONS ==========

@Injectable()
export class CronOptionsService {
  private readonly logger = new Logger(CronOptionsService.name);

  // Option 1: Named cron job (for dynamic management)
  @Cron('0 0 * * *', {
    name: 'dailyBackup',  // Identifier for SchedulerRegistry
  })
  namedCronJob() {
    this.logger.log('Named cron job executing');
  }

  // Option 2: Timezone-aware cron job
  @Cron('0 9 * * *', {
    name: 'morningReport',
    timeZone: 'America/New_York',  // EST/EDT timezone
  })
  easternTime() {
    this.logger.log('Runs at 9 AM Eastern Time');
    // Always runs at 9 AM in New York, regardless of server timezone
  }

  // Option 3: Pacific Time
  @Cron('0 9 * * 1-5', {
    name: 'weekdayAlertPST',
    timeZone: 'America/Los_Angeles',
  })
  pacificTime() {
    this.logger.log('Runs at 9 AM Pacific Time on weekdays');
  }

  // Option 4: London Time
  @Cron('0 17 * * 1-5', {
    name: 'endOfDayUK',
    timeZone: 'Europe/London',
  })
  londonTime() {
    this.logger.log('Runs at 5 PM London time on weekdays');
  }

  // Option 5: Disabled cron job (defined but not active)
  @Cron('0 0 * * *', {
    name: 'disabledJob',
    disabled: true,  // Job won't run
  })
  disabledJob() {
    this.logger.log('This will not execute');
  }

  // Option 6: UTC timezone (no DST)
  @Cron('0 0 * * *', {
    timeZone: 'UTC',
  })
  utcTime() {
    this.logger.log('Runs at midnight UTC');
  }
}

// COMMON IANA TIMEZONES:
// - America/New_York (EST/EDT)
// - America/Los_Angeles (PST/PDT)
// - America/Chicago (CST/CDT)
// - Europe/London (GMT/BST)
// - Europe/Paris (CET/CEST)
// - Asia/Tokyo (JST)
// - Australia/Sydney (AEST/AEDT)
// - UTC (no daylight saving)
```

---

### **Method 5: Async Cron Jobs with Dependency Injection**

```typescript
// ========== ASYNC CRON JOBS WITH DI ==========

@Injectable()
export class DataCleanupService {
  private readonly logger = new Logger(DataCleanupService.name);

  // Inject dependencies via constructor
  constructor(
    private readonly usersRepository: UsersRepository,
    private readonly logsRepository: LogsRepository,
    private readonly configService: ConfigService,
    private readonly emailService: EmailService,
  ) {}

  // Async cron job (returns Promise)
  @Cron(CronExpression.EVERY_DAY_AT_2AM, {
    name: 'dataCleanup',
    timeZone: 'America/New_York',
  })
  async cleanupOldData() {
    this.logger.log('Starting data cleanup job');

    try {
      // Access configuration
      const retentionDays = this.configService.get<number>('DATA_RETENTION_DAYS', 30);
      const cutoffDate = new Date();
      cutoffDate.setDate(cutoffDate.getDate() - retentionDays);

      // Delete old logs
      const logsResult = await this.logsRepository.delete({
        createdAt: LessThan(cutoffDate),
      });
      this.logger.log(`Deleted ${logsResult.affected} old logs`);

      // Delete inactive users
      const usersResult = await this.usersRepository.delete({
        isActive: false,
        lastLoginAt: LessThan(cutoffDate),
      });
      this.logger.log(`Deleted ${usersResult.affected} inactive users`);

      // Send summary email
      await this.emailService.sendEmail({
        to: 'admin@company.com',
        subject: 'Daily Cleanup Report',
        html: `
          <h2>Data Cleanup Summary</h2>
          <p>Logs deleted: ${logsResult.affected}</p>
          <p>Users deleted: ${usersResult.affected}</p>
        `,
      });

      this.logger.log('Data cleanup completed successfully');
      
    } catch (error) {
      this.logger.error('Data cleanup failed', error.stack);
      
      // Send alert on failure
      await this.emailService.sendEmail({
        to: 'ops@company.com',
        subject: 'ALERT: Data Cleanup Failed',
        html: `<p>Error: ${error.message}</p>`,
      });
    }
  }
}

// BENEFITS:
// ✅ Async/await support
// ✅ Full dependency injection
// ✅ Access to repositories, services, config
// ✅ Error handling with try-catch
// ✅ Logging for monitoring
```

---

### **Method 6: Multiple Cron Jobs in One Service**

```typescript
// ========== MULTIPLE CRON JOBS ==========

@Injectable()
export class EcommerceSchedulerService {
  private readonly logger = new Logger(EcommerceSchedulerService.name);

  constructor(
    private readonly ordersRepository: OrdersRepository,
    private readonly inventoryService: InventoryService,
    private readonly reportsService: ReportsService,
    private readonly cacheService: CacheService,
  ) {}

  // Cron 1: Hourly - Cancel expired orders
  @Cron(CronExpression.EVERY_HOUR, { name: 'cancelExpiredOrders' })
  async cancelExpiredOrders() {
    this.logger.log('Cancelling expired orders');
    
    const oneHourAgo = new Date();
    oneHourAgo.setHours(oneHourAgo.getHours() - 1);
    
    const expired = await this.ordersRepository.find({
      where: { status: 'pending', createdAt: LessThan(oneHourAgo) },
    });
    
    for (const order of expired) {
      await this.ordersRepository.update(order.id, { status: 'expired' });
    }
    
    this.logger.log(`Cancelled ${expired.length} expired orders`);
  }

  // Cron 2: Every 15 minutes - Sync inventory
  @Cron('*/15 * * * *', { name: 'syncInventory' })
  async syncInventory() {
    this.logger.log('Syncing inventory with warehouse');
    await this.inventoryService.syncWithWarehouse();
  }

  // Cron 3: Daily at 2 AM - Generate reports
  @Cron(CronExpression.EVERY_DAY_AT_2AM, {
    name: 'dailyReports',
    timeZone: 'America/New_York',
  })
  async generateDailyReports() {
    this.logger.log('Generating daily reports');
    await this.reportsService.generateDailySalesReport();
  }

  // Cron 4: Monthly - Process subscriptions
  @Cron('0 0 1 * *', {
    name: 'monthlySubscriptions',
    timeZone: 'America/New_York',
  })
  async processMonthlySubscriptions() {
    this.logger.log('Processing monthly subscriptions');
    // Charge recurring fees, send invoices
  }

  // Cron 5: Every 5 minutes - Refresh cache
  @Cron(CronExpression.EVERY_5_MINUTES, { name: 'refreshCache' })
  async refreshCache() {
    this.logger.log('Refreshing product cache');
    await this.cacheService.refreshTopProducts();
  }
}

// ALL CRON JOBS COEXIST IN SAME SERVICE:
// - Each runs on its own schedule
// - Share same dependencies (DI)
// - Independent execution
// - Named for dynamic management
```

**Key Takeaway:** Create a cron job by adding `@Cron()` decorator above a method in an `@Injectable()` service class. **Setup steps**: install `@nestjs/schedule` package (`npm install @nestjs/schedule`), import `ScheduleModule.forRoot()` in AppModule (enables scheduling globally and scans for decorated methods), create service with `@Injectable()` decorator (marks as provider), add method with `@Cron(expression)` decorator passing cron expression string or `CronExpression` enum constant, register service in module's `providers` array. **Decorator syntax**: `@Cron('0 0 * * *')` with cron expression string (midnight daily), `@Cron(CronExpression.EVERY_DAY_AT_2AM)` with enum (recommended for type-safety), `@Cron('0 0 * * *', { name: 'backup', timeZone: 'America/New_York' })` with options object. **Options**: `name` string identifier for dynamic management via SchedulerRegistry (add/remove jobs at runtime), `timeZone` IANA timezone like 'America/New_York' or 'Europe/London' (runs at specific local time regardless of server timezone), `disabled` boolean to define but not activate job. **Method features**: can be **async** returning Promise for database/API operations, has full **dependency injection** access (inject repositories, services, ConfigService, HttpService via constructor), supports **error handling** with try-catch for resilient execution, use **Logger** for monitoring execution and failures. **Execution**: cron job starts automatically when application starts (ScheduleModule scans for @Cron decorators on startup), runs based on cron schedule until app stops, method called with no parameters automatically by scheduler. **Multiple jobs**: one service can have multiple `@Cron()` methods each with different schedule (hourly, daily, weekly, monthly), all share same injected dependencies, execute independently on their own schedules, can be named uniquely for management.

</details>

<details>
<summary><strong>6. What is cron syntax and how do you read it?</strong></summary>

**Answer:**

**Cron syntax** is a **time pattern format** with **5 or 6 fields** separated by spaces representing when a task should run: `second minute hour day month dayOfWeek` (6 fields) or `minute hour day month dayOfWeek` (5 fields - standard Unix cron). Each field accepts: **specific values** (0-59 for minutes, 0-23 for hours, 1-31 for days, 1-12 for months, 0-6 for day of week where 0=Sunday), **wildcards** (`*` = any value), **ranges** (`1-5` = Monday to Friday), **lists** (`1,3,5` = Monday, Wednesday, Friday), **steps** (`*/15` = every 15 units). **Reading examples**: `0 0 * * *` = "at minute 0, hour 0, any day, any month, any day of week" = midnight daily. `0 9 * * 1-5` = "at minute 0, hour 9, any day, any month, Monday-Friday" = 9 AM weekdays. `*/15 * * * *` = "every 15 minutes, any hour, any day, any month, any day of week" = every 15 minutes. `0 0 1 * *` = "at minute 0, hour 0, day 1, any month, any day of week" = 1st of month at midnight. **Special characters**: `*` (any), `-` (range), `,` (list), `/` (step), `?` (no specific value in some implementations). **Tips**: read left to right (minute → hour → day → month → dayOfWeek), `*` means "every" or "any", ranges are inclusive (1-5 includes 1,2,3,4,5), steps apply to the field range (`*/15` in minute field = 0,15,30,45).

---

### **Cron Syntax Breakdown:**

```
STANDARD CRON SYNTAX (5 fields):
┌─────────────────────────────────────────────────────┐
│                                                     │
│         *    *    *    *    *                       │
│         │    │    │    │    │                       │
│         │    │    │    │    └─── Day of Week (0-6) │
│         │    │    │    │         (0=Sunday)         │
│         │    │    │    └──────── Month (1-12)       │
│         │    │    │               (1=January)        │
│         │    │    └───────────── Day of Month(1-31) │
│         │    └────────────────── Hour (0-23)        │
│         └─────────────────────── Minute (0-59)      │
│                                                     │
└─────────────────────────────────────────────────────┘

EXTENDED CRON SYNTAX (6 fields) - NestJS supports:
┌─────────────────────────────────────────────────────┐
│                                                     │
│      *    *    *    *    *    *                    │
│      │    │    │    │    │    │                    │
│      │    │    │    │    │    └─ Day of Week (0-6)│
│      │    │    │    │    └────── Month (1-12)      │
│      │    │    │    └─────────── Day of Month(1-31)│
│      │    │    └──────────────── Hour (0-23)       │
│      │    └───────────────────── Minute (0-59)     │
│      └────────────────────────── Second (0-59)     │
│                                                     │
└─────────────────────────────────────────────────────┘

FIELD RANGES:
┌──────────────┬──────────┬─────────────────────────┐
│ Field        │ Range    │ Special Values          │
├──────────────┼──────────┼─────────────────────────┤
│ Second       │ 0-59     │ * , - /                 │
│ Minute       │ 0-59     │ * , - /                 │
│ Hour         │ 0-23     │ * , - /                 │
│ Day of Month │ 1-31     │ * , - /                 │
│ Month        │ 1-12     │ * , - / JAN-DEC         │
│ Day of Week  │ 0-6      │ * , - / SUN-SAT         │
│              │          │ (0=Sunday, 6=Saturday)  │
└──────────────┴──────────┴─────────────────────────┘

SPECIAL CHARACTERS:
┌──────┬───────────────────────────────────────────┐
│ Char │ Meaning                                   │
├──────┼───────────────────────────────────────────┤
│  *   │ Any value (every)                         │
│  ,   │ List of values (1,3,5 = 1 or 3 or 5)     │
│  -   │ Range (1-5 = 1,2,3,4,5)                   │
│  /   │ Step (*/15 = every 15 units)              │
└──────┴───────────────────────────────────────────┘
```

---

### **Method 1: Reading Basic Cron Expressions**

```typescript
// ========== READING CRON EXPRESSIONS STEP BY STEP ==========

// Example 1: '0 0 * * *'
// Position: [minute] [hour] [day] [month] [dayOfWeek]
//              0       0      *     *        *
// 
// Breakdown:
// - Minute: 0 (at minute 0)
// - Hour: 0 (at hour 0 = midnight)
// - Day: * (any day of month)
// - Month: * (any month)
// - Day of Week: * (any day of week)
//
// Result: Runs at midnight (00:00) every day

@Cron('0 0 * * *')
midnightDaily() {
  // Executes at: 00:00:00 every day
}

// Example 2: '0 9 * * 1-5'
// Position: [minute] [hour] [day] [month] [dayOfWeek]
//              0       9      *     *       1-5
//
// Breakdown:
// - Minute: 0 (at minute 0)
// - Hour: 9 (at 9 AM)
// - Day: * (any day of month)
// - Month: * (any month)
// - Day of Week: 1-5 (Monday to Friday)
//
// Result: Runs at 9 AM on weekdays (Monday-Friday)

@Cron('0 9 * * 1-5')
weekdayMornings() {
  // Executes at: 09:00:00 Monday-Friday
}

// Example 3: '*/15 * * * *'
// Position: [minute] [hour] [day] [month] [dayOfWeek]
//            */15      *      *     *        *
//
// Breakdown:
// - Minute: */15 (every 15 minutes: 0, 15, 30, 45)
// - Hour: * (any hour)
// - Day: * (any day)
// - Month: * (any month)
// - Day of Week: * (any day of week)
//
// Result: Runs every 15 minutes (0, 15, 30, 45 minutes of every hour)

@Cron('*/15 * * * *')
every15Minutes() {
  // Executes at: XX:00, XX:15, XX:30, XX:45
}

// Example 4: '0 0 1 * *'
// Position: [minute] [hour] [day] [month] [dayOfWeek]
//              0       0      1     *        *
//
// Breakdown:
// - Minute: 0 (at minute 0)
// - Hour: 0 (at midnight)
// - Day: 1 (1st day of month)
// - Month: * (any month)
// - Day of Week: * (any day of week)
//
// Result: Runs on the 1st of every month at midnight

@Cron('0 0 1 * *')
monthlyFirstDay() {
  // Executes at: 00:00:00 on Jan 1, Feb 1, Mar 1, etc.
}
```

---

### **Method 2: Using Wildcard (*)  - "Any Value"**

```typescript
// ========== WILDCARD (*) = ANY VALUE ==========

// Example 1: Any minute of specific hour
@Cron('* 9 * * *')
everyMinuteAt9AM() {
  // Runs: 09:00, 09:01, 09:02, ..., 09:59
  // Every minute during the 9 AM hour
}

// Example 2: Any hour of specific day
@Cron('0 * 1 * *')
everyHourOnFirstDay() {
  // Runs: 1st of month at 00:00, 01:00, 02:00, ..., 23:00
  // Every hour on the 1st day of month
}

// Example 3: Any day, specific time
@Cron('30 14 * * *')
daily2_30PM() {
  // Runs: 14:30 (2:30 PM) every day
  // Any day of month, any month, any day of week
}

// Example 4: Everything wildcard
@Cron('0 0 * * *')
midnightEveryDay() {
  // Runs: Midnight every day
  // Day: * (any day 1-31)
  // Month: * (any month 1-12)
  // Day of Week: * (any day 0-6)
}

// WILDCARD MEANING:
// - * in minute → every minute (0-59)
// - * in hour → every hour (0-23)
// - * in day → every day (1-31)
// - * in month → every month (1-12)
// - * in dayOfWeek → every day of week (0-6)
```

---

### **Method 3: Using Range (-) - Continuous Values**

```typescript
// ========== RANGE (-) = CONTINUOUS VALUES ==========

// Example 1: Hour range (business hours)
@Cron('0 9-17 * * *')
everyHourBusinessHours() {
  // Runs at: 09:00, 10:00, 11:00, 12:00, 13:00, 14:00, 15:00, 16:00, 17:00
  // Every hour from 9 AM to 5 PM
}

// Example 2: Weekdays (Monday-Friday)
@Cron('0 8 * * 1-5')
weekdaysAt8AM() {
  // Runs at: 08:00 on Monday(1), Tuesday(2), Wednesday(3), Thursday(4), Friday(5)
  // Day of week: 1-5 = Monday to Friday
}

// Example 3: Minute range
@Cron('15-45 * * * *')
minutesQuarter() {
  // Runs at: XX:15, XX:16, XX:17, ..., XX:44, XX:45
  // Every minute from 15 to 45 of every hour
}

// Example 4: Month range (Q1)
@Cron('0 0 1 1-3 *')
quarterlyQ1() {
  // Runs at: Midnight on 1st day of January, February, March
  // Months 1-3 = Jan, Feb, Mar
}

// Example 5: Combined ranges
@Cron('0 9-17 * * 1-5')
businessHoursWeekdays() {
  // Runs at: 09:00-17:00 (every hour) on Monday-Friday
  // Hour: 9-17 (9 AM to 5 PM)
  // Day of Week: 1-5 (Monday to Friday)
}

// RANGE RULES:
// - Inclusive (1-5 includes 1, 2, 3, 4, 5)
// - Can span any valid range
// - Must be ascending (1-5, not 5-1)
```

---

### **Method 4: Using List (,) - Specific Values**

```typescript
// ========== LIST (,) = SPECIFIC VALUES ==========

// Example 1: Specific hours
@Cron('0 0,6,12,18 * * *')
fourTimesDaily() {
  // Runs at: 00:00 (midnight), 06:00 (6 AM), 12:00 (noon), 18:00 (6 PM)
  // Four times per day at specific hours
}

// Example 2: Specific days of week (Mon, Wed, Fri)
@Cron('0 9 * * 1,3,5')
mondayWednesdayFriday() {
  // Runs at: 09:00 on Monday(1), Wednesday(3), Friday(5)
  // Three days per week
}

// Example 3: Specific days of month
@Cron('0 0 1,15 * *')
twiceMonthly() {
  // Runs at: Midnight on 1st and 15th of every month
  // Bi-weekly payroll, reports, etc.
}

// Example 4: Specific months (quarterly)
@Cron('0 0 1 1,4,7,10 *')
quarterly() {
  // Runs at: Midnight on 1st day of January, April, July, October
  // Quarterly reports
}

// Example 5: Combined list and range
@Cron('0 9,12,14-16 * * 1-5')
specificBusinessHours() {
  // Runs at: 09:00, 12:00, 14:00, 15:00, 16:00 on weekdays
  // Hours: 9, 12, and 14-16
  // Days: Monday-Friday
}

// LIST RULES:
// - Comma-separated values
// - Any valid values for the field
// - Can mix with ranges (1-3,5,7-9)
```

---

### **Method 5: Using Step (/) - Intervals**

```typescript
// ========== STEP (/) = INTERVALS ==========

// Example 1: Every 5 minutes
@Cron('*/5 * * * *')
every5Minutes() {
  // Runs at: 00:00, 00:05, 00:10, 00:15, ..., 23:55
  // Minutes: 0, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50, 55
}

// Example 2: Every 15 minutes
@Cron('*/15 * * * *')
every15Minutes() {
  // Runs at: XX:00, XX:15, XX:30, XX:45
  // Minutes: 0, 15, 30, 45
}

// Example 3: Every 2 hours
@Cron('0 */2 * * *')
every2Hours() {
  // Runs at: 00:00, 02:00, 04:00, 06:00, ..., 22:00
  // Hours: 0, 2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22
}

// Example 4: Every 3 hours during business hours
@Cron('0 9-17/3 * * *')
every3HoursBusiness() {
  // Runs at: 09:00, 12:00, 15:00
  // Start at 9, increment by 3, stop at 17
}

// Example 5: Every 10 minutes during peak hours
@Cron('*/10 9-18 * * *')
every10MinutesPeak() {
  // Runs at: 09:00, 09:10, 09:20, ..., 18:00, 18:10, ..., 18:50
  // Every 10 minutes from 9 AM to 6:59 PM
}

// Example 6: Every other day
@Cron('0 0 */2 * *')
everyOtherDay() {
  // Runs at: Midnight on 1st, 3rd, 5th, ..., 31st of month
  // Days: 1, 3, 5, 7, 9, 11, 13, 15, 17, 19, 21, 23, 25, 27, 29, 31
}

// STEP RULES:
// - Format: */N or start-end/N
// - */N starts from min value of field
// - start-end/N steps within range
// - N is the increment
```

---

### **Method 6: Complex Examples - Combining Operators**

```typescript
// ========== COMPLEX CRON EXPRESSIONS ==========

// Example 1: Business hours monitoring (every 30 min, 9-5 PM, weekdays)
@Cron('*/30 9-17 * * 1-5')
businessHoursMonitoring() {
  // Runs at: 09:00, 09:30, 10:00, ..., 17:00, 17:30 on Mon-Fri
  // Combines: step (*/30), range (9-17), range (1-5)
}

// Example 2: Bi-weekly on specific days (1st and 15th at 8 AM)
@Cron('0 8 1,15 * *')
biWeekly() {
  // Runs at: 08:00 on 1st and 15th of every month
  // Combines: list (1,15)
}

// Example 3: Every 15 min during morning rush (7-9 AM, weekdays)
@Cron('*/15 7-9 * * 1-5')
morningRush() {
  // Runs at: 07:00, 07:15, 07:30, ..., 09:00, 09:15, ..., 09:45 on Mon-Fri
  // Combines: step (*/15), range (7-9), range (1-5)
}

// Example 4: Quarterly on specific dates
@Cron('0 0 1 1,4,7,10 *')
quarterlyReports() {
  // Runs at: Midnight on Jan 1, Apr 1, Jul 1, Oct 1
  // Combines: list (1,4,7,10)
}

// Example 5: Every 2 hours on weekends
@Cron('0 */2 * * 0,6')
weekendSchedule() {
  // Runs at: 00:00, 02:00, 04:00, ..., 22:00 on Saturday and Sunday
  // Combines: step (*/2), list (0,6)
}

// Example 6: Specific minutes of specific hours (8:30, 12:30, 16:30)
@Cron('30 8,12,16 * * 1-5')
specificTimesWeekdays() {
  // Runs at: 08:30, 12:30, 16:30 on Monday-Friday
  // Combines: specific minute (30), list (8,12,16), range (1-5)
}

// Example 7: Every 5 minutes during peak hours (9-11 AM, 2-5 PM)
@Cron('*/5 9-11,14-17 * * *')
peakHoursMonitoring() {
  // Runs at: 09:00-11:55 (every 5 min) and 14:00-17:55 (every 5 min)
  // Combines: step (*/5), ranges with list (9-11,14-17)
}
```

**Key Takeaway:** **Cron syntax** is a **time pattern format** with **5 fields** (standard Unix) or **6 fields** (extended with seconds): `second minute hour dayOfMonth month dayOfWeek`. Each field accepts **specific values** (0-59 minutes, 0-23 hours, 1-31 days, 1-12 months, 0-6 day of week where 0=Sunday), **wildcards** `*` for any value, **ranges** with `-` (1-5 = Monday-Friday inclusive), **lists** with `,` (1,3,5 = specific values), **steps** with `/` (*/15 = every 15 units starting from min value, 9-17/2 = 9,11,13,15,17). **Reading process**: read left to right, identify each field value, interpret special characters, combine conditions (AND logic between fields). **Examples breakdown**: `0 0 * * *` means "at minute 0 AND hour 0 AND any day AND any month AND any day of week" = midnight daily. `0 9 * * 1-5` = "at minute 0 AND hour 9 AND any day AND any month AND Monday-Friday" = weekdays 9 AM. `*/15 * * * *` = "every 15 minutes (0,15,30,45) AND any hour AND any day AND any month AND any day of week" = every 15 minutes continuously. `0 0 1 * *` = "at minute 0 AND hour 0 AND day 1 AND any month AND any day of week" = 1st of month at midnight. **Special characters meaning**: `*` matches any value in that field (every minute, every hour, every day), `-` defines inclusive range (1-5 includes 1,2,3,4,5), `,` lists specific values (1,3,5 = Monday Wednesday Friday only), `/` defines step intervals (*/15 in minute field = 0,15,30,45; 9-17/2 in hour field = 9,11,13,15,17). **Complex patterns**: combine operators like `*/30 9-17 * * 1-5` (every 30 minutes from 9 AM-5 PM on weekdays), `0 8,12,16 * * *` (at 8 AM, noon, 4 PM daily), `0 0 1,15 * *` (midnight on 1st and 15th of month).

</details>

<details>
<summary><strong>7. What are common cron patterns (every minute, hourly, daily)?</strong></summary>

**Answer:**

Common cron patterns include: **Every second** (`* * * * * *` or `CronExpression.EVERY_SECOND` - testing only), **Every minute** (`0 * * * * *` or `CronExpression.EVERY_MINUTE`), **Every 5 minutes** (`0 */5 * * * *` or `CronExpression.EVERY_5_MINUTES`), **Every 15 minutes** (`*/15 * * * *`), **Every 30 minutes** (`0 */30 * * * *` or `CronExpression.EVERY_30_MINUTES`), **Every hour** (`0 0 * * * *` or `CronExpression.EVERY_HOUR`), **Every day at midnight** (`0 0 0 * * *` or `CronExpression.EVERY_DAY_AT_MIDNIGHT`), **Every day at specific time** (`0 0 2 * * *` = 2 AM, `CronExpression.EVERY_DAY_AT_2AM`), **Every weekday** (`0 0 9 * * 1-5` = 9 AM Monday-Friday), **Every week** (`0 0 0 * * 0` or `CronExpression.EVERY_WEEK` = Sunday midnight), **Every month** (`0 0 0 1 * *` or `CronExpression.EVERY_MONTH` = 1st at midnight), **Every year** (`0 0 0 1 1 *` or `CronExpression.EVERY_YEAR` = Jan 1st midnight). Use **CronExpression enum** for common patterns (type-safe, readable), **custom strings** for specific needs (weekdays at 9 AM, business hours, quarterly). Patterns serve different purposes: **frequent** (every minute/5 minutes for monitoring), **hourly** (data sync, cache refresh), **daily** (cleanup, reports, backups at off-peak hours), **weekly** (weekly reports, summaries), **monthly** (billing, subscriptions, monthly reports), **yearly** (annual processing, compliance).

---

### **Common Cron Patterns Reference:**

```
FREQUENCY-BASED PATTERNS:
┌────────────────────────────────────────────────────────────────┐
│  Pattern                    │  Cron Expression  │  Enum        │
├─────────────────────────────┼───────────────────┼──────────────┤
│  Every second (testing)     │  * * * * * *      │  EVERY_SECOND│
│  Every 5 seconds            │  */5 * * * * *    │  EVERY_5_... │
│  Every 10 seconds           │  */10 * * * * *   │  EVERY_10... │
│  Every 30 seconds           │  */30 * * * * *   │  EVERY_30... │
│  Every minute               │  0 * * * * *      │  EVERY_MINUTE│
│  Every 5 minutes            │  0 */5 * * * *    │  EVERY_5_... │
│  Every 10 minutes           │  0 */10 * * * *   │  EVERY_10... │
│  Every 15 minutes           │  */15 * * * *     │  (custom)    │
│  Every 30 minutes           │  0 */30 * * * *   │  EVERY_30... │
│  Every hour                 │  0 0 * * * *      │  EVERY_HOUR  │
│  Every 2 hours              │  0 0 */2 * * *    │  (custom)    │
│  Every 3 hours              │  0 0 */3 * * *    │  (custom)    │
└────────────────────────────────────────────────────────────────┘

DAILY PATTERNS:
┌────────────────────────────────────────────────────────────────┐
│  Pattern                    │  Cron Expression  │  Enum        │
├─────────────────────────────┼───────────────────┼──────────────┤
│  Midnight (00:00)           │  0 0 0 * * *      │  EVERY_DAY...│
│  1 AM                       │  0 0 1 * * *      │  EVERY_DAY...│
│  2 AM (off-peak)            │  0 0 2 * * *      │  EVERY_DAY...│
│  Noon (12:00)               │  0 0 12 * * *     │  EVERY_DAY...│
│  9 AM weekdays              │  0 0 9 * * 1-5    │  (custom)    │
│  5 PM weekdays              │  0 0 17 * * 1-5   │  (custom)    │
└────────────────────────────────────────────────────────────────┘

WEEKLY/MONTHLY/YEARLY:
┌────────────────────────────────────────────────────────────────┐
│  Pattern                    │  Cron Expression  │  Enum        │
├─────────────────────────────┼───────────────────┼──────────────┤
│  Every Sunday (weekly)      │  0 0 0 * * 0      │  EVERY_WEEK  │
│  Every Monday 9 AM          │  0 0 9 * * 1      │  (custom)    │
│  1st of month (monthly)     │  0 0 0 1 * *      │  EVERY_MONTH │
│  15th of month              │  0 0 0 15 * *     │  (custom)    │
│  Jan 1st (yearly)           │  0 0 0 1 1 *      │  EVERY_YEAR  │
│  Quarterly (1st day)        │  0 0 0 1 1,4,7,10 │  (custom)    │
└────────────────────────────────────────────────────────────────┘
```

---

### **Method 1: High-Frequency Patterns (Seconds/Minutes)**

```typescript
// ========== HIGH-FREQUENCY PATTERNS ==========

import { Injectable, Logger } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';

@Injectable()
export class HighFrequencyService {
  private readonly logger = new Logger(HighFrequencyService.name);

  // Every second (use ONLY for testing/development)
  @Cron(CronExpression.EVERY_SECOND)
  everySecond() {
    this.logger.debug('Runs every second');
    // WARNING: Very resource-intensive, use for testing only
  }

  // Every 5 seconds
  @Cron(CronExpression.EVERY_5_SECONDS)
  every5Seconds() {
    this.logger.log('Runs every 5 seconds');
    // Use case: Real-time monitoring, health checks
  }

  // Every 10 seconds
  @Cron(CronExpression.EVERY_10_SECONDS)
  every10Seconds() {
    this.logger.log('Runs every 10 seconds');
    // Use case: Frequent polling, status checks
  }

  // Every 30 seconds
  @Cron(CronExpression.EVERY_30_SECONDS)
  every30Seconds() {
    this.logger.log('Runs every 30 seconds');
    // Use case: Metrics collection, queue monitoring
  }

  // Every minute
  @Cron(CronExpression.EVERY_MINUTE)
  everyMinute() {
    this.logger.log('Runs every minute (at second 0)');
    // Use case: Regular monitoring, status updates
  }

  // Every 5 minutes
  @Cron(CronExpression.EVERY_5_MINUTES)
  every5Minutes() {
    this.logger.log('Runs every 5 minutes (0, 5, 10, 15...)');
    // Use case: Cache refresh, data sync
  }

  // Every 10 minutes
  @Cron(CronExpression.EVERY_10_MINUTES)
  every10Minutes() {
    this.logger.log('Runs every 10 minutes (0, 10, 20...)');
    // Use case: Periodic API calls, updates
  }

  // Every 15 minutes (custom)
  @Cron('*/15 * * * *')
  every15Minutes() {
    this.logger.log('Runs every 15 minutes (0, 15, 30, 45)');
    // Use case: External API sync, price updates
  }

  // Every 30 minutes
  @Cron(CronExpression.EVERY_30_MINUTES)
  every30Minutes() {
    this.logger.log('Runs every 30 minutes (0, 30)');
    // Use case: Half-hourly reports, checks
  }
}

// WHEN TO USE:
// - Every second/5sec: Testing only (too frequent for production)
// - Every 10-30 seconds: Real-time monitoring, health checks
// - Every minute: Regular status checks, metrics
// - Every 5-15 minutes: API sync, cache refresh
// - Every 30 minutes: Periodic updates, reports
```

---

### **Method 2: Hourly Patterns**

```typescript
// ========== HOURLY PATTERNS ==========

@Injectable()
export class HourlyPatternsService {
  private readonly logger = new Logger(HourlyPatternsService.name);

  // Every hour (at minute 0)
  @Cron(CronExpression.EVERY_HOUR)
  everyHour() {
    this.logger.log('Runs every hour (XX:00)');
    // Use case: Hourly sync, cache cleanup, reports
  }

  // Every 2 hours
  @Cron('0 */2 * * *')
  every2Hours() {
    this.logger.log('Runs every 2 hours (00:00, 02:00, 04:00...)');
    // Use case: Periodic backups, data aggregation
  }

  // Every 3 hours
  @Cron('0 */3 * * *')
  every3Hours() {
    this.logger.log('Runs every 3 hours (00:00, 03:00, 06:00...)');
    // Use case: External service sync
  }

  // Every 6 hours
  @Cron('0 */6 * * *')
  every6Hours() {
    this.logger.log('Runs every 6 hours (00:00, 06:00, 12:00, 18:00)');
    // Use case: Quarterly daily tasks
  }

  // Every 12 hours (twice daily)
  @Cron('0 */12 * * *')
  every12Hours() {
    this.logger.log('Runs every 12 hours (00:00, 12:00)');
    // Use case: Bi-daily reports
  }

  // Specific hours: 9 AM, 12 PM, 3 PM, 6 PM
  @Cron('0 9,12,15,18 * * *')
  specificHours() {
    this.logger.log('Runs at 9 AM, noon, 3 PM, 6 PM');
    // Use case: Business hours notifications
  }

  // Business hours: every hour 9 AM - 5 PM
  @Cron('0 9-17 * * *')
  businessHours() {
    this.logger.log('Runs every hour from 9 AM to 5 PM');
    // Use case: Office hours monitoring
  }
}

// WHEN TO USE:
// - Every hour: Regular sync, cache updates
// - Every 2-3 hours: Less frequent updates
// - Every 6-12 hours: Bi-daily/quarterly tasks
// - Specific hours: Business-critical times
// - Business hours: Office hours only
```

---

### **Method 3: Daily Patterns**

```typescript
// ========== DAILY PATTERNS ==========

@Injectable()
export class DailyPatternsService {
  private readonly logger = new Logger(DailyPatternsService.name);

  // Midnight every day (00:00)
  @Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT)
  dailyMidnight() {
    this.logger.log('Runs at midnight daily');
    // Use case: Daily cleanup, rollover tasks
  }

  // 1 AM daily
  @Cron(CronExpression.EVERY_DAY_AT_1AM)
  daily1AM() {
    this.logger.log('Runs at 1 AM daily');
    // Use case: Off-peak maintenance
  }

  // 2 AM daily (optimal for heavy tasks)
  @Cron(CronExpression.EVERY_DAY_AT_2AM)
  daily2AM() {
    this.logger.log('Runs at 2 AM daily');
    // Use case: Database backup, heavy processing
  }

  // Noon daily (12:00)
  @Cron(CronExpression.EVERY_DAY_AT_NOON)
  dailyNoon() {
    this.logger.log('Runs at noon daily');
    // Use case: Mid-day reports, notifications
  }

  // 6 AM daily (before business hours)
  @Cron('0 6 * * *')
  daily6AM() {
    this.logger.log('Runs at 6 AM daily');
    // Use case: Pre-work preparation, cache warming
  }

  // 9 AM weekdays only
  @Cron('0 9 * * 1-5')
  weekdays9AM() {
    this.logger.log('Runs at 9 AM Monday-Friday');
    // Use case: Workday start notifications
  }

  // 5 PM weekdays (end of workday)
  @Cron('0 17 * * 1-5')
  weekdays5PM() {
    this.logger.log('Runs at 5 PM Monday-Friday');
    // Use case: End-of-day reports, summaries
  }

  // Twice daily: midnight and noon
  @Cron('0 0,12 * * *')
  twiceDaily() {
    this.logger.log('Runs at midnight and noon');
    // Use case: Bi-daily sync, checks
  }
}

// WHEN TO USE:
// - Midnight-2 AM: Off-peak heavy tasks (backups, cleanup)
// - 6-8 AM: Pre-business prep (cache warming, reports)
// - 9 AM weekdays: Start of workday tasks
// - Noon: Mid-day updates
// - 5-6 PM: End-of-day processing
```

---

### **Method 4: Weekly/Monthly/Yearly Patterns**

```typescript
// ========== WEEKLY/MONTHLY/YEARLY PATTERNS ==========

@Injectable()
export class PeriodicPatternsService {
  private readonly logger = new Logger(PeriodicPatternsService.name);

  // Every Sunday at midnight (weekly)
  @Cron(CronExpression.EVERY_WEEK)
  weekly() {
    this.logger.log('Runs every Sunday at midnight');
    // Use case: Weekly reports, summaries
  }

  // Every Monday at 8 AM
  @Cron('0 8 * * 1')
  mondayMorning() {
    this.logger.log('Runs every Monday at 8 AM');
    // Use case: Week start notifications
  }

  // Every Friday at 5 PM
  @Cron('0 17 * * 5')
  fridayEvening() {
    this.logger.log('Runs every Friday at 5 PM');
    // Use case: Week end reports
  }

  // 1st of month at midnight (monthly)
  @Cron(CronExpression.EVERY_MONTH)
  monthly() {
    this.logger.log('Runs 1st of every month at midnight');
    // Use case: Monthly reports, billing
  }

  // 15th of month (bi-monthly payroll)
  @Cron('0 0 15 * *')
  biMonthly() {
    this.logger.log('Runs on 15th of month');
    // Use case: Bi-monthly processing
  }

  // Last day of month (complex)
  @Cron('0 0 28-31 * *')
  endOfMonth() {
    this.logger.log('Runs on last few days of month');
    const today = new Date();
    const tomorrow = new Date(today);
    tomorrow.setDate(tomorrow.getDate() + 1);
    
    // Check if tomorrow is a different month
    if (tomorrow.getMonth() !== today.getMonth()) {
      this.logger.log('This is the last day of the month!');
      // Month-end processing
    }
  }

  // Quarterly: 1st day of Jan, Apr, Jul, Oct
  @Cron('0 0 1 1,4,7,10 *')
  quarterly() {
    this.logger.log('Runs quarterly on 1st of Jan/Apr/Jul/Oct');
    // Use case: Quarterly reports, reviews
  }

  // Semi-annually: Jan 1 and Jul 1
  @Cron('0 0 1 1,7 *')
  semiAnnually() {
    this.logger.log('Runs on Jan 1 and Jul 1');
    // Use case: Half-yearly processing
  }

  // Yearly: January 1st at midnight
  @Cron(CronExpression.EVERY_YEAR)
  yearly() {
    this.logger.log('Runs every Jan 1 at midnight');
    // Use case: Annual reports, renewals
  }
}

// WHEN TO USE:
// - Weekly: Sunday/Monday tasks, week summaries
// - Bi-weekly: Payroll, reports
// - Monthly: 1st/15th processing, billing
// - Quarterly: Q1/Q2/Q3/Q4 reports
// - Yearly: Annual processing, compliance
```

---

### **Method 5: Business-Specific Patterns**

```typescript
// ========== BUSINESS-SPECIFIC PATTERNS ==========

@Injectable()
export class BusinessPatternsService {
  private readonly logger = new Logger(BusinessPatternsService.name);

  // Every 15 minutes during market hours (9:30 AM - 4 PM EST)
  @Cron('*/15 9-16 * * 1-5', {
    timeZone: 'America/New_York',
  })
  marketHours() {
    this.logger.log('Runs every 15 min during stock market hours');
    // Use case: Stock price updates
  }

  // Business hours: 9 AM - 5 PM, Monday-Friday
  @Cron('0 9-17 * * 1-5')
  officeHours() {
    this.logger.log('Runs every hour during office hours');
    // Use case: Business hours monitoring
  }

  // Every 30 min during peak traffic (11 AM - 2 PM, 5 PM - 8 PM)
  @Cron('*/30 11-14,17-20 * * *')
  peakHours() {
    this.logger.log('Runs every 30 min during peak hours');
    // Use case: Load balancing, scaling
  }

  // Overnight processing: 2 AM - 6 AM
  @Cron('0 2-6 * * *')
  overnightProcessing() {
    this.logger.log('Runs hourly from 2 AM - 6 AM');
    // Use case: Heavy batch processing
  }

  // Subscription billing: 1st of month at 3 AM
  @Cron('0 3 1 * *')
  subscriptionBilling() {
    this.logger.log('Runs 1st of month at 3 AM');
    // Use case: Monthly subscription charges
  }

  // Tax reporting: Last day of quarter at 11 PM
  @Cron('0 23 31 3,6,9,12 *')
  quarterlyTax() {
    this.logger.log('Runs last day of quarter at 11 PM');
    // Use case: Quarterly tax preparation
  }

  // Black Friday: November 4th Friday at 6 AM
  @Cron('0 6 22-28 11 5')  // 4th Friday of November
  blackFriday() {
    const now = new Date();
    const dayOfMonth = now.getDate();
    const dayOfWeek = now.getDay();
    
    // Check if it's the 4th Friday
    if (dayOfWeek === 5 && dayOfMonth >= 22 && dayOfMonth <= 28) {
      this.logger.log('Black Friday preparations!');
      // Special promotions, scaling prep
    }
  }
}

// CUSTOM PATTERNS FOR:
// - Market hours (stock trading, forex)
// - Business hours (9-5 operations)
// - Peak hours (lunch, evening rush)
// - Overnight processing (off-peak batch jobs)
// - Billing cycles (monthly, quarterly)
// - Special events (Black Friday, holidays)
```

**Key Takeaway:** Common cron patterns organized by frequency: **High-frequency** - every second (`* * * * * *` testing only), every 5 seconds (`*/5 * * * * *`), every minute (`0 * * * * *` or `CronExpression.EVERY_MINUTE`), every 5 minutes (`0 */5 * * * *` or `EVERY_5_MINUTES`), every 15 minutes (`*/15 * * * *`), every 30 minutes (`0 */30 * * * *` or `EVERY_30_MINUTES`). **Hourly** - every hour (`0 0 * * * *` or `EVERY_HOUR`), every 2 hours (`0 */2 * * *`), business hours 9-5 (`0 9-17 * * *`), specific hours (`0 9,12,15,18 * * *`). **Daily** - midnight (`0 0 0 * * *` or `EVERY_DAY_AT_MIDNIGHT`), 2 AM for off-peak (`0 0 2 * * *` or `EVERY_DAY_AT_2AM`), noon (`0 0 12 * * *` or `EVERY_DAY_AT_NOON`), weekdays 9 AM (`0 0 9 * * 1-5`), weekdays 5 PM (`0 0 17 * * 1-5`). **Weekly/Monthly/Yearly** - every Sunday (`0 0 0 * * 0` or `EVERY_WEEK`), every Monday 8 AM (`0 0 8 * * 1`), 1st of month (`0 0 0 1 * *` or `EVERY_MONTH`), 15th of month (`0 0 0 15 * *` for bi-monthly), quarterly (`0 0 0 1 1,4,7,10 *` for Jan/Apr/Jul/Oct 1st), yearly (`0 0 0 1 1 *` or `EVERY_YEAR` for Jan 1st). **Use CronExpression enum** for common patterns (type-safe, self-documenting, no syntax errors), **custom strings** for specific needs (weekday mornings, business hours, quarterly schedules). **Purpose-based**: monitoring (every minute/5 minutes), API sync (every 15 minutes), cache refresh (hourly), cleanup (daily 2 AM), backups (daily off-peak), reports (daily/weekly/monthly), billing (monthly 1st), compliance (quarterly/yearly).

</details>

<details>
<summary><strong>8. How do you schedule a job to run at midnight every day?</strong></summary>

**Answer:**

Schedule a job to run at midnight every day using `@Cron()` decorator with **midnight pattern**: `@Cron('0 0 0 * * *')` (6 fields with seconds) or `@Cron('0 0 * * *')` (5 fields standard), or preferably `@Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT)` using the enum constant for type-safety. The expression `0 0 0 * * *` means: second=0, minute=0, hour=0 (midnight), any day of month, any month, any day of week = runs at exactly 00:00:00 every day. **Best practices**: use `CronExpression.EVERY_DAY_AT_MIDNIGHT` enum (readable, type-safe), add **timezone** option if specific timezone needed (`{ timeZone: 'America/New_York' }` runs at midnight in NY regardless of server timezone), use **named job** for management (`{ name: 'dailyCleanup' }`), implement **error handling** with try-catch, add **logging** to track execution, make method **async** for database operations, use **off-peak hours** (midnight-3 AM) for heavy tasks like backups/cleanup/reports. The scheduler automatically triggers the method at midnight daily, no manual invocation needed, runs continuously until app stops.

---

### **Midnight Scheduling - Implementation:**

```
MIDNIGHT CRON EXPRESSION:
┌────────────────────────────────────────────────────┐
│  Expression: 0 0 0 * * *                           │
│              │ │ │ │ │ │                           │
│              │ │ │ │ │ └─── Any day of week       │
│              │ │ │ │ └───── Any month             │
│              │ │ │ └─────── Any day of month      │
│              │ │ └───────── Hour: 0 (midnight)    │
│              │ └─────────── Minute: 0             │
│              └───────────── Second: 0             │
│                                                    │
│  Runs at: 00:00:00 every day                       │
│                                                    │
│  Timeline:                                         │
│  Jan 1, 2024 00:00:00 → ✅ Runs                    │
│  Jan 1, 2024 00:00:01 → ❌ Doesn't run             │
│  Jan 1, 2024 12:00:00 → ❌ Doesn't run             │
│  Jan 2, 2024 00:00:00 → ✅ Runs                    │
│  Jan 3, 2024 00:00:00 → ✅ Runs                    │
└────────────────────────────────────────────────────┘
```

---

### **Method 1: Basic Midnight Cron Job**

```typescript
// ========== BASIC MIDNIGHT CRON ==========

import { Injectable, Logger } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';

@Injectable()
export class MidnightJobsService {
  private readonly logger = new Logger(MidnightJobsService.name);

  // Method 1: Using CronExpression enum (RECOMMENDED)
  @Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT)
  dailyMidnightJob() {
    this.logger.log('Daily midnight job executing');
    this.logger.log(`Executed at: ${new Date().toISOString()}`);
    
    // Your midnight task logic here
    // Example: Data cleanup, report generation, etc.
  }

  // Method 2: Using 6-field cron expression
  @Cron('0 0 0 * * *')  // second minute hour day month dayOfWeek
  midnightWith6Fields() {
    this.logger.log('Midnight job (6 fields)');
  }

  // Method 3: Using 5-field cron expression (standard Unix)
  @Cron('0 0 * * *')  // minute hour day month dayOfWeek
  midnightWith5Fields() {
    this.logger.log('Midnight job (5 fields)');
  }
}

// WHICH TO USE?
// ✅ CronExpression.EVERY_DAY_AT_MIDNIGHT (best: type-safe, readable)
// ✅ '0 0 0 * * *' (explicit 6 fields)
// ✅ '0 0 * * *' (standard 5 fields)
// All three produce identical behavior: run at midnight daily
```

---

### **Method 2: Midnight Job with Timezone**

```typescript
// ========== TIMEZONE-AWARE MIDNIGHT CRON ==========

@Injectable()
export class TimezoneMidnightService {
  private readonly logger = new Logger(TimezoneMidnightService.name);

  // Midnight Eastern Time (EST/EDT)
  @Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT, {
    name: 'midnightEastern',
    timeZone: 'America/New_York',
  })
  midnightEastern() {
    this.logger.log('Midnight Eastern Time');
    // Runs at midnight in New York (EST/EDT)
    // Server could be anywhere, job runs at NY midnight
  }

  // Midnight Pacific Time (PST/PDT)
  @Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT, {
    name: 'midnightPacific',
    timeZone: 'America/Los_Angeles',
  })
  midnightPacific() {
    this.logger.log('Midnight Pacific Time');
    // Runs at midnight in Los Angeles
  }

  // Midnight London Time (GMT/BST)
  @Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT, {
    name: 'midnightLondon',
    timeZone: 'Europe/London',
  })
  midnightLondon() {
    this.logger.log('Midnight London Time');
    // Runs at midnight in London
  }

  // Midnight UTC (no daylight saving)
  @Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT, {
    name: 'midnightUTC',
    timeZone: 'UTC',
  })
  midnightUTC() {
    this.logger.log('Midnight UTC');
    // Runs at midnight UTC, consistent year-round
  }
}

// WHY USE TIMEZONE:
// - Multi-region applications
// - Business operates in specific timezone
// - Compliance requirements (data at local midnight)
// - Consistent timing regardless of server location
```

---

### **Method 3: Production Midnight Jobs with Error Handling**

```typescript
// ========== PRODUCTION MIDNIGHT JOBS ==========

@Injectable()
export class ProductionMidnightService {
  private readonly logger = new Logger(ProductionMidnightService.name);

  constructor(
    private readonly logsRepository: LogsRepository,
    private readonly sessionsRepository: SessionsRepository,
    private readonly emailService: EmailService,
    private readonly configService: ConfigService,
  ) {}

  // Daily cleanup at midnight
  @Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT, {
    name: 'dailyCleanup',
    timeZone: 'America/New_York',
  })
  async dailyCleanup() {
    this.logger.log('Starting daily cleanup at midnight');
    
    const startTime = Date.now();
    let logsDeleted = 0;
    let sessionsDeleted = 0;

    try {
      // Get retention period from config
      const retentionDays = this.configService.get<number>(
        'DATA_RETENTION_DAYS',
        30,
      );

      const cutoffDate = new Date();
      cutoffDate.setDate(cutoffDate.getDate() - retentionDays);

      this.logger.log(`Deleting data older than ${retentionDays} days`);

      // Delete old logs
      const logsResult = await this.logsRepository.delete({
        createdAt: LessThan(cutoffDate),
      });
      logsDeleted = logsResult.affected || 0;
      this.logger.log(`Deleted ${logsDeleted} old logs`);

      // Delete expired sessions
      const sessionsResult = await this.sessionsRepository.delete({
        expiresAt: LessThan(new Date()),
      });
      sessionsDeleted = sessionsResult.affected || 0;
      this.logger.log(`Deleted ${sessionsDeleted} expired sessions`);

      const duration = Date.now() - startTime;
      this.logger.log(
        `Daily cleanup completed in ${duration}ms: ` +
        `${logsDeleted} logs, ${sessionsDeleted} sessions`,
      );

      // Send success notification
      await this.emailService.sendEmail({
        to: 'admin@company.com',
        subject: 'Daily Cleanup Report',
        html: `
          <h2>Cleanup Summary</h2>
          <p>Execution time: ${duration}ms</p>
          <p>Logs deleted: ${logsDeleted}</p>
          <p>Sessions deleted: ${sessionsDeleted}</p>
        `,
      });
      
    } catch (error) {
      this.logger.error('Daily cleanup failed', error.stack);
      
      // Send failure alert
      await this.emailService.sendEmail({
        to: 'ops@company.com',
        subject: '⚠️ ALERT: Daily Cleanup Failed',
        html: `
          <h2>Cleanup Failed</h2>
          <p>Error: ${error.message}</p>
          <p>Stack: ${error.stack}</p>
        `,
      });
      
      // Re-throw to mark job as failed in monitoring
      throw error;
    }
  }
}

// BEST PRACTICES:
// ✅ Use try-catch for error handling
// ✅ Log start, progress, and completion
// ✅ Track execution time
// ✅ Send notifications on success/failure
// ✅ Use async for database operations
// ✅ Make it idempotent (safe to run multiple times)
```

---

### **Method 4: Multiple Midnight Jobs**

```typescript
// ========== MULTIPLE MIDNIGHT JOBS ==========

@Injectable()
export class MultipleMidnightService {
  private readonly logger = new Logger(MultipleMidnightService.name);

  constructor(
    private readonly backupService: BackupService,
    private readonly reportsService: ReportsService,
    private readonly analyticsService: AnalyticsService,
  ) {}

  // Job 1: Database backup at midnight
  @Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT, {
    name: 'databaseBackup',
  })
  async databaseBackup() {
    this.logger.log('Starting database backup');
    await this.backupService.createDailyBackup();
  }

  // Job 2: Generate daily reports at 00:10 (10 min after midnight)
  @Cron('0 10 0 * * *', {
    name: 'dailyReports',
  })
  async generateDailyReports() {
    this.logger.log('Generating daily reports');
    // Wait 10 min after midnight to ensure data is finalized
    await this.reportsService.generateDailySalesReport();
  }

  // Job 3: Process analytics at 00:30
  @Cron('0 30 0 * * *', {
    name: 'dailyAnalytics',
  })
  async processAnalytics() {
    this.logger.log('Processing daily analytics');
    await this.analyticsService.aggregateDailyMetrics();
  }
}

// STAGGERING MIDNIGHT JOBS:
// - 00:00 - Database backup (heaviest task)
// - 00:10 - Daily reports (needs complete data)
// - 00:30 - Analytics processing
// - Prevents resource contention
// - Ensures dependencies are met
```

---

### **Method 5: Midnight Job Variations**

```typescript
// ========== MIDNIGHT VARIATIONS ==========

@Injectable()
export class MidnightVariationsService {
  private readonly logger = new Logger(MidnightVariationsService.name);

  // Midnight weekdays only (Monday-Friday)
  @Cron('0 0 0 * * 1-5', {
    name: 'weekdayMidnight',
  })
  weekdayMidnight() {
    this.logger.log('Midnight on weekdays only');
    // Use case: Business day processing only
  }

  // Midnight weekends only (Saturday-Sunday)
  @Cron('0 0 0 * * 0,6', {
    name: 'weekendMidnight',
  })
  weekendMidnight() {
    this.logger.log('Midnight on weekends only');
    // Use case: Weekend-specific tasks
  }

  // Midnight on 1st of month only
  @Cron('0 0 0 1 * *', {
    name: 'monthlyMidnight',
  })
  monthlyMidnight() {
    this.logger.log('Midnight on 1st of month');
    // Use case: Monthly processing
  }

  // Midnight on specific dates (1st and 15th)
  @Cron('0 0 0 1,15 * *', {
    name: 'biMonthlyMidnight',
  })
  biMonthlyMidnight() {
    this.logger.log('Midnight on 1st and 15th');
    // Use case: Bi-monthly payroll, reports
  }

  // Slightly before midnight (23:55 - 11:55 PM)
  @Cron('0 55 23 * * *', {
    name: 'beforeMidnight',
  })
  beforeMidnight() {
    this.logger.log('Runs 5 minutes before midnight');
    // Use case: Pre-midnight preparation tasks
  }

  // Just after midnight (00:05)
  @Cron('0 5 0 * * *', {
    name: 'afterMidnight',
  })
  afterMidnight() {
    this.logger.log('Runs 5 minutes after midnight');
    // Use case: Post-midnight tasks requiring previous day data
  }
}

// MIDNIGHT VARIATIONS FOR:
// - Weekdays only (business processing)
// - Weekends only (maintenance windows)
// - Monthly (1st of month)
// - Bi-monthly (payroll dates)
// - Before/after midnight (staging tasks)
```

**Key Takeaway:** Schedule a job at midnight daily using `@Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT)` (recommended enum for type-safety and readability) or `@Cron('0 0 0 * * *')` (6-field expression: second=0, minute=0, hour=0=midnight, any day, any month, any dayOfWeek) or `@Cron('0 0 * * *')` (5-field standard Unix). **Expression breakdown**: `0 0 0 * * *` runs at exactly 00:00:00 every day (midnight), `0 0 * * *` is equivalent 5-field format. **Options**: add `timeZone` for specific timezone (`{ timeZone: 'America/New_York' }` runs at midnight in NY regardless of server location, handles DST automatically), add `name` for dynamic management (`{ name: 'dailyCleanup' }` allows add/remove via SchedulerRegistry), set `disabled: true` to define but not activate. **Best practices**: use **async method** for database/API operations, implement **try-catch** for error handling (log failures, send alerts to ops team), add **comprehensive logging** (log start time, progress, completion time, items processed), track **execution metrics** (duration, records affected), send **notifications** on completion or failure, make **idempotent** (safe to run multiple times), use **off-peak hours** (midnight-3 AM for heavy tasks like backups, cleanup, reports). **Variations**: weekdays only (`0 0 0 * * 1-5`), weekends only (`0 0 0 * * 0,6`), 1st of month (`0 0 0 1 * *`), bi-monthly (`0 0 0 1,15 * *`). **Stagger jobs**: if multiple midnight tasks, stagger them (00:00 backup, 00:10 reports, 00:30 analytics) to prevent resource contention and ensure dependencies met. **Use cases**: daily cleanup (delete old logs/sessions), database backups (full backup during off-peak), report generation (daily sales/analytics), data aggregation (consolidate previous day), batch processing (heavy computation when traffic low).

</details>

<details>
<summary><strong>9. How do you use cron names for managing jobs?</strong></summary>

**Answer:**

Use **cron names** by passing a `name` property in the options object of `@Cron()`, `@Interval()`, or `@Timeout()` decorators: `@Cron('0 0 * * *', { name: 'dailyBackup' })`. The name acts as a **unique identifier** allowing **dynamic management** via `SchedulerRegistry` service. With a named job, you can: **get the job** (`schedulerRegistry.getCronJob('dailyBackup')`), **start/stop job** (`job.start()`, `job.stop()`), **delete job** (`schedulerRegistry.deleteCronJob('dailyBackup')`), **check if exists** (`schedulerRegistry.doesExist('cron', 'dailyBackup')`), **get all cron jobs** (`schedulerRegistry.getCronJobs()`). Benefits: **runtime control** (enable/disable jobs based on conditions), **monitoring** (check job status, next execution time), **dynamic configuration** (add/remove jobs based on user settings), **debugging** (pause jobs during maintenance), **graceful shutdown** (stop jobs before app termination). Without a name, jobs are **anonymous** and cannot be accessed dynamically (only managed via decorator). Use descriptive names (camelCase): 'dailyBackup', 'hourlySync', 'weeklyReport'. Names must be **unique** within job type (cron/interval/timeout can share same name across types but not within same type).

---

### **Named Cron Jobs Architecture:**

```
NAMED CRON JOB FLOW:
┌──────────────────────────────────────────────────────┐
│  1. Define Named Job                                 │
│     @Cron('0 0 * * *', { name: 'dailyBackup' })      │
│     dailyBackup() { }                                │
└──────────────────┬───────────────────────────────────┘
                   ↓
┌──────────────────────────────────────────────────────┐
│  2. Job Registered in SchedulerRegistry              │
│     - Stored with unique identifier                  │
│     - Accessible for dynamic management              │
└──────────────────┬───────────────────────────────────┘
                   ↓
┌──────────────────────────────────────────────────────┐
│  3. Dynamic Management via SchedulerRegistry         │
│     - getCronJob(name) → Retrieve job reference      │
│     - job.start() → Start execution                  │
│     - job.stop() → Stop execution                    │
│     - deleteCronJob(name) → Remove job               │
│     - doesExist('cron', name) → Check existence      │
└──────────────────────────────────────────────────────┘

BENEFITS:
✅ Runtime control (start/stop/delete jobs)
✅ Monitoring (check status, next run time)
✅ Dynamic configuration (conditional jobs)
✅ Debugging (pause during maintenance)
✅ Graceful shutdown (stop before termination)
```

---

### **Method 1: Creating Named Cron Jobs**

```typescript
// ========== CREATING NAMED CRON JOBS ==========

import { Injectable, Logger } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';

@Injectable()
export class NamedCronService {
  private readonly logger = new Logger(NamedCronService.name);

  // Named cron job with string name
  @Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT, {
    name: 'dailyBackup',  // Unique identifier
  })
  dailyBackup() {
    this.logger.log('Daily backup job executing');
  }

  // Named job with custom expression
  @Cron('*/15 * * * *', {
    name: 'priceSync',
  })
  priceSync() {
    this.logger.log('Price sync job executing');
  }

  // Named job with timezone
  @Cron(CronExpression.EVERY_HOUR, {
    name: 'hourlyReport',
    timeZone: 'America/New_York',
  })
  hourlyReport() {
    this.logger.log('Hourly report job executing');
  }

  // Multiple named jobs in same service
  @Cron('0 9 * * 1-5', {
    name: 'weekdayMorningAlert',
  })
  weekdayAlert() {
    this.logger.log('Weekday morning alert');
  }

  @Cron('0 17 * * 1-5', {
    name: 'weekdayEveningReport',
  })
  weekdayReport() {
    this.logger.log('Weekday evening report');
  }
}

// NAMING CONVENTIONS:
// ✅ Use camelCase: 'dailyBackup', 'hourlySync'
// ✅ Be descriptive: indicates purpose and frequency
// ✅ Unique per type: can't have two cron jobs with same name
// ✅ Examples: 'databaseBackup', 'priceUpdate', 'emailReminder'
```

---

### **Method 2: Accessing Named Jobs via SchedulerRegistry**

```typescript
// ========== ACCESSING NAMED JOBS ==========

import { Injectable, Logger, OnModuleInit } from '@nestjs/common';
import { SchedulerRegistry } from '@nestjs/schedule';
import { CronJob } from 'cron';

@Injectable()
export class JobManagementService implements OnModuleInit {
  private readonly logger = new Logger(JobManagementService.name);

  constructor(private readonly schedulerRegistry: SchedulerRegistry) {}

  onModuleInit() {
    // Access and inspect jobs after module initialization
    this.inspectJobs();
  }

  inspectJobs() {
    // Get specific cron job by name
    try {
      const dailyBackup = this.schedulerRegistry.getCronJob('dailyBackup');
      this.logger.log('Daily backup job found');
      this.logger.log(`Running: ${dailyBackup.running}`);
      this.logger.log(`Next run: ${dailyBackup.nextDate()}`);
    } catch (error) {
      this.logger.warn('Daily backup job not found');
    }

    // Get all cron jobs
    const cronJobs = this.schedulerRegistry.getCronJobs();
    this.logger.log(`Total cron jobs: ${cronJobs.size}`);

    // Iterate through all jobs
    cronJobs.forEach((job, name) => {
      this.logger.log(`Job: ${name}`);
      this.logger.log(`  Running: ${job.running}`);
      this.logger.log(`  Next: ${job.nextDate()}`);
    });

    // Check if job exists
    const exists = this.schedulerRegistry.doesExist('cron', 'dailyBackup');
    this.logger.log(`dailyBackup exists: ${exists}`);
  }

  // Get job by name
  getJob(name: string): CronJob {
    return this.schedulerRegistry.getCronJob(name);
  }

  // Check if job is running
  isJobRunning(name: string): boolean {
    try {
      const job = this.schedulerRegistry.getCronJob(name);
      return job.running;
    } catch {
      return false;
    }
  }

  // Get next execution time
  getNextRunTime(name: string): Date | null {
    try {
      const job = this.schedulerRegistry.getCronJob(name);
      return job.nextDate().toJSDate();
    } catch {
      return null;
    }
  }

  // Get last execution time
  getLastRunTime(name: string): Date | null {
    try {
      const job = this.schedulerRegistry.getCronJob(name);
      return job.lastDate()?.toJSDate() || null;
    } catch {
      return null;
    }
  }
}

// SCHEDULERREGISTRY METHODS:
// - getCronJob(name) → Returns CronJob instance
// - getCronJobs() → Returns Map<string, CronJob>
// - doesExist('cron', name) → Returns boolean
// - deleteCronJob(name) → Removes job
```

---

### **Method 3: Starting and Stopping Named Jobs**

```typescript
// ========== START/STOP NAMED JOBS ==========

@Injectable()
export class JobControlService {
  private readonly logger = new Logger(JobControlService.name);

  constructor(private readonly schedulerRegistry: SchedulerRegistry) {}

  // Stop a job
  stopJob(name: string): void {
    try {
      const job = this.schedulerRegistry.getCronJob(name);
      job.stop();
      this.logger.log(`Job ${name} stopped`);
    } catch (error) {
      this.logger.error(`Failed to stop job ${name}`, error.message);
      throw new Error(`Job ${name} not found`);
    }
  }

  // Start a job
  startJob(name: string): void {
    try {
      const job = this.schedulerRegistry.getCronJob(name);
      job.start();
      this.logger.log(`Job ${name} started`);
    } catch (error) {
      this.logger.error(`Failed to start job ${name}`, error.message);
      throw new Error(`Job ${name} not found`);
    }
  }

  // Toggle job (stop if running, start if stopped)
  toggleJob(name: string): boolean {
    try {
      const job = this.schedulerRegistry.getCronJob(name);
      
      if (job.running) {
        job.stop();
        this.logger.log(`Job ${name} stopped`);
        return false;
      } else {
        job.start();
        this.logger.log(`Job ${name} started`);
        return true;
      }
    } catch (error) {
      this.logger.error(`Failed to toggle job ${name}`, error.message);
      throw new Error(`Job ${name} not found`);
    }
  }

  // Restart job (stop then start)
  restartJob(name: string): void {
    try {
      const job = this.schedulerRegistry.getCronJob(name);
      job.stop();
      job.start();
      this.logger.log(`Job ${name} restarted`);
    } catch (error) {
      this.logger.error(`Failed to restart job ${name}`, error.message);
      throw new Error(`Job ${name} not found`);
    }
  }
}

// USE CASES:
// - Stop job during maintenance
// - Start job after configuration update
// - Toggle job based on user settings
// - Restart job to reload configuration
```

---

### **Method 4: Deleting and Managing Named Jobs**

```typescript
// ========== DELETE AND MANAGE NAMED JOBS ==========

@Injectable()
export class JobLifecycleService {
  private readonly logger = new Logger(JobLifecycleService.name);

  constructor(private readonly schedulerRegistry: SchedulerRegistry) {}

  // Delete a job
  deleteJob(name: string): void {
    try {
      this.schedulerRegistry.deleteCronJob(name);
      this.logger.log(`Job ${name} deleted`);
    } catch (error) {
      this.logger.error(`Failed to delete job ${name}`, error.message);
      throw new Error(`Job ${name} not found`);
    }
  }

  // Safe delete (check existence first)
  safeDeleteJob(name: string): boolean {
    const exists = this.schedulerRegistry.doesExist('cron', name);
    
    if (exists) {
      this.schedulerRegistry.deleteCronJob(name);
      this.logger.log(`Job ${name} deleted`);
      return true;
    } else {
      this.logger.warn(`Job ${name} does not exist`);
      return false;
    }
  }

  // Delete all jobs
  deleteAllJobs(): void {
    const cronJobs = this.schedulerRegistry.getCronJobs();
    const jobNames = Array.from(cronJobs.keys());

    jobNames.forEach(name => {
      this.schedulerRegistry.deleteCronJob(name);
      this.logger.log(`Job ${name} deleted`);
    });

    this.logger.log(`Deleted ${jobNames.length} jobs`);
  }

  // Delete jobs matching pattern
  deleteJobsByPattern(pattern: RegExp): void {
    const cronJobs = this.schedulerRegistry.getCronJobs();
    const jobNames = Array.from(cronJobs.keys());
    
    const matchingJobs = jobNames.filter(name => pattern.test(name));

    matchingJobs.forEach(name => {
      this.schedulerRegistry.deleteCronJob(name);
      this.logger.log(`Job ${name} deleted (matched pattern)`);
    });

    this.logger.log(`Deleted ${matchingJobs.length} jobs matching pattern`);
  }

  // List all job names
  listAllJobs(): string[] {
    const cronJobs = this.schedulerRegistry.getCronJobs();
    return Array.from(cronJobs.keys());
  }

  // Get job count
  getJobCount(): number {
    const cronJobs = this.schedulerRegistry.getCronJobs();
    return cronJobs.size;
  }

  // Check if any jobs are running
  hasRunningJobs(): boolean {
    const cronJobs = this.schedulerRegistry.getCronJobs();
    return Array.from(cronJobs.values()).some(job => job.running);
  }
}

// LIFECYCLE MANAGEMENT:
// - Delete job when no longer needed
// - Clean up on module destroy
// - Remove jobs based on user configuration
// - Bulk operations for maintenance
```

---

### **Method 5: REST API for Job Management**

```typescript
// ========== REST API FOR JOB MANAGEMENT ==========

import { Controller, Get, Post, Delete, Param, Body } from '@nestjs/common';

@Controller('jobs')
export class JobsController {
  constructor(
    private readonly jobControlService: JobControlService,
    private readonly jobLifecycleService: JobLifecycleService,
    private readonly jobManagementService: JobManagementService,
  ) {}

  // List all jobs
  @Get()
  listJobs() {
    const jobs = this.jobLifecycleService.listAllJobs();
    
    return jobs.map(name => ({
      name,
      running: this.jobManagementService.isJobRunning(name),
      nextRun: this.jobManagementService.getNextRunTime(name),
      lastRun: this.jobManagementService.getLastRunTime(name),
    }));
  }

  // Get job details
  @Get(':name')
  getJob(@Param('name') name: string) {
    const job = this.jobManagementService.getJob(name);
    
    return {
      name,
      running: job.running,
      nextRun: job.nextDate().toJSDate(),
      lastRun: job.lastDate()?.toJSDate() || null,
      cronTime: job.cronTime.source,
    };
  }

  // Start job
  @Post(':name/start')
  startJob(@Param('name') name: string) {
    this.jobControlService.startJob(name);
    return { message: `Job ${name} started`, success: true };
  }

  // Stop job
  @Post(':name/stop')
  stopJob(@Param('name') name: string) {
    this.jobControlService.stopJob(name);
    return { message: `Job ${name} stopped`, success: true };
  }

  // Toggle job
  @Post(':name/toggle')
  toggleJob(@Param('name') name: string) {
    const isRunning = this.jobControlService.toggleJob(name);
    return {
      message: `Job ${name} ${isRunning ? 'started' : 'stopped'}`,
      running: isRunning,
    };
  }

  // Restart job
  @Post(':name/restart')
  restartJob(@Param('name') name: string) {
    this.jobControlService.restartJob(name);
    return { message: `Job ${name} restarted`, success: true };
  }

  // Delete job
  @Delete(':name')
  deleteJob(@Param('name') name: string) {
    this.jobLifecycleService.deleteJob(name);
    return { message: `Job ${name} deleted`, success: true };
  }

  // Get job statistics
  @Get('stats')
  getStats() {
    const totalJobs = this.jobLifecycleService.getJobCount();
    const runningJobs = this.jobLifecycleService.listAllJobs()
      .filter(name => this.jobManagementService.isJobRunning(name))
      .length;

    return {
      total: totalJobs,
      running: runningJobs,
      stopped: totalJobs - runningJobs,
    };
  }
}

// REST API ENDPOINTS:
// GET    /jobs           → List all jobs
// GET    /jobs/:name     → Get job details
// POST   /jobs/:name/start    → Start job
// POST   /jobs/:name/stop     → Stop job
// POST   /jobs/:name/toggle   → Toggle job
// POST   /jobs/:name/restart  → Restart job
// DELETE /jobs/:name     → Delete job
// GET    /jobs/stats     → Get statistics
```

---

### **Method 6: Monitoring and Health Checks**

```typescript
// ========== JOB MONITORING ==========

@Injectable()
export class JobMonitoringService {
  private readonly logger = new Logger(JobMonitoringService.name);

  constructor(private readonly schedulerRegistry: SchedulerRegistry) {}

  // Monitor all jobs
  monitorJobs() {
    const cronJobs = this.schedulerRegistry.getCronJobs();

    cronJobs.forEach((job, name) => {
      const status = {
        name,
        running: job.running,
        nextRun: job.nextDate().toJSDate(),
        lastRun: job.lastDate()?.toJSDate() || null,
      };

      this.logger.log(`Job Monitor: ${JSON.stringify(status)}`);

      // Alert if job is not running (when it should be)
      if (!job.running) {
        this.logger.warn(`⚠️ Job ${name} is not running!`);
      }
    });
  }

  // Health check for critical jobs
  healthCheck(criticalJobs: string[]): boolean {
    let allHealthy = true;

    for (const jobName of criticalJobs) {
      const exists = this.schedulerRegistry.doesExist('cron', jobName);
      
      if (!exists) {
        this.logger.error(`❌ Critical job ${jobName} not found`);
        allHealthy = false;
        continue;
      }

      const job = this.schedulerRegistry.getCronJob(jobName);
      
      if (!job.running) {
        this.logger.error(`❌ Critical job ${jobName} not running`);
        allHealthy = false;
      } else {
        this.logger.log(`✅ Critical job ${jobName} healthy`);
      }
    }

    return allHealthy;
  }

  // Get jobs scheduled to run in next N minutes
  getUpcomingJobs(minutes: number): Array<{ name: string; nextRun: Date }> {
    const cronJobs = this.schedulerRegistry.getCronJobs();
    const now = new Date();
    const cutoff = new Date(now.getTime() + minutes * 60 * 1000);
    const upcoming = [];

    cronJobs.forEach((job, name) => {
      if (!job.running) return;

      const nextRun = job.nextDate().toJSDate();
      
      if (nextRun <= cutoff) {
        upcoming.push({ name, nextRun });
      }
    });

    return upcoming.sort((a, b) => 
      a.nextRun.getTime() - b.nextRun.getTime()
    );
  }
}

// MONITORING USE CASES:
// - Periodic health checks (every 5 minutes)
// - Alert if critical jobs stopped
// - Dashboard showing job status
// - Upcoming job schedule view
```

**Key Takeaway:** Use **cron names** by adding `name` property in decorator options: `@Cron('0 0 * * *', { name: 'dailyBackup' })` creates uniquely identified job accessible via `SchedulerRegistry`. **Benefits**: **runtime control** (start/stop jobs dynamically with `job.start()`/`job.stop()` without redeploying), **monitoring** (check if running with `job.running`, get next execution with `job.nextDate()`, get last execution with `job.lastDate()`), **dynamic management** (delete with `schedulerRegistry.deleteCronJob(name)`, check existence with `schedulerRegistry.doesExist('cron', name)`), **bulk operations** (get all jobs with `schedulerRegistry.getCronJobs()` returns Map, iterate and manage multiple jobs), **graceful shutdown** (stop all jobs before app termination). **SchedulerRegistry methods**: `getCronJob(name)` retrieves job instance, `getCronJobs()` returns Map of all jobs, `deleteCronJob(name)` removes job, `doesExist('cron', name)` checks if job exists. **CronJob instance methods**: `start()` begins execution, `stop()` halts execution, `running` property shows current state (boolean), `nextDate()` returns next run time (Luxon DateTime), `lastDate()` returns last run time or null, `cronTime.source` shows original cron expression. **Naming conventions**: use camelCase descriptive names ('dailyBackup', 'hourlySync', 'priceUpdate'), names must be unique within job type (can't have two cron jobs with same name but cron/interval/timeout can share names across types), be specific about frequency and purpose. **Use cases**: **conditional execution** (stop job during maintenance, start after config update), **user-controlled jobs** (enable/disable via admin panel, REST API for job management), **monitoring dashboards** (show all jobs status, display next run times, alert if critical jobs stopped), **testing/debugging** (pause jobs during development, restart jobs to test changes), **dynamic configuration** (add jobs based on user settings, remove jobs when features disabled), **health checks** (verify critical jobs running, monitor execution times, alert on failures). **Without names**: jobs are anonymous and cannot be accessed or managed dynamically, only controllable via decorator definition, suitable for simple fire-and-forget tasks that never need runtime management.

</details>

## Intervals & Timeouts

<details>
<summary><strong>10. How do you schedule tasks at intervals using `@Interval()`?</strong></summary>

**Answer:**

Schedule tasks at **fixed intervals** using `@Interval()` decorator: `@Interval(milliseconds)` or `@Interval(name, milliseconds)` for named intervals. The decorator accepts **milliseconds** as parameter (not cron expression): `@Interval(5000)` runs every 5 seconds, `@Interval(60000)` runs every minute. **Execution behavior**: runs **immediately on app startup** (first execution at 0 seconds), then repeats every N milliseconds continuously until app stops. Named intervals allow dynamic management: `@Interval('healthCheck', 30000)` creates named interval accessible via `SchedulerRegistry`. To manage: **get interval** (`schedulerRegistry.getInterval('healthCheck')`), **delete interval** (`schedulerRegistry.deleteInterval('healthCheck')`), **add dynamically** (`schedulerRegistry.addInterval(name, setInterval(callback, ms))`). **Difference from Cron**: intervals use **fixed time between executions** (every 30 seconds regardless of clock time), cron uses **calendar-based scheduling** (specific times like midnight, 9 AM). Use intervals for: **polling** (check status every 10 seconds), **health checks** (ping services every 30 seconds), **metrics collection** (gather stats every minute), **monitoring** (continuous checking), when **specific time doesn't matter** (just need regular execution). Avoid for: tasks needing specific times (use Cron), very high frequency (every second strains resources), calendar-based tasks (daily, weekly, monthly use Cron).

---

### **Interval Scheduling Architecture:**

```
@INTERVAL() EXECUTION FLOW:
┌──────────────────────────────────────────────────────┐
│  App Starts at 10:00:00                              │
└──────────────────┬───────────────────────────────────┘
                   ↓
┌──────────────────────────────────────────────────────┐
│  First Execution: Immediate (10:00:00)               │
│  @Interval(10000) // 10 seconds                      │
└──────────────────┬───────────────────────────────────┘
                   ↓
        Wait 10,000 milliseconds
                   ↓
┌──────────────────────────────────────────────────────┐
│  Second Execution: 10:00:10                          │
└──────────────────┬───────────────────────────────────┘
                   ↓
        Wait 10,000 milliseconds
                   ↓
┌──────────────────────────────────────────────────────┐
│  Third Execution: 10:00:20                           │
└──────────────────┬───────────────────────────────────┘
                   ↓
        Continues until app stops...

KEY CHARACTERISTICS:
✅ First run: Immediately on startup
✅ Subsequent runs: Every N milliseconds
✅ Fixed time between executions
✅ Not tied to specific clock times
✅ Runs continuously until stopped
```

---

### **Method 1: Basic Interval Tasks**

```typescript
// ========== BASIC @INTERVAL() USAGE ==========

import { Injectable, Logger } from '@nestjs/common';
import { Interval } from '@nestjs/schedule';

@Injectable()
export class IntervalTasksService {
  private readonly logger = new Logger(IntervalTasksService.name);

  // Every 5 seconds (5000ms)
  @Interval(5000)
  every5Seconds() {
    this.logger.log('Runs every 5 seconds');
    this.logger.log(`Current time: ${new Date().toISOString()}`);
  }

  // Every 10 seconds
  @Interval(10000)
  every10Seconds() {
    this.logger.log('Runs every 10 seconds');
  }

  // Every 30 seconds
  @Interval(30000)
  every30Seconds() {
    this.logger.log('Runs every 30 seconds');
  }

  // Every minute (60000ms)
  @Interval(60000)
  everyMinute() {
    this.logger.log('Runs every minute');
  }

  // Every 5 minutes (5 * 60 * 1000)
  @Interval(5 * 60 * 1000)
  every5Minutes() {
    this.logger.log('Runs every 5 minutes');
  }

  // Every 15 minutes
  @Interval(15 * 60 * 1000)
  every15Minutes() {
    this.logger.log('Runs every 15 minutes');
  }

  // Every hour (60 * 60 * 1000)
  @Interval(60 * 60 * 1000)
  everyHour() {
    this.logger.log('Runs every hour');
  }
}

// MILLISECONDS CONVERSION:
// 1 second  = 1000ms
// 5 seconds = 5000ms
// 10 seconds = 10000ms
// 30 seconds = 30000ms
// 1 minute  = 60000ms
// 5 minutes = 300000ms
// 1 hour    = 3600000ms
```

---

### **Method 2: Named Intervals for Dynamic Management**

```typescript
// ========== NAMED INTERVALS ==========

@Injectable()
export class NamedIntervalService {
  private readonly logger = new Logger(NamedIntervalService.name);

  // Named interval with string identifier
  @Interval('healthCheck', 30000)  // Every 30 seconds
  healthCheck() {
    this.logger.log('Health check running');
    // Check database, Redis, external services
  }

  // Named interval for metrics collection
  @Interval('metricsCollector', 60000)  // Every minute
  collectMetrics() {
    this.logger.log('Collecting metrics');
    // Gather CPU, memory, request count
  }

  // Named interval for queue monitoring
  @Interval('queueMonitor', 10000)  // Every 10 seconds
  monitorQueue() {
    this.logger.log('Monitoring queue depth');
    // Check job queue, alert if too many pending
  }
}

// Dynamic management via SchedulerRegistry:
@Injectable()
export class IntervalControlService {
  constructor(private readonly schedulerRegistry: SchedulerRegistry) {}

  // Get interval by name
  getInterval(name: string): NodeJS.Timeout {
    return this.schedulerRegistry.getInterval(name);
  }

  // Delete interval by name
  deleteInterval(name: string): void {
    this.schedulerRegistry.deleteInterval(name);
  }

  // Check if interval exists
  intervalExists(name: string): boolean {
    return this.schedulerRegistry.doesExist('interval', name);
  }

  // Get all intervals
  getAllIntervals(): Map<string, NodeJS.Timeout> {
    return this.schedulerRegistry.getIntervals();
  }
}

// BENEFITS OF NAMED INTERVALS:
// ✅ Can be stopped/removed dynamically
// ✅ Can be added at runtime
// ✅ Trackable and manageable
// ✅ Conditional execution
```

---

### **Method 3: Practical Interval Use Cases**

```typescript
// ========== REAL-WORLD INTERVAL EXAMPLES ==========

@Injectable()
export class ProductionIntervalService {
  private readonly logger = new Logger(ProductionIntervalService.name);
  private lastCheck = new Date();

  constructor(
    private readonly databaseService: DatabaseService,
    private readonly cacheService: CacheService,
    private readonly queueService: QueueService,
    private readonly alertService: AlertService,
  ) {}

  // Health check every 30 seconds
  @Interval('healthCheck', 30000)
  async healthCheck() {
    this.logger.log('Performing health check');

    try {
      // Check database connectivity
      await this.databaseService.ping();
      
      // Check cache connectivity
      await this.cacheService.ping();
      
      this.logger.log('✅ All systems healthy');
      
    } catch (error) {
      this.logger.error('❌ Health check failed', error.stack);
      
      // Send alert to ops team
      await this.alertService.sendAlert({
        level: 'critical',
        message: 'Health check failed',
        error: error.message,
      });
    }
  }

  // Monitor queue depth every 10 seconds
  @Interval('queueMonitor', 10000)
  async monitorQueue() {
    const depth = await this.queueService.getQueueDepth();
    
    this.logger.log(`Queue depth: ${depth}`);

    // Alert if queue too deep
    if (depth > 1000) {
      this.logger.warn(`⚠️ Queue depth high: ${depth}`);
      
      await this.alertService.sendAlert({
        level: 'warning',
        message: `Queue depth: ${depth} (threshold: 1000)`,
      });
    }
  }

  // Collect metrics every minute
  @Interval('metricsCollector', 60000)
  async collectMetrics() {
    const metrics = {
      timestamp: new Date(),
      cpu: process.cpuUsage(),
      memory: process.memoryUsage(),
      uptime: process.uptime(),
    };

    this.logger.log(`Metrics: ${JSON.stringify(metrics)}`);
    
    // Store metrics for monitoring dashboard
    await this.databaseService.saveMetrics(metrics);
  }

  // Cache refresh every 5 minutes
  @Interval('cacheRefresh', 5 * 60 * 1000)
  async refreshCache() {
    this.logger.log('Refreshing cache');

    try {
      // Refresh frequently accessed data
      await this.cacheService.refreshTopProducts();
      await this.cacheService.refreshPopularCategories();
      
      this.logger.log('Cache refreshed successfully');
      
    } catch (error) {
      this.logger.error('Cache refresh failed', error.stack);
    }
  }

  // Polling external API every 15 seconds
  @Interval('apiPoller', 15000)
  async pollExternalAPI() {
    this.logger.log('Polling external API');

    try {
      const response = await this.externalAPI.getLatestData();
      
      // Process response
      await this.processData(response);
      
    } catch (error) {
      this.logger.error('API polling failed', error.stack);
    }
  }
}

// USE CASES FOR INTERVALS:
// ✅ Health checks (every 30 sec)
// ✅ Queue monitoring (every 10 sec)
// ✅ Metrics collection (every minute)
// ✅ Cache refresh (every 5 min)
// ✅ API polling (every 15 sec)
// ✅ Status updates (continuous)
```

---

### **Method 4: Adding Intervals Dynamically**

```typescript
// ========== DYNAMIC INTERVAL MANAGEMENT ==========

@Injectable()
export class DynamicIntervalService {
  private readonly logger = new Logger(DynamicIntervalService.name);

  constructor(private readonly schedulerRegistry: SchedulerRegistry) {}

  // Add interval dynamically at runtime
  addInterval(name: string, milliseconds: number, callback: () => void) {
    const interval = setInterval(callback, milliseconds);
    this.schedulerRegistry.addInterval(name, interval);
    
    this.logger.log(
      `Interval ${name} added: every ${milliseconds}ms`
    );
  }

  // Remove interval
  removeInterval(name: string) {
    if (this.schedulerRegistry.doesExist('interval', name)) {
      this.schedulerRegistry.deleteInterval(name);
      this.logger.log(`Interval ${name} removed`);
    } else {
      this.logger.warn(`Interval ${name} does not exist`);
    }
  }

  // Update interval (remove and re-add with new timing)
  updateInterval(
    name: string,
    newMilliseconds: number,
    callback: () => void
  ) {
    this.removeInterval(name);
    this.addInterval(name, newMilliseconds, callback);
    this.logger.log(`Interval ${name} updated`);
  }

  // List all intervals
  listIntervals(): string[] {
    const intervals = this.schedulerRegistry.getIntervals();
    return Array.from(intervals.keys());
  }

  // Remove all intervals
  removeAllIntervals() {
    const intervals = this.schedulerRegistry.getIntervals();
    const names = Array.from(intervals.keys());
    
    names.forEach(name => {
      this.schedulerRegistry.deleteInterval(name);
    });
    
    this.logger.log(`Removed ${names.length} intervals`);
  }
}

// Controller for dynamic management
@Controller('intervals')
export class IntervalsController {
  constructor(
    private readonly dynamicIntervalService: DynamicIntervalService,
  ) {}

  @Post()
  addInterval(@Body() dto: { name: string; milliseconds: number }) {
    this.dynamicIntervalService.addInterval(
      dto.name,
      dto.milliseconds,
      () => {
        console.log(`Dynamic interval ${dto.name} executing`);
      }
    );
    return { message: 'Interval added', name: dto.name };
  }

  @Delete(':name')
  removeInterval(@Param('name') name: string) {
    this.dynamicIntervalService.removeInterval(name);
    return { message: 'Interval removed', name };
  }

  @Get()
  listIntervals() {
    return this.dynamicIntervalService.listIntervals();
  }
}

// DYNAMIC MANAGEMENT USE CASES:
// - Add intervals based on user configuration
// - Remove intervals when features disabled
// - Update interval timing without restart
// - Conditional interval creation
```

---

### **Method 5: Interval vs Cron Comparison**

```typescript
// ========== INTERVAL VS CRON ==========

@Injectable()
export class ComparisonService {
  private readonly logger = new Logger(ComparisonService.name);

  // INTERVAL: Fixed time between executions
  @Interval(60000)  // Every minute
  intervalExample() {
    // Runs every 60 seconds from previous execution
    // 10:00:00, 10:01:00, 10:02:00, 10:03:00...
    // Not tied to clock time
    this.logger.log('Interval: every minute');
  }

  // CRON: Specific clock times
  @Cron('0 * * * * *')  // Every minute at second 0
  cronExample() {
    // Runs at specific times: XX:00:00
    // 10:00:00, 10:01:00, 10:02:00, 10:03:00...
    // Tied to clock time
    this.logger.log('Cron: every minute at second 0');
  }

  // SCENARIO 1: Health check (use Interval)
  @Interval(30000)  // Every 30 seconds
  healthCheckInterval() {
    // Don't care about specific time, just check regularly
    this.logger.log('Health check (interval)');
  }

  // SCENARIO 2: Daily report (use Cron)
  @Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT)
  dailyReportCron() {
    // Must run at specific time: midnight
    this.logger.log('Daily report (cron)');
  }
}

// WHEN TO USE INTERVAL:
// ✅ Fixed frequency (every 30 seconds, every 5 minutes)
// ✅ Polling, monitoring, health checks
// ✅ Don't care about specific clock time
// ✅ Continuous regular execution
// ✅ Simple timing requirements

// WHEN TO USE CRON:
// ✅ Specific times (midnight, 9 AM, noon)
// ✅ Calendar-based (daily, weekly, monthly)
// ✅ Business hours (9-5 weekdays)
// ✅ Complex schedules (first of month, quarterly)
// ✅ Time-sensitive tasks (reports at 2 AM)
```

**Key Takeaway:** Schedule tasks at **fixed intervals** using `@Interval(milliseconds)` decorator accepting milliseconds as parameter: `@Interval(5000)` every 5 seconds, `@Interval(60000)` every minute, `@Interval(5 * 60 * 1000)` every 5 minutes. **Execution behavior**: runs **immediately on app startup** (first execution at 0 seconds from start), then repeats every N milliseconds continuously until app stops, **fixed time between executions** not tied to clock time (if app starts at 10:37:42, runs at 10:37:42, 10:38:42, 10:39:42 for 60000ms interval). **Named intervals**: use `@Interval('healthCheck', 30000)` for dynamic management via SchedulerRegistry, access with `schedulerRegistry.getInterval('healthCheck')`, delete with `schedulerRegistry.deleteInterval('healthCheck')`, check existence with `schedulerRegistry.doesExist('interval', 'healthCheck')`, get all with `schedulerRegistry.getIntervals()` returns Map. **Dynamic intervals**: add at runtime with `schedulerRegistry.addInterval(name, setInterval(callback, ms))`, remove with `schedulerRegistry.deleteInterval(name)`, useful for user-configured schedules or conditional intervals. **Use cases**: **health checks** (ping database/Redis every 30 seconds), **queue monitoring** (check depth every 10 seconds, alert if > threshold), **metrics collection** (gather CPU/memory every minute), **polling** (check external API every 15 seconds for updates), **cache refresh** (update frequently accessed data every 5 minutes), **status monitoring** (continuous checking of system state), when **specific time doesn't matter** (just need regular periodic execution). **Difference from Cron**: intervals use fixed duration between executions (mechanical timing), cron uses calendar-based scheduling (specific clock times like midnight, 9 AM), intervals not aware of time zones or daylight saving, intervals simpler for regular polling/monitoring. **Avoid intervals for**: tasks needing specific times (use Cron for "midnight daily" or "9 AM weekdays"), very high frequency every second (use with caution, can strain resources), calendar-based tasks (daily, weekly, monthly better with Cron), time-sensitive operations (reports at 2 AM use Cron). **Milliseconds reference**: 1s=1000ms, 5s=5000ms, 10s=10000ms, 30s=30000ms, 1min=60000ms, 5min=300000ms, 15min=900000ms, 1hr=3600000ms.

</details>

<details>
<summary><strong>11. What is the difference between `@Interval()` and `@Cron()`?</strong></summary>

**Answer:**

`@Interval()` and `@Cron()` differ in **trigger mechanism**: `@Interval()` uses **fixed time intervals** in milliseconds (`@Interval(60000)` = every 60 seconds from last execution completion), while `@Cron()` uses **calendar-based scheduling** with cron expressions (`@Cron('0 * * * *')` = every hour at minute 0). **Key differences**: **Timing** - Interval runs every N milliseconds continuously from app start (first run immediate, subsequent runs after interval), Cron runs at specific times/dates (midnight, 9 AM weekdays, 1st of month). **Syntax** - Interval takes milliseconds number (5000, 60000, 300000), Cron takes expression string or enum ('0 0 * * *', CronExpression.EVERY_HOUR). **Use cases** - Interval for fixed-frequency tasks (health checks every 30 seconds, metrics every minute, polling), Cron for calendar-based tasks (daily reports at 2 AM, weekly summaries, monthly billing). **Precision** - Interval drifts over time (if task takes 2 seconds with 60-second interval, actual interval is 62 seconds), Cron runs at exact time (always at 00:00:00 regardless of previous run duration). **First execution** - Interval runs immediately on app start then after interval, Cron waits until scheduled time. **Business logic** - Interval ignores calendar (weekends, business hours), Cron is calendar-aware (weekdays only, business hours, specific dates).

---

### **@Interval() vs @Cron() Comparison:**

```
@INTERVAL() - FIXED TIME INTERVALS:
┌─────────────────────────────────────────────────────┐
│  Trigger: Every N milliseconds                      │
│  Syntax: @Interval(60000)  // 60 seconds            │
│                                                     │
│  Timeline:                                          │
│  00:00:00 → First run (immediate on startup)        │
│  00:01:00 → Second run (60 seconds later)           │
│  00:02:00 → Third run (60 seconds later)            │
│  00:03:00 → Fourth run (60 seconds later)           │
│  ...continues until app stops                       │
│                                                     │
│  Characteristics:                                   │
│  ✓ Fixed frequency (every N ms)                     │
│  ✓ Runs immediately on startup                      │
│  ✓ Continuous execution                             │
│  ✓ Time drift (if task duration varies)            │
│  ✗ Not calendar-aware                               │
│  ✗ Can't specify exact time                         │
└─────────────────────────────────────────────────────┘

@CRON() - CALENDAR-BASED SCHEDULING:
┌─────────────────────────────────────────────────────┐
│  Trigger: Specific times/dates                      │
│  Syntax: @Cron('0 0 * * *')  // Midnight daily      │
│                                                     │
│  Timeline:                                          │
│  Jan 1, 00:00:00 → First run (at midnight)          │
│  Jan 2, 00:00:00 → Second run (next midnight)       │
│  Jan 3, 00:00:00 → Third run (next midnight)        │
│  ...runs at midnight every day                      │
│                                                     │
│  Characteristics:                                   │
│  ✓ Specific times (9 AM, midnight, etc.)            │
│  ✓ Calendar-aware (weekdays, months)                │
│  ✓ No time drift (exact time always)                │
│  ✓ Waits for scheduled time                         │
│  ✗ Not fixed frequency                              │
│  ✗ Doesn't run until scheduled                      │
└─────────────────────────────────────────────────────┘

COMPARISON TABLE:
┌─────────────────┬──────────────────┬─────────────────┐
│ Feature         │ @Interval()      │ @Cron()         │
├─────────────────┼──────────────────┼─────────────────┤
│ Trigger         │ Fixed ms         │ Calendar time   │
│ Syntax          │ Milliseconds     │ Cron expression │
│ First run       │ Immediate        │ Wait for time   │
│ Precision       │ Relative (drift) │ Absolute (exact)│
│ Calendar-aware  │ No               │ Yes             │
│ Weekday support │ No               │ Yes             │
│ Specific time   │ No               │ Yes             │
│ Fixed frequency │ Yes              │ No              │
└─────────────────┴──────────────────┴─────────────────┘
```

---

### **Method 1: @Interval() - Fixed Time Intervals**

```typescript
// ========== @INTERVAL() EXAMPLES ==========

import { Injectable, Logger } from '@nestjs/common';
import { Interval } from '@nestjs/schedule';

@Injectable()
export class IntervalService {
  private readonly logger = new Logger(IntervalService.name);

  // Every 10 seconds
  @Interval(10000)
  every10Seconds() {
    this.logger.log(`Interval: ${new Date().toISOString()}`);
    // Runs immediately on startup, then every 10 seconds
  }

  // Every 30 seconds (health check)
  @Interval(30000)
  healthCheck() {
    this.logger.log('Health check running');
    // Use case: Monitor service health every 30 seconds
  }

  // Every minute (metrics collection)
  @Interval(60000)
  collectMetrics() {
    this.logger.log('Collecting metrics');
    // Use case: Gather system metrics every minute
  }

  // Every 5 minutes (cache refresh)
  @Interval(5 * 60 * 1000)
  refreshCache() {
    this.logger.log('Refreshing cache');
    // Use case: Update cache every 5 minutes
  }
}

// INTERVAL CHARACTERISTICS:
// - First execution: Immediately on app start (time 0)
// - Second execution: After N milliseconds
// - Third execution: After another N milliseconds
// - Pattern: time 0, time N, time 2N, time 3N, ...
// - No calendar awareness (runs on weekends, holidays, etc.)
// - Time drift: If task takes 2s with 60s interval, actual is 62s

// WHEN TO USE @Interval():
// ✅ Fixed frequency needed (every 30 seconds, every minute)
// ✅ Continuous monitoring (health checks, queue depth)
// ✅ Polling (check external service status)
// ✅ Regular updates (metrics, cache refresh)
// ✅ Don't care about specific time (just regular checks)
```

---

### **Method 2: @Cron() - Calendar-Based Scheduling**

```typescript
// ========== @CRON() EXAMPLES ==========

import { Cron, CronExpression } from '@nestjs/schedule';

@Injectable()
export class CronService {
  private readonly logger = new Logger(CronService.name);

  // Every hour (at minute 0)
  @Cron(CronExpression.EVERY_HOUR)
  everyHour() {
    this.logger.log(`Cron: ${new Date().toISOString()}`);
    // Runs at 00:00, 01:00, 02:00, etc.
  }

  // Midnight daily
  @Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT)
  dailyMidnight() {
    this.logger.log('Daily cleanup at midnight');
    // Use case: Daily cleanup, reports at specific time
  }

  // Weekdays at 9 AM
  @Cron('0 9 * * 1-5')
  weekdayMornings() {
    this.logger.log('Weekday morning task');
    // Use case: Business hours tasks (Monday-Friday only)
  }

  // 1st of month at midnight
  @Cron('0 0 1 * *')
  monthlyBilling() {
    this.logger.log('Monthly billing');
    // Use case: Monthly reports, subscription billing
  }
}

// CRON CHARACTERISTICS:
// - First execution: Waits for scheduled time
// - Subsequent executions: At exact scheduled times
// - Pattern: Specific calendar times (9 AM, midnight, etc.)
// - Calendar-aware (weekdays vs weekends, specific months)
// - No time drift: Always runs at exact time specified
// - Waits until next scheduled time (may be hours or days)

// WHEN TO USE @Cron():
// ✅ Specific time needed (9 AM, midnight, noon)
// ✅ Calendar-based (daily, weekly, monthly, yearly)
// ✅ Business hours (9-5 weekdays only)
// ✅ Off-peak processing (2-4 AM when traffic low)
// ✅ Exact timing matters (reports at midnight sharp)
```

---

### **Method 3: Time Drift Comparison**

```typescript
// ========== TIME DRIFT EXAMPLE ==========

@Injectable()
export class TimeDriftService {
  private readonly logger = new Logger(TimeDriftService.name);

  // @Interval() - TIME DRIFT
  @Interval(60000)  // Every 60 seconds
  async intervalWithDrift() {
    const start = Date.now();
    this.logger.log(`Interval start: ${new Date().toISOString()}`);
    
    // Simulate task that takes 5 seconds
    await new Promise(resolve => setTimeout(resolve, 5000));
    
    const duration = Date.now() - start;
    this.logger.log(`Interval end: ${duration}ms`);
    // Next run: 60 seconds AFTER this completes
    // Actual interval: 60s + 5s = 65 seconds
  }

  // @Cron() - NO TIME DRIFT
  @Cron('0 * * * *')  // Every hour at minute 0
  async cronNoDrift() {
    const start = Date.now();
    this.logger.log(`Cron start: ${new Date().toISOString()}`);
    
    // Even if task takes 5 seconds...
    await new Promise(resolve => setTimeout(resolve, 5000));
    
    const duration = Date.now() - start;
    this.logger.log(`Cron end: ${duration}ms`);
    // Next run: Next hour at :00 (regardless of duration)
    // Always runs at XX:00:00 sharp
  }
}

// DRIFT IMPACT:
// Interval over 24 hours with 60s interval + 5s task:
// - Expected: 1440 executions (24 * 60)
// - Actual: ~1330 executions (drift accumulates)
// 
// Cron over 24 hours with hourly schedule:
// - Expected: 24 executions
// - Actual: 24 executions (no drift, exact times)
```

---

### **Method 4: Use Case Decision Matrix**

```typescript
// ========== WHEN TO USE EACH ==========

// SCENARIO 1: Health check every 30 seconds
// SOLUTION: @Interval() ✅
@Interval(30000)
healthCheck() {
  // Fixed frequency, don't care about specific time
}

// SCENARIO 2: Daily backup at 2 AM
// SOLUTION: @Cron() ✅
@Cron(CronExpression.EVERY_DAY_AT_2AM)
dailyBackup() {
  // Specific time (off-peak hours)
}

// SCENARIO 3: Check queue depth every minute
// SOLUTION: @Interval() ✅
@Interval(60000)
checkQueue() {
  // Continuous monitoring, fixed frequency
}

// SCENARIO 4: Generate weekly reports on Monday 8 AM
// SOLUTION: @Cron() ✅
@Cron('0 8 * * 1')
weeklyReport() {
  // Specific day and time
}

// SCENARIO 5: Refresh cache every 5 minutes
// SOLUTION: @Interval() ✅
@Interval(5 * 60 * 1000)
refreshCache() {
  // Regular updates, don't need exact time
}

// SCENARIO 6: Process monthly subscriptions on 1st
// SOLUTION: @Cron() ✅
@Cron('0 0 1 * *')
monthlySubscriptions() {
  // Specific date (1st of month)
}

// SCENARIO 7: Monitor API endpoint every 15 seconds
// SOLUTION: @Interval() ✅
@Interval(15000)
monitorAPI() {
  // High-frequency monitoring
}

// SCENARIO 8: Send reports during business hours only
// SOLUTION: @Cron() ✅
@Cron('0 9-17 * * 1-5')
businessHoursTask() {
  // Calendar-aware (weekdays 9-5)
}
```

---

### **Method 5: Combining Both Decorators**

```typescript
// ========== USING BOTH IN ONE SERVICE ==========

@Injectable()
export class HybridSchedulerService {
  private readonly logger = new Logger(HybridSchedulerService.name);

  constructor(
    private readonly healthService: HealthService,
    private readonly reportsService: ReportsService,
  ) {}

  // Interval: Continuous monitoring
  @Interval(30000)
  async monitorHealth() {
    this.logger.log('Monitoring health (every 30s)');
    const status = await this.healthService.check();
    
    if (!status.healthy) {
      this.logger.error('Health check failed!');
      // Alert ops team
    }
  }

  // Cron: Daily report at specific time
  @Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT)
  async generateDailyReport() {
    this.logger.log('Generating daily report (midnight)');
    await this.reportsService.generateDailyReport();
  }

  // Interval: Metrics every minute
  @Interval(60000)
  collectMetrics() {
    this.logger.log('Collecting metrics (every minute)');
    // Gather system metrics continuously
  }

  // Cron: Cleanup at off-peak hours
  @Cron(CronExpression.EVERY_DAY_AT_2AM)
  async cleanupData() {
    this.logger.log('Cleaning up data (2 AM)');
    // Heavy cleanup during off-peak hours
  }
}

// BEST PRACTICE:
// - Use @Interval() for continuous, fixed-frequency tasks
// - Use @Cron() for calendar-based, time-specific tasks
// - Both can coexist in same service
// - Choose based on requirements, not preference
```

**Key Takeaway:** `@Interval()` and `@Cron()` differ fundamentally in **trigger mechanism**: `@Interval(milliseconds)` uses **fixed time intervals** (every N milliseconds continuously: `@Interval(60000)` = every 60 seconds, `@Interval(5 * 60 * 1000)` = every 5 minutes), runs immediately on app start then every interval, creates **time drift** if task duration varies (60-second interval with 5-second task = 65-second actual interval). `@Cron(expression)` uses **calendar-based scheduling** (specific times/dates: `@Cron('0 0 * * *')` = midnight daily, `@Cron('0 9 * * 1-5')` = weekdays 9 AM), waits for scheduled time before first run, **no time drift** as always runs at exact time regardless of previous duration. **Key differences**: **Timing** - Interval is relative to last execution (N milliseconds after completion), Cron is absolute calendar time (9 AM, midnight, 1st of month). **Syntax** - Interval takes milliseconds number (simple numeric value), Cron takes expression string or enum (complex time pattern). **First execution** - Interval runs immediately at startup (time 0), Cron waits until next scheduled time (may be hours away). **Precision** - Interval accumulates drift over time (if task takes longer/shorter, interval shifts), Cron maintains exact timing (always at specified time sharp). **Calendar awareness** - Interval ignores calendar (runs 24/7 including weekends/holidays), Cron is calendar-aware (weekdays only, business hours, specific months). **Use @Interval() for**: fixed-frequency tasks (health checks every 30 seconds), continuous monitoring (queue depth every minute), polling (API status checks), regular updates (cache refresh every 5 minutes), when exact time doesn't matter. **Use @Cron() for**: specific times (daily at 2 AM, weekdays at 9 AM), calendar-based schedules (weekly Monday, monthly 1st), business hours (9-5 weekdays only), off-peak processing (2-4 AM), when exact timing critical (reports at midnight sharp).

</details>

<details>
<summary><strong>12. How do you schedule one-time delayed tasks using `@Timeout()`?</strong></summary>

**Answer:**

Schedule one-time delayed tasks using `@Timeout(milliseconds)` decorator: `@Timeout(5000)` runs the method **once** after 5 seconds from app startup. The decorator takes **milliseconds** (5000 = 5 seconds, 60000 = 1 minute, 600000 = 10 minutes) or **name and milliseconds** (`@Timeout('initCache', 10000)`). Unlike `@Interval()` (repeats continuously) or `@Cron()` (repeats on schedule), `@Timeout()` executes **exactly once** then never again until app restarts. **Use cases**: **initialization tasks** (pre-load cache, warm connections, seed data), **delayed startup** (wait for dependencies ready, give time for services to start), **one-time operations** (send welcome email after delay, cleanup temp files after processing). The method can be **async** for database/API operations, has access to **dependency injection** (inject services via constructor), supports **named timeouts** for dynamic management via SchedulerRegistry (delete timeout: `schedulerRegistry.deleteTimeout('initCache')`). **Timing**: starts counting from app startup (time 0), executes after N milliseconds, never executes again. For recurring tasks use `@Interval()` or `@Cron()`, for one-time delayed use `@Timeout()`.

---

### **@Timeout() - One-Time Delayed Execution:**

```
TIMEOUT EXECUTION FLOW:
┌─────────────────────────────────────────────────────┐
│  App starts at time 0                               │
│  @Timeout(5000) registered                          │
└──────────────────┬──────────────────────────────────┘
                   ↓
         ⏰ Wait 5000ms (5 seconds)
                   ↓
┌─────────────────────────────────────────────────────┐
│  Time: 5 seconds                                    │
│  Method executes ONCE                               │
│  Task completed                                     │
└──────────────────┬──────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────┐
│  Method NEVER runs again                            │
│  (until app restarts)                               │
└─────────────────────────────────────────────────────┘

COMPARISON:
┌───────────────────┬────────────┬─────────────────────┐
│                   │ Executions │ Use Case            │
├───────────────────┼────────────┼─────────────────────┤
│ @Timeout(5000)    │ Once       │ Initialization      │
│ @Interval(5000)   │ Continuous │ Monitoring          │
│ @Cron('*/5 * * *')│ Scheduled  │ Periodic tasks      │
└───────────────────┴────────────┴─────────────────────┘
```

---

### **Method 1: Basic @Timeout() Usage**

```typescript
// ========== BASIC TIMEOUT ==========

import { Injectable, Logger } from '@nestjs/common';
import { Timeout } from '@nestjs/schedule';

@Injectable()
export class TimeoutService {
  private readonly logger = new Logger(TimeoutService.name);

  // Basic timeout - 5 seconds after startup
  @Timeout(5000)
  afterFiveSeconds() {
    this.logger.log('Timeout executed after 5 seconds');
    this.logger.log(`Executed at: ${new Date().toISOString()}`);
    // Runs once, 5 seconds after app starts
  }

  // 10 second timeout
  @Timeout(10000)
  afterTenSeconds() {
    this.logger.log('Timeout executed after 10 seconds');
  }

  // 1 minute timeout
  @Timeout(60000)
  afterOneMinute() {
    this.logger.log('Timeout executed after 1 minute');
  }

  // 5 minute timeout
  @Timeout(5 * 60 * 1000)
  afterFiveMinutes() {
    this.logger.log('Timeout executed after 5 minutes');
  }
}

// EXECUTION TIMELINE:
// 00:00:00 - App starts
// 00:00:05 - afterFiveSeconds() runs
// 00:00:10 - afterTenSeconds() runs
// 00:01:00 - afterOneMinute() runs
// 00:05:00 - afterFiveMinutes() runs
// 00:05:01+ - Nothing runs (all timeouts completed)
```

---

### **Method 2: Named Timeouts for Dynamic Management**

```typescript
// ========== NAMED TIMEOUTS ==========

@Injectable()
export class NamedTimeoutService {
  private readonly logger = new Logger(NamedTimeoutService.name);

  // Named timeout for management
  @Timeout('cacheInitialization', 10000)
  initializeCache() {
    this.logger.log('Initializing cache after 10 seconds');
    // Can be cancelled before execution if needed
  }

  // Another named timeout
  @Timeout('warmupConnections', 5000)
  warmupDatabaseConnections() {
    this.logger.log('Warming up database connections');
  }
}

// Dynamic management via SchedulerRegistry
@Injectable()
export class TimeoutManagerService {
  constructor(private readonly schedulerRegistry: SchedulerRegistry) {}

  // Cancel a timeout before it executes
  cancelTimeout(name: string) {
    try {
      this.schedulerRegistry.deleteTimeout(name);
      console.log(`Timeout ${name} cancelled`);
    } catch (error) {
      console.log(`Timeout ${name} not found or already executed`);
    }
  }

  // Check if timeout exists (hasn't executed yet)
  timeoutExists(name: string): boolean {
    return this.schedulerRegistry.doesExist('timeout', name);
  }
}

// USE CASE: Cancel initialization if condition met
@Injectable()
export class ConditionalTimeoutService {
  constructor(
    private readonly timeoutManager: TimeoutManagerService,
    private readonly configService: ConfigService,
  ) {}

  @Timeout('conditionalInit', 30000)
  conditionalInitialization() {
    // This will run after 30 seconds
    // UNLESS cancelled before then
  }

  onModuleInit() {
    // If fast startup enabled, cancel the timeout
    if (this.configService.get('FAST_STARTUP')) {
      this.timeoutManager.cancelTimeout('conditionalInit');
    }
  }
}
```

---

### **Method 3: Initialization Tasks with @Timeout()**

```typescript
// ========== INITIALIZATION USE CASES ==========

@Injectable()
export class InitializationService {
  private readonly logger = new Logger(InitializationService.name);

  constructor(
    private readonly cacheService: CacheService,
    private readonly dbService: DatabaseService,
    private readonly emailService: EmailService,
  ) {}

  // Cache warming: Pre-load frequently accessed data
  @Timeout(5000)
  async warmCache() {
    this.logger.log('Warming cache after 5 seconds');
    
    try {
      // Load top products into cache
      const topProducts = await this.dbService.getTopProducts(100);
      await this.cacheService.setMany('products:top', topProducts);
      
      // Load configuration
      const config = await this.dbService.getConfig();
      await this.cacheService.set('config', config);
      
      this.logger.log('Cache warmed successfully');
    } catch (error) {
      this.logger.error('Cache warming failed', error.stack);
    }
  }

  // Connection pool warm-up
  @Timeout(3000)
  async warmupConnections() {
    this.logger.log('Warming up database connections');
    
    // Execute dummy queries to establish connections
    await this.dbService.query('SELECT 1');
    
    this.logger.log('Database connections ready');
  }

  // Send startup notification
  @Timeout(10000)
  async sendStartupNotification() {
    this.logger.log('Sending startup notification');
    
    await this.emailService.sendEmail({
      to: 'ops@company.com',
      subject: 'Application Started',
      text: `Application started at ${new Date().toISOString()}`,
    });
  }

  // Initial data sync (after app fully started)
  @Timeout(30000)
  async initialDataSync() {
    this.logger.log('Performing initial data sync');
    
    // Wait 30 seconds for app to fully start
    // Then sync initial data from external API
    await this.syncFromExternalAPI();
  }
}

// WHY USE TIMEOUTS FOR INITIALIZATION:
// ✅ Doesn't block app startup (runs after delay)
// ✅ Gives time for dependencies to be ready
// ✅ Can stagger initialization tasks (5s, 10s, 30s)
// ✅ Non-critical tasks don't delay app availability
```

---

### **Method 4: Async Timeout with Error Handling**

```typescript
// ========== ASYNC TIMEOUT WITH ERROR HANDLING ==========

@Injectable()
export class AsyncTimeoutService {
  private readonly logger = new Logger(AsyncTimeoutService.name);

  constructor(
    private readonly dataService: DataService,
    private readonly notificationService: NotificationService,
  ) {}

  @Timeout('dataPreload', 15000)
  async preloadData() {
    this.logger.log('Starting data preload after 15 seconds');
    
    const startTime = Date.now();
    let recordsLoaded = 0;

    try {
      // Preload user data
      const users = await this.dataService.getAllActiveUsers();
      await this.cacheData('users', users);
      recordsLoaded += users.length;
      this.logger.log(`Loaded ${users.length} users`);

      // Preload product catalog
      const products = await this.dataService.getAllProducts();
      await this.cacheData('products', products);
      recordsLoaded += products.length;
      this.logger.log(`Loaded ${products.length} products`);

      // Preload categories
      const categories = await this.dataService.getCategories();
      await this.cacheData('categories', categories);
      recordsLoaded += categories.length;
      this.logger.log(`Loaded ${categories.length} categories`);

      const duration = Date.now() - startTime;
      this.logger.log(
        `Data preload completed: ${recordsLoaded} records in ${duration}ms`,
      );

      // Send success notification
      await this.notificationService.send({
        message: `Data preload successful: ${recordsLoaded} records`,
      });
      
    } catch (error) {
      this.logger.error('Data preload failed', error.stack);
      
      // Send failure alert
      await this.notificationService.sendAlert({
        level: 'error',
        message: `Data preload failed: ${error.message}`,
      });
      
      // Don't throw - timeout failures shouldn't crash app
    }
  }

  private async cacheData(key: string, data: any[]): Promise<void> {
    // Cache implementation
  }
}
```

---

### **Method 5: Timeout vs Interval vs Cron Comparison**

```typescript
// ========== COMPARISON SCENARIOS ==========

@Injectable()
export class ComparisonService {
  private readonly logger = new Logger(ComparisonService.name);

  // SCENARIO 1: One-time cache initialization
  // USE: @Timeout() ✅
  @Timeout(5000)
  initCache() {
    this.logger.log('Initialize cache once after startup');
    // Runs once, 5 seconds after app starts
  }

  // SCENARIO 2: Continuous health monitoring
  // USE: @Interval() ✅
  @Interval(30000)
  monitorHealth() {
    this.logger.log('Monitor health every 30 seconds');
    // Runs continuously: 0s, 30s, 60s, 90s, ...
  }

  // SCENARIO 3: Daily backup at 2 AM
  // USE: @Cron() ✅
  @Cron(CronExpression.EVERY_DAY_AT_2AM)
  dailyBackup() {
    this.logger.log('Daily backup at 2 AM');
    // Runs every day at exactly 02:00:00
  }

  // SCENARIO 4: Send welcome email after user signup (one-time)
  // USE: @Timeout() ✅ (but better use message queue)
  @Timeout(60000)
  sendDelayedEmail() {
    // Actually, for user-triggered one-time tasks, use message queue
    // @Timeout() is for app-level one-time tasks
  }

  // SCENARIO 5: Warm up connections on startup
  // USE: @Timeout() ✅
  @Timeout(3000)
  warmConnections() {
    this.logger.log('Warm up connections once');
    // Perfect for one-time initialization
  }
}

// DECISION MATRIX:
// ┌──────────────────────┬─────────────┬───────────────┬──────────┐
// │ Requirement          │ @Timeout()  │ @Interval()   │ @Cron()  │
// ├──────────────────────┼─────────────┼───────────────┼──────────┤
// │ Run once             │ ✅ Yes      │ ❌ No          │ ❌ No     │
// │ Delayed execution    │ ✅ Yes      │ ❌ Immediate   │ ⏰ Wait   │
// │ Repeat execution     │ ❌ No       │ ✅ Yes         │ ✅ Yes    │
// │ Fixed frequency      │ ❌ N/A      │ ✅ Yes         │ ❌ No     │
// │ Calendar-based       │ ❌ N/A      │ ❌ No          │ ✅ Yes    │
// │ Initialization       │ ✅ Perfect  │ ❌ Wrong tool  │ ❌ Wrong  │
// └──────────────────────┴─────────────┴───────────────┴──────────┘
```

**Key Takeaway:** Schedule one-time delayed tasks using `@Timeout(milliseconds)` decorator which executes method **exactly once** after specified delay from app startup: `@Timeout(5000)` runs after 5 seconds, `@Timeout(60000)` after 1 minute, `@Timeout(5 * 60 * 1000)` after 5 minutes. **Syntax**: `@Timeout(milliseconds)` for anonymous timeout or `@Timeout('name', milliseconds)` for named timeout manageable via SchedulerRegistry (can cancel before execution with `schedulerRegistry.deleteTimeout('name')`, check existence with `schedulerRegistry.doesExist('timeout', 'name')`). **Execution behavior**: timer starts at app startup (time 0), waits N milliseconds, executes method once, **never executes again** until app restarts (unlike `@Interval()` which repeats continuously or `@Cron()` which repeats on schedule). **Use cases**: **initialization tasks** (cache warming after 5 seconds to pre-load frequently accessed data, connection pool warmup after 3 seconds, initial data sync after 30 seconds), **delayed startup** (give time for dependencies to be ready, stagger initialization tasks to avoid startup bottleneck, non-critical tasks that shouldn't block app availability), **one-time operations** (send startup notification after 10 seconds, perform initial health check after delay, cleanup temporary startup files). **Method features**: can be **async** for database/API operations with await, has full **dependency injection** access (inject repositories, services, cache, email via constructor), supports **error handling** with try-catch (failures shouldn't crash app, log errors and continue), use **Logger** for tracking execution. **Comparison**: **@Timeout()** runs once after delay (one-time initialization), **@Interval()** runs continuously every N ms (monitoring, polling), **@Cron()** runs at specific times repeatedly (daily reports, scheduled tasks). **Best practices**: use for app-level initialization not user-triggered tasks (for user actions use message queues), stagger multiple timeouts (3s, 5s, 10s, 30s) to spread load, don't throw errors from timeout methods (log and continue), combine with named timeouts for cancellation capability if conditions change.

</details>

## Dynamic Scheduling

<details>
<summary><strong>13. How do you dynamically schedule/unschedule jobs?</strong></summary>

**Answer:**

Dynamically schedule/unschedule jobs using **SchedulerRegistry** service: inject `SchedulerRegistry` from `@nestjs/schedule`, use methods **addCronJob**/deleteCronJob, **addInterval**/deleteInterval, **addTimeout**/deleteTimeout to manage jobs at runtime. **Scheduling**: create job with `new CronJob()` (cron), `setInterval()` (interval), `setTimeout()` (timeout), add to registry with `schedulerRegistry.addCronJob(name, job)`, start with `job.start()`. **Unscheduling**: retrieve job with `schedulerRegistry.getCronJob(name)`, stop with `job.stop()`, delete with `schedulerRegistry.deleteCronJob(name)`. **Use cases**: **user preferences** (users enable/disable scheduled reports), **configuration changes** (admin modifies backup schedule), **conditional scheduling** (enable jobs based on feature flags), **dynamic frequency** (adjust monitoring interval based on load), **multi-tenant** (each tenant has custom schedule). Unlike static decorators (`@Cron()`, `@Interval()`, `@Timeout()`) which are **compile-time** (defined in code, always active), dynamic scheduling is **runtime** (add/remove jobs while app running based on data/conditions). Methods return void (no confirmation), must check existence with `doesExist()` before operations to avoid errors. **Pattern**: create job → add to registry → start → stop when needed → delete from registry.

---

### **Dynamic Scheduling Architecture:**

```
STATIC DECORATORS (Compile-time):
┌─────────────────────────────────────────────────────┐
│  @Cron('0 0 * * *')                                 │
│  dailyBackup() { ... }                              │
│  ↓                                                  │
│  Defined in code                                    │
│  Always active when app runs                        │
│  Cannot change without redeploying                  │
└─────────────────────────────────────────────────────┘

DYNAMIC SCHEDULING (Runtime):
┌─────────────────────────────────────────────────────┐
│  SchedulerRegistry                                  │
│  ↓                                                  │
│  addCronJob('userBackup', cronJob) ← Create         │
│  getCronJob('userBackup').start()  ← Start         │
│  getCronJob('userBackup').stop()   ← Stop          │
│  deleteCronJob('userBackup')       ← Delete        │
│  ↓                                                  │
│  Jobs managed at runtime                            │
│  Add/remove based on conditions                     │
│  No redeployment needed                             │
└─────────────────────────────────────────────────────┘

SCHEDULER REGISTRY OPERATIONS:
┌─────────────────────────────────────────────────────┐
│                                                     │
│  CREATE → ADD → START → STOP → DELETE              │
│                                                     │
│  1. new CronJob(pattern, callback)                 │
│  2. schedulerRegistry.addCronJob(name, job)        │
│  3. job.start()                                     │
│  4. job.stop()                                      │
│  5. schedulerRegistry.deleteCronJob(name)          │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

### **Method 1: Dynamic Cron Job Management**

```typescript
// ========== DYNAMIC CRON JOBS ==========

import { Injectable, Logger } from '@nestjs/common';
import { SchedulerRegistry } from '@nestjs/schedule';
import { CronJob } from 'cron';

@Injectable()
export class DynamicSchedulerService {
  private readonly logger = new Logger(DynamicSchedulerService.name);

  constructor(private readonly schedulerRegistry: SchedulerRegistry) {}

  // Add a cron job dynamically
  addDynamicCronJob(name: string, cronTime: string) {
    // Create CronJob instance
    const job = new CronJob(cronTime, () => {
      this.logger.log(`Dynamic job ${name} executing at ${new Date().toISOString()}`);
      // Your job logic here
    });

    // Add to registry
    this.schedulerRegistry.addCronJob(name, job);
    
    // Start the job
    job.start();
    
    this.logger.log(`Cron job ${name} added with pattern: ${cronTime}`);
  }

  // Remove a cron job dynamically
  removeDynamicCronJob(name: string) {
    // Check if job exists
    if (this.schedulerRegistry.doesExist('cron', name)) {
      // Stop the job
      const job = this.schedulerRegistry.getCronJob(name);
      job.stop();
      
      // Delete from registry
      this.schedulerRegistry.deleteCronJob(name);
      
      this.logger.log(`Cron job ${name} removed`);
    } else {
      this.logger.warn(`Cron job ${name} does not exist`);
    }
  }

  // Pause a cron job (stop without deleting)
  pauseCronJob(name: string) {
    if (this.schedulerRegistry.doesExist('cron', name)) {
      const job = this.schedulerRegistry.getCronJob(name);
      job.stop();
      this.logger.log(`Cron job ${name} paused`);
    }
  }

  // Resume a cron job
  resumeCronJob(name: string) {
    if (this.schedulerRegistry.doesExist('cron', name)) {
      const job = this.schedulerRegistry.getCronJob(name);
      job.start();
      this.logger.log(`Cron job ${name} resumed`);
    }
  }

  // Get all cron jobs
  getAllCronJobs(): Map<string, CronJob> {
    return this.schedulerRegistry.getCronJobs();
  }

  // List all cron job names
  listCronJobNames(): string[] {
    const jobs = this.schedulerRegistry.getCronJobs();
    return Array.from(jobs.keys());
  }
}

// USAGE EXAMPLE:
// dynamicScheduler.addDynamicCronJob('userBackup', '0 0 * * *');
// dynamicScheduler.pauseCronJob('userBackup');
// dynamicScheduler.resumeCronJob('userBackup');
// dynamicScheduler.removeDynamicCronJob('userBackup');
```

---

### **Method 2: Dynamic Interval Management**

```typescript
// ========== DYNAMIC INTERVALS ==========

@Injectable()
export class DynamicIntervalService {
  private readonly logger = new Logger(DynamicIntervalService.name);

  constructor(private readonly schedulerRegistry: SchedulerRegistry) {}

  // Add interval dynamically
  addDynamicInterval(name: string, milliseconds: number) {
    const callback = () => {
      this.logger.log(`Interval ${name} executing every ${milliseconds}ms`);
      // Your interval logic here
    };

    // Create and add interval
    const interval = setInterval(callback, milliseconds);
    this.schedulerRegistry.addInterval(name, interval);
    
    this.logger.log(`Interval ${name} added: every ${milliseconds}ms`);
  }

  // Remove interval dynamically
  removeDynamicInterval(name: string) {
    if (this.schedulerRegistry.doesExist('interval', name)) {
      // Delete interval (automatically clears it)
      this.schedulerRegistry.deleteInterval(name);
      
      this.logger.log(`Interval ${name} removed`);
    } else {
      this.logger.warn(`Interval ${name} does not exist`);
    }
  }

  // Get all intervals
  getAllIntervals(): Map<string, any> {
    return this.schedulerRegistry.getIntervals();
  }

  // List all interval names
  listIntervalNames(): string[] {
    const intervals = this.schedulerRegistry.getIntervals();
    return Array.from(intervals.keys());
  }
}

// USAGE EXAMPLE:
// intervalService.addDynamicInterval('healthCheck', 30000); // Every 30s
// intervalService.removeDynamicInterval('healthCheck');
```

---

### **Method 3: Dynamic Timeout Management**

```typescript
// ========== DYNAMIC TIMEOUTS ==========

@Injectable()
export class DynamicTimeoutService {
  private readonly logger = new Logger(DynamicTimeoutService.name);

  constructor(private readonly schedulerRegistry: SchedulerRegistry) {}

  // Add timeout dynamically
  addDynamicTimeout(name: string, milliseconds: number) {
    const callback = () => {
      this.logger.log(`Timeout ${name} executed after ${milliseconds}ms`);
      // Your timeout logic here
      
      // Auto-cleanup after execution
      this.schedulerRegistry.deleteTimeout(name);
    };

    // Create and add timeout
    const timeout = setTimeout(callback, milliseconds);
    this.schedulerRegistry.addTimeout(name, timeout);
    
    this.logger.log(`Timeout ${name} added: executes in ${milliseconds}ms`);
  }

  // Cancel timeout before execution
  cancelDynamicTimeout(name: string) {
    if (this.schedulerRegistry.doesExist('timeout', name)) {
      // Delete timeout (automatically clears it)
      this.schedulerRegistry.deleteTimeout(name);
      
      this.logger.log(`Timeout ${name} cancelled`);
    } else {
      this.logger.warn(`Timeout ${name} does not exist or already executed`);
    }
  }

  // Get all timeouts
  getAllTimeouts(): Map<string, any> {
    return this.schedulerRegistry.getTimeouts();
  }
}

// USAGE EXAMPLE:
// timeoutService.addDynamicTimeout('delayedInit', 60000); // After 60s
// timeoutService.cancelDynamicTimeout('delayedInit');
```

---

### **Method 4: User-Based Dynamic Scheduling**

```typescript
// ========== USER-BASED SCHEDULING ==========

import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { CronJob } from 'cron';

interface UserScheduleSettings {
  userId: string;
  reportSchedule: string; // Cron expression
  enabled: boolean;
}

@Injectable()
export class UserSchedulingService {
  private readonly logger = new Logger(UserSchedulingService.name);

  constructor(
    private readonly schedulerRegistry: SchedulerRegistry,
    @InjectRepository(UserScheduleSettings)
    private readonly settingsRepo: Repository<UserScheduleSettings>,
    private readonly reportService: ReportService,
  ) {}

  // Enable user's scheduled report
  async enableUserReport(userId: string, cronExpression: string) {
    const jobName = `user-report-${userId}`;

    // Create job that generates report for this user
    const job = new CronJob(cronExpression, async () => {
      this.logger.log(`Generating report for user ${userId}`);
      
      try {
        await this.reportService.generateForUser(userId);
        this.logger.log(`Report generated successfully for user ${userId}`);
      } catch (error) {
        this.logger.error(`Report generation failed for user ${userId}`, error.stack);
      }
    });

    // Add and start job
    this.schedulerRegistry.addCronJob(jobName, job);
    job.start();

    // Save to database
    await this.settingsRepo.save({
      userId,
      reportSchedule: cronExpression,
      enabled: true,
    });

    this.logger.log(`Scheduled report enabled for user ${userId}: ${cronExpression}`);
  }

  // Disable user's scheduled report
  async disableUserReport(userId: string) {
    const jobName = `user-report-${userId}`;

    // Remove job if exists
    if (this.schedulerRegistry.doesExist('cron', jobName)) {
      const job = this.schedulerRegistry.getCronJob(jobName);
      job.stop();
      this.schedulerRegistry.deleteCronJob(jobName);
    }

    // Update database
    await this.settingsRepo.update(
      { userId },
      { enabled: false },
    );

    this.logger.log(`Scheduled report disabled for user ${userId}`);
  }

  // Update user's schedule
  async updateUserSchedule(userId: string, newCronExpression: string) {
    // Disable existing schedule
    await this.disableUserReport(userId);
    
    // Enable with new schedule
    await this.enableUserReport(userId, newCronExpression);
  }

  // Load all user schedules on app startup
  async onModuleInit() {
    this.logger.log('Loading user schedules...');
    
    const settings = await this.settingsRepo.find({ where: { enabled: true } });
    
    for (const setting of settings) {
      await this.enableUserReport(setting.userId, setting.reportSchedule);
    }
    
    this.logger.log(`Loaded ${settings.length} user schedules`);
  }
}

// USAGE:
// await userScheduling.enableUserReport('user123', '0 9 * * *'); // Daily 9 AM
// await userScheduling.disableUserReport('user123');
// await userScheduling.updateUserSchedule('user123', '0 12 * * *'); // Change to noon
```

---

### **Method 5: Configuration-Based Dynamic Scheduling**

```typescript
// ========== CONFIG-BASED SCHEDULING ==========

interface ScheduleConfig {
  name: string;
  type: 'cron' | 'interval' | 'timeout';
  pattern: string | number; // Cron expression or milliseconds
  enabled: boolean;
  handler: string; // Handler method name
}

@Injectable()
export class ConfigBasedSchedulerService {
  private readonly logger = new Logger(ConfigBasedSchedulerService.name);

  constructor(
    private readonly schedulerRegistry: SchedulerRegistry,
    private readonly configService: ConfigService,
  ) {}

  // Load schedules from configuration
  async loadSchedulesFromConfig() {
    const schedules: ScheduleConfig[] = this.configService.get('schedules', []);
    
    this.logger.log(`Loading ${schedules.length} schedules from config`);

    for (const schedule of schedules) {
      if (schedule.enabled) {
        await this.addScheduleFromConfig(schedule);
      }
    }
  }

  // Add schedule based on config
  private async addScheduleFromConfig(config: ScheduleConfig) {
    switch (config.type) {
      case 'cron':
        const cronJob = new CronJob(config.pattern as string, () => {
          this.executeHandler(config.handler);
        });
        this.schedulerRegistry.addCronJob(config.name, cronJob);
        cronJob.start();
        break;

      case 'interval':
        const interval = setInterval(() => {
          this.executeHandler(config.handler);
        }, config.pattern as number);
        this.schedulerRegistry.addInterval(config.name, interval);
        break;

      case 'timeout':
        const timeout = setTimeout(() => {
          this.executeHandler(config.handler);
          this.schedulerRegistry.deleteTimeout(config.name);
        }, config.pattern as number);
        this.schedulerRegistry.addTimeout(config.name, timeout);
        break;
    }

    this.logger.log(`Loaded schedule: ${config.name} (${config.type})`);
  }

  // Execute handler by name
  private executeHandler(handlerName: string) {
    this.logger.log(`Executing handler: ${handlerName}`);
    // Implement handler execution logic
  }

  // Reload schedules (useful for hot-reload)
  async reloadSchedules() {
    // Clear all existing schedules
    this.clearAllSchedules();
    
    // Reload from config
    await this.loadSchedulesFromConfig();
  }

  // Clear all schedules
  private clearAllSchedules() {
    // Clear cron jobs
    const cronJobs = this.schedulerRegistry.getCronJobs();
    cronJobs.forEach((job, name) => {
      job.stop();
      this.schedulerRegistry.deleteCronJob(name);
    });

    // Clear intervals
    const intervals = this.schedulerRegistry.getIntervals();
    intervals.forEach((interval, name) => {
      this.schedulerRegistry.deleteInterval(name);
    });

    // Clear timeouts
    const timeouts = this.schedulerRegistry.getTimeouts();
    timeouts.forEach((timeout, name) => {
      this.schedulerRegistry.deleteTimeout(name);
    });
  }
}

// EXAMPLE CONFIG (config.yaml):
// schedules:
//   - name: dailyBackup
//     type: cron
//     pattern: '0 0 * * *'
//     enabled: true
//     handler: backupHandler
//   - name: healthCheck
//     type: interval
//     pattern: 30000
//     enabled: true
//     handler: healthCheckHandler
```

**Key Takeaway:** Dynamically schedule/unschedule jobs using **SchedulerRegistry** service from `@nestjs/schedule` which manages jobs at runtime: inject `SchedulerRegistry` via constructor, use **addCronJob**(name, job)/deleteCronJob(name) for cron jobs, **addInterval**(name, interval)/deleteInterval(name) for intervals, **addTimeout**(name, timeout)/deleteTimeout(name) for timeouts. **Dynamic cron workflow**: create job with `new CronJob(cronPattern, callback)`, add to registry with `schedulerRegistry.addCronJob('jobName', job)`, start execution with `job.start()`, stop with `job.stop()`, delete with `schedulerRegistry.deleteCronJob('jobName')`. **Dynamic interval workflow**: create with `setInterval(callback, milliseconds)`, add to registry with `schedulerRegistry.addInterval('name', interval)`, remove with `schedulerRegistry.deleteInterval('name')` (automatically clears interval). **Dynamic timeout workflow**: create with `setTimeout(callback, milliseconds)`, add to registry with `schedulerRegistry.addTimeout('name', timeout)`, cancel with `schedulerRegistry.deleteTimeout('name')`. **Advantages over static decorators**: static `@Cron()/@Interval()/@Timeout()` are **compile-time** (defined in code, always active when app runs, cannot change without redeploying), dynamic scheduling is **runtime** (add/remove jobs while app running based on database data, user preferences, configuration changes, feature flags, conditional logic). **Use cases**: **user preferences** (users enable/disable scheduled reports: `enableUserReport(userId, '0 9 * * *')`), **multi-tenant** (each tenant custom schedule), **configuration-driven** (load schedules from config file/database on startup, hot-reload without restart), **conditional scheduling** (enable monitoring jobs only in production, disable during maintenance), **dynamic frequency** (adjust polling interval based on system load). **Helper methods**: check existence with `schedulerRegistry.doesExist('cron'|'interval'|'timeout', name)` before operations, get all jobs with `schedulerRegistry.getCronJobs()|getIntervals()|getTimeouts()` returns Map<string, JobType>, list names with `Array.from(map.keys())`. **Best practices**: always check `doesExist()` before stop/delete to avoid errors, store schedule metadata in database for persistence across restarts, load saved schedules in `onModuleInit()` lifecycle hook, use descriptive job names with prefixes (`user-report-${userId}`, `tenant-${tenantId}-backup`), cleanup jobs when no longer needed to prevent memory leaks, handle errors in job callbacks (don't throw, log and continue).

</details>

<details>
<summary><strong>14. What is `SchedulerRegistry` and how do you use it?</strong></summary>

**Answer:**

`SchedulerRegistry` is a **central registry service** from `@nestjs/schedule` that provides **programmatic access** to manage all scheduled jobs (cron jobs, intervals, timeouts) at runtime. It acts as a **job store** where all scheduled tasks are registered by name, allowing **dynamic management** (add, retrieve, delete, list jobs). **Injection**: import from `@nestjs/schedule` and inject via constructor (`constructor(private schedulerRegistry: SchedulerRegistry)`). **Methods**: **getCronJob**(name)/getCronJobs() retrieves cron jobs, **addCronJob**(name, job)/deleteCronJob(name) manages cron jobs, **getInterval**(name)/getIntervals() retrieves intervals, **addInterval**(name, interval)/deleteInterval(name) manages intervals, **getTimeout**(name)/getTimeouts() retrieves timeouts, **addTimeout**(name, timeout)/deleteTimeout(name) manages timeouts, **doesExist**(type, name) checks if job exists. **Use cases**: **dynamic scheduling** (add jobs based on user input), **runtime control** (pause/resume jobs), **monitoring** (list all active jobs, check next execution), **cleanup** (remove jobs when no longer needed), **debugging** (inspect job state, manually trigger). **Static decorator integration**: jobs created with `@Cron(expression, { name: 'jobName' })` are automatically registered in SchedulerRegistry and accessible via `getCronJob('jobName')`. **Pattern**: named jobs (always use name option for management) → access via registry → perform operations (start/stop/delete) → verify with doesExist.

---

### **SchedulerRegistry Architecture:**

```
SCHEDULER REGISTRY - CENTRAL JOB MANAGEMENT:
┌─────────────────────────────────────────────────────┐
│                 SchedulerRegistry                   │
│                                                     │
│  ┌───────────────────────────────────────────┐    │
│  │         Cron Jobs (Map)                   │    │
│  │  'dailyBackup'    → CronJob               │    │
│  │  'hourlyCleanup'  → CronJob               │    │
│  │  'user-123-report'→ CronJob               │    │
│  └───────────────────────────────────────────┘    │
│                                                     │
│  ┌───────────────────────────────────────────┐    │
│  │         Intervals (Map)                   │    │
│  │  'healthCheck'    → NodeJS.Timer          │    │
│  │  'metricsCollect' → NodeJS.Timer          │    │
│  └───────────────────────────────────────────┘    │
│                                                     │
│  ┌───────────────────────────────────────────┐    │
│  │         Timeouts (Map)                    │    │
│  │  'delayedInit'    → NodeJS.Timeout        │    │
│  │  'warmCache'      → NodeJS.Timeout        │    │
│  └───────────────────────────────────────────┘    │
│                                                     │
└─────────────────────────────────────────────────────┘
         ↓                ↓                 ↓
    addCronJob      addInterval      addTimeout
    getCronJob      getInterval      getTimeout
    deleteCronJob   deleteInterval   deleteTimeout
    getCronJobs()   getIntervals()   getTimeouts()
    doesExist('cron', 'name')

INTEGRATION WITH DECORATORS:
┌─────────────────────────────────────────────────────┐
│  @Cron('0 0 * * *', { name: 'dailyBackup' })        │
│  dailyBackup() { ... }                              │
│            ↓                                        │
│  Automatically registered in SchedulerRegistry      │
│            ↓                                        │
│  Accessible via:                                    │
│  schedulerRegistry.getCronJob('dailyBackup')        │
└─────────────────────────────────────────────────────┘
```

---

### **Method 1: Basic SchedulerRegistry Usage**

```typescript
// ========== BASIC USAGE ==========

import { Injectable, Logger } from '@nestjs/common';
import { SchedulerRegistry } from '@nestjs/schedule';
import { Cron, CronExpression } from '@nestjs/schedule';

@Injectable()
export class SchedulerRegistryService {
  private readonly logger = new Logger(SchedulerRegistryService.name);

  constructor(private readonly schedulerRegistry: SchedulerRegistry) {}

  // Access decorator-defined cron job
  @Cron(CronExpression.EVERY_MINUTE, { name: 'myMinuteJob' })
  handleMinuteJob() {
    this.logger.log('Minute job executed');
  }

  // Control the decorator-defined job
  pauseMinuteJob() {
    const job = this.schedulerRegistry.getCronJob('myMinuteJob');
    job.stop();
    this.logger.log('Minute job paused');
  }

  resumeMinuteJob() {
    const job = this.schedulerRegistry.getCronJob('myMinuteJob');
    job.start();
    this.logger.log('Minute job resumed');
  }

  // Get job status
  getMinuteJobStatus() {
    const job = this.schedulerRegistry.getCronJob('myMinuteJob');
    return {
      running: job.running,
      lastDate: job.lastDate(),
      nextDate: job.nextDate().toJSDate(),
    };
  }

  // List all cron jobs
  listAllCronJobs() {
    const jobs = this.schedulerRegistry.getCronJobs();
    const jobList = [];

    jobs.forEach((job, name) => {
      jobList.push({
        name,
        running: job.running,
        nextRun: job.nextDate().toJSDate(),
      });
    });

    return jobList;
  }

  // Check if job exists
  jobExists(name: string): boolean {
    return this.schedulerRegistry.doesExist('cron', name);
  }
}
```

---

### **Method 2: SchedulerRegistry Methods Reference**

```typescript
// ========== ALL SCHEDULER REGISTRY METHODS ==========

@Injectable()
export class SchedulerRegistryMethodsService {
  constructor(private readonly schedulerRegistry: SchedulerRegistry) {}

  // ========== CRON JOB METHODS ==========

  // Add cron job
  addCron(name: string, cronTime: string, callback: () => void) {
    const job = new CronJob(cronTime, callback);
    this.schedulerRegistry.addCronJob(name, job);
    job.start();
  }

  // Get single cron job
  getCron(name: string): CronJob {
    return this.schedulerRegistry.getCronJob(name);
  }

  // Get all cron jobs
  getAllCrons(): Map<string, CronJob> {
    return this.schedulerRegistry.getCronJobs();
  }

  // Delete cron job
  deleteCron(name: string) {
    const job = this.schedulerRegistry.getCronJob(name);
    job.stop(); // Stop before deleting
    this.schedulerRegistry.deleteCronJob(name);
  }

  // ========== INTERVAL METHODS ==========

  // Add interval
  addInterval(name: string, milliseconds: number, callback: () => void) {
    const interval = setInterval(callback, milliseconds);
    this.schedulerRegistry.addInterval(name, interval);
  }

  // Delete interval
  deleteInterval(name: string) {
    this.schedulerRegistry.deleteInterval(name);
    // Automatically calls clearInterval
  }

  // Get all intervals
  getAllIntervals(): Map<string, any> {
    return this.schedulerRegistry.getIntervals();
  }

  // ========== TIMEOUT METHODS ==========

  // Add timeout
  addTimeout(name: string, milliseconds: number, callback: () => void) {
    const timeout = setTimeout(callback, milliseconds);
    this.schedulerRegistry.addTimeout(name, timeout);
  }

  // Delete timeout
  deleteTimeout(name: string) {
    this.schedulerRegistry.deleteTimeout(name);
    // Automatically calls clearTimeout
  }

  // Get all timeouts
  getAllTimeouts(): Map<string, any> {
    return this.schedulerRegistry.getTimeouts();
  }

  // ========== UTILITY METHODS ==========

  // Check if job exists
  exists(type: 'cron' | 'interval' | 'timeout', name: string): boolean {
    return this.schedulerRegistry.doesExist(type, name);
  }

  // Get all job names
  getAllJobNames() {
    return {
      crons: Array.from(this.schedulerRegistry.getCronJobs().keys()),
      intervals: Array.from(this.schedulerRegistry.getIntervals().keys()),
      timeouts: Array.from(this.schedulerRegistry.getTimeouts().keys()),
    };
  }

  // Clear all jobs (useful for testing/shutdown)
  clearAll() {
    // Clear crons
    const crons = this.schedulerRegistry.getCronJobs();
    crons.forEach((job, name) => {
      job.stop();
      this.schedulerRegistry.deleteCronJob(name);
    });

    // Clear intervals
    const intervals = this.schedulerRegistry.getIntervals();
    intervals.forEach((interval, name) => {
      this.schedulerRegistry.deleteInterval(name);
    });

    // Clear timeouts
    const timeouts = this.schedulerRegistry.getTimeouts();
    timeouts.forEach((timeout, name) => {
      this.schedulerRegistry.deleteTimeout(name);
    });
  }
}
```

---

### **Method 3: Monitoring and Debugging with SchedulerRegistry**

```typescript
// ========== MONITORING & DEBUGGING ==========

@Injectable()
export class SchedulerMonitoringService {
  private readonly logger = new Logger(SchedulerMonitoringService.name);

  constructor(private readonly schedulerRegistry: SchedulerRegistry) {}

  // Get comprehensive job status
  getJobsStatus() {
    const cronJobs = this.schedulerRegistry.getCronJobs();
    const intervals = this.schedulerRegistry.getIntervals();
    const timeouts = this.schedulerRegistry.getTimeouts();

    return {
      summary: {
        totalCronJobs: cronJobs.size,
        totalIntervals: intervals.size,
        totalTimeouts: timeouts.size,
        totalJobs: cronJobs.size + intervals.size + timeouts.size,
      },
      cronJobs: this.getCronJobDetails(),
      intervals: Array.from(intervals.keys()),
      timeouts: Array.from(timeouts.keys()),
    };
  }

  // Get detailed cron job information
  getCronJobDetails() {
    const jobs = this.schedulerRegistry.getCronJobs();
    const details = [];

    jobs.forEach((job, name) => {
      try {
        details.push({
          name,
          running: job.running,
          lastRun: job.lastDate() || 'Never',
          nextRun: job.nextDate() ? job.nextDate().toJSDate() : 'N/A',
          nextRunIn: job.nextDate() 
            ? this.getTimeUntil(job.nextDate().toJSDate())
            : 'N/A',
        });
      } catch (error) {
        details.push({
          name,
          error: 'Unable to get job details',
        });
      }
    });

    return details;
  }

  // Calculate time until next execution
  private getTimeUntil(date: Date): string {
    const now = new Date();
    const diffMs = date.getTime() - now.getTime();
    
    if (diffMs < 0) return 'Overdue';
    
    const diffMins = Math.floor(diffMs / 60000);
    const diffHours = Math.floor(diffMins / 60);
    const diffDays = Math.floor(diffHours / 24);

    if (diffDays > 0) return `${diffDays}d ${diffHours % 24}h`;
    if (diffHours > 0) return `${diffHours}h ${diffMins % 60}m`;
    return `${diffMins}m`;
  }

  // Health check endpoint
  async healthCheck() {
    const status = this.getJobsStatus();
    
    // Check if critical jobs are running
    const criticalJobs = ['dailyBackup', 'hourlyCleanup'];
    const missingJobs = criticalJobs.filter(
      name => !this.schedulerRegistry.doesExist('cron', name)
    );

    return {
      healthy: missingJobs.length === 0,
      status,
      issues: missingJobs.length > 0 
        ? `Missing critical jobs: ${missingJobs.join(', ')}`
        : null,
    };
  }

  // Log all job statuses (useful for debugging)
  logAllJobs() {
    this.logger.log('========== SCHEDULED JOBS STATUS ==========');
    
    const details = this.getCronJobDetails();
    details.forEach(job => {
      this.logger.log(
        `[${job.name}] Running: ${job.running}, Next: ${job.nextRunIn}`
      );
    });

    const intervals = Array.from(this.schedulerRegistry.getIntervals().keys());
    this.logger.log(`Active Intervals: ${intervals.join(', ')}`);

    const timeouts = Array.from(this.schedulerRegistry.getTimeouts().keys());
    this.logger.log(`Pending Timeouts: ${timeouts.join(', ')}`);
  }

  // Manually trigger a cron job (useful for testing)
  async manuallyTriggerJob(name: string) {
    if (!this.schedulerRegistry.doesExist('cron', name)) {
      throw new Error(`Job ${name} not found`);
    }

    const job = this.schedulerRegistry.getCronJob(name);
    
    // Execute the job's callback manually
    // Note: This accesses private API, use with caution
    this.logger.log(`Manually triggering job: ${name}`);
    
    // Better approach: Expose job logic in separate method
    // and call that method directly
  }
}
```

---

### **Method 4: Runtime Job Controller**

```typescript
// ========== RUNTIME JOB CONTROLLER ==========

import { Controller, Post, Delete, Get, Param, Body } from '@nestjs/common';

interface CreateJobDto {
  name: string;
  type: 'cron' | 'interval';
  pattern: string | number;
}

@Controller('scheduler')
export class SchedulerController {
  constructor(
    private readonly schedulerRegistry: SchedulerRegistry,
    private readonly logger: Logger,
  ) {}

  // Create a new scheduled job
  @Post('jobs')
  createJob(@Body() dto: CreateJobDto) {
    const { name, type, pattern } = dto;

    // Check if job already exists
    if (this.schedulerRegistry.doesExist(type, name)) {
      return { error: `Job ${name} already exists` };
    }

    try {
      if (type === 'cron') {
        const job = new CronJob(pattern as string, () => {
          this.logger.log(`Executing dynamic job: ${name}`);
          // Job logic here
        });
        this.schedulerRegistry.addCronJob(name, job);
        job.start();
      } else if (type === 'interval') {
        const interval = setInterval(() => {
          this.logger.log(`Executing dynamic interval: ${name}`);
          // Interval logic here
        }, pattern as number);
        this.schedulerRegistry.addInterval(name, interval);
      }

      return { success: true, message: `Job ${name} created` };
    } catch (error) {
      return { error: error.message };
    }
  }

  // Delete a scheduled job
  @Delete('jobs/:type/:name')
  deleteJob(@Param('type') type: 'cron' | 'interval', @Param('name') name: string) {
    if (!this.schedulerRegistry.doesExist(type, name)) {
      return { error: `Job ${name} not found` };
    }

    try {
      if (type === 'cron') {
        const job = this.schedulerRegistry.getCronJob(name);
        job.stop();
        this.schedulerRegistry.deleteCronJob(name);
      } else if (type === 'interval') {
        this.schedulerRegistry.deleteInterval(name);
      }

      return { success: true, message: `Job ${name} deleted` };
    } catch (error) {
      return { error: error.message };
    }
  }

  // Pause a cron job
  @Post('jobs/cron/:name/pause')
  pauseJob(@Param('name') name: string) {
    if (!this.schedulerRegistry.doesExist('cron', name)) {
      return { error: `Job ${name} not found` };
    }

    const job = this.schedulerRegistry.getCronJob(name);
    job.stop();
    return { success: true, message: `Job ${name} paused` };
  }

  // Resume a cron job
  @Post('jobs/cron/:name/resume')
  resumeJob(@Param('name') name: string) {
    if (!this.schedulerRegistry.doesExist('cron', name)) {
      return { error: `Job ${name} not found` };
    }

    const job = this.schedulerRegistry.getCronJob(name);
    job.start();
    return { success: true, message: `Job ${name} resumed` };
  }

  // List all jobs
  @Get('jobs')
  listJobs() {
    const cronJobs = this.schedulerRegistry.getCronJobs();
    const intervals = this.schedulerRegistry.getIntervals();
    const timeouts = this.schedulerRegistry.getTimeouts();

    return {
      crons: Array.from(cronJobs.keys()),
      intervals: Array.from(intervals.keys()),
      timeouts: Array.from(timeouts.keys()),
    };
  }

  // Get job details
  @Get('jobs/cron/:name')
  getJobDetails(@Param('name') name: string) {
    if (!this.schedulerRegistry.doesExist('cron', name)) {
      return { error: `Job ${name} not found` };
    }

    const job = this.schedulerRegistry.getCronJob(name);
    
    return {
      name,
      running: job.running,
      lastRun: job.lastDate() || null,
      nextRun: job.nextDate() ? job.nextDate().toJSDate() : null,
    };
  }
}

// USAGE:
// POST /scheduler/jobs
// { "name": "customJob", "type": "cron", "pattern": "0 0 * * *" }
//
// DELETE /scheduler/jobs/cron/customJob
// POST /scheduler/jobs/cron/customJob/pause
// POST /scheduler/jobs/cron/customJob/resume
// GET /scheduler/jobs
```

---

### **Method 5: Testing with SchedulerRegistry**

```typescript
// ========== TESTING WITH SCHEDULER REGISTRY ==========

describe('SchedulerRegistry Tests', () => {
  let service: SchedulerRegistryService;
  let schedulerRegistry: SchedulerRegistry;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      imports: [ScheduleModule.forRoot()],
      providers: [SchedulerRegistryService],
    }).compile();

    service = module.get<SchedulerRegistryService>(SchedulerRegistryService);
    schedulerRegistry = module.get<SchedulerRegistry>(SchedulerRegistry);
  });

  afterEach(() => {
    // Clean up all jobs after each test
    const crons = schedulerRegistry.getCronJobs();
    crons.forEach((job, name) => {
      job.stop();
      schedulerRegistry.deleteCronJob(name);
    });

    const intervals = schedulerRegistry.getIntervals();
    intervals.forEach((interval, name) => {
      schedulerRegistry.deleteInterval(name);
    });
  });

  it('should add and retrieve a cron job', () => {
    // Add job
    const job = new CronJob('0 0 * * *', () => {});
    schedulerRegistry.addCronJob('testJob', job);

    // Verify exists
    expect(schedulerRegistry.doesExist('cron', 'testJob')).toBe(true);

    // Retrieve job
    const retrievedJob = schedulerRegistry.getCronJob('testJob');
    expect(retrievedJob).toBe(job);
  });

  it('should delete a cron job', () => {
    // Add job
    const job = new CronJob('0 0 * * *', () => {});
    schedulerRegistry.addCronJob('testJob', job);
    job.start();

    // Delete job
    job.stop();
    schedulerRegistry.deleteCronJob('testJob');

    // Verify removed
    expect(schedulerRegistry.doesExist('cron', 'testJob')).toBe(false);
  });

  it('should list all cron jobs', () => {
    // Add multiple jobs
    schedulerRegistry.addCronJob('job1', new CronJob('0 0 * * *', () => {}));
    schedulerRegistry.addCronJob('job2', new CronJob('0 1 * * *', () => {}));

    // Get all jobs
    const jobs = schedulerRegistry.getCronJobs();
    
    expect(jobs.size).toBe(2);
    expect(jobs.has('job1')).toBe(true);
    expect(jobs.has('job2')).toBe(true);
  });
});
```

**Key Takeaway:** `SchedulerRegistry` is a **central registry service** from `@nestjs/schedule` that provides programmatic access to manage all scheduled jobs at runtime - it acts as a **job store** maintaining named collections (Maps) of cron jobs, intervals, and timeouts with methods for **CRUD operations**: **addCronJob**(name, job)/deleteCronJob(name), **addInterval**(name, interval)/deleteInterval(name), **addTimeout**(name, timeout)/deleteTimeout(name). **Injection**: import from `@nestjs/schedule` and inject via constructor (`constructor(private readonly schedulerRegistry: SchedulerRegistry) {}`), available after importing `ScheduleModule.forRoot()` in module. **Retrieval methods**: **getCronJob**(name) returns single CronJob instance for control (job.start()/stop()/running/lastDate()/nextDate()), **getCronJobs**() returns `Map<string, CronJob>` of all cron jobs, **getIntervals**() returns `Map<string, NodeJS.Timer>` of all intervals, **getTimeouts**() returns `Map<string, NodeJS.Timeout>` of all timeouts. **Utility method**: **doesExist**(type, name) where type is 'cron'|'interval'|'timeout' checks if named job exists (prevents errors before operations). **Static decorator integration**: jobs defined with decorators including name option (`@Cron('0 0 * * *', { name: 'dailyBackup' })`) are **automatically registered** in SchedulerRegistry and accessible via getCronJob('dailyBackup') for runtime control. **Use cases**: **dynamic scheduling** (add/remove jobs based on database data, user preferences, configuration), **runtime control** (pause/resume jobs via stop()/start(): `schedulerRegistry.getCronJob('backup').stop()`), **monitoring** (list all active jobs for health checks, check next execution times, log job statuses), **debugging** (inspect job state, verify jobs running, manually trigger for testing), **cleanup** (remove obsolete jobs when features disabled, clear all jobs on shutdown). **CronJob API**: retrieved job has properties/methods - **running** boolean (is job active), **lastDate**() returns last execution time, **nextDate**() returns luxon DateTime of next execution, **start**() begins execution, **stop**() pauses execution. **Best practices**: always use named jobs for management (pass `{ name: 'jobName' }` in decorator options), check existence with `doesExist()` before operations to prevent errors, iterate Maps with `forEach((job, name) => {})` or `Array.from(map.keys())`, cleanup jobs in `onModuleDestroy()` lifecycle hook (stop and delete all), use for monitoring/debugging not business logic.

</details>

<details>
<summary><strong>15. How do you add a cron job dynamically at runtime?</strong></summary>

**Answer:**

Add a cron job dynamically at runtime by: **1)** Import `CronJob` from `cron` package (`import { CronJob } from 'cron'`), **2)** Create new CronJob instance with pattern and callback (`const job = new CronJob('0 * * * *', callback)`), **3)** Inject `SchedulerRegistry`, **4)** Add job to registry with unique name (`schedulerRegistry.addCronJob('jobName', job)`), **5)** Start the job (`job.start()`). The **CronJob constructor** accepts: first parameter is cron expression string ('0 0 * * *'), second is callback function (sync or async), optional third is onComplete callback, fourth is start boolean (default false), fifth is timezone string ('America/New_York'). **Use cases**: **database-driven schedules** (load schedules from database on startup), **user preferences** (users configure their report schedules), **multi-tenant** (each tenant custom backup time), **A/B testing** (enable experimental jobs for subset), **feature flags** (activate jobs based on configuration). Unlike static `@Cron()` decorator (compile-time, code-defined, requires redeployment), dynamic jobs are **runtime** (created from data, can be added anytime, no code changes). **Best practices**: validate cron expression before creating job (use cron-parser), store job metadata in database (for persistence across restarts), load saved jobs in `onModuleInit()`, use descriptive names with identifiers (`user-${userId}-report`), handle errors in callback (try-catch, don't throw), check if job name exists before adding (avoid duplicates with `doesExist()`).

---

### **Dynamic Cron Job Creation Flow:**

```
RUNTIME CRON JOB CREATION:
┌─────────────────────────────────────────────────────┐
│  1. User/Config provides schedule data              │
│     { userId: '123', schedule: '0 9 * * *' }        │
└──────────────────┬──────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────┐
│  2. Validate cron expression                        │
│     - Check syntax validity                         │
│     - Verify timezone if provided                   │
└──────────────────┬──────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────┐
│  3. Create CronJob instance                         │
│     const job = new CronJob(                        │
│       '0 9 * * *',           // Pattern             │
│       async () => { ... },   // Callback            │
│       null,                  // onComplete          │
│       false,                 // start               │
│       'America/New_York'     // timezone            │
│     )                                               │
└──────────────────┬──────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────┐
│  4. Add to SchedulerRegistry                        │
│     schedulerRegistry.addCronJob(                   │
│       'user-123-report',    // Unique name          │
│       job                   // CronJob instance     │
│     )                                               │
└──────────────────┬──────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────┐
│  5. Start the job                                   │
│     job.start()                                     │
│     // Job now active and will run on schedule     │
└──────────────────┬──────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────┐
│  6. Save metadata to database (optional)            │
│     await db.save({                                 │
│       userId: '123',                                │
│       jobName: 'user-123-report',                   │
│       schedule: '0 9 * * *',                        │
│       enabled: true                                 │
│     })                                              │
└─────────────────────────────────────────────────────┘

STATIC VS DYNAMIC:
┌──────────────────┬─────────────────┬─────────────────┐
│ Feature          │ @Cron() Static  │ Dynamic Runtime │
├──────────────────┼─────────────────┼─────────────────┤
│ Definition       │ Code (decorator)│ Data (database) │
│ When created     │ Compile-time    │ Runtime         │
│ Modification     │ Redeploy app    │ Instant         │
│ User control     │ No              │ Yes             │
│ Database-driven  │ No              │ Yes             │
│ Multi-tenant     │ Difficult       │ Easy            │
└──────────────────┴─────────────────┴─────────────────┘
```

---

### **Method 1: Basic Dynamic Cron Job**

```typescript
// ========== BASIC DYNAMIC CRON JOB ==========

import { Injectable, Logger } from '@nestjs/common';
import { SchedulerRegistry } from '@nestjs/schedule';
import { CronJob } from 'cron';

@Injectable()
export class DynamicCronService {
  private readonly logger = new Logger(DynamicCronService.name);

  constructor(private readonly schedulerRegistry: SchedulerRegistry) {}

  // Add cron job dynamically
  addCronJob(name: string, cronExpression: string) {
    // 1. Create CronJob instance
    const job = new CronJob(
      cronExpression,      // Cron pattern: '0 * * * *' = hourly
      () => {              // Callback function
        this.logger.log(`Job ${name} executing at ${new Date().toISOString()}`);
        this.executeJobLogic(name);
      },
      null,                // onComplete callback (optional)
      false,               // Start immediately? (false = manual start)
      'America/New_York',  // Timezone (optional)
    );

    // 2. Add to registry with unique name
    this.schedulerRegistry.addCronJob(name, job);

    // 3. Start the job
    job.start();

    this.logger.log(`Cron job added: ${name} with pattern ${cronExpression}`);
  }

  // Job logic
  private executeJobLogic(name: string) {
    // Implement your job logic here
    this.logger.log(`Executing logic for job: ${name}`);
  }
}

// USAGE:
// dynamicCronService.addCronJob('hourlyReport', '0 * * * *');
// dynamicCronService.addCronJob('dailyBackup', '0 0 * * *');
// dynamicCronService.addCronJob('weeklyCleanup', '0 0 * * 0');
```

---

### **Method 2: User-Configured Dynamic Cron Jobs**

```typescript
// ========== USER-CONFIGURED SCHEDULES ==========

import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';

// Entity to store user schedules
@Entity()
class UserSchedule {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  userId: string;

  @Column()
  jobName: string;

  @Column()
  cronExpression: string;

  @Column()
  reportType: string;

  @Column({ default: true })
  enabled: boolean;

  @Column()
  timezone: string;
}

@Injectable()
export class UserScheduleService {
  private readonly logger = new Logger(UserScheduleService.name);

  constructor(
    private readonly schedulerRegistry: SchedulerRegistry,
    @InjectRepository(UserSchedule)
    private readonly scheduleRepo: Repository<UserSchedule>,
    private readonly reportService: ReportService,
  ) {}

  // User enables scheduled report
  async createUserSchedule(
    userId: string,
    reportType: string,
    cronExpression: string,
    timezone: string = 'UTC',
  ) {
    // Validate cron expression (using cron-parser)
    if (!this.isValidCronExpression(cronExpression)) {
      throw new Error(`Invalid cron expression: ${cronExpression}`);
    }

    const jobName = `user-${userId}-${reportType}`;

    // Check if job already exists
    if (this.schedulerRegistry.doesExist('cron', jobName)) {
      throw new Error(`Schedule already exists for user ${userId}`);
    }

    // Create cron job
    const job = new CronJob(
      cronExpression,
      async () => {
        this.logger.log(`Generating ${reportType} report for user ${userId}`);
        
        try {
          await this.reportService.generateReport(userId, reportType);
          this.logger.log(`Report generated successfully for user ${userId}`);
        } catch (error) {
          this.logger.error(
            `Report generation failed for user ${userId}`,
            error.stack,
          );
        }
      },
      null,
      false,
      timezone,
    );

    // Add to registry and start
    this.schedulerRegistry.addCronJob(jobName, job);
    job.start();

    // Save to database for persistence
    await this.scheduleRepo.save({
      userId,
      jobName,
      cronExpression,
      reportType,
      enabled: true,
      timezone,
    });

    this.logger.log(
      `Schedule created for user ${userId}: ${cronExpression} (${timezone})`,
    );

    return {
      jobName,
      cronExpression,
      timezone,
      nextRun: job.nextDate().toJSDate(),
    };
  }

  // Validate cron expression
  private isValidCronExpression(expression: string): boolean {
    try {
      // Use cron-parser library
      const parser = require('cron-parser');
      parser.parseExpression(expression);
      return true;
    } catch (error) {
      return false;
    }
  }

  // Load all user schedules on app startup
  async onModuleInit() {
    this.logger.log('Loading user schedules from database...');

    const schedules = await this.scheduleRepo.find({
      where: { enabled: true },
    });

    for (const schedule of schedules) {
      try {
        // Recreate cron job
        const job = new CronJob(
          schedule.cronExpression,
          async () => {
            await this.reportService.generateReport(
              schedule.userId,
              schedule.reportType,
            );
          },
          null,
          false,
          schedule.timezone,
        );

        this.schedulerRegistry.addCronJob(schedule.jobName, job);
        job.start();

        this.logger.log(`Loaded schedule: ${schedule.jobName}`);
      } catch (error) {
        this.logger.error(
          `Failed to load schedule ${schedule.jobName}`,
          error.stack,
        );
      }
    }

    this.logger.log(`Loaded ${schedules.length} user schedules`);
  }
}

// USAGE:
// await userScheduleService.createUserSchedule(
//   'user123',
//   'sales',
//   '0 9 * * 1-5',  // Weekdays at 9 AM
//   'America/New_York'
// );
```

---

### **Method 3: Multi-Tenant Dynamic Scheduling**

```typescript
// ========== MULTI-TENANT SCHEDULING ==========

interface TenantConfig {
  tenantId: string;
  backupSchedule: string;
  reportSchedule: string;
  cleanupSchedule: string;
  timezone: string;
}

@Injectable()
export class MultiTenantSchedulerService {
  private readonly logger = new Logger(MultiTenantSchedulerService.name);

  constructor(
    private readonly schedulerRegistry: SchedulerRegistry,
    private readonly tenantService: TenantService,
  ) {}

  // Setup schedules for a tenant
  async setupTenantSchedules(config: TenantConfig) {
    const { tenantId, backupSchedule, reportSchedule, cleanupSchedule, timezone } = config;

    // Backup job
    const backupJob = new CronJob(
      backupSchedule,
      async () => {
        this.logger.log(`Running backup for tenant ${tenantId}`);
        await this.tenantService.backupData(tenantId);
      },
      null,
      false,
      timezone,
    );
    this.schedulerRegistry.addCronJob(`tenant-${tenantId}-backup`, backupJob);
    backupJob.start();

    // Report job
    const reportJob = new CronJob(
      reportSchedule,
      async () => {
        this.logger.log(`Generating report for tenant ${tenantId}`);
        await this.tenantService.generateReport(tenantId);
      },
      null,
      false,
      timezone,
    );
    this.schedulerRegistry.addCronJob(`tenant-${tenantId}-report`, reportJob);
    reportJob.start();

    // Cleanup job
    const cleanupJob = new CronJob(
      cleanupSchedule,
      async () => {
        this.logger.log(`Running cleanup for tenant ${tenantId}`);
        await this.tenantService.cleanupData(tenantId);
      },
      null,
      false,
      timezone,
    );
    this.schedulerRegistry.addCronJob(`tenant-${tenantId}-cleanup`, cleanupJob);
    cleanupJob.start();

    this.logger.log(`Setup complete for tenant ${tenantId}`);
  }

  // Remove all schedules for a tenant
  async removeTenantSchedules(tenantId: string) {
    const jobNames = [
      `tenant-${tenantId}-backup`,
      `tenant-${tenantId}-report`,
      `tenant-${tenantId}-cleanup`,
    ];

    for (const jobName of jobNames) {
      if (this.schedulerRegistry.doesExist('cron', jobName)) {
        const job = this.schedulerRegistry.getCronJob(jobName);
        job.stop();
        this.schedulerRegistry.deleteCronJob(jobName);
        this.logger.log(`Removed job: ${jobName}`);
      }
    }
  }

  // Update tenant schedule
  async updateTenantSchedule(
    tenantId: string,
    scheduleType: 'backup' | 'report' | 'cleanup',
    newCronExpression: string,
  ) {
    const jobName = `tenant-${tenantId}-${scheduleType}`;

    // Remove existing job
    if (this.schedulerRegistry.doesExist('cron', jobName)) {
      const job = this.schedulerRegistry.getCronJob(jobName);
      job.stop();
      this.schedulerRegistry.deleteCronJob(jobName);
    }

    // Create new job with updated schedule
    const job = new CronJob(
      newCronExpression,
      async () => {
        await this.executeScheduleType(tenantId, scheduleType);
      },
    );

    this.schedulerRegistry.addCronJob(jobName, job);
    job.start();

    this.logger.log(`Updated ${scheduleType} schedule for tenant ${tenantId}`);
  }

  private async executeScheduleType(
    tenantId: string,
    type: 'backup' | 'report' | 'cleanup',
  ) {
    switch (type) {
      case 'backup':
        await this.tenantService.backupData(tenantId);
        break;
      case 'report':
        await this.tenantService.generateReport(tenantId);
        break;
      case 'cleanup':
        await this.tenantService.cleanupData(tenantId);
        break;
    }
  }
}
```

---

### **Method 4: Feature Flag Controlled Dynamic Jobs**

```typescript
// ========== FEATURE FLAG CONTROLLED JOBS ==========

@Injectable()
export class FeatureFlagSchedulerService {
  private readonly logger = new Logger(FeatureFlagSchedulerService.name);

  constructor(
    private readonly schedulerRegistry: SchedulerRegistry,
    private readonly configService: ConfigService,
  ) {}

  // Conditionally add jobs based on feature flags
  async setupConditionalJobs() {
    // Analytics job (only if analytics enabled)
    if (this.configService.get('ENABLE_ANALYTICS')) {
      this.addAnalyticsJob();
    }

    // Experimental feature job (only in beta)
    if (this.configService.get('ENABLE_BETA_FEATURES')) {
      this.addExperimentalJob();
    }

    // Production-only jobs
    if (this.configService.get('NODE_ENV') === 'production') {
      this.addProductionJobs();
    }
  }

  private addAnalyticsJob() {
    const job = new CronJob('0 0 * * *', async () => {
      this.logger.log('Running analytics aggregation');
      // Analytics logic
    });

    this.schedulerRegistry.addCronJob('analytics', job);
    job.start();
    this.logger.log('Analytics job enabled');
  }

  private addExperimentalJob() {
    const job = new CronJob('0 */6 * * *', async () => {
      this.logger.log('Running experimental feature');
      // Experimental logic
    });

    this.schedulerRegistry.addCronJob('experimental', job);
    job.start();
    this.logger.log('Experimental job enabled');
  }

  private addProductionJobs() {
    // Backup job
    const backupJob = new CronJob('0 2 * * *', async () => {
      this.logger.log('Running production backup');
      // Backup logic
    });
    this.schedulerRegistry.addCronJob('prod-backup', backupJob);
    backupJob.start();

    // Monitoring job
    const monitorJob = new CronJob('*/5 * * * *', async () => {
      // Monitor every 5 minutes
    });
    this.schedulerRegistry.addCronJob('prod-monitor', monitorJob);
    monitorJob.start();

    this.logger.log('Production jobs enabled');
  }

  // Hot reload jobs when config changes
  async reloadJobsFromConfig() {
    this.logger.log('Reloading jobs from configuration...');
    
    // Clear existing conditional jobs
    this.clearConditionalJobs();
    
    // Reload based on new config
    await this.setupConditionalJobs();
    
    this.logger.log('Jobs reloaded');
  }

  private clearConditionalJobs() {
    const conditionalJobNames = [
      'analytics',
      'experimental',
      'prod-backup',
      'prod-monitor',
    ];

    conditionalJobNames.forEach(name => {
      if (this.schedulerRegistry.doesExist('cron', name)) {
        const job = this.schedulerRegistry.getCronJob(name);
        job.stop();
        this.schedulerRegistry.deleteCronJob(name);
      }
    });
  }
}
```

---

### **Method 5: Async Callback with Dependency Injection**

```typescript
// ========== ASYNC CALLBACK WITH DI ==========

@Injectable()
export class DynamicJobWithDIService {
  private readonly logger = new Logger(DynamicJobWithDIService.name);

  constructor(
    private readonly schedulerRegistry: SchedulerRegistry,
    private readonly dataService: DataService,
    private readonly emailService: EmailService,
    private readonly cacheService: CacheService,
  ) {}

  // Add job with async callback and full DI access
  addJobWithDependencies(
    name: string,
    cronExpression: string,
    jobType: 'email' | 'data' | 'cache',
  ) {
    // Create job with async callback
    const job = new CronJob(
      cronExpression,
      async () => {
        const startTime = Date.now();
        this.logger.log(`Starting job ${name} (${jobType})`);

        try {
          // Execute job based on type, with full DI access
          switch (jobType) {
            case 'email':
              await this.emailService.sendScheduledEmails();
              break;
            case 'data':
              await this.dataService.processData();
              break;
            case 'cache':
              await this.cacheService.refreshCache();
              break;
          }

          const duration = Date.now() - startTime;
          this.logger.log(`Job ${name} completed in ${duration}ms`);
        } catch (error) {
          this.logger.error(`Job ${name} failed`, error.stack);
          // Don't throw - let job continue on next schedule
        }
      },
      null,   // onComplete
      false,  // start
    );

    // Add and start
    this.schedulerRegistry.addCronJob(name, job);
    job.start();

    this.logger.log(`Job ${name} added with pattern ${cronExpression}`);
  }

  // Add complex job with multiple dependencies
  addComplexJob(name: string, cronExpression: string) {
    const job = new CronJob(cronExpression, async () => {
      try {
        // Step 1: Fetch data
        const data = await this.dataService.fetchLatestData();
        
        // Step 2: Process data
        const processed = await this.dataService.processData(data);
        
        // Step 3: Cache results
        await this.cacheService.set(`job-${name}-result`, processed);
        
        // Step 4: Send notification
        await this.emailService.sendNotification({
          subject: `Job ${name} completed`,
          body: `Processed ${processed.length} items`,
        });
        
      } catch (error) {
        this.logger.error(`Complex job ${name} failed`, error.stack);
      }
    });

    this.schedulerRegistry.addCronJob(name, job);
    job.start();
  }
}
```

**Key Takeaway:** Add a cron job dynamically at runtime by creating `CronJob` instance and registering with `SchedulerRegistry`: **1)** Import CronJob from cron package (`import { CronJob } from 'cron'`), **2)** Create job with `new CronJob(cronExpression, callback, onComplete?, start?, timezone?)` where cronExpression is cron pattern string ('0 * * * *' for hourly), callback is function to execute (can be async with await for database/API operations), onComplete is optional callback after execution, start is boolean to auto-start (false = manual start recommended), timezone is IANA timezone string ('America/New_York', 'Europe/London'), **3)** Inject SchedulerRegistry via constructor (`constructor(private schedulerRegistry: SchedulerRegistry)`), **4)** Add job to registry with `schedulerRegistry.addCronJob(uniqueName, job)` where uniqueName is string identifier for management (use descriptive names like `user-${userId}-report`, `tenant-${tenantId}-backup`), **5)** Start job execution with `job.start()` to activate scheduling. **Callback features**: can be async function with full dependency injection access (inject services via constructor, use in callback), handle errors with try-catch (don't throw, log and continue to next execution), return values ignored (side-effects only), execution is sequential (waits for previous completion before next). **Best practices**: **validate cron expression** before creating job (use cron-parser library: `require('cron-parser').parseExpression(expr)` throws if invalid), **check existence** before adding (`schedulerRegistry.doesExist('cron', name)` prevents duplicates), **persist to database** (save job metadata for recreation after app restart, load in `onModuleInit()` lifecycle hook), **use unique naming** (include identifiers: user ID, tenant ID, type to avoid collisions), **error handling** (wrap callback in try-catch, log errors, send alerts for critical failures), **cleanup on delete** (stop job before removing, delete database record). **Use cases**: **user preferences** (users configure report schedules: daily at 9 AM, weekly Monday, monthly 1st), **multi-tenant** (each tenant custom backup/report times in their timezone), **database-driven** (load schedules from configuration table on startup), **feature flags** (conditionally enable jobs based on environment/config), **A/B testing** (enable experimental scheduled features for subset of users). **Advantages over static @Cron()**: static decorators are compile-time (defined in code, always active, require redeployment to change), dynamic jobs are runtime (created from database data, can add/modify/delete instantly without code changes, support per-user/per-tenant customization).

</details>

<details>
<summary><strong>16. How do you delete a scheduled job?</strong></summary>

**Answer:**

Delete a scheduled job by: **1)** Inject `SchedulerRegistry` from `@nestjs/schedule`, **2)** Check if job exists with `schedulerRegistry.doesExist(type, name)` where type is 'cron'|'interval'|'timeout', **3)** For cron jobs: retrieve with `getCronJob(name)`, stop with `job.stop()`, delete with `deleteCronJob(name)`, **4)** For intervals: delete with `deleteInterval(name)` (automatically clears interval), **5)** For timeouts: delete with `deleteTimeout(name)` (automatically clears timeout). **Stopping vs Deleting**: **stop()** pauses execution but keeps job in registry (can resume with `job.start()`), **delete** removes from registry permanently (cannot resume, must recreate). **Best practices**: always check existence before delete (prevents errors), stop cron jobs before deleting (clean shutdown), update database if job metadata stored (maintain consistency), log deletion for audit trail, handle errors gracefully (job may not exist). **Use cases**: **user disables feature** (delete scheduled reports), **tenant cancellation** (remove all tenant jobs), **configuration change** (delete old job, add new one with updated schedule), **cleanup on shutdown** (delete all jobs in `onModuleDestroy()`), **job replacement** (delete existing, create updated version). **Cleanup pattern**: check existence → stop (cron only) → delete from registry → delete from database.

---

### **Job Deletion Flow:**

```
DELETION WORKFLOW:
┌─────────────────────────────────────────────────────┐
│  1. Check if job exists                             │
│     schedulerRegistry.doesExist('cron', 'jobName')  │
│     ↓                                               │
│     if false → Job not found, log warning           │
│     if true  → Continue to step 2                   │
└──────────────────┬──────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────┐
│  2. Retrieve job (cron only)                        │
│     const job = schedulerRegistry.getCronJob(name)  │
└──────────────────┬──────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────┐
│  3. Stop job (cron only)                            │
│     job.stop()                                      │
│     // Prevents next execution                      │
└──────────────────┬──────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────┐
│  4. Delete from registry                            │
│     schedulerRegistry.deleteCronJob(name)           │
│     // Removes from SchedulerRegistry               │
└──────────────────┬──────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────┐
│  5. Delete from database (if persisted)             │
│     await db.delete({ jobName: name })              │
│     // Maintain consistency                         │
└──────────────────┬──────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────┐
│  Job completely removed                             │
│  Cannot be resumed (must recreate)                  │
└─────────────────────────────────────────────────────┘

STOP VS DELETE:
┌────────────────────┬───────────────┬─────────────────┐
│ Operation          │ job.stop()    │ delete()        │
├────────────────────┼───────────────┼─────────────────┤
│ Execution          │ Paused        │ Removed         │
│ In registry        │ Yes           │ No              │
│ Can resume         │ Yes (start()) │ No (recreate)   │
│ Memory             │ Uses memory   │ Freed           │
│ Use case           │ Pause         │ Permanent del   │
└────────────────────┴───────────────┴─────────────────┘
```

---

### **Method 1: Delete Cron Job**

```typescript
// ========== DELETE CRON JOB ==========

import { Injectable, Logger } from '@nestjs/common';
import { SchedulerRegistry } from '@nestjs/schedule';

@Injectable()
export class CronDeletionService {
  private readonly logger = new Logger(CronDeletionService.name);

  constructor(private readonly schedulerRegistry: SchedulerRegistry) {}

  // Delete a cron job
  deleteCronJob(name: string): boolean {
    // 1. Check if job exists
    if (!this.schedulerRegistry.doesExist('cron', name)) {
      this.logger.warn(`Cron job ${name} does not exist`);
      return false;
    }

    try {
      // 2. Get the job
      const job = this.schedulerRegistry.getCronJob(name);

      // 3. Stop the job (prevents next execution)
      job.stop();

      // 4. Delete from registry
      this.schedulerRegistry.deleteCronJob(name);

      this.logger.log(`Cron job ${name} deleted successfully`);
      return true;
    } catch (error) {
      this.logger.error(`Failed to delete cron job ${name}`, error.stack);
      return false;
    }
  }

  // Delete multiple cron jobs
  deleteCronJobs(names: string[]): { deleted: string[]; failed: string[] } {
    const deleted: string[] = [];
    const failed: string[] = [];

    for (const name of names) {
      if (this.deleteCronJob(name)) {
        deleted.push(name);
      } else {
        failed.push(name);
      }
    }

    return { deleted, failed };
  }

  // Delete all cron jobs
  deleteAllCronJobs(): number {
    const jobs = this.schedulerRegistry.getCronJobs();
    let deletedCount = 0;

    jobs.forEach((job, name) => {
      try {
        job.stop();
        this.schedulerRegistry.deleteCronJob(name);
        deletedCount++;
        this.logger.log(`Deleted cron job: ${name}`);
      } catch (error) {
        this.logger.error(`Failed to delete job ${name}`, error.stack);
      }
    });

    this.logger.log(`Deleted ${deletedCount} cron jobs`);
    return deletedCount;
  }
}

// USAGE:
// cronDeletionService.deleteCronJob('dailyBackup');
// cronDeletionService.deleteCronJobs(['job1', 'job2', 'job3']);
// cronDeletionService.deleteAllCronJobs();
```

---

### **Method 2: Delete Interval**

```typescript
// ========== DELETE INTERVAL ==========

@Injectable()
export class IntervalDeletionService {
  private readonly logger = new Logger(IntervalDeletionService.name);

  constructor(private readonly schedulerRegistry: SchedulerRegistry) {}

  // Delete an interval
  deleteInterval(name: string): boolean {
    // Check if interval exists
    if (!this.schedulerRegistry.doesExist('interval', name)) {
      this.logger.warn(`Interval ${name} does not exist`);
      return false;
    }

    try {
      // Delete interval (automatically calls clearInterval)
      this.schedulerRegistry.deleteInterval(name);

      this.logger.log(`Interval ${name} deleted successfully`);
      return true;
    } catch (error) {
      this.logger.error(`Failed to delete interval ${name}`, error.stack);
      return false;
    }
  }

  // Delete all intervals
  deleteAllIntervals(): number {
    const intervals = this.schedulerRegistry.getIntervals();
    let deletedCount = 0;

    intervals.forEach((interval, name) => {
      try {
        this.schedulerRegistry.deleteInterval(name);
        deletedCount++;
        this.logger.log(`Deleted interval: ${name}`);
      } catch (error) {
        this.logger.error(`Failed to delete interval ${name}`, error.stack);
      }
    });

    this.logger.log(`Deleted ${deletedCount} intervals`);
    return deletedCount;
  }
}

// USAGE:
// intervalDeletionService.deleteInterval('healthCheck');
// intervalDeletionService.deleteAllIntervals();
```

---

### **Method 3: Delete Timeout**

```typescript
// ========== DELETE TIMEOUT ==========

@Injectable()
export class TimeoutDeletionService {
  private readonly logger = new Logger(TimeoutDeletionService.name);

  constructor(private readonly schedulerRegistry: SchedulerRegistry) {}

  // Cancel a timeout (before it executes)
  cancelTimeout(name: string): boolean {
    // Check if timeout exists (may have already executed)
    if (!this.schedulerRegistry.doesExist('timeout', name)) {
      this.logger.warn(
        `Timeout ${name} does not exist (may have already executed)`,
      );
      return false;
    }

    try {
      // Delete timeout (automatically calls clearTimeout)
      this.schedulerRegistry.deleteTimeout(name);

      this.logger.log(`Timeout ${name} cancelled successfully`);
      return true;
    } catch (error) {
      this.logger.error(`Failed to cancel timeout ${name}`, error.stack);
      return false;
    }
  }

  // Cancel all pending timeouts
  cancelAllTimeouts(): number {
    const timeouts = this.schedulerRegistry.getTimeouts();
    let cancelledCount = 0;

    timeouts.forEach((timeout, name) => {
      try {
        this.schedulerRegistry.deleteTimeout(name);
        cancelledCount++;
        this.logger.log(`Cancelled timeout: ${name}`);
      } catch (error) {
        this.logger.error(`Failed to cancel timeout ${name}`, error.stack);
      }
    });

    this.logger.log(`Cancelled ${cancelledCount} timeouts`);
    return cancelledCount;
  }
}

// USAGE:
// timeoutDeletionService.cancelTimeout('delayedInit');
// timeoutDeletionService.cancelAllTimeouts();
```

---

### **Method 4: User Schedule Deletion with Database**

```typescript
// ========== DELETE WITH DATABASE PERSISTENCE ==========

import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';

@Injectable()
export class UserScheduleDeletionService {
  private readonly logger = new Logger(UserScheduleDeletionService.name);

  constructor(
    private readonly schedulerRegistry: SchedulerRegistry,
    @InjectRepository(UserSchedule)
    private readonly scheduleRepo: Repository<UserSchedule>,
  ) {}

  // Delete user's scheduled job
  async deleteUserSchedule(userId: string, reportType: string): Promise<void> {
    const jobName = `user-${userId}-${reportType}`;

    // 1. Delete from SchedulerRegistry
    if (this.schedulerRegistry.doesExist('cron', jobName)) {
      try {
        const job = this.schedulerRegistry.getCronJob(jobName);
        job.stop();
        this.schedulerRegistry.deleteCronJob(jobName);
        this.logger.log(`Deleted job from registry: ${jobName}`);
      } catch (error) {
        this.logger.error(
          `Failed to delete job from registry: ${jobName}`,
          error.stack,
        );
      }
    }

    // 2. Delete from database
    try {
      await this.scheduleRepo.delete({
        userId,
        reportType,
      });
      this.logger.log(`Deleted schedule from database for user ${userId}`);
    } catch (error) {
      this.logger.error(
        `Failed to delete schedule from database`,
        error.stack,
      );
      throw error;
    }
  }

  // Delete all schedules for a user
  async deleteAllUserSchedules(userId: string): Promise<void> {
    this.logger.log(`Deleting all schedules for user ${userId}`);

    // 1. Get all user schedules from database
    const schedules = await this.scheduleRepo.find({ where: { userId } });

    // 2. Delete each job from registry
    for (const schedule of schedules) {
      if (this.schedulerRegistry.doesExist('cron', schedule.jobName)) {
        try {
          const job = this.schedulerRegistry.getCronJob(schedule.jobName);
          job.stop();
          this.schedulerRegistry.deleteCronJob(schedule.jobName);
        } catch (error) {
          this.logger.error(
            `Failed to delete job ${schedule.jobName}`,
            error.stack,
          );
        }
      }
    }

    // 3. Delete all from database
    await this.scheduleRepo.delete({ userId });

    this.logger.log(
      `Deleted ${schedules.length} schedules for user ${userId}`,
    );
  }

  // Soft delete (disable instead of delete)
  async disableUserSchedule(userId: string, reportType: string): Promise<void> {
    const jobName = `user-${userId}-${reportType}`;

    // Stop and remove from registry
    if (this.schedulerRegistry.doesExist('cron', jobName)) {
      const job = this.schedulerRegistry.getCronJob(jobName);
      job.stop();
      this.schedulerRegistry.deleteCronJob(jobName);
    }

    // Mark as disabled in database (not deleted)
    await this.scheduleRepo.update(
      { userId, reportType },
      { enabled: false },
    );

    this.logger.log(`Disabled schedule for user ${userId}: ${reportType}`);
  }
}

// USAGE:
// await userScheduleDeletion.deleteUserSchedule('user123', 'sales');
// await userScheduleDeletion.deleteAllUserSchedules('user123');
// await userScheduleDeletion.disableUserSchedule('user123', 'sales');
```

---

### **Method 5: Graceful Shutdown with Job Cleanup**

```typescript
// ========== CLEANUP ON SHUTDOWN ==========

import { OnModuleDestroy } from '@nestjs/common';

@Injectable()
export class GracefulShutdownService implements OnModuleDestroy {
  private readonly logger = new Logger(GracefulShutdownService.name);

  constructor(private readonly schedulerRegistry: SchedulerRegistry) {}

  // Clean up all jobs on shutdown
  async onModuleDestroy() {
    this.logger.log('Application shutting down, cleaning up scheduled jobs...');

    const stats = {
      cronJobs: 0,
      intervals: 0,
      timeouts: 0,
    };

    // 1. Stop and delete all cron jobs
    const cronJobs = this.schedulerRegistry.getCronJobs();
    cronJobs.forEach((job, name) => {
      try {
        job.stop();
        this.schedulerRegistry.deleteCronJob(name);
        stats.cronJobs++;
      } catch (error) {
        this.logger.error(`Failed to cleanup cron job ${name}`, error.stack);
      }
    });

    // 2. Delete all intervals
    const intervals = this.schedulerRegistry.getIntervals();
    intervals.forEach((interval, name) => {
      try {
        this.schedulerRegistry.deleteInterval(name);
        stats.intervals++;
      } catch (error) {
        this.logger.error(`Failed to cleanup interval ${name}`, error.stack);
      }
    });

    // 3. Cancel all timeouts
    const timeouts = this.schedulerRegistry.getTimeouts();
    timeouts.forEach((timeout, name) => {
      try {
        this.schedulerRegistry.deleteTimeout(name);
        stats.timeouts++;
      } catch (error) {
        this.logger.error(`Failed to cleanup timeout ${name}`, error.stack);
      }
    });

    this.logger.log(
      `Cleanup complete: ${stats.cronJobs} cron jobs, ` +
      `${stats.intervals} intervals, ${stats.timeouts} timeouts`,
    );
  }

  // Force cleanup (can be called manually)
  forceCleanup(): void {
    this.onModuleDestroy();
  }

  // Cleanup specific pattern (e.g., all user jobs)
  cleanupByPattern(pattern: string): number {
    let cleanedCount = 0;

    // Cleanup cron jobs matching pattern
    const cronJobs = this.schedulerRegistry.getCronJobs();
    cronJobs.forEach((job, name) => {
      if (name.includes(pattern)) {
        try {
          job.stop();
          this.schedulerRegistry.deleteCronJob(name);
          cleanedCount++;
          this.logger.log(`Cleaned up job: ${name}`);
        } catch (error) {
          this.logger.error(`Failed to cleanup ${name}`, error.stack);
        }
      }
    });

    // Cleanup intervals matching pattern
    const intervals = this.schedulerRegistry.getIntervals();
    intervals.forEach((interval, name) => {
      if (name.includes(pattern)) {
        try {
          this.schedulerRegistry.deleteInterval(name);
          cleanedCount++;
        } catch (error) {
          this.logger.error(`Failed to cleanup ${name}`, error.stack);
        }
      }
    });

    this.logger.log(`Cleaned up ${cleanedCount} jobs matching pattern: ${pattern}`);
    return cleanedCount;
  }
}

// USAGE:
// Automatic cleanup on app shutdown (implements OnModuleDestroy)
// gracefulShutdown.forceCleanup(); // Manual cleanup
// gracefulShutdown.cleanupByPattern('user-'); // Cleanup all user jobs
// gracefulShutdown.cleanupByPattern('tenant-123'); // Cleanup tenant jobs
```

**Key Takeaway:** Delete a scheduled job using `SchedulerRegistry` with proper cleanup workflow: **1)** Check existence with `schedulerRegistry.doesExist(type, name)` where type is 'cron'|'interval'|'timeout' and name is job identifier (prevents errors if job doesn't exist), **2)** For **cron jobs**: retrieve with `const job = schedulerRegistry.getCronJob(name)`, stop execution with `job.stop()` (prevents next scheduled run), delete from registry with `schedulerRegistry.deleteCronJob(name)` (removes permanently, frees memory), **3)** For **intervals**: delete directly with `schedulerRegistry.deleteInterval(name)` (automatically calls clearInterval, no stop needed), **4)** For **timeouts**: cancel with `schedulerRegistry.deleteTimeout(name)` (automatically calls clearTimeout, prevents execution if not yet fired), **5)** If job metadata persisted to database: delete database record to maintain consistency (`await scheduleRepo.delete({ jobName: name })`). **Stop vs Delete difference**: **job.stop()** pauses execution but keeps job in registry (job still exists, can resume with `job.start()`, uses memory, useful for temporary pause), **deleteCronJob()** removes from registry permanently (job no longer exists, cannot resume without recreating, frees memory, requires full recreation to reactivate). **Error handling**: wrap in try-catch (deletion may fail if job doesn't exist or already deleted), log warnings for missing jobs (not errors, may be expected), handle database errors separately (transaction may be needed for consistency). **Best practices**: **always check existence** before delete operations (use `doesExist()` to prevent throws), **stop before delete** for cron jobs (clean shutdown: job.stop() then deleteCronJob()), **maintain consistency** (if job in database, delete both registry and database in same operation), **log deletions** for audit trail (who deleted, when, why), **cleanup on shutdown** (implement OnModuleDestroy lifecycle hook, stop and delete all jobs for graceful shutdown), **pattern-based cleanup** (delete multiple jobs by pattern: all user jobs `cleanupByPattern('user-')`, all tenant jobs `cleanupByPattern('tenant-123')`). **Use cases**: **user disables feature** (delete scheduled reports when user turns off notifications), **tenant cancellation** (remove all tenant jobs when subscription ends: backup, reports, cleanup), **job replacement** (delete old job, create new with updated schedule for configuration changes), **temporary removal** (use stop() for maintenance mode, delete() for permanent removal), **bulk cleanup** (delete all jobs matching pattern for mass operations). **Soft delete alternative**: instead of deleting, mark as disabled in database and remove from registry (preserves history, can re-enable later: `await repo.update({ enabled: false })` then `deleteCronJob(name)`).

</details>

## Use Cases

17. What are common use cases for scheduled jobs?
18. How do you implement data cleanup jobs?
19. How do you send scheduled email notifications?
20. How do you implement report generation?
21. How do you sync data with external APIs periodically?

## Queue-Based Jobs

22. What is a job queue and when should you use it?
23. What is Bull queue library?
24. How do you integrate Bull with NestJS using `@nestjs/bull`?
25. What is Redis and why is it used with Bull?
26. How do you create a queue processor?
27. How do you add jobs to a queue?
28. What is the difference between cron jobs and queues?

## Job Processing

29. How do you handle job failures?
30. How do you implement retry logic for failed jobs?
31. How do you implement job prioritization?
32. How do you process jobs concurrently?

## Background Workers

33. What are background workers?
34. When should you use background workers vs cron jobs?
35. How do you ensure jobs don't run twice in clustered environments?

## Monitoring & Logging

36. How do you monitor scheduled jobs?
37. How do you log job execution?
38. How do you handle long-running jobs?

## Best Practices

39. Should scheduled jobs contain heavy logic or delegate to services?
40. How do you test scheduled jobs?
41. How do you handle timezone considerations?
42. How do you prevent jobs from overlapping (job locking)?
43. Should you run scheduled jobs in production on all instances?
44. How do you use distributed locks to ensure single execution in multi-instance setups?
45. What are the performance implications of many scheduled jobs?
