# NestJS Scheduling & Background Jobs - Top Interview Questions

## Scheduling Fundamentals

<details>
<summary><strong>1. What is task scheduling and when do you need it?</strong></summary>

**Answer:**

**Task scheduling** — quick overview:

- **Definition:** run code automatically at specific times or intervals without manual triggers.
- **How it works:** the system executes background tasks on time-based triggers (examples: daily at midnight, every 5 minutes, hourly, or on specific dates).
- **NestJS tool:** use `@nestjs/schedule` and decorators (`@Cron()`, `@Interval()`, `@Timeout()`) to declare scheduled methods.
- **Common use cases:** data cleanup (daily), report generation (monthly/weekly), email notifications (hourly/weekly digests), API synchronization (every 15 minutes), cache warming (pre-load before peak), health checks (every minute), backups (nightly), billing (monthly), token expiration cleanup.
- **Benefits:** automation (no manual intervention), reliability & consistency (predictable runs), resource optimization (schedule heavy jobs during off-peak hours).
- **Alternatives:** manual triggers (not scalable), external cron services or cloud schedulers (extra infrastructure/cost).

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

**Key Takeaway:** Task scheduling automates code execution at defined times or intervals using NestJS `@nestjs/schedule`. Quick reference:

- **What it is:** declarative scheduling via decorators (`@Cron()`, `@Interval()`, `@Timeout()`) backed by `node-cron` and native timers.

- **Common use cases:**
  - **Data cleanup:** delete expired sessions daily at 2 AM, remove old logs weekly, purge soft-deleted records monthly.
  - **Report generation:** monthly sales reports (1st of month), weekly analytics (every Monday), quarterly summaries.
  - **Email notifications:** appointment reminders hourly, weekly digests Monday 9 AM, scheduled promotional emails.
  - **API synchronization:** sync prices every 15 minutes, update inventory hourly, fetch external data periodically.
  - **Cache warming:** pre-load frequently accessed data before peak hours, refresh cached reports nightly.
  - **Health checks:** ping services every minute, verify database connectivity every 5 minutes.
  - **Backups & billing:** nightly backups (e.g., 3 AM), periodic billing/subscription processing.

- **Benefits:**
  - **Automation:** runs without manual triggers, 24/7.
  - **Reliability & consistency:** predictable schedules and repeatable runs.
  - **Resource optimization:** schedule heavy work during off-peak windows (e.g., 2–4 AM).

- **When to use:** time-based automation, periodic data processing, maintenance tasks, off-peak heavy jobs.

- **When NOT to use:**
  - **User-triggered workflows:** use API endpoints instead.
  - **Low-latency or real-time processing:** prefer message queues (e.g., Bull) or event-driven systems.
  - **Event-driven tasks:** use webhooks or event handlers when tasks depend on external events.

</details>

<details>
<summary><strong>2. What is `@nestjs/schedule` package?</strong></summary>

**Answer:**

`@nestjs/schedule` — package overview:

- **What it is:** official NestJS package for declarative scheduling using decorators: `@Cron()`, `@Interval()`, `@Timeout()`.
- **Quick install & setup:**
  - `npm install @nestjs/schedule`
  - Import `ScheduleModule.forRoot()` in `AppModule` to enable scanning and registration of decorated methods.
- **Core features:**
  - **Cron jobs:** calendar-based schedules using cron syntax (e.g., `0 0 * * *` = midnight daily).
  - **Intervals:** repeated execution every N milliseconds (`@Interval(5000)`).
  - **Timeouts:** one-time delayed execution (`@Timeout(10000)`).
  - **Named jobs & dynamic scheduling:** manage jobs at runtime with `SchedulerRegistry` (add/get/delete cron, interval, timeout).
- **Under the hood:** uses `node-cron` for cron parsing and native `setTimeout`/`setInterval` for timeouts/intervals; `ScheduleModule` registers jobs at app startup.
- **Advantages:**
  - Declarative syntax (decorators) — no manual cron wiring.
  - Seamless NestJS DI integration (inject services, config, logger, queues inside jobs).
  - TypeScript-friendly and testable (mock scheduled methods in tests).
  - Dynamic runtime control for user-driven schedules.

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

**Key Takeaway:** `@nestjs/schedule` provides declarative task scheduling with full NestJS integration. Quick summary:

- **Package & setup:** `npm install @nestjs/schedule`; import `ScheduleModule.forRoot()` in `AppModule` to enable scheduling and scan for decorators.
- **Decorators:** use `@Cron()`, `@Interval()`, `@Timeout()` on service methods to define scheduled tasks.
- **Cron jobs:** schedule calendar-based tasks with `@Cron('0 0 * * *')` or `@Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT)` (supports second/minute/hour/day/month/dayOfWeek).
- **Intervals:** `@Interval(10000)` runs repeatedly every N milliseconds (first run occurs immediately on startup, then repeats).
- **Timeouts:** `@Timeout(5000)` runs once after N milliseconds (one-time delayed execution).
- **Named jobs & dynamic management:** add `name` option (e.g., `@Cron('0 0 * * *', { name: 'dailyBackup' })`) and use `SchedulerRegistry` to add/remove/get jobs at runtime (`addCronJob`, `deleteCronJob`, `addInterval`, `deleteInterval`, `addTimeout`, `deleteTimeout`).
- **Timezone support:** specify `{ timeZone: 'America/New_York' }` on cron jobs to run in a specific timezone.
- **NestJS integration:** scheduled methods have full DI access (repositories, services, `ConfigService`, `Logger`, `HttpService`, queues); methods may be `private`/`public`/`async` and are type-safe in TypeScript.
- **Implementation details:** uses `node-cron` for cron parsing and native `setTimeout`/`setInterval` for timeouts/intervals; `ScheduleModule` registers tasks at startup.
- **Benefits:** declarative syntax, type-safety, testability, centralized scheduling logic, dynamic runtime control, and seamless DI/logging integration.

</details>

<details>
<summary><strong>3. What types of scheduled tasks can you create (cron, intervals, timeouts)?</strong></summary>

**Answer:**

`@nestjs/schedule` provides three scheduling primitives — Cron jobs, Intervals, and Timeouts — summarized below:

- **Cron jobs (`@Cron()`) — calendar-based scheduling:**
  - Use cron expressions (e.g., `0 0 * * *` = midnight daily, `*/15 * * * *` = every 15 minutes).
  - Best for specific times/dates (daily at 2 AM, weekdays at 9 AM, first of month).
  - Support options like `name` and `timeZone`.

- **Intervals (`@Interval()`) — fixed-frequency execution:**
  - Accept milliseconds (e.g., `@Interval(5000)` = every 5 seconds).
  - First run occurs immediately on startup, then repeats every interval.
  - Good for polling, health checks, metrics collection, cache refresh.

- **Timeouts (`@Timeout()`) — one-time delayed execution:**
  - Accept milliseconds (e.g., `@Timeout(10000)` = runs once after 10 seconds).
  - Runs only once (use for cache warmup, delayed initialization tasks).

- **Shared features:**
  - **Named tasks**: give tasks a `name` to manage them via `SchedulerRegistry` (add/get/delete).
  - **Full NestJS DI**: scheduled methods can inject services, repositories, `ConfigService`, `Logger`, etc.
  - **Dynamic control**: add/remove jobs at runtime using `SchedulerRegistry`.

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

**Key Takeaway:** `@nestjs/schedule` provides three scheduling primitives—Cron jobs, Intervals, and Timeouts—choose based on timing needs:

- **Cron jobs (`@Cron()`):** calendar-based schedules using cron expressions. Examples:
  - `@Cron('0 0 * * *')` — midnight daily
  - `@Cron('0 9 * * 1-5')` — weekdays at 9 AM
  - `@Cron(CronExpression.EVERY_DAY_AT_2AM)` — enum constant (preferred)
  - Use for: daily cleanup, monthly reports, business-hour tasks
- **Intervals (`@Interval(milliseconds)`):** fixed-frequency execution every N milliseconds.
  - `@Interval(5000)` → every 5 seconds
  - `@Interval(60000)` → every minute
  - Behavior: first run occurs on startup, then repeats every interval
  - Use for: health checks, polling, monitoring
- **Timeouts (`@Timeout(milliseconds)`):** one-time delayed execution after N milliseconds.
  - `@Timeout(5000)` → runs once after 5 seconds
  - Use for: initialization tasks (cache warming), delayed startup actions
- **Shared features:**
  - Named jobs: `name` option (e.g., `@Cron('..', { name: 'jobName' })`) for `SchedulerRegistry` management
  - Full NestJS DI support (inject services, repositories, config, logger)
  - Timezone option: `{ timeZone: 'America/New_York' }` when local time matters
  - Dynamic add/remove via `SchedulerRegistry` at runtime
- **When to choose:**
  - Use Cron for calendar/date-specific schedules
  - Use Interval for steady, frequent polling or monitoring
  - Use Timeout for one-off startup or delayed tasks

</details>

## Cron Jobs

<details>
<summary><strong>4. What are cron jobs?</strong></summary>

**Answer:**

**Cron jobs** — scheduled tasks that run at specific times or dates using cron expressions. Quick summary:

- **Origin & purpose:** Named after the Unix `cron` daemon (chronos = time). Use cron jobs for calendar-based schedules (daily, weekly, monthly).
- **Cron expression format:** 5- or 6-field string (minute hour day month dayOfWeek; optional `second` at front). Example:
  - `0 0 * * *` → runs at minute=0, hour=0 every day (midnight)
- **How to use in NestJS:** Decorate a service method with `@Cron()` using either a cron string or the `CronExpression` enum for clarity (e.g., `@Cron(CronExpression.EVERY_DAY_AT_2AM)`).
- **Common patterns:**
  - Hourly: `0 * * * *`
  - Daily (midnight): `0 0 * * *`
  - Weekdays 9 AM: `0 9 * * 1-5`
  - Every 15 minutes: `*/15 * * * *`
  - 1st of month: `0 0 1 * *`
- **Use cases:** scheduled reports, data cleanup, nightly backups, batch processing, time-specific notifications.
- **Benefits:** human-readable schedules, calendar-aware (weekdays/month boundaries), precise timing, flexible patterns, timezone support.

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

**Key Takeaway:** Cron jobs are scheduled tasks that run at specific times or dates using cron expressions. Quick reference:

- **What they are:** Scheduled tasks triggered by a cron expression (calendar-based timing).
- **Cron expression (5–6 fields):** `minute hour day month dayOfWeek` (optional `second` at start for 6-field). Examples:
  - `0 0 * * *` → midnight daily
  - `0 9 * * 1-5` → 9 AM on weekdays
  - `*/15 * * * *` → every 15 minutes
- **Special characters:** `*` (every), `-` (range, e.g. `1-5`), `,` (list, e.g. `1,3,5`), `/` (step, e.g. `*/15`).
- **How to use in NestJS:** Use `@Cron()` on a service method with either a cron string or the `CronExpression` enum for type-safety (e.g. `@Cron(CronExpression.EVERY_DAY_AT_2AM)`).
- **Common options:**
  - `name` — give a job an identifier for dynamic management
  - `timeZone` — run at local times regardless of server TZ (IANA names)
  - `disabled` — define job but keep it inactive
- **Common patterns:** hourly (`0 * * * *`), daily midnight (`0 0 * * *`), weekdays 9 AM (`0 9 * * 1-5`), every 15 minutes (`*/15 * * * *`), 1st of month (`0 0 1 * *`), business hours (`0 9-17 * * 1-5`), quarterly (`0 0 1 1,4,7,10 *`).
- **Use cases:** scheduled reports, data cleanup, nightly backups, hourly batch processing, time-specific notifications.
- **Benefits:** human-readable, calendar-aware (weekdays/months), precise timing, flexible patterns, timezone support.

</details>

<details>
<summary><strong>5. How do you create a cron job using `@Cron()` decorator?</strong></summary>

**Answer:**

- **Quick steps:**
  - Install: `npm install @nestjs/schedule`
  - Enable scheduling: import `ScheduleModule.forRoot()` in `AppModule`
  - Create a service class and mark it `@Injectable()`
  - Add a method decorated with `@Cron()` using a cron string or `CronExpression` enum
  - Register the service in the module `providers` array
- **Decorator options & examples:**
  - Cron string: `@Cron('0 0 * * *')` (midnight daily)
  - Enum: `@Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT)` (preferred for clarity)
  - With options: `@Cron('0 0 * * *', { name: 'backup', timeZone: 'America/New_York' })`
- **Method features:**
  - Methods may be `async` and return a `Promise`
  - Full NestJS DI: inject repositories/services via constructor
  - Use `Logger` and wrap logic in try/catch for reliability
- **Options explained:**
  - `name` — identifier for dynamic management via `SchedulerRegistry`
  - `timeZone` — IANA timezone string to run at local times regardless of server TZ
  - `disabled` — define the job but keep it inactive until enabled
- **Lifecycle:** Registered jobs start automatically on app startup and run until the app stops

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

**Key Takeaway:** Creating a `@Cron()` job — quick steps

- Setup:
  - Install package: `npm install @nestjs/schedule`
  - Import `ScheduleModule.forRoot()` in `AppModule`
- Create a cron job:
  - Add `@Injectable()` to your service class
  - Define a method and decorate it with `@Cron(expression)`
  - Register the service in the module's `providers` array
- Decorator syntax examples:
  - `@Cron('0 0 * * *')` — cron string (midnight daily)
  - `@Cron(CronExpression.EVERY_DAY_AT_2AM)` — enum (type-safe)
  - `@Cron('0 0 * * *', { name: 'backup', timeZone: 'America/New_York' })` — with options
- Options:
  - `name` — identifier for dynamic management via `SchedulerRegistry`
  - `timeZone` — IANA timezone (runs at local time regardless of server TZ)
  - `disabled` — define but don't activate
- Method features & best practices:
  - Methods can be `async` and return Promise for DB/API work
  - Use dependency injection (services, repositories, ConfigService)
  - Wrap logic in `try/catch` and use `Logger` for failures
  - Make jobs idempotent where possible
- Execution & lifecycle:
  - `ScheduleModule` scans for decorators on startup and registers jobs
  - Methods are invoked automatically by the scheduler (no params)
  - Jobs run until the app stops
- Multiple jobs:
  - A single service may contain multiple `@Cron()` methods with different schedules
  - They share DI dependencies and run independently

</details>

<details>
<summary><strong>6. What is cron syntax and how do you read it?</strong></summary>

**Answer:**

- Cron syntax — quick reference

- Fields:
  - 6-field format (with seconds): `second minute hour day month dayOfWeek`
  - 5-field format (standard Unix): `minute hour day month dayOfWeek`
- Field ranges:
  - Minutes: `0-59`
  - Hours: `0-23`
  - Day of month: `1-31`
  - Month: `1-12`
  - Day of week: `0-6` (0 = Sunday)
- Operators:
  - `*` — wildcard (every value)
  - `-` — range (e.g., `1-5`)
  - `,` — list (e.g., `1,3,5`)
  - `/` — step (e.g., `*/15` = every 15 units)
  - `?` — no specific value (used in some cron flavors)
- Reading tips:
  - Read left → right (minute → hour → day → month → dayOfWeek)
  - Combine fields with AND logic
  - `*` means "every" for that field
  - Ranges are inclusive (e.g., `1-5` includes 1..5)
  - Steps apply within the field range (e.g., `*/15` → 0,15,30,45)
- Examples:
  - `0 0 * * *` → midnight daily
  - `0 9 * * 1-5` → weekdays at 9 AM
  - `*/15 * * * *` → every 15 minutes
  - `0 0 1 * *` → 1st of month at midnight

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

**Key Takeaway:** Cron syntax — quick reference

- Format:
  - 6-field (with seconds): `second minute hour dayOfMonth month dayOfWeek`
  - 5-field (standard Unix): `minute hour dayOfMonth month dayOfWeek`
- Field value ranges:
  - Minutes: `0-59`
  - Hours: `0-23`
  - Day of month: `1-31`
  - Month: `1-12`
  - Day of week: `0-6` (0 = Sunday)
- Operators:
  - `*` — wildcard (any value)
  - `-` — range (e.g., `1-5` = 1 through 5)
  - `,` — list (e.g., `1,3,5`)
  - `/` — step (e.g., `*/15` = every 15 units)
- Reading approach: read left→right, interpret each field, then combine with AND logic
- Examples:
  - `0 0 * * *` — midnight daily
  - `0 9 * * 1-5` — weekdays at 9 AM
  - `*/15 * * * *` — every 15 minutes
  - `0 0 1 * *` — 1st of month at midnight
- Complex patterns examples:
  - `*/30 9-17 * * 1-5` — every 30 minutes between 9 AM–5 PM on weekdays
  - `0 8,12,16 * * *` — at 8:00, 12:00, and 16:00 daily
  - `0 0 1,15 * *` — midnight on 1st and 15th of each month

</details>

<details>
<summary><strong>7. What are common cron patterns (every minute, hourly, daily)?</strong></summary>

**Answer:**

- Common cron patterns (examples):
  - Every second: `* * * * * *` (`CronExpression.EVERY_SECOND`) — testing only
  - Every minute: `0 * * * * *` (`CronExpression.EVERY_MINUTE`)
  - Every 5 minutes: `0 */5 * * * *` (`CronExpression.EVERY_5_MINUTES`)
  - Every 15 minutes: `*/15 * * * *`
  - Every 30 minutes: `0 */30 * * * *` (`CronExpression.EVERY_30_MINUTES`)
  - Every hour: `0 0 * * * *` (`CronExpression.EVERY_HOUR`)
  - Every day at midnight: `0 0 0 * * *` (`CronExpression.EVERY_DAY_AT_MIDNIGHT`)
  - Every day at specific time (e.g., 2 AM): `0 0 2 * * *` (`CronExpression.EVERY_DAY_AT_2AM`)
  - Weekdays at 9 AM: `0 0 9 * * 1-5`
  - Weekly (Sunday midnight): `0 0 0 * * 0` (`CronExpression.EVERY_WEEK`)
  - Monthly (1st at midnight): `0 0 0 1 * *` (`CronExpression.EVERY_MONTH`)
  - Yearly (Jan 1st at midnight): `0 0 0 1 1 *` (`CronExpression.EVERY_YEAR`)
- Use `CronExpression` enum for common patterns (type-safe, readable); use custom strings for specific needs (e.g., business hours, quarterly)
- Typical purposes by frequency:
  - Frequent (every minute/5 minutes): monitoring, alerting
  - Hourly: data sync, cache refresh
  - Daily: cleanup, reports, off-peak backups
  - Weekly: summaries, weekly reports
  - Monthly: billing, subscription processing
  - Yearly: annual tasks, compliance

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

**Key Takeaway:**

- Common cron patterns organized by frequency:
  - **High-frequency:**
    - Every second: `* * * * * *` (testing only)
    - Every 5 seconds: `*/5 * * * * *`
    - Every minute: `0 * * * * *` or `CronExpression.EVERY_MINUTE`
    - Every 5 minutes: `0 */5 * * * *` or `EVERY_5_MINUTES`
    - Every 15 minutes: `*/15 * * * *`
    - Every 30 minutes: `0 */30 * * * *` or `EVERY_30_MINUTES`
  - **Hourly:**
    - Every hour: `0 0 * * * *` or `EVERY_HOUR`
    - Every 2 hours: `0 */2 * * *`
    - Business hours 9-5: `0 9-17 * * *`
    - Specific hours: `0 9,12,15,18 * * *`
  - **Daily:**
    - Midnight: `0 0 0 * * *` or `EVERY_DAY_AT_MIDNIGHT`
    - 2 AM for off-peak: `0 0 2 * * *` or `EVERY_DAY_AT_2AM`
    - Noon: `0 0 12 * * *` or `EVERY_DAY_AT_NOON`
    - Weekdays 9 AM: `0 0 9 * * 1-5`
    - Weekdays 5 PM: `0 0 17 * * 1-5`
  - **Weekly/Monthly/Yearly:**
    - Every Sunday: `0 0 0 * * 0` or `EVERY_WEEK`
    - Every Monday 8 AM: `0 0 8 * * 1`
    - 1st of month: `0 0 0 1 * *` or `EVERY_MONTH`
    - 15th of month: `0 0 0 15 * *` (for bi-monthly)
    - Quarterly: `0 0 0 1 1,4,7,10 *` (for Jan/Apr/Jul/Oct 1st)
    - Yearly: `0 0 0 1 1 *` or `EVERY_YEAR` (for Jan 1st)
- **Use CronExpression enum** for common patterns (type-safe, self-documenting, no syntax errors)
- **Custom strings** for specific needs (weekday mornings, business hours, quarterly schedules)
- **Purpose-based:**
  - Monitoring: every minute/5 minutes
  - API sync: every 15 minutes
  - Cache refresh: hourly
  - Cleanup: daily 2 AM
  - Backups: daily off-peak
  - Reports: daily/weekly/monthly
  - Billing: monthly 1st
  - Compliance: quarterly/yearly

</details>

<details>
<summary><strong>8. How do you schedule a job to run at midnight every day?</strong></summary>

**Answer:**

- Schedule a job to run at midnight every day using `@Cron()` decorator with **midnight pattern**:
  - `@Cron('0 0 0 * * *')` (6 fields with seconds)
  - `@Cron('0 0 * * *')` (5 fields standard)
  - Prefer `@Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT)` for type-safety and readability
- **Expression meaning:** `0 0 0 * * *` → second=0, minute=0, hour=0 (midnight) — runs at 00:00:00 every day
- **Options:**
  - `timeZone` to run at local midnight (e.g., `{ timeZone: 'America/New_York' }`)
  - `name` for dynamic management (e.g., `{ name: 'dailyCleanup' }`)
  - `disabled: true` to define but not activate
- **Best practices:**
  - Use an **async** method for DB/API work
  - Wrap logic in `try/catch` and log start/completion/errors
  - Track execution metrics (duration, records processed)
  - Make tasks **idempotent** (safe to re-run)
  - Stagger heavy midnight tasks to avoid contention
  - Prefer off-peak windows (midnight–3 AM) for heavy work
- Scheduler runs the job automatically at midnight; no manual invocation required

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

**Key Takeaway:**

- Schedule a job at midnight daily using:
  - `@Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT)` (recommended enum for type-safety and readability)
  - `@Cron('0 0 0 * * *')` (6-field expression: second=0, minute=0, hour=0=midnight, any day, any month, any dayOfWeek)
  - `@Cron('0 0 * * *')` (5-field standard Unix)
- **Expression breakdown:**
  - `0 0 0 * * *` runs at exactly 00:00:00 every day (midnight)
  - `0 0 * * *` is equivalent 5-field format
- **Options:**
  - Add `timeZone` for specific timezone (e.g., `{ timeZone: 'America/New_York' }` runs at midnight in NY regardless of server location, handles DST automatically)
  - Add `name` for dynamic management (e.g., `{ name: 'dailyCleanup' }` allows add/remove via SchedulerRegistry)
  - Set `disabled: true` to define but not activate
- **Best practices:**
  - Use **async method** for database/API operations
  - Implement **try-catch** for error handling (log failures, send alerts to ops team)
  - Add **comprehensive logging** (log start time, progress, completion time, items processed)
  - Track **execution metrics** (duration, records affected)
  - Send **notifications** on completion or failure
  - Make **idempotent** (safe to run multiple times)
  - Use **off-peak hours** (midnight-3 AM for heavy tasks like backups, cleanup, reports)
- **Variations:**
  - Weekdays only: `0 0 0 * * 1-5`
  - Weekends only: `0 0 0 * * 0,6`
  - 1st of month: `0 0 0 1 * *`
  - Bi-monthly: `0 0 0 1,15 * *`
- **Stagger jobs:** If multiple midnight tasks, stagger them (00:00 backup, 00:10 reports, 00:30 analytics) to prevent resource contention and ensure dependencies met
- **Use cases:**
  - Daily cleanup (delete old logs/sessions)
  - Database backups (full backup during off-peak)
  - Report generation (daily sales/analytics)
  - Data aggregation (consolidate previous day)
  - Batch processing (heavy computation when traffic low)

</details>

<details>
<summary><strong>9. How do you use cron names for managing jobs?</strong></summary>

**Answer:**

- Use **cron names** by passing a `name` property in the options object of `@Cron()`, `@Interval()`, or `@Timeout()` decorators:
  - Example: `@Cron('0 0 * * *', { name: 'dailyBackup' })`
- The name acts as a **unique identifier** allowing **dynamic management** via `SchedulerRegistry` service
- With a named job, you can:
  - **Get the job:** `schedulerRegistry.getCronJob('dailyBackup')`
  - **Start/stop job:** `job.start()`, `job.stop()`
  - **Delete job:** `schedulerRegistry.deleteCronJob('dailyBackup')`
  - **Check if exists:** `schedulerRegistry.doesExist('cron', 'dailyBackup')`
  - **Get all cron jobs:** `schedulerRegistry.getCronJobs()`
- **Benefits:**
  - **Runtime control:** enable/disable jobs based on conditions
  - **Monitoring:** check job status, next execution time
  - **Dynamic configuration:** add/remove jobs based on user settings
  - **Debugging:** pause jobs during maintenance
  - **Graceful shutdown:** stop jobs before app termination
- Without a name, jobs are **anonymous** and cannot be accessed dynamically (only managed via decorator)
- **Naming conventions:**
  - Use descriptive names (camelCase): 'dailyBackup', 'hourlySync', 'weeklyReport'
  - Names must be **unique** within job type (cron/interval/timeout can share same name across types but not within same type)

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

**Key Takeaway:**

- Use **cron names** by adding a `name` property in decorator options:
  - Example: `@Cron('0 0 * * *', { name: 'dailyBackup' })` creates a uniquely identified job accessible via `SchedulerRegistry`.

- **Benefits:**
  - **Runtime control:** Start/stop jobs dynamically with `job.start()`/`job.stop()` without redeploying
  - **Monitoring:** Check if running with `job.running`, get next execution with `job.nextDate()`, get last execution with `job.lastDate()`
  - **Dynamic management:** Delete with `schedulerRegistry.deleteCronJob(name)`, check existence with `schedulerRegistry.doesExist('cron', name)`
  - **Bulk operations:** Get all jobs with `schedulerRegistry.getCronJobs()` (returns Map), iterate and manage multiple jobs
  - **Graceful shutdown:** Stop all jobs before app termination

- **SchedulerRegistry methods:**
  - `getCronJob(name)`: retrieves job instance
  - `getCronJobs()`: returns Map of all jobs
  - `deleteCronJob(name)`: removes job
  - `doesExist('cron', name)`: checks if job exists

- **CronJob instance methods:**
  - `start()`: begins execution
  - `stop()`: halts execution
  - `running`: property shows current state (boolean)
  - `nextDate()`: returns next run time (Luxon DateTime)
  - `lastDate()`: returns last run time or null
  - `cronTime.source`: shows original cron expression

- **Naming conventions:**
  - Use camelCase descriptive names (e.g., 'dailyBackup', 'hourlySync', 'priceUpdate')
  - Names must be unique within job type (can't have two cron jobs with same name, but cron/interval/timeout can share names across types)
  - Be specific about frequency and purpose

- **Use cases:**
  - **Conditional execution:** Stop job during maintenance, start after config update
  - **User-controlled jobs:** Enable/disable via admin panel, REST API for job management
  - **Monitoring dashboards:** Show all jobs status, display next run times, alert if critical jobs stopped
  - **Testing/debugging:** Pause jobs during development, restart jobs to test changes
  - **Dynamic configuration:** Add jobs based on user settings, remove jobs when features disabled
  - **Health checks:** Verify critical jobs running, monitor execution times, alert on failures

- **Without names:**
  - Jobs are anonymous and cannot be accessed or managed dynamically
  - Only controllable via decorator definition
  - Suitable for simple fire-and-forget tasks that never need runtime management

</details>

## Intervals & Timeouts

<details>
<summary><strong>10. How do you schedule tasks at intervals using `@Interval()`?</strong></summary>

**Answer:**

- Schedule tasks at **fixed intervals** using the `@Interval()` decorator:
  - `@Interval(milliseconds)` or `@Interval(name, milliseconds)` for named intervals
  - Accepts **milliseconds** as parameter (not cron expression)
  - Examples: `@Interval(5000)` runs every 5 seconds, `@Interval(60000)` runs every minute
- **Execution behavior:**
  - Runs **immediately on app startup** (first execution at 0 seconds)
  - Repeats every N milliseconds continuously until app stops
- **Named intervals:**
  - Allow dynamic management: `@Interval('healthCheck', 30000)` creates named interval accessible via `SchedulerRegistry`
  - Management: **get interval** (`schedulerRegistry.getInterval('healthCheck')`), **delete interval** (`schedulerRegistry.deleteInterval('healthCheck')`), **add dynamically** (`schedulerRegistry.addInterval(name, setInterval(callback, ms))`)
- **Difference from Cron:**
  - Intervals use **fixed time between executions** (every 30 seconds regardless of clock time)
  - Cron uses **calendar-based scheduling** (specific times like midnight, 9 AM)
- **Use intervals for:**
  - **Polling:** check status every 10 seconds
  - **Health checks:** ping services every 30 seconds
  - **Metrics collection:** gather stats every minute
  - **Monitoring:** continuous checking
  - When **specific time doesn't matter** (just need regular execution)
- **Avoid intervals for:**
  - Tasks needing specific times (use Cron)
  - Very high frequency (every second strains resources)
  - Calendar-based tasks (daily, weekly, monthly use Cron)

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

**Key Takeaway:**

- Schedule tasks at **fixed intervals** using the `@Interval(milliseconds)` decorator:
  - Examples: `@Interval(5000)` (every 5 seconds), `@Interval(60000)` (every minute), `@Interval(5 * 60 * 1000)` (every 5 minutes)
- **Execution behavior:**
  - Runs **immediately on app startup** (first execution at 0 seconds)
  - Repeats every N milliseconds continuously until the app stops
  - **Fixed time between executions** (not tied to clock time)
  - Example: If app starts at 10:37:42, with a 60000ms interval, runs at 10:37:42, 10:38:42, 10:39:42, etc.
- **Named intervals:**
  - Use `@Interval('healthCheck', 30000)` for dynamic management via SchedulerRegistry
  - Access: `schedulerRegistry.getInterval('healthCheck')`
  - Delete: `schedulerRegistry.deleteInterval('healthCheck')`
  - Check existence: `schedulerRegistry.doesExist('interval', 'healthCheck')`
  - Get all: `schedulerRegistry.getIntervals()` (returns Map)
- **Dynamic intervals:**
  - Add at runtime: `schedulerRegistry.addInterval(name, setInterval(callback, ms))`
  - Remove: `schedulerRegistry.deleteInterval(name)`
  - Useful for user-configured schedules or conditional intervals
- **Use cases:**
  - **Health checks:** ping database/Redis every 30 seconds
  - **Queue monitoring:** check depth every 10 seconds, alert if above threshold
  - **Metrics collection:** gather CPU/memory every minute
  - **Polling:** check external API every 15 seconds for updates
  - **Cache refresh:** update frequently accessed data every 5 minutes
  - **Status monitoring:** continuous checking of system state
  - When **specific time doesn't matter** (just need regular periodic execution)
- **Difference from Cron:**
  - Intervals use fixed duration between executions (mechanical timing)
  - Cron uses calendar-based scheduling (specific clock times like midnight, 9 AM)
  - Intervals are not aware of time zones or daylight saving
  - Intervals are simpler for regular polling/monitoring
- **Avoid intervals for:**
  - Tasks needing specific times (use Cron for "midnight daily" or "9 AM weekdays")
  - Very high frequency (every second; use with caution, can strain resources)
  - Calendar-based tasks (daily, weekly, monthly are better with Cron)
  - Time-sensitive operations (e.g., reports at 2 AM; use Cron)
- **Milliseconds reference:**
  - 1s = 1000ms
  - 5s = 5000ms
  - 10s = 10000ms
  - 30s = 30000ms
  - 1min = 60000ms
  - 5min = 300000ms
  - 15min = 900000ms
  - 1hr = 3600000ms

</details>

<details>
<summary><strong>11. What is the difference between `@Interval()` and `@Cron()`?</strong></summary>

**Answer:**

`@Interval()` and `@Cron()` differ in their **trigger mechanism**:

- `@Interval()` uses **fixed time intervals** in milliseconds:
  - Example: `@Interval(60000)` runs every 60 seconds from the last execution completion
- `@Cron()` uses **calendar-based scheduling** with cron expressions:
  - Example: `@Cron('0 * * * *')` runs every hour at minute 0

**Key differences:**
- **Timing:**
  - Interval: runs every N milliseconds continuously from app start (first run immediate, subsequent runs after interval)
  - Cron: runs at specific times/dates (e.g., midnight, 9 AM weekdays, 1st of month)
- **Syntax:**
  - Interval: takes a milliseconds number (e.g., 5000, 60000, 300000)
  - Cron: takes an expression string or enum (e.g., '0 0 * * *', `CronExpression.EVERY_HOUR`)
- **Use cases:**
  - Interval: fixed-frequency tasks (health checks every 30 seconds, metrics every minute, polling)
  - Cron: calendar-based tasks (daily reports at 2 AM, weekly summaries, monthly billing)
- **Precision:**
  - Interval: can drift over time (if task takes 2 seconds with a 60-second interval, actual interval is 62 seconds)
  - Cron: runs at exact time (always at 00:00:00 regardless of previous run duration)
- **First execution:**
  - Interval: runs immediately on app start, then after each interval
  - Cron: waits until the scheduled time for first execution
- **Business logic:**
  - Interval: ignores calendar (runs on weekends, holidays, business hours, etc.)
  - Cron: is calendar-aware (can target weekdays only, business hours, specific dates)

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

**Key Takeaway:**

- `@Interval()` and `@Cron()` differ fundamentally in their **trigger mechanism**:
  - `@Interval(milliseconds)` uses **fixed time intervals**:
    - Runs every N milliseconds continuously (e.g., `@Interval(60000)` = every 60 seconds, `@Interval(5 * 60 * 1000)` = every 5 minutes)
    - Runs immediately on app start, then every interval
    - Creates **time drift** if task duration varies (e.g., 60-second interval with 5-second task = 65-second actual interval)
  - `@Cron(expression)` uses **calendar-based scheduling**:
    - Runs at specific times/dates (e.g., `@Cron('0 0 * * *')` = midnight daily, `@Cron('0 9 * * 1-5')` = weekdays 9 AM)
    - Waits for scheduled time before first run
    - **No time drift**: always runs at exact time regardless of previous duration

- **Key differences:**
  - **Timing:**
    - Interval: relative to last execution (N milliseconds after completion)
    - Cron: absolute calendar time (9 AM, midnight, 1st of month)
  - **Syntax:**
    - Interval: takes milliseconds number (simple numeric value)
    - Cron: takes expression string or enum (complex time pattern)
  - **First execution:**
    - Interval: runs immediately at startup (time 0)
    - Cron: waits until next scheduled time (may be hours away)
  - **Precision:**
    - Interval: accumulates drift over time (if task takes longer/shorter, interval shifts)
    - Cron: maintains exact timing (always at specified time sharp)
  - **Calendar awareness:**
    - Interval: ignores calendar (runs 24/7 including weekends/holidays)
    - Cron: calendar-aware (weekdays only, business hours, specific months)

- **Use `@Interval()` for:**
  - Fixed-frequency tasks (health checks every 30 seconds)
  - Continuous monitoring (queue depth every minute)
  - Polling (API status checks)
  - Regular updates (cache refresh every 5 minutes)
  - When exact time doesn't matter

- **Use `@Cron()` for:**
  - Specific times (daily at 2 AM, weekdays at 9 AM)
  - Calendar-based schedules (weekly Monday, monthly 1st)
  - Business hours (9-5 weekdays only)
  - Off-peak processing (2-4 AM)
  - When exact timing is critical (reports at midnight sharp)

</details>

<details>
<summary><strong>12. How do you schedule one-time delayed tasks using `@Timeout()`?</strong></summary>

**Answer:**


- Schedule one-time delayed tasks using the `@Timeout(milliseconds)` decorator:
  - `@Timeout(5000)` runs the method **once** after 5 seconds from app startup
  - The decorator takes **milliseconds** (e.g., 5000 = 5 seconds, 60000 = 1 minute, 600000 = 10 minutes)
  - Or use **name and milliseconds**: `@Timeout('initCache', 10000)`

- Unlike `@Interval()` (repeats continuously) or `@Cron()` (repeats on schedule), `@Timeout()` executes **exactly once** then never again until app restarts.

- **Use cases:**
  - **Initialization tasks:** pre-load cache, warm connections, seed data
  - **Delayed startup:** wait for dependencies to be ready, give time for services to start
  - **One-time operations:** send welcome email after delay, cleanup temp files after processing

- The method can be **async** for database/API operations
- Has access to **dependency injection** (inject services via constructor)
- Supports **named timeouts** for dynamic management via SchedulerRegistry (delete timeout: `schedulerRegistry.deleteTimeout('initCache')`)

- **Timing:**
  - Starts counting from app startup (time 0)
  - Executes after N milliseconds
  - Never executes again

- For recurring tasks use `@Interval()` or `@Cron()`, for one-time delayed use `@Timeout()`

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

**Key Takeaway:**

- Schedule one-time delayed tasks using the `@Timeout(milliseconds)` decorator, which executes a method **exactly once** after a specified delay from app startup.
  - Examples:
    - `@Timeout(5000)` runs after 5 seconds
    - `@Timeout(60000)` after 1 minute
    - `@Timeout(5 * 60 * 1000)` after 5 minutes

- **Syntax:**
  - `@Timeout(milliseconds)` for anonymous timeout
  - `@Timeout('name', milliseconds)` for named timeout manageable via SchedulerRegistry
    - Can cancel before execution with `schedulerRegistry.deleteTimeout('name')`
    - Check existence with `schedulerRegistry.doesExist('timeout', 'name')`

- **Execution behavior:**
  - Timer starts at app startup (time 0)
  - Waits N milliseconds
  - Executes method once
  - **Never executes again** until app restarts (unlike `@Interval()` or `@Cron()`)

- **Use cases:**
  - **Initialization tasks:**
    - Cache warming after 5 seconds to pre-load frequently accessed data
    - Connection pool warmup after 3 seconds
    - Initial data sync after 30 seconds
  - **Delayed startup:**
    - Give time for dependencies to be ready
    - Stagger initialization tasks to avoid startup bottleneck
    - Non-critical tasks that shouldn't block app availability
  - **One-time operations:**
    - Send startup notification after 10 seconds
    - Perform initial health check after delay
    - Cleanup temporary startup files

- **Method features:**
  - Can be **async** for database/API operations with `await`
  - Has full **dependency injection** access (inject repositories, services, cache, email via constructor)
  - Supports **error handling** with try-catch (failures shouldn't crash app, log errors and continue)
  - Use **Logger** for tracking execution

- **Comparison:**
  - `@Timeout()` runs once after delay (one-time initialization)
  - `@Interval()` runs continuously every N ms (monitoring, polling)
  - `@Cron()` runs at specific times repeatedly (daily reports, scheduled tasks)

- **Best practices:**
  - Use for app-level initialization, not user-triggered tasks (for user actions use message queues)
  - Stagger multiple timeouts (3s, 5s, 10s, 30s) to spread load
  - Don't throw errors from timeout methods (log and continue)
  - Combine with named timeouts for cancellation capability if conditions change

</details>

## Dynamic Scheduling

<details>
<summary><strong>13. How do you dynamically schedule/unschedule jobs?</strong></summary>

**Answer:**


- Dynamically schedule/unschedule jobs using the **SchedulerRegistry** service:
  - Inject `SchedulerRegistry` from `@nestjs/schedule` via constructor.
  - Use methods:
    - **addCronJob** / **deleteCronJob** for cron jobs
    - **addInterval** / **deleteInterval** for intervals
    - **addTimeout** / **deleteTimeout** for timeouts

- **Scheduling:**
  - Create job:
    - `new CronJob()` for cron jobs
    - `setInterval()` for intervals
    - `setTimeout()` for timeouts
  - Add to registry: `schedulerRegistry.addCronJob(name, job)`
  - Start: `job.start()`

- **Unscheduling:**
  - Retrieve job: `schedulerRegistry.getCronJob(name)`
  - Stop: `job.stop()`
  - Delete: `schedulerRegistry.deleteCronJob(name)`

- **Use cases:**
  - **User preferences:** Users enable/disable scheduled reports
  - **Configuration changes:** Admin modifies backup schedule
  - **Conditional scheduling:** Enable jobs based on feature flags
  - **Dynamic frequency:** Adjust monitoring interval based on load
  - **Multi-tenant:** Each tenant has a custom schedule

- Unlike static decorators (`@Cron()`, `@Interval()`, `@Timeout()`) which are **compile-time** (defined in code, always active), dynamic scheduling is **runtime** (add/remove jobs while app is running based on data/conditions).

- Methods return void (no confirmation); always check existence with `doesExist()` before operations to avoid errors.

- **Pattern:**
  - Create job → add to registry → start → stop when needed → delete from registry

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

**Key Takeaway:**

- Dynamically schedule/unschedule jobs using the **SchedulerRegistry** service from `@nestjs/schedule`, which manages jobs at runtime.
  - Inject `SchedulerRegistry` via constructor.
  - Use:
    - **addCronJob(name, job)** / **deleteCronJob(name)** for cron jobs
    - **addInterval(name, interval)** / **deleteInterval(name)** for intervals
    - **addTimeout(name, timeout)** / **deleteTimeout(name)** for timeouts

- **Dynamic cron workflow:**
  - Create job with `new CronJob(cronPattern, callback)`
  - Add to registry: `schedulerRegistry.addCronJob('jobName', job)`
  - Start execution: `job.start()`
  - Stop: `job.stop()`
  - Delete: `schedulerRegistry.deleteCronJob('jobName')`

- **Dynamic interval workflow:**
  - Create with `setInterval(callback, milliseconds)`
  - Add to registry: `schedulerRegistry.addInterval('name', interval)`
  - Remove: `schedulerRegistry.deleteInterval('name')` (automatically clears interval)

- **Dynamic timeout workflow:**
  - Create with `setTimeout(callback, milliseconds)`
  - Add to registry: `schedulerRegistry.addTimeout('name', timeout)`
  - Cancel: `schedulerRegistry.deleteTimeout('name')`

- **Advantages over static decorators:**
  - Static `@Cron()` / `@Interval()` / `@Timeout()` are **compile-time** (defined in code, always active when app runs, cannot change without redeploying)
  - Dynamic scheduling is **runtime** (add/remove jobs while app is running based on database data, user preferences, configuration changes, feature flags, conditional logic)

- **Use cases:**
  - **User preferences:** Users enable/disable scheduled reports (`enableUserReport(userId, '0 9 * * *')`)
  - **Multi-tenant:** Each tenant has a custom schedule
  - **Configuration-driven:** Load schedules from config file/database on startup, hot-reload without restart
  - **Conditional scheduling:** Enable monitoring jobs only in production, disable during maintenance
  - **Dynamic frequency:** Adjust polling interval based on system load

- **Helper methods:**
  - Check existence with `schedulerRegistry.doesExist('cron'|'interval'|'timeout', name)` before operations
  - Get all jobs with `schedulerRegistry.getCronJobs()` / `getIntervals()` / `getTimeouts()` (returns `Map<string, JobType>`)
  - List names with `Array.from(map.keys())`

- **Best practices:**
  - Always check `doesExist()` before stop/delete to avoid errors
  - Store schedule metadata in database for persistence across restarts
  - Load saved schedules in `onModuleInit()` lifecycle hook
  - Use descriptive job names with prefixes (`user-report-${userId}`, `tenant-${tenantId}-backup`)
  - Cleanup jobs when no longer needed to prevent memory leaks
  - Handle errors in job callbacks (don't throw, log and continue)

</details>

<details>
<summary><strong>14. What is `SchedulerRegistry` and how do you use it?</strong></summary>

**Answer:**


**SchedulerRegistry Overview:**

- `SchedulerRegistry` is a **central registry service** from `@nestjs/schedule` that provides programmatic access to manage all scheduled jobs (cron jobs, intervals, timeouts) at runtime.
- Acts as a **job store** where all scheduled tasks are registered by name, allowing **dynamic management** (add, retrieve, delete, list jobs).

- **Injection:**
  - Import from `@nestjs/schedule` and inject via constructor: `constructor(private schedulerRegistry: SchedulerRegistry)`

- **Methods:**
  - `getCronJob(name)` / `getCronJobs()`: Retrieve cron jobs
  - `addCronJob(name, job)` / `deleteCronJob(name)`: Manage cron jobs
  - `getInterval(name)` / `getIntervals()`: Retrieve intervals
  - `addInterval(name, interval)` / `deleteInterval(name)`: Manage intervals
  - `getTimeout(name)` / `getTimeouts()`: Retrieve timeouts
  - `addTimeout(name, timeout)` / `deleteTimeout(name)`: Manage timeouts
  - `doesExist(type, name)`: Check if job exists (`type` is `'cron' | 'interval' | 'timeout'`)

- **Use cases:**
  - Dynamic scheduling (add jobs based on user input)
  - Runtime control (pause/resume jobs)
  - Monitoring (list all active jobs, check next execution)
  - Cleanup (remove jobs when no longer needed)
  - Debugging (inspect job state, manually trigger)

- **Static decorator integration:**
  - Jobs created with `@Cron(expression, { name: 'jobName' })` are automatically registered in SchedulerRegistry and accessible via `getCronJob('jobName')`.

- **Pattern:**
  - Named jobs (always use name option for management)
  - Access via registry
  - Perform operations (start/stop/delete)
  - Verify with `doesExist()`

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


**Key Takeaway:**

- `SchedulerRegistry` is a **central registry service** from `@nestjs/schedule` that provides programmatic access to manage all scheduled jobs at runtime.
- Acts as a **job store** maintaining named collections (Maps) of:
  - Cron jobs
  - Intervals
  - Timeouts
- **CRUD operations:**
  - `addCronJob(name, job)` / `deleteCronJob(name)`
  - `addInterval(name, interval)` / `deleteInterval(name)`
  - `addTimeout(name, timeout)` / `deleteTimeout(name)`
- **Injection:**
  - Import from `@nestjs/schedule` and inject via constructor: `constructor(private readonly schedulerRegistry: SchedulerRegistry) {}`
  - Available after importing `ScheduleModule.forRoot()` in your module
- **Retrieval methods:**
  - `getCronJob(name)`: Returns single CronJob instance for control (`job.start()`, `job.stop()`, `job.running`, `job.lastDate()`, `job.nextDate()`)
  - `getCronJobs()`: Returns `Map<string, CronJob>` of all cron jobs
  - `getIntervals()`: Returns `Map<string, NodeJS.Timer>` of all intervals
  - `getTimeouts()`: Returns `Map<string, NodeJS.Timeout>` of all timeouts
- **Utility method:**
  - `doesExist(type, name)`: Where type is `'cron' | 'interval' | 'timeout'`, checks if named job exists (prevents errors before operations)
- **Static decorator integration:**
  - Jobs defined with decorators including the name option (e.g., `@Cron('0 0 * * *', { name: 'dailyBackup' })`) are **automatically registered** in SchedulerRegistry and accessible via `getCronJob('dailyBackup')` for runtime control
- **Use cases:**
  - Dynamic scheduling (add/remove jobs based on database data, user preferences, configuration)
  - Runtime control (pause/resume jobs via `stop()`/`start()`)
  - Monitoring (list all active jobs for health checks, check next execution times, log job statuses)
  - Debugging (inspect job state, verify jobs running, manually trigger for testing)
  - Cleanup (remove obsolete jobs when features disabled, clear all jobs on shutdown)
- **CronJob API:**
  - Retrieved job has properties/methods:
    - `running` (is job active)
    - `lastDate()` (returns last execution time)
    - `nextDate()` (returns luxon DateTime of next execution)
    - `start()` (begins execution)
    - `stop()` (pauses execution)
- **Best practices:**
  - Always use named jobs for management (pass `{ name: 'jobName' }` in decorator options)
  - Check existence with `doesExist()` before operations to prevent errors
  - Iterate Maps with `forEach((job, name) => { ... })` or `Array.from(map.keys())`
  - Cleanup jobs in `onModuleDestroy()` lifecycle hook (stop and delete all)
  - Use for monitoring/debugging, not business logic

</details>

<details>
<summary><strong>15. How do you add a cron job dynamically at runtime?</strong></summary>

**Answer:**


**How to add a cron job dynamically at runtime:**

- **Step-by-step process:**
  - **Import `CronJob` from `cron` package:**
    - `import { CronJob } from 'cron'`
  - **Create a new CronJob instance:**
    - Example: `const job = new CronJob('0 * * * *', callback)`
    - **CronJob constructor parameters:**
      - 1st: Cron expression string (e.g., `'0 0 * * *'`)
      - 2nd: Callback function (sync or async)
      - 3rd: (Optional) onComplete callback
      - 4th: (Optional) start boolean (default `false`)
      - 5th: (Optional) timezone string (e.g., `'America/New_York'`)
  - **Inject `SchedulerRegistry`:**
    - Via constructor injection
  - **Add job to registry with unique name:**
    - `schedulerRegistry.addCronJob('jobName', job)`
    - Use descriptive names with identifiers (e.g., `user-${userId}-report`)
  - **Start the job:**
    - `job.start()`

- **Use cases:**
  - Database-driven schedules (load from database on startup)
  - User preferences (users configure their report schedules)
  - Multi-tenant (each tenant custom backup time)
  - A/B testing (enable experimental jobs for subset)
  - Feature flags (activate jobs based on configuration)

- **Advantages over static `@Cron()` decorator:**
  - Static: Compile-time, code-defined, requires redeployment
  - Dynamic: Runtime, created from data, can be added anytime, no code changes

- **Best practices:**
  - Validate cron expression before creating job (use cron-parser)
  - Store job metadata in database (for persistence across restarts)
  - Load saved jobs in `onModuleInit()`
  - Use descriptive names with identifiers
  - Handle errors in callback (try-catch, don't throw)
  - Check if job name exists before adding (avoid duplicates with `doesExist()`)

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

**Key Takeaway:** Add a cron job dynamically at runtime by creating a `CronJob` instance and registering it with `SchedulerRegistry`:

- **Step-by-step process:**
  - **Import CronJob:**
    - `import { CronJob } from 'cron'`
  - **Create the job:**
    - `new CronJob(cronExpression, callback, onComplete?, start?, timezone?)`
    - **Parameters:**
      - `cronExpression`: Cron pattern string (e.g., `'0 * * * *'` for hourly)
      - `callback`: Function to execute (can be async, supports database/API operations)
      - `onComplete`: (Optional) Callback after execution
      - `start`: Boolean to auto-start (usually `false` for manual start)
      - `timezone`: IANA timezone string (e.g., `'America/New_York'`, `'Europe/London'`)
  - **Inject SchedulerRegistry:**
    - Via constructor: `constructor(private schedulerRegistry: SchedulerRegistry)`
  - **Add job to registry:**
    - `schedulerRegistry.addCronJob(uniqueName, job)`
    - Use descriptive, unique names (e.g., `user-${userId}-report`, `tenant-${tenantId}-backup`)
  - **Start job execution:**
    - `job.start()` to activate scheduling

- **Callback features:**
  - Can be an async function with full dependency injection (inject services via constructor, use in callback)
  - Handle errors with try-catch (do not throw; log and continue to next execution)
  - Return values are ignored (side-effects only)
  - Execution is sequential (waits for previous completion before next)

- **Best practices:**
  - **Validate cron expression** before creating job (use `cron-parser`: `require('cron-parser').parseExpression(expr)` throws if invalid)
  - **Check existence** before adding (`schedulerRegistry.doesExist('cron', name)`) to prevent duplicates
  - **Persist to database:** Save job metadata for recreation after app restart; load in `onModuleInit()` lifecycle hook
  - **Use unique naming:** Include identifiers (user ID, tenant ID, type) to avoid collisions
  - **Error handling:** Wrap callback in try-catch, log errors, send alerts for critical failures
  - **Cleanup on delete:** Stop job before removing, delete database record

- **Use cases:**
  - **User preferences:** Users configure report schedules (daily at 9 AM, weekly Monday, monthly 1st)
  - **Multi-tenant:** Each tenant has custom backup/report times in their timezone
  - **Database-driven:** Load schedules from configuration table on startup
  - **Feature flags:** Conditionally enable jobs based on environment/config
  - **A/B testing:** Enable experimental scheduled features for a subset of users

- **Advantages over static `@Cron()`:**
  - Static decorators are compile-time (defined in code, always active, require redeployment to change)
  - Dynamic jobs are runtime (created from database data, can add/modify/delete instantly without code changes, support per-user/per-tenant customization)

</details>

<details>
<summary><strong>16. How do you delete a scheduled job?</strong></summary>

**Answer:**


**How to delete a scheduled job:**

- **Step-by-step process:**
  - **Inject `SchedulerRegistry`:**
    - Import from `@nestjs/schedule` and inject via constructor.
  - **Check if job exists:**
    - Use `schedulerRegistry.doesExist(type, name)` where type is `'cron' | 'interval' | 'timeout'`.
  - **For cron jobs:**
    - Retrieve with `getCronJob(name)`
    - Stop with `job.stop()`
    - Delete with `deleteCronJob(name)`
  - **For intervals:**
    - Delete with `deleteInterval(name)` (automatically clears interval)
  - **For timeouts:**
    - Delete with `deleteTimeout(name)` (automatically clears timeout)

- **Stopping vs Deleting:**
  - `stop()`: Pauses execution but keeps job in registry (can resume with `job.start()`)
  - `delete`: Removes from registry permanently (cannot resume, must recreate)

- **Best practices:**
  - Always check existence before delete (prevents errors)
  - Stop cron jobs before deleting (clean shutdown)
  - Update database if job metadata is stored (maintain consistency)
  - Log deletion for audit trail
  - Handle errors gracefully (job may not exist)

- **Use cases:**
  - User disables feature (delete scheduled reports)
  - Tenant cancellation (remove all tenant jobs)
  - Configuration change (delete old job, add new one with updated schedule)
  - Cleanup on shutdown (delete all jobs in `onModuleDestroy()`)
  - Job replacement (delete existing, create updated version)

- **Cleanup pattern:**
  - Check existence → Stop (cron only) → Delete from registry → Delete from database

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

**Key Takeaway:**

- **Delete a scheduled job using `SchedulerRegistry` with this workflow:**
  - **Check existence:** Use `schedulerRegistry.doesExist(type, name)` where type is `'cron' | 'interval' | 'timeout'` and name is the job identifier (prevents errors if job doesn't exist).
  - **For cron jobs:**
    - Retrieve with `const job = schedulerRegistry.getCronJob(name)`
    - Stop execution with `job.stop()` (prevents next scheduled run)
    - Delete from registry with `schedulerRegistry.deleteCronJob(name)` (removes permanently, frees memory)
  - **For intervals:** Delete directly with `schedulerRegistry.deleteInterval(name)` (automatically calls `clearInterval`, no stop needed)
  - **For timeouts:** Cancel with `schedulerRegistry.deleteTimeout(name)` (automatically calls `clearTimeout`, prevents execution if not yet fired)
  - **If job metadata is persisted to database:** Delete the database record to maintain consistency (`await scheduleRepo.delete({ jobName: name })`).

- **Stop vs Delete difference:**
  - `job.stop()`: Pauses execution but keeps job in registry (job still exists, can resume with `job.start()`, uses memory, useful for temporary pause)
  - `deleteCronJob()`: Removes from registry permanently (job no longer exists, cannot resume without recreating, frees memory, requires full recreation to reactivate)

- **Error handling:**
  - Wrap in try-catch (deletion may fail if job doesn't exist or already deleted)
  - Log warnings for missing jobs (not errors, may be expected)
  - Handle database errors separately (transaction may be needed for consistency)

- **Best practices:**
  - Always check existence before delete operations (use `doesExist()` to prevent throws)
  - Stop before delete for cron jobs (clean shutdown: `job.stop()` then `deleteCronJob()`)
  - Maintain consistency (if job in database, delete both registry and database in same operation)
  - Log deletions for audit trail (who deleted, when, why)
  - Cleanup on shutdown (implement `OnModuleDestroy` lifecycle hook, stop and delete all jobs for graceful shutdown)
  - Pattern-based cleanup (delete multiple jobs by pattern: all user jobs `cleanupByPattern('user-')`, all tenant jobs `cleanupByPattern('tenant-123')`)

- **Use cases:**
  - User disables feature (delete scheduled reports when user turns off notifications)
  - Tenant cancellation (remove all tenant jobs when subscription ends: backup, reports, cleanup)
  - Job replacement (delete old job, create new with updated schedule for configuration changes)
  - Temporary removal (use `stop()` for maintenance mode, `delete()` for permanent removal)
  - Bulk cleanup (delete all jobs matching pattern for mass operations)

- **Soft delete alternative:** Instead of deleting, mark as disabled in database and remove from registry (preserves history, can re-enable later: `await repo.update({ enabled: false })` then `deleteCronJob(name)`).

</details>

## Use Cases

<details>
<summary><strong>17. What are common use cases for scheduled jobs?</strong></summary>

**Answer:**


**Common use cases for scheduled jobs include:**

- **Data cleanup:** Delete expired records, archive old data, purge temporary files
- **Report generation:** Daily/weekly/monthly reports, analytics dashboards, business intelligence
- **Email notifications:** Reminders, digests, scheduled newsletters
- **Data synchronization:** Sync with external APIs, update cached data, replicate databases
- **Backup and maintenance:** Database backups, log rotation, system health checks
- **Billing and subscriptions:** Process monthly subscriptions, generate invoices, expire trials
- **Monitoring and alerts:** Check system health, monitor API endpoints, send alerts
- **Cache management:** Refresh stale cache, pre-warm cache, invalidate expired entries
- **Batch processing:** Process pending orders, update user statistics, aggregate metrics
- **Content management:** Publish scheduled posts, expire content, update search indexes

**Scheduling patterns:**
- **High frequency:** Every 30s–5min (health checks, queue monitoring)
- **Hourly:** Metrics aggregation, cache refresh
- **Daily:** Cleanup at 2 AM, reports at 9 AM
- **Weekly:** Sunday night maintenance, Monday morning summaries
- **Monthly:** Billing on 1st, reports on last day

**Best practices:**
- Use off-peak hours (2–4 AM) for heavy operations
- Stagger multiple jobs to avoid resource contention
- Make jobs idempotent (safe to run multiple times)
- Implement proper error handling and retry logic
- Log execution for monitoring and debugging

---

### **Common Use Cases Overview:**

```
SCHEDULED JOB USE CASES BY CATEGORY:

┌─────────────────────────────────────────────────────┐
│  1. DATA MANAGEMENT                                 │
│  ├─ Cleanup expired records (daily)                │
│  ├─ Archive old data (weekly)                      │
│  ├─ Purge temporary files (hourly)                 │
│  └─ Database maintenance (nightly)                 │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│  2. REPORTING & ANALYTICS                           │
│  ├─ Daily business reports (9 AM)                  │
│  ├─ Weekly summaries (Monday 8 AM)                 │
│  ├─ Monthly analytics (1st at midnight)            │
│  └─ Real-time metrics aggregation (every 5 min)    │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│  3. COMMUNICATION                                   │
│  ├─ Email digests (daily at user's preferred time) │
│  ├─ Reminder notifications (hourly check)          │
│  ├─ Newsletter dispatch (weekly)                   │
│  └─ Alert notifications (every minute)             │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│  4. SYNCHRONIZATION                                 │
│  ├─ External API sync (every 15 minutes)           │
│  ├─ Cache refresh (every 5 minutes)                │
│  ├─ Database replication (continuous)              │
│  └─ Search index update (hourly)                   │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│  5. MAINTENANCE & MONITORING                        │
│  ├─ Database backups (daily 2 AM)                  │
│  ├─ Health checks (every 30 seconds)               │
│  ├─ Log rotation (daily midnight)                  │
│  └─ System cleanup (weekly Sunday night)           │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│  6. BUSINESS OPERATIONS                             │
│  ├─ Process subscriptions (monthly 1st)            │
│  ├─ Generate invoices (monthly 1st)                │
│  ├─ Expire trials (daily check)                    │
│  └─ Update user tiers (daily)                      │
└─────────────────────────────────────────────────────┘

SCHEDULING FREQUENCY GUIDE:
┌─────────────────┬──────────────────────────────────┐
│ Frequency       │ Use Cases                        │
├─────────────────┼──────────────────────────────────┤
│ Every 30s-1min  │ Health checks, queue monitoring  │
│ Every 5-15min   │ API sync, metrics collection     │
│ Hourly          │ Cache refresh, log processing    │
│ Daily (2-4 AM)  │ Cleanup, backups, maintenance    │
│ Daily (9 AM)    │ Business reports, summaries      │
│ Weekly          │ Analytics, major cleanup         │
│ Monthly (1st)   │ Billing, subscriptions, invoices │
└─────────────────┴──────────────────────────────────┘
```

---

### **Method 1: Data Cleanup Jobs**

```typescript
// ========== DATA CLEANUP ==========

import { Injectable, Logger } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository, LessThan } from 'typeorm';

@Injectable()
export class DataCleanupService {
  private readonly logger = new Logger(DataCleanupService.name);

  constructor(
    @InjectRepository(User)
    private readonly userRepo: Repository<User>,
    @InjectRepository(Session)
    private readonly sessionRepo: Repository<Session>,
    @InjectRepository(TempFile)
    private readonly tempFileRepo: Repository<TempFile>,
  ) {}

  // Delete expired sessions daily at 2 AM
  @Cron(CronExpression.EVERY_DAY_AT_2AM)
  async cleanupExpiredSessions() {
    this.logger.log('Starting expired sessions cleanup');
    
    const thirtyDaysAgo = new Date();
    thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);

    try {
      const result = await this.sessionRepo.delete({
        lastActivity: LessThan(thirtyDaysAgo),
      });

      this.logger.log(`Deleted ${result.affected} expired sessions`);
    } catch (error) {
      this.logger.error('Failed to cleanup sessions', error.stack);
    }
  }

  // Delete temporary files hourly
  @Cron(CronExpression.EVERY_HOUR)
  async cleanupTempFiles() {
    this.logger.log('Starting temp files cleanup');
    
    const oneHourAgo = new Date();
    oneHourAgo.setHours(oneHourAgo.getHours() - 1);

    try {
      const files = await this.tempFileRepo.find({
        where: { createdAt: LessThan(oneHourAgo) },
      });

      for (const file of files) {
        // Delete physical file
        await fs.unlink(file.path);
        // Delete database record
        await this.tempFileRepo.remove(file);
      }

      this.logger.log(`Deleted ${files.length} temporary files`);
    } catch (error) {
      this.logger.error('Failed to cleanup temp files', error.stack);
    }
  }

  // Archive old records weekly
  @Cron('0 0 * * 0') // Sunday midnight
  async archiveOldRecords() {
    this.logger.log('Starting data archival');
    
    const sixMonthsAgo = new Date();
    sixMonthsAgo.setMonth(sixMonthsAgo.getMonth() - 6);

    try {
      // Move old data to archive table
      const oldRecords = await this.userRepo.find({
        where: { lastLogin: LessThan(sixMonthsAgo) },
      });

      // Archive logic here
      this.logger.log(`Archived ${oldRecords.length} old records`);
    } catch (error) {
      this.logger.error('Failed to archive data', error.stack);
    }
  }
}
```

---

### **Method 2: Report Generation**

```typescript
// ========== REPORT GENERATION ==========

@Injectable()
export class ReportGenerationService {
  private readonly logger = new Logger(ReportGenerationService.name);

  constructor(
    private readonly analyticsService: AnalyticsService,
    private readonly emailService: EmailService,
    private readonly storageService: StorageService,
  ) {}

  // Daily business report at 9 AM
  @Cron('0 9 * * 1-5') // Weekdays at 9 AM
  async generateDailyReport() {
    this.logger.log('Generating daily business report');

    try {
      const yesterday = new Date();
      yesterday.setDate(yesterday.getDate() - 1);

      // Gather metrics
      const metrics = await this.analyticsService.getDailyMetrics(yesterday);

      // Generate report
      const report = {
        date: yesterday.toISOString().split('T')[0],
        totalSales: metrics.totalSales,
        newUsers: metrics.newUsers,
        activeUsers: metrics.activeUsers,
        revenue: metrics.revenue,
      };

      // Save to storage
      await this.storageService.saveReport('daily', report);

      // Email to stakeholders
      await this.emailService.sendReport({
        to: 'management@company.com',
        subject: `Daily Report - ${report.date}`,
        report,
      });

      this.logger.log('Daily report generated successfully');
    } catch (error) {
      this.logger.error('Failed to generate daily report', error.stack);
    }
  }

  // Weekly summary on Monday morning
  @Cron('0 8 * * 1') // Monday 8 AM
  async generateWeeklyReport() {
    this.logger.log('Generating weekly summary');

    try {
      const lastWeek = await this.analyticsService.getWeeklyMetrics();

      const report = {
        week: lastWeek.weekNumber,
        summary: lastWeek.summary,
        trends: lastWeek.trends,
        topProducts: lastWeek.topProducts,
      };

      await this.emailService.sendReport({
        to: 'team@company.com',
        subject: `Weekly Summary - Week ${report.week}`,
        report,
      });

      this.logger.log('Weekly report sent successfully');
    } catch (error) {
      this.logger.error('Failed to generate weekly report', error.stack);
    }
  }

  // Monthly report on 1st at midnight
  @Cron('0 0 1 * *')
  async generateMonthlyReport() {
    this.logger.log('Generating monthly report');

    try {
      const lastMonth = new Date();
      lastMonth.setMonth(lastMonth.getMonth() - 1);

      const metrics = await this.analyticsService.getMonthlyMetrics(lastMonth);

      const report = {
        month: lastMonth.toLocaleString('default', { month: 'long' }),
        year: lastMonth.getFullYear(),
        revenue: metrics.totalRevenue,
        growth: metrics.growthPercentage,
        customers: metrics.totalCustomers,
      };

      await this.storageService.saveReport('monthly', report);
      await this.emailService.sendReport({
        to: 'executives@company.com',
        subject: `Monthly Report - ${report.month} ${report.year}`,
        report,
      });

      this.logger.log('Monthly report generated successfully');
    } catch (error) {
      this.logger.error('Failed to generate monthly report', error.stack);
    }
  }
}
```

---

### **Method 3: Email Notifications**

```typescript
// ========== EMAIL NOTIFICATIONS ==========

@Injectable()
export class EmailNotificationService {
  private readonly logger = new Logger(EmailNotificationService.name);

  constructor(
    private readonly userRepo: Repository<User>,
    private readonly emailService: EmailService,
    private readonly reminderService: ReminderService,
  ) {}

  // Send daily email digest at user's preferred time
  @Cron('0 9 * * *') // 9 AM
  async sendDailyDigest() {
    this.logger.log('Sending daily email digests');

    try {
      const users = await this.userRepo.find({
        where: { emailDigestEnabled: true },
      });

      for (const user of users) {
        const digest = await this.prepareDigest(user);
        
        await this.emailService.send({
          to: user.email,
          subject: 'Your Daily Digest',
          html: digest,
        });
      }

      this.logger.log(`Sent digest to ${users.length} users`);
    } catch (error) {
      this.logger.error('Failed to send daily digests', error.stack);
    }
  }

  // Check for reminders every hour
  @Cron(CronExpression.EVERY_HOUR)
  async processReminders() {
    this.logger.log('Processing reminders');

    try {
      const now = new Date();
      const oneHourLater = new Date(now.getTime() + 60 * 60 * 1000);

      const reminders = await this.reminderService.getDueReminders(
        now,
        oneHourLater,
      );

      for (const reminder of reminders) {
        await this.emailService.send({
          to: reminder.userEmail,
          subject: `Reminder: ${reminder.title}`,
          text: reminder.message,
        });

        await this.reminderService.markAsSent(reminder.id);
      }

      this.logger.log(`Processed ${reminders.length} reminders`);
    } catch (error) {
      this.logger.error('Failed to process reminders', error.stack);
    }
  }

  // Weekly newsletter on Friday
  @Cron('0 10 * * 5') // Friday 10 AM
  async sendWeeklyNewsletter() {
    this.logger.log('Sending weekly newsletter');

    try {
      const subscribers = await this.userRepo.find({
        where: { newsletterSubscribed: true },
      });

      const newsletter = await this.prepareNewsletter();

      // Send in batches to avoid overwhelming email service
      const batchSize = 100;
      for (let i = 0; i < subscribers.length; i += batchSize) {
        const batch = subscribers.slice(i, i + batchSize);
        
        await Promise.all(
          batch.map(user =>
            this.emailService.send({
              to: user.email,
              subject: 'Weekly Newsletter',
              html: newsletter,
            }),
          ),
        );

        // Wait 1 second between batches
        await new Promise(resolve => setTimeout(resolve, 1000));
      }

      this.logger.log(`Newsletter sent to ${subscribers.length} subscribers`);
    } catch (error) {
      this.logger.error('Failed to send newsletter', error.stack);
    }
  }

  private async prepareDigest(user: User): Promise<string> {
    // Prepare personalized digest
    return '<html>...</html>';
  }

  private async prepareNewsletter(): Promise<string> {
    // Prepare newsletter content
    return '<html>...</html>';
  }
}
```

---

### **Method 4: Data Synchronization**

```typescript
// ========== DATA SYNCHRONIZATION ==========

@Injectable()
export class DataSyncService {
  private readonly logger = new Logger(DataSyncService.name);

  constructor(
    private readonly httpService: HttpService,
    private readonly cacheService: CacheService,
    private readonly productRepo: Repository<Product>,
  ) {}

  // Sync with external API every 15 minutes
  @Cron('*/15 * * * *')
  async syncExternalData() {
    this.logger.log('Syncing data from external API');

    try {
      const response = await this.httpService.axiosRef.get(
        'https://api.external.com/data',
      );

      const externalData = response.data;

      // Update local database
      for (const item of externalData) {
        await this.productRepo.upsert(item, ['externalId']);
      }

      this.logger.log(`Synced ${externalData.length} items`);
    } catch (error) {
      this.logger.error('Failed to sync external data', error.stack);
    }
  }

  // Refresh cache every 5 minutes
  @Cron('*/5 * * * *')
  async refreshCache() {
    this.logger.log('Refreshing application cache');

    try {
      // Refresh product cache
      const products = await this.productRepo.find({ where: { active: true } });
      await this.cacheService.set('products:active', products, 600);

      // Refresh configuration cache
      const config = await this.getAppConfig();
      await this.cacheService.set('app:config', config, 300);

      this.logger.log('Cache refreshed successfully');
    } catch (error) {
      this.logger.error('Failed to refresh cache', error.stack);
    }
  }

  // Update search index hourly
  @Cron(CronExpression.EVERY_HOUR)
  async updateSearchIndex() {
    this.logger.log('Updating search index');

    try {
      const products = await this.productRepo.find();
      
      // Update Elasticsearch or similar
      // await this.searchService.bulkIndex(products);

      this.logger.log(`Indexed ${products.length} products`);
    } catch (error) {
      this.logger.error('Failed to update search index', error.stack);
    }
  }

  private async getAppConfig() {
    // Fetch application configuration
    return {};
  }
}
```

---

### **Method 5: Monitoring and Maintenance**

```typescript
// ========== MONITORING & MAINTENANCE ==========

@Injectable()
export class MonitoringService {
  private readonly logger = new Logger(MonitoringService.name);

  constructor(
    private readonly healthService: HealthService,
    private readonly alertService: AlertService,
    private readonly backupService: BackupService,
  ) {}

  // Health check every 30 seconds
  @Interval(30000)
  async performHealthCheck() {
    try {
      const health = await this.healthService.check();

      if (!health.healthy) {
        this.logger.error('Health check failed', health.details);
        
        await this.alertService.send({
          level: 'critical',
          message: 'Application health check failed',
          details: health.details,
        });
      }
    } catch (error) {
      this.logger.error('Health check error', error.stack);
    }
  }

  // Database backup daily at 2 AM
  @Cron(CronExpression.EVERY_DAY_AT_2AM)
  async performDatabaseBackup() {
    this.logger.log('Starting database backup');

    try {
      const backupPath = await this.backupService.backupDatabase();
      
      this.logger.log(`Database backed up to ${backupPath}`);
      
      await this.alertService.send({
        level: 'info',
        message: 'Database backup completed successfully',
      });
    } catch (error) {
      this.logger.error('Database backup failed', error.stack);
      
      await this.alertService.send({
        level: 'critical',
        message: 'Database backup failed',
        error: error.message,
      });
    }
  }

  // Log rotation daily at midnight
  @Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT)
  async rotateLogs() {
    this.logger.log('Rotating application logs');

    try {
      // Implement log rotation logic
      this.logger.log('Logs rotated successfully');
    } catch (error) {
      this.logger.error('Log rotation failed', error.stack);
    }
  }

  // System cleanup weekly
  @Cron('0 3 * * 0') // Sunday 3 AM
  async performSystemCleanup() {
    this.logger.log('Performing system cleanup');

    try {
      // Clear old logs
      // Vacuum database
      // Clean temp directories
      
      this.logger.log('System cleanup completed');
    } catch (error) {
      this.logger.error('System cleanup failed', error.stack);
    }
  }
}
```


**Key Takeaway:**

- **Common use cases for scheduled jobs span multiple categories:**
  - **Data management:**
    - Cleanup expired sessions daily at 2 AM with `LessThan(thirtyDaysAgo)`
    - Delete temporary files hourly
    - Archive old records weekly (Sunday midnight)
    - Purge soft-deleted items
  - **Report generation:**
    - Daily business reports weekdays at 9 AM (yesterday's metrics)
    - Weekly summaries Monday 8 AM
    - Monthly reports on 1st at midnight (billing/analytics)
    - Real-time metrics aggregation every 5 minutes
  - **Email communications:**
    - Daily email digests at user's preferred time
    - Reminder notifications checked hourly with time window
    - Weekly newsletters Friday 10 AM (in batches of 100 to avoid overwhelming email service)
    - Scheduled alerts
  - **Data synchronization:**
    - External API sync every 15 minutes (with upsert operations)
    - Cache refresh every 5 minutes (for active products and config)
    - Search index update hourly (with bulk indexing)
    - Database replication
  - **Backup and maintenance:**
    - Database backups daily at 2 AM (off-peak, with success alerts)
    - Log rotation daily at midnight
    - System cleanup weekly (Sunday 3 AM, vacuum and temp cleanup)
  - **Business operations:**
    - Process monthly subscriptions on 1st
    - Generate invoices
    - Expire trials (daily check)
    - Update user tiers based on usage
  - **Monitoring:**
    - Health checks every 30 seconds (with alerting on failures)
    - API endpoint monitoring
    - Queue depth checks
    - Resource usage tracking

- **Scheduling patterns by frequency:**
  - **High-frequency:** 30s–1min (health checks using `@Interval(30000)`, queue monitoring)
  - **Medium-frequency:** 5–15min (API sync `@Cron('*/15 * * * *')`, metrics collection, cache refresh)
  - **Hourly:** Log processing, search index updates, batch operations
  - **Daily off-peak:** 2–4 AM (cleanup, backups, maintenance using `EVERY_DAY_AT_2AM`)
  - **Daily business hours:** 9 AM (reports, summaries)
  - **Weekly:** Sunday/Monday (analytics, major cleanup, summaries)
  - **Monthly:** 1st (billing `@Cron('0 0 1 * *')`, subscriptions, invoices)

- **Best practices:**
  - **Timing:** Use off-peak hours (2–4 AM) for resource-intensive operations
  - **Stagger jobs:** Stagger multiple jobs by 10–30 minutes to prevent contention
  - **Idempotency:** Make jobs safe to run multiple times, check before processing to avoid duplicates
  - **Error handling:** Wrap in try-catch, log failures, send alerts for critical jobs, don't throw errors that stop scheduling
  - **Performance:** Batch operations in chunks of 100–1000 items, use pagination for large datasets, add delays between batches to prevent overload
  - **Monitoring:** Log start/completion/duration, track success/failure rates, alert on anomalies
  - **Database operations:** Use efficient queries with indexes, batch updates, avoid N+1 queries, use transactions for consistency

</details>

<details>
<summary><strong>18. How do you implement data cleanup jobs?</strong></summary>

**Answer:**


**Implement data cleanup jobs:**

- **Identify cleanup targets:**
  - Expired sessions
  - Old logs
  - Temporary files
  - Soft-deleted records
  - Archived data
- **Choose appropriate schedule:**
  - Daily (2–4 AM) for off-peak processing
  - Hourly for temp files
  - Weekly for major cleanup
- **Use `@Cron()` decorator with specific timing:**
  - Example: `@Cron(CronExpression.EVERY_DAY_AT_2AM)`
- **Query data to clean using TypeORM operators:**
  - `LessThan()`, `IsNull()`, date comparisons
- **Delete/archive in batches to avoid locking:**
  - Process 1000 records at a time
  - Commit per batch
- **Log metrics:**
  - Records deleted
  - Duration
  - Errors
- **Handle errors gracefully:**
  - Continue on failure
  - Retry failed batches
  - Alert on critical issues

**Cleanup types:**
- **Hard delete:** Permanently remove (`repo.delete()`, cascade deletes)
- **Soft delete:** Mark as deleted (`deletedAt` timestamp, filter in queries)
- **Archive:** Move to separate table/storage for historical data, keep for compliance

**Best practices:**
- Make idempotent (check before delete, use WHERE conditions)
- Run during off-peak hours (2–4 AM minimal traffic)
- Use database transactions (rollback on error)
- Add safety limits (max records per run to prevent accidents)
- Dry-run mode (log without deleting for testing)
- Maintain audit trail (log what was deleted when)

---

### **Data Cleanup Architecture:**

```
CLEANUP JOB WORKFLOW:
┌─────────────────────────────────────────────────────┐
│  1. IDENTIFY DATA TO CLEAN                          │
│     ↓                                               │
│     - Expired sessions (> 30 days inactive)        │
│     - Temporary files (> 1 hour old)               │
│     - Soft-deleted records (> 90 days)             │
│     - Old logs (> 1 year)                          │
│     - Archived orders (> 7 years for compliance)   │
└──────────────────┬──────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────┐
│  2. QUERY DATA IN BATCHES                           │
│     ↓                                               │
│     SELECT * FROM sessions                          │
│     WHERE last_activity < NOW() - INTERVAL 30 DAY   │
│     LIMIT 1000  -- Process in batches              │
└──────────────────┬──────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────┐
│  3. DELETE/ARCHIVE BATCH                            │
│     ↓                                               │
│     BEGIN TRANSACTION                               │
│       DELETE FROM sessions WHERE id IN (...)        │
│       Log to audit_log                             │
│     COMMIT                                          │
└──────────────────┬──────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────┐
│  4. REPEAT UNTIL ALL PROCESSED                      │
│     ↓                                               │
│     Process next batch                              │
│     Sleep between batches (avoid load)             │
└──────────────────┬──────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────┐
│  5. LOG COMPLETION METRICS                          │
│     ↓                                               │
│     Total deleted: 15,420 records                   │
│     Duration: 45 seconds                            │
│     Batches processed: 16                           │
└─────────────────────────────────────────────────────┘

CLEANUP STRATEGIES:
┌──────────────────┬────────────────────────────────────┐
│ Strategy         │ When to Use                        │
├──────────────────┼────────────────────────────────────┤
│ Hard Delete      │ Truly disposable data (temp files) │
│ Soft Delete      │ Need recovery option (user data)   │
│ Archive          │ Compliance/historical (orders)     │
│ Truncate         │ Complete table cleanup (logs)      │
└──────────────────┴────────────────────────────────────┘
```

---

### **Method 1: Basic Cleanup with Batch Processing**

```typescript
// ========== BASIC CLEANUP ==========

import { Injectable, Logger } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository, LessThan } from 'typeorm';

@Injectable()
export class BasicCleanupService {
  private readonly logger = new Logger(BasicCleanupService.name);
  private readonly BATCH_SIZE = 1000;

  constructor(
    @InjectRepository(Session)
    private readonly sessionRepo: Repository<Session>,
  ) {}

  // Clean expired sessions daily at 2 AM
  @Cron(CronExpression.EVERY_DAY_AT_2AM)
  async cleanupExpiredSessions() {
    this.logger.log('Starting expired sessions cleanup');
    const startTime = Date.now();

    try {
      const expiryDate = new Date();
      expiryDate.setDate(expiryDate.getDate() - 30); // 30 days ago

      let totalDeleted = 0;
      let hasMore = true;

      // Process in batches
      while (hasMore) {
        // Find batch to delete
        const expiredSessions = await this.sessionRepo.find({
          where: { lastActivity: LessThan(expiryDate) },
          take: this.BATCH_SIZE,
        });

        if (expiredSessions.length === 0) {
          hasMore = false;
          break;
        }

        // Delete batch
        await this.sessionRepo.remove(expiredSessions);
        totalDeleted += expiredSessions.length;

        this.logger.log(`Deleted batch of ${expiredSessions.length} sessions`);

        // Small delay to avoid overwhelming database
        await new Promise(resolve => setTimeout(resolve, 100));
      }

      const duration = Date.now() - startTime;
      this.logger.log(
        `Cleanup complete: ${totalDeleted} sessions deleted in ${duration}ms`,
      );
    } catch (error) {
      this.logger.error('Session cleanup failed', error.stack);
    }
  }
}
```

---

### **Method 2: Advanced Cleanup with Multiple Targets**

```typescript
// ========== MULTI-TARGET CLEANUP ==========

@Injectable()
export class AdvancedCleanupService {
  private readonly logger = new Logger(AdvancedCleanupService.name);

  constructor(
    @InjectRepository(Session)
    private readonly sessionRepo: Repository<Session>,
    @InjectRepository(TempFile)
    private readonly tempFileRepo: Repository<TempFile>,
    @InjectRepository(Notification)
    private readonly notificationRepo: Repository<Notification>,
    @InjectRepository(AuditLog)
    private readonly auditRepo: Repository<AuditLog>,
  ) {}

  // Comprehensive cleanup daily at 3 AM
  @Cron('0 3 * * *')
  async performDailyCleanup() {
    this.logger.log('========== STARTING DAILY CLEANUP ==========');
    const metrics = {
      sessions: 0,
      tempFiles: 0,
      notifications: 0,
      auditLogs: 0,
      duration: 0,
    };

    const startTime = Date.now();

    try {
      // 1. Clean expired sessions (> 30 days)
      metrics.sessions = await this.cleanExpiredSessions(30);

      // 2. Clean temporary files (> 24 hours)
      metrics.tempFiles = await this.cleanTempFiles(24);

      // 3. Clean read notifications (> 90 days)
      metrics.notifications = await this.cleanOldNotifications(90);

      // 4. Archive old audit logs (> 1 year)
      metrics.auditLogs = await this.archiveAuditLogs(365);

      metrics.duration = Date.now() - startTime;

      this.logger.log('========== CLEANUP COMPLETE ==========');
      this.logger.log(`Sessions cleaned: ${metrics.sessions}`);
      this.logger.log(`Temp files deleted: ${metrics.tempFiles}`);
      this.logger.log(`Notifications cleaned: ${metrics.notifications}`);
      this.logger.log(`Audit logs archived: ${metrics.auditLogs}`);
      this.logger.log(`Total duration: ${metrics.duration}ms`);
    } catch (error) {
      this.logger.error('Daily cleanup failed', error.stack);
    }
  }

  private async cleanExpiredSessions(daysOld: number): Promise<number> {
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - daysOld);

    const result = await this.sessionRepo.delete({
      lastActivity: LessThan(cutoffDate),
    });

    return result.affected || 0;
  }

  private async cleanTempFiles(hoursOld: number): Promise<number> {
    const cutoffDate = new Date();
    cutoffDate.setHours(cutoffDate.getHours() - hoursOld);

    const files = await this.tempFileRepo.find({
      where: { createdAt: LessThan(cutoffDate) },
    });

    for (const file of files) {
      try {
        // Delete physical file
        await fs.unlink(file.path);
      } catch (error) {
        this.logger.warn(`Failed to delete file: ${file.path}`);
      }
    }

    // Delete database records
    await this.tempFileRepo.remove(files);

    return files.length;
  }

  private async cleanOldNotifications(daysOld: number): Promise<number> {
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - daysOld);

    const result = await this.notificationRepo.delete({
      read: true,
      readAt: LessThan(cutoffDate),
    });

    return result.affected || 0;
  }

  private async archiveAuditLogs(daysOld: number): Promise<number> {
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - daysOld);

    // In production, would move to archive storage (S3, etc.)
    const oldLogs = await this.auditRepo.find({
      where: { createdAt: LessThan(cutoffDate) },
      take: 10000,
    });

    // Archive to external storage
    // await this.storageService.archiveLogs(oldLogs);

    // Then delete from main database
    await this.auditRepo.remove(oldLogs);

    return oldLogs.length;
  }
}
```

---

### **Method 3: Soft Delete Cleanup**

```typescript
// ========== SOFT DELETE CLEANUP ==========

@Entity()
class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  @Column({ nullable: true })
  @DeleteDateColumn()  // TypeORM soft delete
  deletedAt: Date;
}

@Injectable()
export class SoftDeleteCleanupService {
  private readonly logger = new Logger(SoftDeleteCleanupService.name);

  constructor(
    @InjectRepository(User)
    private readonly userRepo: Repository<User>,
  ) {}

  // Permanently delete soft-deleted users after 90 days
  @Cron('0 4 * * *') // Daily at 4 AM
  async cleanupSoftDeletedUsers() {
    this.logger.log('Cleaning up soft-deleted users');

    try {
      const ninetyDaysAgo = new Date();
      ninetyDaysAgo.setDate(ninetyDaysAgo.getDate() - 90);

      // Find soft-deleted users older than 90 days
      const deletedUsers = await this.userRepo
        .createQueryBuilder('user')
        .withDeleted()  // Include soft-deleted records
        .where('user.deletedAt IS NOT NULL')
        .andWhere('user.deletedAt < :cutoffDate', {
          cutoffDate: ninetyDaysAgo,
        })
        .getMany();

      if (deletedUsers.length === 0) {
        this.logger.log('No soft-deleted users to clean');
        return;
      }

      // Permanently delete (hard delete)
      for (const user of deletedUsers) {
        // Clean up related data first
        // await this.cleanupUserData(user.id);

        // Hard delete
        await this.userRepo
          .createQueryBuilder()
          .delete()
          .from(User)
          .where('id = :id', { id: user.id })
          .execute();
      }

      this.logger.log(`Permanently deleted ${deletedUsers.length} users`);
    } catch (error) {
      this.logger.error('Soft delete cleanup failed', error.stack);
    }
  }
}
```

---

### **Method 4: Cleanup with Transaction and Safety Limits**

```typescript
// ========== SAFE CLEANUP WITH LIMITS ==========

@Injectable()
export class SafeCleanupService {
  private readonly logger = new Logger(SafeCleanupService.name);
  private readonly MAX_DELETES_PER_RUN = 50000;  // Safety limit
  private readonly BATCH_SIZE = 1000;

  constructor(
    @InjectRepository(Order)
    private readonly orderRepo: Repository<Order>,
    private readonly dataSource: DataSource,
  ) {}

  @Cron('0 2 * * *')
  async cleanupOldOrders() {
    this.logger.log('Starting order cleanup with safety limits');

    const queryRunner = this.dataSource.createQueryRunner();
    await queryRunner.connect();

    try {
      const twoYearsAgo = new Date();
      twoYearsAgo.setFullYear(twoYearsAgo.getFullYear() - 2);

      let totalDeleted = 0;

      while (totalDeleted < this.MAX_DELETES_PER_RUN) {
        // Start transaction for batch
        await queryRunner.startTransaction();

        try {
          // Find batch to delete
          const oldOrders = await queryRunner.manager.find(Order, {
            where: {
              status: 'completed',
              completedAt: LessThan(twoYearsAgo),
            },
            take: this.BATCH_SIZE,
          });

          if (oldOrders.length === 0) {
            await queryRunner.commitTransaction();
            break;
          }

          // Archive to external storage before deleting
          // await this.archiveOrders(oldOrders);

          // Delete batch
          await queryRunner.manager.remove(oldOrders);

          // Commit transaction
          await queryRunner.commitTransaction();

          totalDeleted += oldOrders.length;
          this.logger.log(
            `Deleted batch: ${oldOrders.length} orders (total: ${totalDeleted})`,
          );

          // Safety check
          if (totalDeleted >= this.MAX_DELETES_PER_RUN) {
            this.logger.warn(
              `Reached safety limit of ${this.MAX_DELETES_PER_RUN} deletes`,
            );
            break;
          }

          // Delay between batches
          await new Promise(resolve => setTimeout(resolve, 500));
        } catch (error) {
          // Rollback transaction on error
          await queryRunner.rollbackTransaction();
          this.logger.error('Batch deletion failed, rolled back', error.stack);
          throw error;
        }
      }

      this.logger.log(`Cleanup complete: ${totalDeleted} orders deleted`);
    } catch (error) {
      this.logger.error('Order cleanup failed', error.stack);
    } finally {
      await queryRunner.release();
    }
  }
}
```

---

### **Method 5: Dry-Run Mode for Testing**

```typescript
// ========== DRY-RUN CLEANUP ==========

@Injectable()
export class DryRunCleanupService {
  private readonly logger = new Logger(DryRunCleanupService.name);
  private readonly DRY_RUN = process.env.CLEANUP_DRY_RUN === 'true';

  constructor(
    @InjectRepository(Product)
    private readonly productRepo: Repository<Product>,
  ) {}

  @Cron('0 5 * * *')
  async cleanupInactiveProducts() {
    const mode = this.DRY_RUN ? '[DRY-RUN]' : '[LIVE]';
    this.logger.log(`${mode} Starting inactive products cleanup`);

    try {
      const sixMonthsAgo = new Date();
      sixMonthsAgo.setMonth(sixMonthsAgo.getMonth() - 6);

      const inactiveProducts = await this.productRepo.find({
        where: {
          active: false,
          deactivatedAt: LessThan(sixMonthsAgo),
        },
      });

      this.logger.log(
        `${mode} Found ${inactiveProducts.length} inactive products`,
      );

      if (this.DRY_RUN) {
        // In dry-run mode, just log what would be deleted
        for (const product of inactiveProducts) {
          this.logger.log(
            `${mode} Would delete: ${product.id} - ${product.name}`,
          );
        }
        this.logger.log(`${mode} Dry-run complete, no data deleted`);
      } else {
        // Actually delete in live mode
        await this.productRepo.remove(inactiveProducts);
        this.logger.log(
          `${mode} Deleted ${inactiveProducts.length} inactive products`,
        );
      }
    } catch (error) {
      this.logger.error(`${mode} Cleanup failed`, error.stack);
    }
  }
}

// USAGE:
// Set environment variable:
// CLEANUP_DRY_RUN=true  # Test mode, no deletion
// CLEANUP_DRY_RUN=false # Live mode, actually delete
```


**Key Takeaway:**

- **Implement data cleanup jobs by:**
  - Identifying cleanup targets (expired sessions, temporary files, soft-deleted records, old logs)
  - Using `@Cron()` decorator with off-peak scheduling (`@Cron(CronExpression.EVERY_DAY_AT_2AM)` or `@Cron('0 3 * * *')`)

- **Query data efficiently:**
  - Use TypeORM operators: `LessThan(cutoffDate)` for date comparisons, `IsNull()` for null checks
  - Create indexes on cleanup criteria columns (lastActivity, createdAt, deletedAt)
  - Use `take: BATCH_SIZE` to limit query results

- **Process in batches (1000 records per batch recommended):**
  - Prevents database locking
  - Avoids memory issues
  - Allows progress tracking
  - Enables graceful cancellation (iterate with `while(hasMore)` loop, check `results.length === 0` to stop, add 100–500ms delay between batches with `setTimeout(resolve, 100)`)

- **Deletion strategies:**
  - **Hard delete:** `repo.delete()` or `repo.remove()` for truly disposable data like temporary files
  - **Soft delete:** Set `deletedAt` timestamp, use TypeORM `@DeleteDateColumn()`, filter with `withDeleted()` in queries, permanently delete after grace period (e.g., 90 days)
  - **Archive:** Move to separate table or external storage (S3/archive database) before deleting, maintain for compliance/historical analysis

- **Use transactions for data consistency:**
  - Create `queryRunner`, wrap batch in `startTransaction()`/`commitTransaction()`, `rollbackTransaction()` on error to prevent partial deletes

- **Safety practices:**
  - **Limits:** Set `MAX_DELETES_PER_RUN` (e.g., 50,000) to prevent accidental mass deletion, break loop when limit reached
  - **Dry-run mode:** Set environment variable `CLEANUP_DRY_RUN=true` to log what would be deleted without actual deletion, test cleanup logic safely in production
  - **Audit trail:** Log deleted record IDs/count to audit table, track who initiated cleanup, enable recovery if needed
  - **Validation:** Verify date ranges are reasonable, confirm target table before deleting, add WHERE conditions to prevent full table delete

- **Error handling:**
  - Wrap in try-catch, log detailed errors with context
  - Continue processing other batches on failure
  - Send alerts for critical cleanup failures
  - Implement retry logic for transient failures

- **Monitoring:**
  - Log start/completion time and duration
  - Track records processed per batch and total
  - Calculate cleanup rate (records/second)
  - Alert if cleanup takes too long or deletes unexpectedly high/low counts
  - Expose metrics endpoint for monitoring dashboards

</details>


<details>
<summary><strong>19. How do you send scheduled email notifications?</strong></summary>

**Pattern:** Scheduled email notifications (digests, reminders, newsletters)

**Architecture (production-grade):**
- Cron scheduler → enqueues email jobs → Bull queue → separate processor workers send emails
- Provides resilience, rate limiting, retries, and monitoring

**Implementation:**

```typescript
// Scheduler: Enqueue jobs (lightweight)
@Injectable()
export class NotificationsScheduler {
  constructor(
    @InjectQueue('email') private emailQueue: Queue,
    private usersRepo: UsersRepository,
  ) {}

  @Cron('0 9 * * *', { name: 'daily-digest' })
  async scheduleDailyDigests() {
    // Query active users with digest enabled
    const users = await this.usersRepo.find({
      where: { emailPreferences: { dailyDigest: true }, isActive: true },
    });

    // Batch enqueue (100 at a time)
    for (let i = 0; i < users.length; i += 100) {
      const batch = users.slice(i, i + 100).map(user => ({
        name: 'daily-digest',
        data: { userId: user.id, email: user.email },
        opts: {
          attempts: 3,
          backoff: { type: 'exponential', delay: 2000 },
        },
      }));
      await this.emailQueue.addBulk(batch);
    }
  }
}

// Processor: Send individual emails
@Processor('email')
export class EmailProcessor {
  constructor(private emailService: EmailService) {}

  @Process({ name: 'daily-digest', concurrency: 5 })
  async processDailyDigest(job: Job) {
    const { email, userId } = job.data;
    await this.emailService.sendEmail({
      to: email,
      subject: 'Daily Digest',
      html: await this.generateDigestHTML(userId),
    });
    return { success: true };
  }
}
```

**Best practices:**
- **Rate limiting:** Control concurrency (5-10), respect provider limits
- **Idempotency:** Mark as queued before sending, use dedup keys
- **Personalization:** User timezone, language, unsubscribe preferences
- **Monitoring:** Track sent/failed counts, delivery rates
- **Testing:** Dry-run mode, test recipients list

</details>

<details>
<summary><strong>20. How do you implement report generation?</strong></summary>

**Pattern:** Scheduled report generation (daily/weekly/monthly analytics)

**Architecture:**
- Cron triggers generation → process aggregations → store in S3 → deliver via email/API
- Run during off-peak hours (2-5 AM)

**Implementation:**

```typescript
// Scheduler
@Injectable()
export class ReportsScheduler {
  constructor(@InjectQueue('reports') private queue: Queue) {}

  // Daily sales report - weekdays 2 AM
  @Cron('0 2 * * 1-5', { name: 'daily-sales', timeZone: 'America/New_York' })
  async scheduleDailySalesReport() {
    const yesterday = this.getYesterday();
    await this.queue.add('generate-report', {
      type: 'daily-sales',
      startDate: yesterday.start,
      endDate: yesterday.end,
      recipients: ['sales@company.com'],
    }, { timeout: 600000 }); // 10 min timeout
  }

  // Monthly report - 1st of month at 3 AM
  @Cron('0 3 1 * *', { name: 'monthly-report' })
  async scheduleMonthlyReport() {
    const lastMonth = this.getLastMonth();
    await this.queue.add('generate-report', {
      type: 'monthly-analytics',
      startDate: lastMonth.start,
      endDate: lastMonth.end,
      recipients: ['board@company.com'],
      format: 'pdf',
    });
  }
}

// Processor
@Processor('reports')
export class ReportsProcessor {
  constructor(
    private ordersRepo: OrdersRepository,
    private s3Service: S3Service,
    private emailService: EmailService,
  ) {}

  @Process({ name: 'generate-report', concurrency: 2 })
  async generateReport(job: Job) {
    const { type, startDate, endDate, recipients } = job.data;

    // 1. Aggregate metrics
    await job.progress(20);
    const metrics = await this.aggregateMetrics(startDate, endDate);

    // 2. Calculate comparisons
    await job.progress(40);
    const previousPeriod = this.getPreviousPeriod(startDate, endDate);
    const comparison = await this.calculateComparisons(metrics, previousPeriod);

    // 3. Generate CSV
    await job.progress(60);
    const csvContent = this.generateCSV(metrics, comparison);

    // 4. Upload to S3
    await job.progress(80);
    const key = `reports/${type}-${startDate}.csv`;
    await this.s3Service.upload(key, csvContent);
    const downloadUrl = await this.s3Service.getPresignedUrl(key, 604800); // 7 days

    // 5. Email with link
    await this.emailService.send({
      to: recipients,
      subject: `${type} Report`,
      html: this.generateEmailHTML(metrics, downloadUrl),
    });

    return { success: true, s3Key: key };
  }

  private async aggregateMetrics(start, end) {
    // Use materialized views for performance
    return {
      totalOrders: await this.ordersRepo.count({ where: { createdAt: Between(start, end) } }),
      totalRevenue: await this.ordersRepo.sum('total', { createdAt: Between(start, end) }),
      avgOrderValue: /* calculate */,
    };
  }
}
```

**Best practices:**
- **Performance:** Use materialized views, pre-aggregate data, stream large reports
- **Storage:** S3 with lifecycle policies (auto-delete after 90 days), presigned URLs
- **Scheduling:** Off-peak hours, stagger multiple reports, timezone-aware
- **Error handling:** Retry transient failures, alert on persistent issues, DLQ
- **Monitoring:** Track duration, row counts, alert on anomalies

**Report types:**
- Daily operational: yesterday's sales/activity
- Weekly summaries: 7-day trends
- Monthly analytics: full month metrics with YoY comparison

</details>

<details>
<summary><strong>21. How do you sync data with external APIs periodically?</strong></summary>

**Pattern:** Periodic data synchronization from external sources

**Architecture:**
- Cron scheduler triggers sync → fetch pages from API → upsert to database
- Store `lastSyncedAt` timestamp for incremental syncs
- Use queues for heavy processing per record

**Implementation:**

```typescript
@Injectable()
export class ExternalSyncService {
  constructor(
    private http: HttpService,
    private productsRepo: ProductsRepository,
    private syncStateRepo: SyncStateRepository,
  ) {}

  // Sync product prices every 15 minutes
  @Cron('*/15 * * * *', { name: 'price-sync' })
  async syncProductPrices() {
    const startTime = Date.now();
    let synced = 0;

    try {
      // Get last sync timestamp
      const lastSync = await this.syncStateRepo.findOne({
        where: { resource: 'products' },
      });

      // Fetch changes since last sync (incremental)
      const response = await this.http.axiosRef.get(
        'https://api.supplier.com/products',
        {
          params: { updated_since: lastSync?.timestamp },
          headers: { 'API-Key': process.env.SUPPLIER_API_KEY },
          timeout: 10000,
        },
      );

      // Upsert products (batch of 100)
      for (let i = 0; i < response.data.products.length; i += 100) {
        const batch = response.data.products.slice(i, i + 100);
        await this.productsRepo.upsert(
          batch.map(p => ({
            externalId: p.id,
            name: p.name,
            price: p.price,
            stock: p.stock,
            lastSyncedAt: new Date(),
          })),
          ['externalId'], // Conflict target
        );
        synced += batch.length;
      }

      // Update sync state
      await this.syncStateRepo.upsert(
        { resource: 'products', timestamp: new Date(), recordsProcessed: synced },
        ['resource'],
      );

      this.logger.log(
        `Synced ${synced} products in ${Date.now() - startTime}ms`,
      );
    } catch (error) {
      this.logger.error(`Sync failed: ${error.message}`);
      // Alert ops team for persistent failures
      throw error;
    }
  }
}
```

**Best practices:**
- **Incremental sync:** Use `since` timestamps or cursors, avoid full syncs
- **Pagination:** Process large datasets in chunks (100-500 records)
- **Rate limiting:** Add delays between requests, track quota usage
- **Idempotency:** Use `upsert` with external IDs as conflict keys
- **Retries:** Exponential backoff for transient failures, circuit breaker for persistent issues
- **Caching:** Store raw responses temporarily for debugging
- **Monitoring:** Track sync duration, records processed, API response times

**Sync patterns:**
- **Full sync:** Fetch all data periodically (simple but expensive)
- **Incremental sync:** Fetch only changes since last sync (efficient)
- **Webhook + polling hybrid:** Webhooks for real-time + polling as fallback

</details>

<details>
<summary><strong>22. What is a job queue and when should you use it?</strong></summary>

**Definition:**
- Asynchronous task processing system that decouples job creation from execution
- Components: Queue (Redis storage), Producer (adds jobs), Consumer/Worker (processes jobs)

**Architecture:**
```
API Request → Producer (enqueue job) → Queue (Redis) → Worker (process)
     ↓                                                        ↓
Return immediately                                     Execute async
```

**When to use:**
- **User-triggered async operations:** Send email after signup, process uploaded file, generate PDF
- **High-volume batch processing:** Process thousands of orders, bulk imports
- **Tasks requiring retries:** External API calls, payment processing, webhook deliveries
- **Resource-intensive operations:** Video transcoding, image processing, ML inference
- **Rate-limited operations:** Control throughput to external services

**Benefits over cron:**
- **Event-driven:** Triggered by actions, not time
- **Priority support:** Urgent jobs processed first
- **Retry mechanisms:** Automatic exponential backoff
- **Rate limiting:** Control concurrent execution
- **Progress tracking:** Monitor job status (pending/active/completed/failed)
- **Horizontal scaling:** Add more workers dynamically

**When NOT to use:**
- **Simple time-based tasks:** Use `@Cron()` instead
- **Synchronous operations:** User waiting for immediate result
- **Very high throughput real-time:** Queue overhead may add latency

**Common implementation (Bull + Redis):**
```typescript
// Producer
await emailQueue.add('welcome-email', { userId: user.id });

// Worker
@Processor('email')
class EmailProcessor {
  @Process('welcome-email')
  async sendWelcome(job: Job) {
    await this.emailService.send(job.data.userId);
  }
}
```

</details>

## Queue-Based Jobs

<details>
<summary><strong>23. What is Bull queue library?</strong></summary>

**Overview:**
- Production-ready Node.js job queue library built on Redis
- Provides robust, distributed job processing with enterprise features

**Core features:**
- **Persistence:** Jobs survive app restarts (stored in Redis)
- **Automatic retries:** Exponential backoff for failed jobs
- **Priority queues:** Process urgent jobs first (1=highest)
- **Delayed jobs:** Execute after specified delay (`delay: 5000ms`)
- **Rate limiting:** Control throughput (`limiter: { max: 100, duration: 1000 }`)
- **Job events:** Lifecycle hooks (completed, failed, progress, active)
- **Concurrency control:** Multiple workers process jobs simultaneously
- **Progress tracking:** Monitor job status in real-time

**Architecture:**
```
Queue (Bull) → manages jobs
Job → unit of work with data payload
Processor → function that executes job logic
Events → lifecycle hooks for monitoring
Redis → persistent storage for job data and state
```

**Installation:**
```bash
npm install @nestjs/bull bull
npm install -D @types/bull
```

**Common use cases:**
- Send emails after user signup
- Process file uploads (images, videos, documents)
- Generate reports on demand
- Webhook delivery with retries
- Image/video transcoding
- Batch data imports and exports

**Advantages:**
- Battle-tested in production (used by major companies)
- Built-in retry with exponential backoff
- Horizontal scaling (add workers dynamically)
- Monitoring UI available (Bull Board, Bull MQ Board)
- TypeScript support with type-safe job data
- Active community and maintenance

**Simple example:**
```typescript
// Add job
await queue.add('process-image', { imageId: 123 }, {
  attempts: 3,
  backoff: { type: 'exponential', delay: 2000 },
});

// Process job
@Processor('image')
class ImageProcessor {
  @Process('process-image')
  async process(job: Job) {
    await this.imageService.resize(job.data.imageId);
  }
}
```

</details>

<details>
<summary><strong>24. How do you integrate Bull with NestJS using `@nestjs/bull`?</strong></summary>

**Integration steps:**

**1. Install dependencies:**
```bash
npm install @nestjs/bull bull
npm install -D @types/bull
```

**2. Configure Redis connection:**
```typescript
// app.module.ts
@Module({
  imports: [
    // Global Bull configuration
    BullModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: (config: ConfigService) => ({
        redis: {
          host: config.get('REDIS_HOST'),
          port: config.get('REDIS_PORT'),
          password: config.get('REDIS_PASSWORD'),
        },
        defaultJobOptions: {
          attempts: 3,
          backoff: { type: 'exponential', delay: 1000 },
          removeOnComplete: true,
        },
      }),
      inject: [ConfigService],
    }),
    EmailModule,
  ],
})
export class AppModule {}
```

**3. Register queues in feature modules:**
```typescript
// email.module.ts
@Module({
  imports: [
    BullModule.registerQueue({
      name: 'email', // Queue name
    }),
  ],
  providers: [EmailService, EmailProcessor],
  exports: [BullModule],
})
export class EmailModule {}
```

**4. Create processor (consumer/worker):**
```typescript
// email.processor.ts
import { Processor, Process } from '@nestjs/bull';
import { Job } from 'bull';

@Processor('email') // Match queue name
export class EmailProcessor {
  constructor(private emailService: EmailService) {}

  @Process({ name: 'welcome-email', concurrency: 5 })
  async sendWelcomeEmail(job: Job) {
    const { userId, email } = job.data;
    await this.emailService.sendWelcome(email);
    return { success: true };
  }

  @Process('password-reset')
  async sendPasswordReset(job: Job) {
    await this.emailService.sendPasswordReset(job.data);
  }

  // Handle all jobs without specific name
  @Process()
  async handleDefault(job: Job) {
    console.log(`Processing job ${job.name}:`, job.data);
  }
}
```

**5. Add jobs (producer):**
```typescript
// users.service.ts
@Injectable()
export class UsersService {
  constructor(
    @InjectQueue('email') private emailQueue: Queue,
  ) {}

  async registerUser(dto: CreateUserDto) {
    const user = await this.usersRepo.save(dto);

    // Add welcome email job
    await this.emailQueue.add('welcome-email', {
      userId: user.id,
      email: user.email,
    }, {
      attempts: 5,
      priority: 1, // High priority
      delay: 2000, // Send after 2 seconds
    });

    return user;
  }
}
```

**Configuration options:**

- **Global (forRoot):**
  - Redis connection settings
  - Default job options (attempts, backoff, timeout)
  - Prefix for Redis keys

- **Queue-specific (registerQueue):**
  - Queue name (must be unique)
  - Custom Redis connection (override global)
  - Queue-specific job options

**Module structure (best practice):**
```
AppModule
├── BullModule.forRoot() ← Global config (once)
├── EmailModule
│   ├── BullModule.registerQueue({ name: 'email' })
│   ├── EmailProcessor ← Worker
│   └── EmailService
└── NotificationsModule
    ├── BullModule.registerQueue({ name: 'notifications' })
    └── NotificationsProcessor
```

**Environment configuration:**
```env
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=secret
```

</details>


<details>
<summary><strong>25. What is Redis and why is it used with Bull?</strong></summary>

**What is Redis:**
- In-memory data structure store
- Used as database, cache, and message broker
- Provides persistence (RDB snapshots + AOF logs)

**Why Bull uses Redis:**

- **Persistent job storage:**
  - Jobs survive app restarts
  - Data stored on disk (configurable persistence)
  - Queue state preserved across crashes

- **Job locking (atomic operations):**
  - `RPOPLPUSH` ensures exactly-once processing
  - Prevents duplicate job execution across workers
  - Concurrent-safe operations

- **Pub/Sub for notifications:**
  - Workers instantly notified of new jobs
  - No polling overhead
  - Event-driven job pickup

- **Fast in-memory operations:**
  - Sub-millisecond queue operations
  - High throughput (10k+ ops/sec)
  - Minimal latency for job processing

- **Data structures:**
  - Lists for job queues (FIFO)
  - Sorted sets for delayed/priority jobs
  - Hashes for job data and metadata

**Setup options:**

```typescript
// Local Redis
BullModule.forRoot({
  redis: {
    host: 'localhost',
    port: 6379,
  },
});

// Cloud Redis (AWS ElastiCache, Redis Cloud, Upstash)
BullModule.forRoot({
  redis: {
    host: 'redis.cloud.provider.com',
    port: 6380,
    password: process.env.REDIS_PASSWORD,
    tls: {}, // Enable TLS for cloud
  },
});

// Redis Cluster
BullModule.forRoot({
  redis: {
    cluster: [
      { host: 'node1', port: 6379 },
      { host: 'node2', port: 6379 },
      { host: 'node3', port: 6379 },
    ],
  },
});
```

**Production considerations:**

- **Persistence strategy:**
  - RDB (snapshots): Fast, periodic saves
  - AOF (append-only file): More durable, every write
  - Both: Maximum durability

- **Memory management:**
  - Configure `maxmemory` policy (prevent OOM)
  - Monitor queue sizes (alert on buildup)
  - Use `removeOnComplete` to auto-cleanup finished jobs

- **Security:**
  - Enable authentication (`requirepass`)
  - Use TLS for encrypted connections
  - Network isolation (VPC, firewall rules)

- **High availability:**
  - Redis Sentinel for automatic failover
  - Redis Cluster for horizontal scaling
  - Dedicated instance (don't share with cache)

- **Monitoring:**
  - Track memory usage (`INFO memory`)
  - Monitor connection count
  - Alert on queue backlog growth
  - Track job throughput metrics

</details>

<details>
<summary><strong>26. How do you create a queue processor?</strong></summary>

**Processor:** Worker service that executes jobs from a queue

**Implementation steps:**

**1. Create processor service:**
```typescript
import { Processor, Process } from '@nestjs/bull';
import { Job } from 'bull';

@Processor('email') // Match queue name
export class EmailProcessor {
  constructor(
    private emailService: EmailService,
    private logger: Logger,
  ) {}

  // Named job processor
  @Process({ name: 'welcome-email', concurrency: 5 })
  async sendWelcomeEmail(job: Job) {
    const { email, userId } = job.data;
    
    try {
      // Access job data
      await this.emailService.sendWelcome(email);
      
      // Report progress (0-100)
      await job.progress(100);
      
      // Return success
      return { success: true, email };
    } catch (error) {
      this.logger.error(`Job ${job.id} failed: ${error.message}`);
      throw error; // Triggers retry
    }
  }

  // Another named processor
  @Process('password-reset')
  async sendPasswordReset(job: Job) {
    await this.emailService.sendPasswordReset(job.data.email);
  }

  // Default processor (handles all other jobs)
  @Process()
  async handleAllOthers(job: Job) {
    console.log(`Processing ${job.name}:`, job.data);
  }
}
```

**2. Register in module:**
```typescript
@Module({
  imports: [BullModule.registerQueue({ name: 'email' })],
  providers: [EmailProcessor, EmailService],
})
export class EmailModule {}
```

**Key features:**

- **Job object properties:**
  - `job.id` - Unique job identifier
  - `job.data` - Job payload data
  - `job.opts` - Job options (attempts, priority, etc.)
  - `job.attemptsMade` - Current retry count
  - `job.progress()` - Report progress percentage
  - `job.log()` - Add log entry to job

- **Return value:**
  - Return data on success
  - Throw error to trigger retry
  - Return Promise for async operations

- **Dependency injection:**
  - Full NestJS DI support
  - Inject repositories, services, config, logger
  - Access any provider from constructor

- **Concurrency control:**
  - `@Process({ concurrency: 5 })` - Process 5 jobs simultaneously
  - Default concurrency: 1 (sequential)
  - Use higher concurrency for I/O-bound tasks

**Multiple processors in one service:**
```typescript
@Processor('notifications')
export class NotificationsProcessor {
  @Process('email')
  async sendEmail(job: Job) { }

  @Process('sms')
  async sendSMS(job: Job) { }

  @Process('push')
  async sendPush(job: Job) { }
}
```

**Best practices:**

- **Stateless processors:**
  - Don't store job state in instance variables
  - Use `job.data` for all inputs
  - Each job execution should be independent

- **Validation:**
  - Validate job.data structure before processing
  - Use class-validator with DTOs
  - Handle invalid data gracefully

- **Logging:**
  - Log job start/completion with job.id
  - Log errors with stack traces
  - Use structured logging for monitoring

- **Error handling:**
  - Throw errors to trigger retry
  - Return success to mark complete
  - Log errors before throwing

- **Type safety:**
  - Define TypeScript interfaces for job.data
  - Use generics: `Job<EmailJobData>`

```typescript
interface EmailJobData {
  userId: string;
  email: string;
  template: string;
}

@Process('send-email')
async sendEmail(job: Job<EmailJobData>) {
  const { userId, email, template } = job.data; // Type-safe
}
```

- **Idempotency:**
  - Make processors safe to run multiple times
  - Check if work already done before processing
  - Use unique transaction IDs

</details>


<details>
<summary><strong>27. How do you add jobs to a queue?</strong></summary>

**Adding jobs (producer pattern):**

**1. Inject Queue:**
```typescript
import { InjectQueue } from '@nestjs/bull';
import { Queue } from 'bull';

@Injectable()
export class UsersService {
  constructor(
    @InjectQueue('email') private emailQueue: Queue,
  ) {}
}
```

**2. Add jobs:**
```typescript
// Simple add
await this.emailQueue.add({ email: 'user@example.com', userId: 123 });

// Named job
await this.emailQueue.add('welcome-email', { 
  email: 'user@example.com',
  userId: 123,
});

// With options
await this.emailQueue.add('welcome-email', 
  { email: 'user@example.com' },
  {
    attempts: 3,                           // Retry count
    backoff: {
      type: 'exponential',                // or 'fixed'
      delay: 2000,                        // Initial delay (ms)
    },
    delay: 5000,                          // Delay execution by 5s
    priority: 1,                          // 1=highest
    timeout: 30000,                       // Max execution time (30s)
    removeOnComplete: true,               // Auto-cleanup
    removeOnFail: false,                  // Keep failed jobs
  }
);
```

**Job options explained:**

- **attempts** (`number`):
  - Number of retry attempts after failures
  - Default: 1 (no retries)
  - Example: `attempts: 3` - retry up to 3 times

- **backoff** (`object`):
  - Retry delay strategy
  - `{ type: 'exponential', delay: 2000 }` - 2s, 4s, 8s, 16s
  - `{ type: 'fixed', delay: 5000 }` - 5s between each retry
  - Custom function: `(attemptsMade) => attemptsMade * 1000`

- **delay** (`number`):
  - Milliseconds to wait before processing
  - Use for scheduled execution
  - Example: `delay: 60000` - process after 1 minute

- **priority** (`number`):
  - Job priority (1 = highest)
  - Lower numbers processed first
  - Default: no priority (FIFO)

- **timeout** (`number`):
  - Maximum execution time in milliseconds
  - Job fails if exceeds timeout
  - Example: `timeout: 300000` - 5 minute limit

- **removeOnComplete** (`boolean | number`):
  - Auto-delete completed jobs
  - `true` - remove immediately
  - `number` - keep last N completed jobs
  - Prevents memory buildup

- **removeOnFail** (`boolean | number`):
  - Auto-delete failed jobs
  - Usually `false` for debugging
  - `number` - keep last N failed jobs

**Adding patterns:**

```typescript
// Bulk add (efficient for many jobs)
const jobs = users.map(user => ({
  name: 'welcome-email',
  data: { userId: user.id, email: user.email },
  opts: { attempts: 3 },
}));
await this.emailQueue.addBulk(jobs);

// Delayed job (scheduled for future)
await this.emailQueue.add(
  'reminder',
  { appointmentId: 123 },
  { delay: 24 * 60 * 60 * 1000 } // 24 hours
);

// High priority job (urgent)
await this.emailQueue.add(
  'alert',
  { type: 'critical-error' },
  { priority: 1, attempts: 5 }
);

// Repeatable job (cron-like in queue)
await this.emailQueue.add(
  'daily-report',
  { type: 'sales' },
  {
    repeat: {
      cron: '0 9 * * *', // Daily at 9 AM
      tz: 'America/New_York',
    },
  }
);
```

**Return value:**
```typescript
const job = await this.emailQueue.add('welcome', { userId: 123 });

console.log(job.id); // Unique job ID
console.log(job.data); // Job payload
console.log(job.opts); // Job options

// Wait for completion (blocking - avoid in API handlers)
const result = await job.finished();
console.log(result); // Processor return value
```

**Best practices:**

- **Don't block requests:**
  - Add job and return immediately
  - Don't await `job.finished()` in API handlers
  - Use webhooks/polling for status updates

- **Job data structure:**
  - Keep data payload small
  - Store references (IDs) not full objects
  - Use TypeScript interfaces for type safety

- **Error handling:**
  - Wrap add() in try-catch
  - Handle queue connection failures
  - Log job IDs for tracking

- **Monitoring:**
  - Log job additions with metadata
  - Track queue length metrics
  - Alert on queue buildup

</details>

<details>
<summary><strong>28. What is the difference between cron jobs and queues?</strong></summary>

**Fundamental differences:**

| Aspect | Cron Jobs (`@Cron()`) | Queues (Bull) |
|--------|----------------------|---------------|
| **Trigger** | Time-based schedule | Event-based (user action, API call) |
| **Execution** | Runs at specific times | Processes when jobs added |
| **Use case** | Scheduled recurring tasks | Async user-triggered operations |
| **Retries** | No built-in retry | Automatic retry with backoff |
| **Scaling** | Single instance (needs locks) | Horizontal (multiple workers) |
| **Tracking** | Limited (logs only) | Full status tracking |
| **Priority** | No priority support | Job prioritization |
| **Delay** | Fixed schedule times | Dynamic delays |

**Detailed comparison:**

**Trigger mechanism:**
- **Cron:** `@Cron('0 2 * * *')` - runs daily at 2 AM regardless of events
- **Queue:** `queue.add(data)` - triggered by user signup, file upload, API call

**Execution timing:**
- **Cron:** Predictable schedule (hourly, daily, weekly)
- **Queue:** Immediate or delayed processing of queued jobs

**Use cases:**
- **Cron examples:**
  - Daily cleanup (delete old logs)
  - Weekly reports (send every Monday)
  - Hourly API sync (fetch external data)
  - Monthly billing (charge subscriptions)

- **Queue examples:**
  - Send email after user signup
  - Process uploaded file (image resize, video transcode)
  - Generate PDF report on demand
  - Webhook delivery with retries

**Retries:**
- **Cron:** Job fails → wait until next scheduled time (no automatic retry)
- **Queue:** Job fails → automatic retry with exponential backoff

**Scaling:**
- **Cron:** Runs on single instance (needs distributed locks for multi-instance)
- **Queue:** Multiple workers process jobs concurrently, horizontal scaling

**Job tracking:**
- **Cron:** Limited tracking (execution logs, duration)
- **Queue:** Full lifecycle tracking (pending/active/completed/failed/progress)

**Priority:**
- **Cron:** All jobs equal priority, no way to prioritize
- **Queue:** Support priority levels (1=highest, urgent jobs first)

**Delay:**
- **Cron:** Can't say "run this in 5 minutes" (fixed schedule only)
- **Queue:** Support dynamic delays (`delay: 5000` - execute after 5 seconds)

**Best practices:**

**Use Cron when:**
- Time-based recurring tasks (daily at 2 AM, every hour)
- Scheduled maintenance (cleanup, backups)
- Periodic data syncing
- Calendar-driven operations

**Use Queue when:**
- User-triggered async operations
- Tasks needing retries
- Resource-intensive operations (offload from API)
- High-volume batch processing
- Priority-based processing

**Combine both (hybrid pattern):**
```typescript
// Cron adds jobs to queue (best of both worlds)
@Cron('0 2 * * *')
async scheduleDailyCleanup() {
  // Add cleanup job to queue
  await this.cleanupQueue.add('cleanup', {
    type: 'old-logs',
    cutoffDays: 30,
  }, {
    attempts: 3,
    timeout: 600000, // 10 min
  });
}

// Queue processor handles heavy work
@Processor('cleanup')
class CleanupProcessor {
  @Process('cleanup')
  async processCleanup(job: Job) {
    // Heavy cleanup logic with retries
    await this.deleteOldLogs(job.data.cutoffDays);
  }
}
```

**Why combine:**
- Cron provides scheduled triggering
- Queue provides retry, monitoring, scaling
- Separates scheduling from execution
- Better fault tolerance

</details>

## Job Processing

<details>
<summary><strong>29. How do you handle job failures?</strong></summary>

**Failure handling strategies:**

**1. In processor (trigger retry):**
```typescript
@Processor('payments')
export class PaymentsProcessor {
  @Process('process-payment')
  async processPayment(job: Job) {
    try {
      await this.paymentService.charge(job.data);
      return { success: true };
    } catch (error) {
      // Log error with context
      this.logger.error(`Payment failed for job ${job.id}: ${error.message}`, {
        jobId: job.id,
        attempt: job.attemptsMade,
        maxAttempts: job.opts.attempts,
        data: job.data,
      });

      // Throw to trigger retry
      throw error;
    }
  }
}
```

**2. Event listeners (handle permanent failures):**
```typescript
// Decorator approach
@OnQueueFailed()
handleFailed(job: Job, error: Error) {
  this.logger.error(`Job ${job.id} permanently failed:`, error);
  
  // Check if retries exhausted
  if (job.attemptsMade >= job.opts.attempts) {
    // Move to dead letter queue
    await this.dlqQueue.add('failed-payment', job.data);
    
    // Alert ops team
    await this.alertService.sendSlack({
      channel: '#ops-alerts',
      message: `Critical: Payment job ${job.id} failed permanently`,
    });
  }
}

// Programmatic approach
constructor(@InjectQueue('payments') private queue: Queue) {
  this.queue.on('failed', async (job, error) => {
    await this.handleFailure(job, error);
  });
}
```

**3. Dead letter queue pattern:**
```typescript
@Module({
  imports: [
    BullModule.registerQueue({ name: 'payments' }),
    BullModule.registerQueue({ name: 'dead-letter' }), // DLQ
  ],
})
export class PaymentsModule {}

// In failure handler
if (job.attemptsMade >= job.opts.attempts) {
  await this.dlqQueue.add('failed-job', {
    originalQueue: 'payments',
    originalJobId: job.id,
    data: job.data,
    error: error.message,
    failedAt: new Date(),
  }, {
    removeOnComplete: false, // Keep for manual review
  });
}
```

**4. Error classification:**
```typescript
@Process('api-sync')
async syncWithAPI(job: Job) {
  try {
    await this.apiClient.fetch(job.data.endpoint);
  } catch (error) {
    // Transient errors: retry
    if (error.code === 'ETIMEDOUT' || error.status === 503) {
      throw error; // Triggers retry
    }
    
    // Permanent errors: don't retry
    if (error.status === 404 || error.status === 401) {
      this.logger.error('Permanent error, not retrying');
      return { success: false, reason: 'permanent_error' };
    }
    
    // Rate limit: longer backoff
    if (error.status === 429) {
      await job.moveToDelayed(Date.now() + 60000); // Wait 1 min
      throw error;
    }
    
    throw error; // Unknown error, retry
  }
}
```

**5. Failure tracking and alerting:**
```typescript
@Injectable()
export class JobMonitoringService {
  private failureCount = new Map<string, number>();

  async trackFailure(job: Job, error: Error) {
    const key = `${job.queue.name}:${job.name}`;
    const count = (this.failureCount.get(key) || 0) + 1;
    this.failureCount.set(key, count);

    // Alert if too many failures
    if (count > 10) {
      await this.alertService.send({
        severity: 'critical',
        message: `High failure rate for ${key}: ${count} failures`,
      });
    }

    // Store in DB for analysis
    await this.failuresRepo.save({
      queue: job.queue.name,
      jobName: job.name,
      jobId: job.id,
      error: error.message,
      stackTrace: error.stack,
      data: job.data,
      attemptsMade: job.attemptsMade,
      failedAt: new Date(),
    });
  }
}
```

**Best practices:**

- **Distinguish error types:** Retry transient (network, timeout), don't retry permanent (invalid data, auth)
- **Set appropriate retry limits:** `attempts: 3-5` for most cases
- **Use exponential backoff:** Prevents overwhelming failing services
- **Implement DLQ:** Prevents job loss, enables manual review/reprocessing
- **Alert on critical failures:** Payment processing, security operations
- **Log with context:** Job ID, attempt number, error details, job data
- **Monitor failure rates:** Track metrics, alert on spikes
- **Circuit breaker:** Stop calling repeatedly failing external services

</details>

<details>
<summary><strong>30. How do you implement retry logic for failed jobs?</strong></summary>

**Retry configuration:**

**1. Set retry options when adding job:**
```typescript
await queue.add('send-email', { email: 'user@example.com' }, {
  attempts: 5, // Total attempts (original + retries)
  backoff: {
    type: 'exponential', // or 'fixed', or function
    delay: 2000, // Initial delay in ms
  },
});
```

**2. Backoff strategies:**

**Exponential (recommended):**
```typescript
// Delays: 2s, 4s, 8s, 16s, 32s
backoff: {
  type: 'exponential',
  delay: 2000,
}

// Execution timeline:
// Attempt 1: Immediate
// Attempt 2: After 2 seconds
// Attempt 3: After 4 seconds
// Attempt 4: After 8 seconds
// Attempt 5: After 16 seconds
```

**Fixed delay:**
```typescript
// Same delay between each retry
backoff: {
  type: 'fixed',
  delay: 5000, // Wait 5s between each attempt
}
```

**Custom function:**
```typescript
// Calculate delay based on attempt number
backoff: (attemptsMade: number) => {
  // Linear: 1s, 2s, 3s, 4s
  return attemptsMade * 1000;
  
  // Exponential with cap: max 60s
  return Math.min(Math.pow(2, attemptsMade) * 1000, 60000);
  
  // Fibonacci: 1s, 1s, 2s, 3s, 5s, 8s
  // return fibonacci(attemptsMade) * 1000;
}
```

**3. In processor:**
```typescript
@Process('api-call')
async makeAPICall(job: Job) {
  const { attemptsMade, opts } = job;
  
  this.logger.log(`Attempt ${attemptsMade + 1}/${opts.attempts}`);

  try {
    const result = await this.apiClient.call(job.data);
    return result;
  } catch (error) {
    // Log retry info
    this.logger.warn(`Attempt ${attemptsMade + 1} failed: ${error.message}`);
    
    // Check if retries remaining
    if (attemptsMade < opts.attempts - 1) {
      this.logger.log(`Will retry (${opts.attempts - attemptsMade - 1} attempts left)`);
    } else {
      this.logger.error('No retries left, job will fail');
    }
    
    throw error; // Triggers retry
  }
}
```

**4. Conditional retry logic:**
```typescript
@Process('payment')
async processPayment(job: Job) {
  try {
    await this.paymentGateway.charge(job.data);
  } catch (error) {
    // Retry transient errors
    if (error.code === 'NETWORK_ERROR' || error.code === 'TIMEOUT') {
      throw error; // Will retry with backoff
    }
    
    // Don't retry permanent errors
    if (error.code === 'INVALID_CARD' || error.code === 'INSUFFICIENT_FUNDS') {
      this.logger.error('Permanent error, not retrying');
      
      // Notify user
      await this.notifyPaymentFailed(job.data.userId, error.code);
      
      // Return success to prevent retry
      return { success: false, reason: error.code };
    }
    
    // Rate limit: custom delay
    if (error.code === 'RATE_LIMIT') {
      await job.moveToDelayed(Date.now() + 60000); // Wait 1 min
      throw error;
    }
    
    throw error; // Unknown error, retry
  }
}
```

**5. Monitor retry metrics:**
```typescript
@Injectable()
export class RetryMonitoringService {
  constructor(@InjectQueue('tasks') private queue: Queue) {
    this.queue.on('failed', this.trackRetry.bind(this));
  }

  private trackRetry(job: Job, error: Error) {
    const metrics = {
      jobName: job.name,
      attemptsMade: job.attemptsMade,
      maxAttempts: job.opts.attempts,
      retriesRemaining: job.opts.attempts - job.attemptsMade,
      errorType: error.name,
    };

    this.logger.log('Job retry metrics:', metrics);

    // Send to monitoring service
    this.metricsService.increment('job.retries', {
      queue: job.queue.name,
      job: job.name,
    });

    // Alert if frequently hitting max retries
    if (job.attemptsMade >= job.opts.attempts) {
      this.metricsService.increment('job.max_retries_reached');
    }
  }
}
```

**Advanced patterns:**

**Jittered exponential backoff:**
```typescript
// Adds randomness to prevent thundering herd
backoff: (attemptsMade: number) => {
  const exponential = Math.pow(2, attemptsMade) * 1000;
  const jitter = Math.random() * 1000;
  return exponential + jitter;
}
```

**Max backoff cap:**
```typescript
// Prevent excessively long delays
backoff: (attemptsMade: number) => {
  const exponential = Math.pow(2, attemptsMade) * 1000;
  return Math.min(exponential, 60000); // Cap at 60 seconds
}
```

**Best practices:**

- **Use exponential backoff:** Prevents overwhelming failing services
- **Set appropriate retry limits:** 3-5 attempts for most cases, higher (5-10) for critical operations
- **Log each attempt:** Track progress, diagnose issues
- **Distinguish error types:** Don't retry permanent errors
- **Monitor retry rates:** High retry rates indicate systemic issues
- **Add jitter:** Prevents thundering herd problem
- **Cap backoff delays:** Prevent jobs waiting hours

</details>

<details>
<summary><strong>31. How do you implement job prioritization?</strong></summary>

**Priority system:**

**1. Set priority when adding job:**
```typescript
// Higher priority = processed first (1-5 scale)
await queue.add('send-email', { to: 'user@example.com' }, {
  priority: 1, // Highest priority (urgent)
});

await queue.add('welcome-email', { to: 'new@example.com' }, {
  priority: 5, // Lowest priority (can wait)
});

// Priority scale:
// 1 = Critical (security alerts, payment failures)
// 2 = High (password reset, important notifications)
// 3 = Normal (default - regular emails, reports)
// 4 = Low (newsletters, welcome emails)
// 5 = Lowest (cleanup, analytics, background tasks)
```

**2. Priority-based processing:**
```typescript
@Injectable()
export class NotificationService {
  constructor(@InjectQueue('notifications') private queue: Queue) {}

  async sendSecurityAlert(userId: string, message: string) {
    return this.queue.add('security-alert', { userId, message }, {
      priority: 1, // Process immediately
      attempts: 5, // Critical, retry aggressively
    });
  }

  async sendPasswordReset(email: string, token: string) {
    return this.queue.add('password-reset', { email, token }, {
      priority: 2, // High priority
      attempts: 3,
    });
  }

  async sendNewsletter(emails: string[]) {
    return this.queue.add('newsletter', { emails }, {
      priority: 5, // Low priority, process when idle
      attempts: 2,
    });
  }
}
```

**3. Processor handles all priorities:**
```typescript
@Processor('notifications')
export class NotificationsProcessor {
  @Process({ concurrency: 10 }) // Process 10 jobs concurrently
  async handle(job: Job) {
    const { priority, name, data } = job.opts;
    
    this.logger.log(`Processing ${name} with priority ${priority}`);
    
    // Same processor, Bull handles priority ordering
    switch (job.name) {
      case 'security-alert':
        return this.sendSecurityAlert(data);
      case 'password-reset':
        return this.sendPasswordReset(data);
      case 'newsletter':
        return this.sendNewsletter(data);
    }
  }
}
```

**4. Priority queue behavior:**
```typescript
// Execution order (highest priority first)
// Queue state:
// - Job A: priority 1 (added at 10:00)
// - Job B: priority 3 (added at 09:55)
// - Job C: priority 1 (added at 10:01)
// - Job D: priority 5 (added at 09:50)

// Processing order: A → C → B → D
// (priority 1 jobs first, then by age within same priority)
```

**5. Dynamic priority based on conditions:**
```typescript
async addPaymentJob(order: Order) {
  const priority = this.calculatePriority(order);
  
  return this.queue.add('process-payment', { orderId: order.id }, {
    priority,
    attempts: 5,
  });
}

private calculatePriority(order: Order): number {
  // VIP customers: highest priority
  if (order.customer.isVIP) return 1;
  
  // Large orders: high priority
  if (order.total > 1000) return 2;
  
  // Express shipping: high priority
  if (order.shippingMethod === 'express') return 2;
  
  // Normal orders: default
  return 3;
}
```

**Use cases:**

| Priority | Use Case | Examples |
|----------|----------|----------|
| 1 (Critical) | Security, payments | Security alerts, payment processing, fraud detection |
| 2 (High) | Time-sensitive user actions | Password reset, order confirmations, 2FA codes |
| 3 (Normal) | Regular operations | Standard emails, reports, notifications |
| 4 (Low) | Non-urgent tasks | Welcome emails, newsletters, recommendations |
| 5 (Lowest) | Background maintenance | Cleanup, analytics, optimization |

**Best practices:**

- **Use sparingly:** Don't make everything high priority (defeats the purpose)
- **Define clear criteria:** Document what priority means for your system
- **Monitor priority distribution:** Too many high-priority jobs indicates misuse
- **Combine with separate queues:** Critical operations in dedicated queue
- **Default to normal (3):** Only deviate when truly needed
- **Consider delay instead:** Low-priority jobs can use delay option instead

**Limitations:**

- Priority evaluated when job is fetched (not when added)
- Within same priority, FIFO order
- Doesn't preempt running jobs (existing jobs finish first)
- High concurrency reduces priority impact

</details>

<details>
<summary><strong>32. How do you process jobs concurrently?</strong></summary>

**Concurrency configuration:**

**1. Set concurrency in processor:**
```typescript
@Processor('emails')
export class EmailProcessor {
  // Process up to 10 jobs simultaneously
  @Process({ concurrency: 10 })
  async sendEmail(job: Job) {
    await this.emailService.send(job.data);
    return { sent: true };
  }
  
  // Different concurrency for different job types
  @Process({ name: 'bulk-email', concurrency: 20 })
  async sendBulkEmail(job: Job) {
    // I/O-bound, can handle more concurrency
    await this.emailService.sendBulk(job.data.emails);
  }
  
  @Process({ name: 'image-processing', concurrency: 2 })
  async processImage(job: Job) {
    // CPU-bound, limit concurrency to CPU cores
    await this.imageService.process(job.data.imageUrl);
  }
}
```

**2. Choosing concurrency based on workload:**

**I/O-bound tasks (high concurrency):**
```typescript
@Process({ concurrency: 20 }) // Network calls, mostly waiting
async callExternalAPI(job: Job) {
  // Spends most time waiting for network response
  const response = await this.httpService.get(job.data.url);
  return response.data;
}

// Examples: API calls, database queries, file I/O, email sending
// Can handle 10-50 concurrent jobs (mostly waiting, not processing)
```

**CPU-bound tasks (low concurrency):**
```typescript
@Process({ concurrency: 2 }) // Match CPU cores
async encodeVideo(job: Job) {
  // Intensive CPU work
  const result = await this.videoService.encode(job.data.videoPath);
  return result;
}

// Examples: Image/video processing, encryption, data transformation
// Limit to 1-4 (number of CPU cores)
```

**Mixed workload:**
```typescript
@Process({ concurrency: 5 }) // Balanced
async generateReport(job: Job) {
  // Database query (I/O) + calculation (CPU) + save file (I/O)
  const data = await this.reportsRepo.find(job.data.filters); // I/O
  const metrics = this.calculateMetrics(data); // CPU
  await this.storageService.save(metrics); // I/O
  return metrics;
}
```

**3. Horizontal scaling:**
```typescript
// Worker Instance 1: concurrency 10
// Worker Instance 2: concurrency 10
// Worker Instance 3: concurrency 10
// Total concurrent processing: 30 jobs

// Redis coordinates to prevent duplicate processing
// Each worker independently fetches jobs from queue
```

**4. Monitor and adjust:**
```typescript
@Injectable()
export class ConcurrencyMonitoringService {
  constructor(@InjectQueue('tasks') private queue: Queue) {
    this.setupMonitoring();
  }

  private async setupMonitoring() {
    // Check active jobs count
    const counts = await this.queue.getJobCounts();
    this.logger.log(`Active jobs: ${counts.active}`);
    
    // Track processing rate
    this.queue.on('completed', () => {
      this.metricsService.increment('jobs.completed');
    });
    
    // Alert if queue backing up
    if (counts.waiting > 1000) {
      this.alertService.send('Queue backlog detected, consider scaling');
    }
  }
}
```

**5. Resource management:**
```typescript
// Database connection pooling
@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      // Match pool size to concurrency
      poolSize: 15, // Slightly more than concurrency (10)
    }),
  ],
})

// Rate limiting for external APIs
@Process({ concurrency: 10 })
async callAPI(job: Job) {
  // Even with 10 concurrent, respect API rate limits
  await this.rateLimiter.acquire(); // Max 5 req/sec
  const result = await this.apiClient.call(job.data);
  return result;
}
```

**Concurrency formula:**

| Workload Type | Recommended Concurrency | Reasoning |
|---------------|------------------------|-----------|
| Pure I/O (API calls, DB queries) | 10-50 | Mostly waiting, minimal CPU |
| Mixed (some CPU, some I/O) | 5-10 | Balanced approach |
| CPU-intensive (encoding, encryption) | 1-4 | Limited by CPU cores |
| Memory-intensive (large data) | 2-5 | Prevent OOM errors |

**Scaling strategy:**

**Vertical (single worker):**
- Increase concurrency
- Limited by server resources
- Good for: Small to medium workloads

**Horizontal (multiple workers):**
```bash
# Start 3 worker instances
pm2 start npm --name worker-1 -- run worker
pm2 start npm --name worker-2 -- run worker
pm2 start npm --name worker-3 -- run worker

# Each with concurrency 10 = total 30 concurrent jobs
```

**Best practices:**

- **Start low:** Begin with concurrency 3-5, increase based on monitoring
- **Match resources:** Don't exceed CPU cores for CPU-bound tasks
- **Connection pools:** Size database pool to match or exceed concurrency
- **Rate limits:** Respect external API limits regardless of concurrency
- **Monitor metrics:** Track CPU, memory, job throughput
- **Horizontal scaling:** Add workers instead of increasing concurrency infinitely
- **Test under load:** Simulate production volume to find optimal settings

</details>

## Background Workers

<details>
<summary><strong>33. What are background workers?</strong></summary>

**Definition:**

- **Separate processes** that consume and process jobs from queues asynchronously
- **Independent** of web server request/response cycle
- **Long-running** tasks without blocking API responses

**Architecture:**

```typescript
// Producer (API Server)
@Injectable()
export class OrdersController {
  constructor(@InjectQueue('orders') private ordersQueue: Queue) {}

  @Post('/')
  async createOrder(@Body() data: CreateOrderDto) {
    // Save to database
    const order = await this.ordersRepo.save(data);
    
    // Add job to queue (async processing)
    await this.ordersQueue.add('process-order', {
      orderId: order.id,
    });
    
    // Return immediately
    return { success: true, orderId: order.id };
  }
}

// Consumer (Background Worker)
@Processor('orders')
export class OrdersProcessor {
  @Process('process-order')
  async processOrder(job: Job) {
    const { orderId } = job.data;
    
    // Heavy processing (doesn't block API)
    await this.paymentService.charge(orderId);
    await this.inventoryService.reserve(orderId);
    await this.emailService.sendConfirmation(orderId);
    await this.shippingService.createLabel(orderId);
    
    return { success: true };
  }
}
```

**Deployment patterns:**

**1. Separate worker servers (production):**
```bash
# API Server (no workers)
API_ONLY=true npm start

# Worker Server (no API)
WORKER_ONLY=true npm run worker

# In app.module.ts
@Module({
  imports: [
    process.env.API_ONLY !== 'true' && BullModule.registerQueue({ name: 'orders' }),
    process.env.WORKER_ONLY !== 'true' && HttpModule,
  ].filter(Boolean),
})
```

**2. Combined (development/small deployments):**
```bash
# Single process runs both API and workers
npm start
```

**3. Process managers:**
```bash
# PM2 ecosystem file
module.exports = {
  apps: [
    {
      name: 'api',
      script: 'dist/main.js',
      instances: 4,
      env: { API_ONLY: 'true' },
    },
    {
      name: 'worker',
      script: 'dist/worker.js',
      instances: 2,
      env: { WORKER_ONLY: 'true' },
    },
  ],
};
```

**Use cases:**

| Use Case | Why Background Worker | Example |
|----------|----------------------|----------|
| Email sending | Don't block API response | Welcome email after signup |
| File processing | CPU/time intensive | Resize image, convert video |
| External API calls | Unreliable, slow | Sync with third-party |
| Report generation | Large data processing | Monthly analytics PDF |
| Data ETL | Batch operations | Import 1M records |
| Scheduled tasks | Cron adds to queue | Daily cleanup job |

**Benefits:**

- **Decoupling:** API responds fast, heavy work offloaded
- **Reliability:** Automatic retries, persistent storage (Redis)
- **Scalability:** Add more workers independently
- **Monitoring:** Track job status, progress, failures
- **Priority:** Urgent jobs processed first
- **Resource control:** Limit concurrency, prevent overload

**Best practices:**

- **Separate in production:** Dedicated worker servers, scale independently
- **Use queues:** Don't process heavy logic synchronously in API
- **Monitor queue depth:** Alert on backlogs, scale workers
- **Implement retries:** Handle transient failures
- **Log with context:** Job ID, user ID, operation type

</details>

<details>
<summary><strong>34. When should you use background workers vs cron jobs?</strong></summary>

**Decision matrix:**

| Criteria | Background Worker (Queue) | Cron Job |
|----------|---------------------------|----------|
| **Trigger** | Event-driven (user action) | Time-based (schedule) |
| **Volume** | Variable (scales with traffic) | Predictable (fixed schedule) |
| **Execution** | Immediate or delayed | Fixed time |
| **Retries** | Built-in retry logic | Manual implementation |
| **Priority** | Supports prioritization | All equal |
| **Tracking** | Job status, progress | Simple logging |
| **Scaling** | Horizontal (add workers) | Limited |
| **Use when** | User-triggered, needs reliability | Periodic maintenance |

**Background worker use cases:**

```typescript
// User action triggers job
@Post('/upload')
async uploadFile(@UploadedFile() file) {
  // Queue for processing
  await this.filesQueue.add('process-upload', {
    fileId: file.id,
    userId: req.user.id,
  }, {
    attempts: 3, // Retry on failure
    priority: file.isPriority ? 1 : 3,
  });
  
  return { uploaded: true };
}

// Examples:
// - Send email after signup
// - Process uploaded image (resize, thumbnail)
// - Payment processing
// - External API sync
// - File conversion
```

**Cron job use cases:**

```typescript
// Time-based recurring task
@Cron('0 2 * * *', { timeZone: 'UTC' }) // Daily at 2 AM
async dailyCleanup() {
  await this.cleanupService.deleteOldLogs();
  await this.cleanupService.archiveData();
  this.logger.log('Daily cleanup completed');
}

// Examples:
// - Daily data cleanup
// - Weekly reports
// - Hourly cache refresh
// - Monthly billing
// - Database maintenance
```

**Hybrid approach (recommended for heavy jobs):**

```typescript
// Cron schedules, queue processes
@Cron('0 2 * * *')
async scheduleNightlyReports() {
  // Add job to queue (not process directly)
  await this.reportsQueue.add('generate-report', {
    date: new Date(),
    type: 'nightly',
  }, {
    attempts: 3,
    timeout: 600000, // 10 minutes
  });
  
  this.logger.log('Report job queued');
}

@Processor('reports')
export class ReportsProcessor {
  @Process('generate-report')
  async generateReport(job: Job) {
    // Heavy processing with retry support
    const data = await this.fetchData();
    const report = await this.generate(data);
    await this.save(report);
    return { reportId: report.id };
  }
}

// Benefits:
// - Cron ensures timing (runs at 2 AM)
// - Queue provides retries and tracking
// - Workers can scale independently
// - Progress monitoring available
```

**Anti-patterns:**

**❌ Don't use cron for user-triggered:**
```typescript
// BAD: Can't scale, delays processing
@Post('/send-email')
async sendEmail() {
  // Stores in DB, cron checks every minute
  await this.emailsRepo.save({ status: 'pending' });
}
```

**✅ Use queue instead:**
```typescript
// GOOD: Immediate processing, scales
@Post('/send-email')
async sendEmail() {
  await this.emailQueue.add('send', emailData);
}
```

**❌ Don't use queue for simple time-based:**
```typescript
// BAD: Overkill for simple task
@Cron('*/5 * * * *')
async healthCheck() {
  await this.healthQueue.add('check', {});
}
```

**✅ Use cron directly:**
```typescript
// GOOD: Simple, no queue overhead
@Cron('*/5 * * * *')
async healthCheck() {
  const status = await this.checkHealth();
  this.logger.log('Health:', status);
}
```

**Quick decision guide:**

- **User uploads file?** → Queue
- **Daily at 2 AM?** → Cron
- **Needs retries?** → Queue
- **Heavy processing?** → Cron + Queue
- **Variable volume?** → Queue
- **Simple periodic check?** → Cron

</details>

<details>
<summary><strong>35. How do you ensure jobs don't run twice in clustered environments?</strong></summary>

**Problem:**

- Multiple instances run same `@Cron()` simultaneously
- 5 servers = 5 executions of daily cleanup
- Queue workers auto-coordinated by Bull/Redis
- **Cron jobs need explicit coordination**

**Solution 1: Distributed locks (Redlock):**

```typescript
// Install: npm install redlock ioredis
import Redlock from 'redlock';
import Redis from 'ioredis';

@Injectable()
export class TasksService {
  private redlock: Redlock;

  constructor() {
    const redis = new Redis(process.env.REDIS_URL);
    this.redlock = new Redlock([redis], {
      driftFactor: 0.01,
      retryCount: 3,
      retryDelay: 200,
    });
  }

  @Cron('0 2 * * *') // Daily at 2 AM
  async dailyCleanup() {
    const resource = 'locks:daily-cleanup';
    const ttl = 3600000; // 1 hour

    let lock;
    try {
      // Try to acquire lock
      lock = await this.redlock.acquire([resource], ttl);
      
      // Only this instance executes
      this.logger.log('Lock acquired, running cleanup');
      await this.performCleanup();
      
    } catch (error) {
      // Another instance has lock
      this.logger.log('Lock not acquired, skipping (another instance running)');
    } finally {
      // Release lock
      if (lock) {
        await lock.release();
      }
    }
  }
}
```

**Solution 2: Simple Redis flag:**

```typescript
@Injectable()
export class ReportsService {
  constructor(@InjectRedis() private readonly redis: Redis) {}

  @Cron('0 * * * *') // Hourly
  async generateHourlyReport() {
    const lockKey = 'lock:hourly-report';
    const lockTTL = 3600; // 1 hour in seconds

    // Try to set lock (NX = set if not exists)
    const acquired = await this.redis.set(
      lockKey,
      Date.now().toString(),
      'EX',
      lockTTL,
      'NX'
    );

    if (!acquired) {
      this.logger.log('Job already running elsewhere');
      return;
    }

    try {
      // Execute job
      await this.generateReport();
    } finally {
      // Clean up lock
      await this.redis.del(lockKey);
    }
  }
}
```

**Solution 3: Cron + Queue hybrid (recommended):**

```typescript
// Only ONE instance adds job to queue
@Cron('0 2 * * *')
async scheduleCleanup() {
  const lockKey = 'lock:schedule-cleanup';
  const acquired = await this.redis.set(lockKey, '1', 'EX', 60, 'NX');
  
  if (acquired) {
    // This instance won the race
    await this.cleanupQueue.add('daily-cleanup', {}, {
      jobId: `cleanup-${new Date().toISOString().split('T')[0]}`, // Unique per day
      attempts: 3,
    });
    this.logger.log('Cleanup job queued');
  }
}

// Bull ensures single processing
@Processor('cleanup')
export class CleanupProcessor {
  @Process('daily-cleanup')
  async handle(job: Job) {
    await this.performCleanup();
    return { success: true };
  }
}
```

**Solution 4: Job deduplication:**

```typescript
// Prevent duplicate jobs in queue
async addJob(userId: string) {
  const jobId = `user-${userId}-welcome`; // Unique ID
  
  // Check if job already exists
  const existing = await this.queue.getJob(jobId);
  if (existing) {
    this.logger.log(`Job ${jobId} already exists`);
    return existing;
  }
  
  // Add with unique jobId
  return this.queue.add('send-welcome', { userId }, {
    jobId, // Bull prevents duplicates with same ID
    removeOnComplete: true,
  });
}
```

**Comparison:**

| Approach | Pros | Cons | Use When |
|----------|------|------|----------|
| Redlock | Reliable, tested algorithm | Complex setup | Production cron jobs |
| Simple Redis flag | Easy to implement | Less reliable | Small deployments |
| Cron + Queue | Best practices | Requires queue | Heavy jobs |
| Job deduplication | Prevents queue duplicates | Only for queues | User-triggered |

**Queue workers (automatic):**

```typescript
// NO COORDINATION NEEDED - Bull uses Redis RPOPLPUSH (atomic)
@Processor('emails')
export class EmailProcessor {
  @Process({ concurrency: 10 })
  async sendEmail(job: Job) {
    // Each worker gets different jobs, no duplicates
    await this.send(job.data);
  }
}
```

**Best practices:**

- **Queue workers:** No coordination needed (Bull handles it)
- **Cron jobs:** Use distributed locks or Cron+Queue
- **Make idempotent:** Safe to run twice
- **Set appropriate TTL:** Longer than max job duration
- **Monitor lock acquisition:** Alert if failing

</details>

## Monitoring & Logging

<details>
<summary><strong>36. How do you monitor scheduled jobs?</strong></summary>

**1. Bull Board (web UI):**

```typescript
// Install: npm install @bull-board/express @bull-board/api
import { BullAdapter } from '@bull-board/api/bullAdapter';
import { ExpressAdapter } from '@bull-board/express';
import { createBullBoard } from '@bull-board/api';

// In main.ts or dedicated controller
const serverAdapter = new ExpressAdapter();
serverAdapter.setBasePath('/admin/queues');

createBullBoard({
  queues: [
    new BullAdapter(emailQueue),
    new BullAdapter(reportsQueue),
    new BullAdapter(uploadsQueue),
  ],
  serverAdapter,
});

app.use('/admin/queues', serverAdapter.getRouter());

// Access at: http://localhost:3000/admin/queues
// Shows: waiting, active, completed, failed, delayed counts
// Can: retry failed jobs, view job data, see logs
```

**2. Queue event listeners:**

```typescript
@Injectable()
export class QueueMonitoringService implements OnModuleInit {
  constructor(
    @InjectQueue('orders') private ordersQueue: Queue,
    private metricsService: MetricsService,
    private alertService: AlertService,
  ) {}

  onModuleInit() {
    // Track completions
    this.ordersQueue.on('completed', (job, result) => {
      this.logger.log(`Job ${job.id} completed in ${Date.now() - job.timestamp}ms`);
      this.metricsService.increment('jobs.completed', {
        queue: 'orders',
        jobName: job.name,
      });
    });

    // Track failures
    this.ordersQueue.on('failed', (job, error) => {
      this.logger.error(`Job ${job.id} failed:`, error);
      this.metricsService.increment('jobs.failed', {
        queue: 'orders',
        jobName: job.name,
        error: error.name,
      });
      
      // Alert on critical jobs
      if (job.name === 'process-payment') {
        this.alertService.send({
          severity: 'high',
          message: `Payment job ${job.id} failed: ${error.message}`,
        });
      }
    });

    // Track stalled jobs (worker crashed)
    this.ordersQueue.on('stalled', (job) => {
      this.logger.warn(`Job ${job.id} stalled`);
      this.alertService.send({
        severity: 'medium',
        message: `Job ${job.id} stalled, worker may have crashed`,
      });
    });
  }
}
```

**3. Job metrics:**

```typescript
@Injectable()
export class JobMetricsService {
  @Cron('*/1 * * * *') // Every minute
  async collectMetrics() {
    const queues = [this.emailQueue, this.ordersQueue, this.uploadsQueue];

    for (const queue of queues) {
      const counts = await queue.getJobCounts();
      
      // Queue depth
      this.metricsService.gauge(`queue.${queue.name}.waiting`, counts.waiting);
      this.metricsService.gauge(`queue.${queue.name}.active`, counts.active);
      this.metricsService.gauge(`queue.${queue.name}.failed`, counts.failed);
      
      // Alert on backlog
      if (counts.waiting > 1000) {
        this.alertService.send({
          severity: 'high',
          message: `Queue ${queue.name} backlog: ${counts.waiting} jobs waiting`,
        });
      }
      
      // Alert on high failure rate
      const failureRate = counts.failed / (counts.completed + counts.failed);
      if (failureRate > 0.05) { // >5%
        this.alertService.send({
          severity: 'critical',
          message: `Queue ${queue.name} high failure rate: ${(failureRate * 100).toFixed(1)}%`,
        });
      }
    }
  }
}
```

**4. Health check endpoint:**

```typescript
@Controller('health')
export class HealthController {
  constructor(@InjectQueue('orders') private ordersQueue: Queue) {}

  @Get('/jobs')
  async checkJobs() {
    const counts = await this.ordersQueue.getJobCounts();
    const waiting = await this.ordersQueue.getWaiting(0, 0);
    
    const isHealthy =
      counts.waiting < 1000 && // Not too many waiting
      counts.active < 100 && // Not too many active
      (waiting.length === 0 || Date.now() - waiting[0].timestamp < 300000); // Oldest < 5 min

    return {
      status: isHealthy ? 'healthy' : 'unhealthy',
      counts,
      oldestWaitingMs: waiting.length > 0 ? Date.now() - waiting[0].timestamp : 0,
    };
  }
}
```

**Alerting rules:**

- **Failure rate > 5%:** Critical alert
- **Queue backlog > 1000:** High alert
- **Stalled jobs detected:** Medium alert
- **Job duration > 10 min:** Warning
- **No jobs completed in 1 hour:** Critical (workers down)

**Best practices:**

- **Use Bull Board:** Easy visual monitoring
- **Track all events:** completed, failed, stalled, active
- **Monitor queue depth:** Alert on backlogs
- **Track success rate:** Detect systemic issues
- **Measure latency:** Time from added to completed
- **Structured logging:** Enable searching/filtering
- **Export to Prometheus:** Create Grafana dashboards
- **Health checks:** Integrate with load balancers

</details>


<details>
<summary><strong>37. How do you log job execution?</strong></summary>

**Structured logging:**

**1. For cron jobs:**
```typescript
@Injectable()
export class TasksService {
  private readonly logger = new Logger(TasksService.name);

  @Cron('0 2 * * *', { name: 'daily-cleanup' })
  async dailyCleanup() {
    const startTime = Date.now();
    const jobName = 'daily-cleanup';
    
    // Log start with context
    this.logger.log({
      event: 'cron.started',
      jobName,
      schedule: '0 2 * * *',
      timestamp: new Date().toISOString(),
    });

    try {
      // Execute job
      await this.cleanupService.performCleanup();
      
      // Log success
      this.logger.log({
        event: 'cron.completed',
        jobName,
        duration: Date.now() - startTime,
        status: 'success',
      });
    } catch (error) {
      // Log failure
      this.logger.error({
        event: 'cron.failed',
        jobName,
        duration: Date.now() - startTime,
        error: error.message,
        stack: error.stack,
      });
      
      throw error;
    }
  }
}
```

**2. For queue processors:**
```typescript
@Processor('orders')
export class OrdersProcessor {
  private readonly logger = new Logger(OrdersProcessor.name);

  @Process('process-order')
  async processOrder(job: Job) {
    const startTime = Date.now();
    
    // Log start with full context
    this.logger.log({
      event: 'job.started',
      jobId: job.id,
      jobName: job.name,
      queue: job.queue.name,
      attempt: job.attemptsMade + 1,
      maxAttempts: job.opts.attempts,
      data: this.sanitizeData(job.data), // Remove sensitive fields
    });

    try {
      const result = await this.orderService.process(job.data);
      
      // Log completion
      this.logger.log({
        event: 'job.completed',
        jobId: job.id,
        duration: Date.now() - startTime,
        result: { orderId: result.id, status: result.status },
      });
      
      return result;
    } catch (error) {
      // Log failure with retry context
      this.logger.error({
        event: 'job.failed',
        jobId: job.id,
        duration: Date.now() - startTime,
        attempt: job.attemptsMade + 1,
        retriesRemaining: job.opts.attempts - job.attemptsMade - 1,
        error: error.message,
        stack: error.stack,
      });
      
      throw error;
    }
  }

  private sanitizeData(data: any) {
    // Remove sensitive fields
    const { password, creditCard, ssn, ...safe } = data;
    return safe;
  }
}
```

**3. JSON logging with Winston:**
```typescript
// logger.config.ts
import { WinstonModule } from 'nest-winston';
import * as winston from 'winston';

export const winstonConfig = WinstonModule.createLogger({
  transports: [
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json(),
      ),
    }),
    new winston.transports.File({
      filename: 'logs/jobs.log',
      format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json(),
      ),
    }),
  ],
});

// Usage
this.logger.log({
  event: 'job.started',
  jobId: job.id,
  timestamp: new Date().toISOString(),
  userId: job.data.userId,
  correlationId: job.data.correlationId,
});
```

**4. Progress logging for long jobs:**
```typescript
@Process('generate-report')
async generateReport(job: Job) {
  const totalRecords = job.data.recordCount;
  let processed = 0;

  this.logger.log(`Starting report generation for ${totalRecords} records`);

  for (const batch of this.getBatches(job.data)) {
    await this.processBatch(batch);
    processed += batch.length;
    
    const progress = Math.floor((processed / totalRecords) * 100);
    await job.progress(progress);
    
    // Log milestone (every 25%)
    if (progress % 25 === 0) {
      this.logger.log({
        event: 'job.progress',
        jobId: job.id,
        progress,
        processed,
        totalRecords,
      });
    }
  }

  this.logger.log(`Report generation complete: ${processed} records processed`);
}
```

**5. Error logging with Sentry:**
```typescript
import * as Sentry from '@sentry/node';

@Process('critical-task')
async handleCriticalTask(job: Job) {
  try {
    await this.taskService.execute(job.data);
  } catch (error) {
    // Log to Sentry with context
    Sentry.withScope((scope) => {
      scope.setContext('job', {
        id: job.id,
        name: job.name,
        queue: job.queue.name,
        attempt: job.attemptsMade,
        data: job.data,
      });
      scope.setTag('job_type', job.name);
      scope.setLevel('error');
      
      Sentry.captureException(error);
    });
    
    throw error;
  }
}
```

**Log levels:**

| Level | Use When | Example |
|-------|----------|---------|
| DEBUG | Development details | Variable values, intermediate steps |
| INFO | Normal execution | Job started, job completed |
| WARN | Recoverable issues | Retry triggered, slow performance |
| ERROR | Failures | Job failed, external API error |

**Best practices:**

- **Log at key points:** Start, completion, failure, milestones
- **Use structured format:** JSON for machine parsing
- **Include context:** jobId, userId, correlationId
- **Sanitize sensitive data:** Remove passwords, tokens, PII
- **Track duration:** Performance monitoring
- **Use correlation IDs:** Trace across services
- **Log sampling:** High-volume jobs (every Nth execution)
- **Appropriate levels:** DEBUG/INFO/WARN/ERROR
- **Error details:** Stack trace, attempt number
- **Business context:** OrderId, operation type

</details>

<details>
<summary><strong>38. How do you handle long-running jobs?</strong></summary>

**Strategies:**

**1. Progress reporting:**
```typescript
@Process('data-migration')
async migrateData(job: Job) {
  const totalRecords = await this.getTotalRecords();
  const batchSize = 1000;
  let processed = 0;

  this.logger.log(`Starting migration: ${totalRecords} records`);

  for (let offset = 0; offset < totalRecords; offset += batchSize) {
    // Process batch
    await this.processBatch(offset, batchSize);
    processed += batchSize;
    
    // Update progress
    const progress = Math.min(100, Math.floor((processed / totalRecords) * 100));
    await job.progress(progress);
    
    this.logger.log(`Progress: ${progress}% (${processed}/${totalRecords})`);
  }

  return { processed };
}
```

**2. Chunking strategy:**
```typescript
@Process('bulk-email')
async sendBulkEmails(job: Job) {
  const { userIds } = job.data;
  const chunkSize = 100;
  
  // Process in chunks to prevent memory buildup
  for (let i = 0; i < userIds.length; i += chunkSize) {
    const chunk = userIds.slice(i, i + chunkSize);
    
    // Process chunk
    await Promise.all(
      chunk.map(userId => this.sendEmail(userId))
    );
    
    // Allow event loop to process other tasks
    await new Promise(resolve => setImmediate(resolve));
    
    // Report progress
    await job.progress(Math.floor((i / userIds.length) * 100));
  }
}
```

**3. Checkpointing:**
```typescript
interface MigrationJobData {
  startId?: number;
  lastProcessedId?: number;
}

@Process('large-migration')
async migrate(job: Job<MigrationJobData>) {
  const startId = job.data.lastProcessedId || job.data.startId || 0;
  const batchSize = 1000;

  this.logger.log(`Resuming from ID: ${startId}`);

  try {
    let currentId = startId;
    
    while (true) {
      const records = await this.getRecords(currentId, batchSize);
      if (records.length === 0) break;
      
      await this.processRecords(records);
      
      // Save checkpoint
      currentId = records[records.length - 1].id;
      await job.update({
        ...job.data,
        lastProcessedId: currentId,
      });
      
      await job.progress(this.calculateProgress(currentId));
    }
    
    return { completed: true, lastId: currentId };
  } catch (error) {
    // Job can resume from last checkpoint on retry
    this.logger.error(`Failed at ID ${job.data.lastProcessedId}`, error);
    throw error;
  }
}
```

**4. Timeout configuration:**
```typescript
// When adding job
await queue.add('long-task', data, {
  timeout: 1800000, // 30 minutes
  attempts: 3,
  backoff: {
    type: 'exponential',
    delay: 60000,
  },
});

// In processor
@Process({ timeout: 1800000 })
async handleLongTask(job: Job) {
  // Job will fail if exceeds 30 minutes
  await this.performLongOperation();
}
```

**5. Separate queue for long jobs:**
```typescript
// Module setup
@Module({
  imports: [
    BullModule.registerQueue({ name: 'quick-tasks' }),
    BullModule.registerQueue({ name: 'long-tasks' }),
  ],
})

// Quick tasks queue
@Processor('quick-tasks')
export class QuickTasksProcessor {
  @Process({ concurrency: 20 }) // High concurrency
  async handleQuick(job: Job) {
    // Fast operations (< 1 minute)
  }
}

// Long tasks queue (dedicated workers)
@Processor('long-tasks')
export class LongTasksProcessor {
  @Process({ concurrency: 2 }) // Low concurrency
  async handleLong(job: Job) {
    // Long operations (> 10 minutes)
  }
}
```

**6. Status endpoint:**
```typescript
@Controller('jobs')
export class JobsController {
  constructor(@InjectQueue('tasks') private queue: Queue) {}

  @Get(':id/status')
  async getStatus(@Param('id') jobId: string) {
    const job = await this.queue.getJob(jobId);
    
    if (!job) {
      throw new NotFoundException('Job not found');
    }

    const state = await job.getState();
    
    return {
      id: job.id,
      state, // waiting, active, completed, failed
      progress: job.progress(),
      data: job.data,
      result: job.returnvalue,
      failedReason: job.failedReason,
      finishedOn: job.finishedOn,
      processedOn: job.processedOn,
    };
  }
}
```

**7. WebSocket updates:**
```typescript
@WebSocketGateway()
export class JobsGateway {
  constructor(@InjectQueue('tasks') private queue: Queue) {
    this.setupJobEvents();
  }

  private setupJobEvents() {
    this.queue.on('progress', (job, progress) => {
      this.server.emit('job:progress', {
        jobId: job.id,
        progress,
      });
    });

    this.queue.on('completed', (job, result) => {
      this.server.emit('job:completed', {
        jobId: job.id,
        result,
      });
    });
  }
}
```

**Best practices:**

- **Report progress:** Every 5-10% or major milestone
- **Use chunking:** Batch size 100-1000 records
- **Set realistic timeout:** 2-3x expected duration
- **Implement checkpoints:** Save state every N records
- **Separate queues:** Dedicated workers for long jobs
- **Allow event loop:** `setImmediate` between chunks
- **Monitor stalls:** Bull auto-retries stalled jobs
- **Provide status API:** Users can check progress
- **Graceful shutdown:** Finish current chunk before exit
- **Log milestones:** Track progress in logs

</details>

## Best Practices

<details>
<summary><strong>39. Should scheduled jobs contain heavy logic or delegate to services?</strong></summary>

**Answer: Delegate to services (thin orchestrators)**

**Why delegate:**

- **Reusability:** Same logic callable from jobs, API endpoints, CLI commands
- **Testability:** Test services independently without scheduling concerns
- **Separation of concerns:** Jobs handle timing, services handle business logic
- **Maintainability:** Business logic changes don't require job refactoring

**Job responsibilities:**

- Trigger execution at scheduled time
- Gather context/parameters for service call
- Call service method
- Handle job-specific errors (log, retry, alert)
- Report progress for long operations
- Log execution (start, completion, duration)

**Service responsibilities:**

- Implement actual business logic
- Validate inputs
- Perform database operations
- Call external APIs
- Return results or throw errors
- Remain unaware of caller (job vs API)

**❌ Anti-pattern:**

```typescript
@Cron('0 2 * * *')
async dailyReport() {
  // 200 lines of business logic in job method
  const users = await this.usersRepo.find({ active: true });
  const orders = await this.ordersRepo.find({
    createdAt: Between(startDate, endDate),
  });
  
  // Complex calculations
  const metrics = {
    revenue: orders.reduce((sum, o) => sum + o.total, 0),
    avgOrderValue: orders.length > 0 ? revenue / orders.length : 0,
    // ... 50 more lines of calculations
  };
  
  // Format and send
  const pdf = await this.pdfGenerator.create(metrics);
  await this.s3.upload('reports', pdf);
  await this.emailService.send(adminEmail, pdf);
  
  // Hard to test, not reusable, tightly coupled
}
```

**✅ Good pattern:**

```typescript
// Job: thin orchestrator
@Cron('0 2 * * *')
async dailyReport() {
  this.logger.log('Starting daily report generation');
  
  try {
    const date = new Date();
    const report = await this.reportsService.generateDailyReport(date);
    
    this.logger.log(`Report generated: ${report.id}`);
  } catch (error) {
    this.logger.error('Report generation failed', error);
    await this.alertService.send('Daily report failed');
    throw error;
  }
}

// Service: all business logic
@Injectable()
export class ReportsService {
  async generateDailyReport(date: Date) {
    // All logic here - testable, reusable
    const metrics = await this.calculateMetrics(date);
    const pdf = await this.formatReport(metrics);
    const url = await this.storageService.upload(pdf);
    await this.notifyAdmins(url);
    
    return { id: uuid(), url, metrics };
  }
  
  // Can be called from:
  // - Cron job (scheduled)
  // - API endpoint (on-demand: POST /reports/generate)
  // - CLI command (manual trigger)
}
```

**Benefits:**

- **Service can be called anywhere:** Job, API controller, CLI, tests
- **Easy to test:** Mock dependencies, no scheduler needed
- **Clean code:** Job method stays small (<10 lines)
- **Versioning:** Can have v1/v2 services without touching job
- **Dependency injection:** Services can inject other services
- **Multiple jobs:** Different jobs can call same service with different params

**For queue processors:**

```typescript
// Same principle
@Process('send-email')
async sendEmail(job: Job) {
  // Thin orchestrator
  await this.emailService.send(job.data);
  await job.progress(100);
}

// Service contains all logic
@Injectable()
export class EmailService {
  async send(data: EmailData) {
    // Validate, format, send, track - all here
  }
}
```

**Best practices:**

- Jobs: <20 lines, just orchestration
- Services: All business logic, database operations, API calls
- Test services independently with unit tests
- Test jobs with mocked services
- Services return results, jobs handle job-specific concerns (progress, logging)

</details>

<details>
<summary><strong>40. How do you test scheduled jobs?</strong></summary>

**Testing approaches:**

**1. Unit test services (not scheduler):**

```typescript
// reports.service.spec.ts
describe('ReportsService', () => {
  let service: ReportsService;
  let mockUsersRepo: jest.Mocked<UsersRepository>;
  let mockEmailService: jest.Mocked<EmailService>;

  beforeEach(async () => {
    mockUsersRepo = {
      find: jest.fn(),
    } as any;
    
    mockEmailService = {
      send: jest.fn(),
    } as any;

    const module = await Test.createTestingModule({
      providers: [
        ReportsService,
        { provide: UsersRepository, useValue: mockUsersRepo },
        { provide: EmailService, useValue: mockEmailService },
      ],
    }).compile();

    service = module.get<ReportsService>(ReportsService);
  });

  it('should generate daily report', async () => {
    // Mock data
    mockUsersRepo.find.mockResolvedValue([
      { id: 1, email: 'user1@example.com' },
      { id: 2, email: 'user2@example.com' },
    ]);

    // Execute service method directly (no scheduler)
    const result = await service.generateDailyReport(new Date());

    // Assert
    expect(mockUsersRepo.find).toHaveBeenCalledWith({ active: true });
    expect(result.userCount).toBe(2);
  });

  it('should handle errors gracefully', async () => {
    mockUsersRepo.find.mockRejectedValue(new Error('Database error'));

    await expect(service.generateDailyReport(new Date())).rejects.toThrow('Database error');
  });
});
```

**2. Test job methods (manually trigger):**

```typescript
// tasks.service.spec.ts
describe('TasksService', () => {
  let service: TasksService;
  let mockReportsService: jest.Mocked<ReportsService>;
  let mockLogger: jest.Mocked<Logger>;

  beforeEach(async () => {
    mockReportsService = {
      generateDailyReport: jest.fn(),
    } as any;

    mockLogger = {
      log: jest.fn(),
      error: jest.fn(),
    } as any;

    const module = await Test.createTestingModule({
      providers: [
        TasksService,
        { provide: ReportsService, useValue: mockReportsService },
        { provide: Logger, useValue: mockLogger },
      ],
    }).compile();

    service = module.get<TasksService>(TasksService);
  });

  it('should call service when cron job executes', async () => {
    mockReportsService.generateDailyReport.mockResolvedValue({ id: '123' });

    // Call job method directly (bypass scheduler)
    await service.dailyReport();

    // Assert service was called
    expect(mockReportsService.generateDailyReport).toHaveBeenCalledTimes(1);
    expect(mockLogger.log).toHaveBeenCalledWith(
      expect.stringContaining('Report generated')
    );
  });

  it('should log error when service fails', async () => {
    mockReportsService.generateDailyReport.mockRejectedValue(
      new Error('Report failed')
    );

    await expect(service.dailyReport()).rejects.toThrow('Report failed');
    expect(mockLogger.error).toHaveBeenCalledWith(
      expect.stringContaining('Report generation failed'),
      expect.any(Error)
    );
  });
});
```

**3. Test with SchedulerRegistry (manual trigger):**

```typescript
import { SchedulerRegistry } from '@nestjs/schedule';

describe('TasksService with Scheduler', () => {
  let schedulerRegistry: SchedulerRegistry;
  let service: TasksService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      imports: [ScheduleModule.forRoot()],
      providers: [TasksService, ReportsService],
    }).compile();

    schedulerRegistry = module.get<SchedulerRegistry>(SchedulerRegistry);
    service = module.get<TasksService>(TasksService);
  });

  it('should manually trigger cron job', async () => {
    const spy = jest.spyOn(service, 'dailyReport');

    // Get cron job from registry
    const cronJob = schedulerRegistry.getCronJob('dailyReport');
    
    // Manually trigger
    await cronJob.fireOnTick();

    expect(spy).toHaveBeenCalledTimes(1);
  });
});
```

**4. Mock time with Jest:**

```typescript
describe('Scheduled timing', () => {
  beforeEach(() => {
    jest.useFakeTimers();
  });

  afterEach(() => {
    jest.useRealTimers();
  });

  it('should trigger job at scheduled time', async () => {
    const mockCallback = jest.fn();
    
    // Create cron job for every minute
    const cronJob = new CronJob('*/1 * * * *', mockCallback);
    cronJob.start();

    // Advance time by 1 minute
    jest.advanceTimersByTime(60000);

    expect(mockCallback).toHaveBeenCalledTimes(1);

    // Advance another minute
    jest.advanceTimersByTime(60000);

    expect(mockCallback).toHaveBeenCalledTimes(2);
  });
});
```

**5. Test queue processors:**

```typescript
import { Queue } from 'bull';

describe('EmailProcessor', () => {
  let queue: Queue;
  let processor: EmailProcessor;
  let mockEmailService: jest.Mocked<EmailService>;

  beforeEach(async () => {
    mockEmailService = {
      send: jest.fn().mockResolvedValue({ messageId: '123' }),
    } as any;

    const module = await Test.createTestingModule({
      imports: [
        BullModule.registerQueue({ name: 'emails' }),
      ],
      providers: [
        EmailProcessor,
        { provide: EmailService, useValue: mockEmailService },
      ],
    }).compile();

    processor = module.get<EmailProcessor>(EmailProcessor);
    queue = module.get<Queue>('BullQueue_emails');
  });

  it('should process email job successfully', async () => {
    // Add job to queue
    const job = await queue.add('send-email', {
      to: 'user@example.com',
      subject: 'Test',
      body: 'Hello',
    });

    // Wait for processing
    await job.finished();

    // Assert
    expect(mockEmailService.send).toHaveBeenCalledWith({
      to: 'user@example.com',
      subject: 'Test',
      body: 'Hello',
    });
    
    const state = await job.getState();
    expect(state).toBe('completed');
  });

  it('should retry failed jobs', async () => {
    // Mock failure then success
    mockEmailService.send
      .mockRejectedValueOnce(new Error('Network error'))
      .mockResolvedValueOnce({ messageId: '123' });

    const job = await queue.add('send-email', 
      { to: 'user@example.com' },
      { attempts: 3 }
    );

    await job.finished();

    // Should have retried
    expect(mockEmailService.send).toHaveBeenCalledTimes(2);
  });
});
```

**6. Integration tests:**

```typescript
describe('Tasks Integration', () => {
  let app: INestApplication;
  let tasksService: TasksService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      imports: [AppModule],
    })
      .overrideProvider(EmailService)
      .useValue({
        send: jest.fn().mockResolvedValue({ sent: true }),
      })
      .compile();

    app = module.createNestApplication();
    await app.init();

    tasksService = module.get<TasksService>(TasksService);
  });

  afterEach(async () => {
    await app.close();
  });

  it('should execute scheduled job and persist results', async () => {
    // Trigger job
    await tasksService.dailyReport();

    // Verify side effects in database
    const reportRepo = app.get(ReportsRepository);
    const reports = await reportRepo.find({
      where: { date: new Date() },
    });

    expect(reports).toHaveLength(1);
    expect(reports[0].status).toBe('completed');
  });
});
```

**7. Test idempotency:**

```typescript
it('should be idempotent (safe to run twice)', async () => {
  const userId = 'test-user-123';

  // Run job twice
  await service.sendWelcomeEmail(userId);
  await service.sendWelcomeEmail(userId);

  // Should only send one email
  const emailLogs = await emailLogsRepo.find({ userId, type: 'welcome' });
  expect(emailLogs).toHaveLength(1);
});
```

**Best practices:**

- **Test services, not schedulers:** Unit test business logic separately
- **Don't wait for actual schedule:** Too slow for CI, use manual triggers
- **Mock time:** Use `jest.useFakeTimers()` for time-based tests
- **Test error scenarios:** Service failures, retries exhausted, timeouts
- **Test idempotency:** Safe to run multiple times
- **Use test database:** Clean state between tests
- **Mock external services:** APIs, email, S3
- **Integration tests:** Verify end-to-end with real side effects
- **Test retry logic:** Mock failures to verify retry behavior
- **Measure coverage:** Aim for >80% on business logic

</details>

<details>
<summary><strong>41. How do you handle timezone considerations?</strong></summary>

**Best practice: Use UTC everywhere**

**1. Schedule all cron jobs in UTC:**

```typescript
@Injectable()
export class TasksService {
  // ✅ Good: Explicit UTC
  @Cron('0 2 * * *', { 
    name: 'daily-cleanup',
    timeZone: 'UTC' 
  })
  async dailyCleanup() {
    // Runs at 2 AM UTC every day
    // Consistent across all servers regardless of their timezone
  }

  // ❌ Bad: No timezone specified (uses server timezone)
  @Cron('0 2 * * *')
  async inconsistentJob() {
    // Time varies based on server timezone
    // Different behavior in dev vs production
  }
}
```

**2. Region-specific jobs:**

```typescript
@Injectable()
export class ReportsService {
  // US business hours (9 AM Eastern)
  @Cron('0 9 * * *', { 
    name: 'us-morning-report',
    timeZone: 'America/New_York' 
  })
  async usMorningReport() {
    // Always 9 AM ET, adjusts for DST automatically
  }

  // EU business hours (9 AM London)
  @Cron('0 9 * * *', { 
    name: 'eu-morning-report',
    timeZone: 'Europe/London' 
  })
  async euMorningReport() {
    // Always 9 AM GMT/BST, handles DST
  }

  // Asia business hours (9 AM Tokyo)
  @Cron('0 9 * * *', { 
    name: 'asia-morning-report',
    timeZone: 'Asia/Tokyo' 
  })
  async asiaMorningReport() {
    // Always 9 AM JST (no DST in Japan)
  }
}
```

**3. User-specific scheduling (convert to UTC):**

```typescript
@Injectable()
export class UserSchedulingService {
  constructor(
    private schedulerRegistry: SchedulerRegistry,
  ) {}

  async scheduleUserReport(userId: string, localTime: string, timezone: string) {
    // User wants report at 9 AM in their timezone
    // Convert to UTC for storage and scheduling
    
    const cronExpression = this.convertToCron(localTime, timezone);
    
    const job = new CronJob(cronExpression, async () => {
      await this.generateReport(userId);
    }, null, true, 'UTC'); // Always schedule in UTC

    this.schedulerRegistry.addCronJob(`user-report-${userId}`, job);
    job.start();

    // Store in database
    await this.userSchedulesRepo.save({
      userId,
      localTime, // "09:00"
      timezone,  // "America/Los_Angeles"
      cronExpression, // Converted to UTC
      createdAt: new Date(), // Store in UTC
    });
  }

  private convertToCron(localTime: string, timezone: string): string {
    // Convert local time to UTC cron expression
    const [hours, minutes] = localTime.split(':').map(Number);
    
    // Use library like moment-timezone for conversion
    const moment = require('moment-timezone');
    const utcTime = moment.tz({ hours, minutes }, timezone).utc();
    
    return `${utcTime.minutes()} ${utcTime.hours()} * * *`;
  }
}
```

**4. Store UTC, display in user timezone:**

```typescript
@Injectable()
export class JobHistoryService {
  async getJobHistory(userId: string, userTimezone: string) {
    // Fetch from database (stored in UTC)
    const jobs = await this.jobHistoryRepo.find({ userId });

    // Convert to user timezone for display
    return jobs.map(job => ({
      ...job,
      executedAt: this.convertToUserTimezone(job.executedAt, userTimezone),
      scheduledFor: this.convertToUserTimezone(job.scheduledFor, userTimezone),
    }));
  }

  private convertToUserTimezone(utcDate: Date, timezone: string): string {
    const moment = require('moment-timezone');
    return moment(utcDate).tz(timezone).format('YYYY-MM-DD HH:mm:ss z');
  }
}
```

**5. DST handling:**

```typescript
@Injectable()
export class SchedulingService {
  // ✅ Good: UTC doesn't have DST
  @Cron('0 2 * * *', { timeZone: 'UTC' })
  async utcJob() {
    // Always runs at 2 AM UTC
    // No DST issues
  }

  // ⚠️ Careful: Local timezone has DST transitions
  @Cron('0 2 * * *', { timeZone: 'America/New_York' })
  async localJob() {
    // On DST transition days (March/November):
    // - Spring forward: 2 AM doesn't exist (skipped)
    // - Fall back: 2 AM happens twice
    // NestJS/node-cron handles this, but be aware
  }
}
```

**6. Multi-region deployments:**

```typescript
// config/scheduling.config.ts
export const schedulingConfig = {
  // All instances use UTC
  defaultTimezone: 'UTC',
  
  // Region-specific jobs
  regions: {
    us: {
      timezone: 'America/New_York',
      businessHoursStart: '09:00',
      businessHoursEnd: '17:00',
    },
    eu: {
      timezone: 'Europe/London',
      businessHoursStart: '09:00',
      businessHoursEnd: '17:00',
    },
    asia: {
      timezone: 'Asia/Tokyo',
      businessHoursStart: '09:00',
      businessHoursEnd: '17:00',
    },
  },
};

// tasks.service.ts
@Injectable()
export class TasksService {
  @Cron('0 2 * * *', { timeZone: 'UTC' })
  async globalCleanup() {
    // Runs once globally at 2 AM UTC
    // Use distributed lock to prevent duplicate execution
    // See Q35 for lock implementation
  }

  @Cron('0 9 * * 1-5', { timeZone: 'America/New_York' })
  async usBusinessHoursJob() {
    // Runs only during US business hours (9 AM ET, Mon-Fri)
    // Won't execute in other regions
  }
}
```

**7. Testing timezone logic:**

```typescript
describe('Timezone handling', () => {
  it('should schedule job in correct timezone', () => {
    process.env.TZ = 'America/New_York';

    const service = new SchedulingService();
    const scheduledTime = service.getNextExecution('0 9 * * *', 'America/New_York');

    expect(scheduledTime.hours()).toBe(9);
  });

  it('should handle DST transition', () => {
    // Test spring forward (2 AM → 3 AM)
    const springForward = new Date('2025-03-09T02:00:00-05:00');
    // Test fall back (2 AM happens twice)
    const fallBack = new Date('2025-11-02T02:00:00-04:00');

    // Verify job behavior during DST transitions
  });

  it('should convert user timezone to UTC', () => {
    const result = service.convertToCron('09:00', 'America/Los_Angeles');
    
    // 9 AM PST = 5 PM UTC (during standard time)
    // 9 AM PDT = 4 PM UTC (during daylight time)
    expect(result).toMatch(/^0 (16|17) \* \* \*$/);
  });
});
```

**8. Database schema:**

```typescript
@Entity()
export class ScheduledJob {
  @Column({ type: 'timestamp with time zone' })
  createdAt: Date; // Always UTC

  @Column({ type: 'timestamp with time zone' })
  nextExecutionAt: Date; // Always UTC

  @Column({ type: 'timestamp with time zone', nullable: true })
  lastExecutedAt: Date; // Always UTC

  @Column()
  userTimezone: string; // For display: 'America/New_York'

  @Column()
  userLocalTime: string; // For display: '09:00'

  @Column()
  cronExpression: string; // Always in UTC: '0 14 * * *'
}
```

**Common pitfalls:**

- ❌ **No timezone specified:** Uses server timezone (inconsistent across environments)
- ❌ **Hardcoding offsets:** `UTC+5` breaks during DST
- ❌ **Mixing UTC and local:** Store UTC, convert only for display
- ❌ **Ignoring DST:** Jobs can skip or duplicate during transitions
- ❌ **Wrong IANA name:** Use `America/New_York`, not `EST` or `EDT`

**Best practices:**

- **Use UTC internally:** All cron expressions, database timestamps
- **Use IANA timezone names:** `America/New_York`, not abbreviations
- **Explicit timezone option:** Always specify `{ timeZone: 'UTC' }`
- **Convert for display only:** Show users their local time
- **Test DST transitions:** Verify behavior on transition days
- **Document timezone:** Comment which timezone each job uses
- **Consistent across environments:** Same timezone in dev/staging/prod

</details>

<details>
<summary><strong>42. How do you prevent jobs from overlapping (job locking)?</strong></summary>

**Problem:** Long-running job still executing when next schedule triggers

**Solutions:**

**1. Redis flag (simple approach):**

```typescript
import { Injectable } from '@nestjs/common';
import { Cron } from '@nestjs/schedule';
import { InjectRedis } from '@liaoliaots/nestjs-redis';
import Redis from 'ioredis';

@Injectable()
export class TasksService {
  constructor(@InjectRedis() private readonly redis: Redis) {}

  @Cron('*/5 * * * *') // Every 5 minutes
  async processData() {
    const lockKey = 'lock:process-data';
    const lockTTL = 300; // 5 minutes in seconds

    // Try to acquire lock
    const acquired = await this.redis.set(
      lockKey,
      Date.now().toString(),
      'EX',
      lockTTL,
      'NX' // Set if Not eXists
    );

    if (!acquired) {
      this.logger.log('Job already running, skipping this execution');
      return;
    }

    try {
      // Execute job
      await this.performDataProcessing();
    } finally {
      // Release lock
      await this.redis.del(lockKey);
    }
  }

  private async performDataProcessing() {
    // Long-running operation
    this.logger.log('Processing data...');
    await this.heavyOperation();
    this.logger.log('Processing complete');
  }
}
```

**2. Redlock (production-grade):**

```typescript
import Redlock from 'redlock';
import Redis from 'ioredis';

@Injectable()
export class LockingService {
  private redlock: Redlock;

  constructor() {
    const redis = new Redis(process.env.REDIS_URL);
    
    this.redlock = new Redlock([redis], {
      driftFactor: 0.01,
      retryCount: 3,
      retryDelay: 200,
      retryJitter: 200,
    });
  }

  @Cron('0 2 * * *') // Daily at 2 AM
  async dailyReport() {
    const resource = 'locks:daily-report';
    const ttl = 3600000; // 1 hour

    let lock;
    try {
      // Acquire lock with TTL
      lock = await this.redlock.acquire([resource], ttl);
      
      this.logger.log('Lock acquired, generating report');
      await this.generateReport();
      
    } catch (error) {
      // Another instance has lock or Redis error
      if (error instanceof Redlock.LockError) {
        this.logger.log('Could not acquire lock, job already running');
      } else {
        this.logger.error('Lock acquisition failed', error);
      }
    } finally {
      // Always release lock
      if (lock) {
        try {
          await lock.release();
          this.logger.log('Lock released');
        } catch (error) {
          this.logger.error('Lock release failed', error);
        }
      }
    }
  }
}
```

**3. Database flag (no Redis):**

```typescript
@Injectable()
export class DatabaseLockingService {
  constructor(private jobLockRepo: JobLockRepository) {}

  @Cron('0 * * * *') // Hourly
  async hourlySync() {
    const jobName = 'hourly-sync';

    // Atomic update: set is_running = true WHERE is_running = false
    const result = await this.jobLockRepo.query(`
      UPDATE job_locks 
      SET 
        is_running = true,
        started_at = NOW()
      WHERE 
        job_name = $1 
        AND is_running = false
      RETURNING id
    `, [jobName]);

    if (result.length === 0) {
      this.logger.log('Job already running (database lock held)');
      return;
    }

    try {
      await this.performSync();
    } finally {
      // Release lock
      await this.jobLockRepo.update(
        { jobName },
        { 
          isRunning: false,
          completedAt: new Date(),
        }
      );
    }
  }
}

// Entity
@Entity('job_locks')
export class JobLock {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  jobName: string;

  @Column({ default: false })
  isRunning: boolean;

  @Column({ type: 'timestamp', nullable: true })
  startedAt: Date;

  @Column({ type: 'timestamp', nullable: true })
  completedAt: Date;
}
```

**4. Bull queue (concurrency control):**

```typescript
// Instead of direct cron execution, use queue
@Injectable()
export class SchedulingService {
  constructor(
    @InjectQueue('reports') private reportsQueue: Queue,
  ) {}

  @Cron('0 2 * * *')
  async scheduleReport() {
    // Cron just adds job to queue
    await this.reportsQueue.add('daily-report', {}, {
      jobId: `daily-report-${new Date().toISOString().split('T')[0]}`, // Unique per day
      removeOnComplete: true,
    });
    
    this.logger.log('Report job queued');
  }
}

// Processor with concurrency: 1 (sequential)
@Processor('reports')
export class ReportsProcessor {
  @Process({ name: 'daily-report', concurrency: 1 })
  async generateReport(job: Job) {
    // Bull ensures only one job processes at a time
    // If job still running, next one waits in queue
    await this.performReportGeneration();
    return { success: true };
  }
}
```

**5. Lock with timeout extension:**

```typescript
@Injectable()
export class ExtendableLockService {
  private redlock: Redlock;

  @Cron('0 3 * * *')
  async veryLongJob() {
    const resource = 'locks:long-job';
    let ttl = 1800000; // 30 minutes initial

    let lock = await this.redlock.acquire([resource], ttl);

    try {
      const chunks = await this.getDataChunks();
      
      for (let i = 0; i < chunks.length; i++) {
        // Check if approaching TTL expiration
        const timeElapsed = Date.now() - lock.expiration + ttl;
        if (timeElapsed > ttl * 0.8) { // 80% of TTL
          // Extend lock by another 30 minutes
          lock = await lock.extend(1800000);
          this.logger.log('Lock extended');
        }

        await this.processChunk(chunks[i]);
      }
    } finally {
      await lock.release();
    }
  }
}
```

**6. Skip vs Wait strategy:**

```typescript
@Injectable()
export class StrategyService {
  // Skip strategy: Job can be skipped
  @Cron('*/1 * * * *') // Every minute
  async healthCheck() {
    const acquired = await this.redis.set('lock:health', '1', 'EX', 60, 'NX');
    
    if (!acquired) {
      // Skip this execution, next one will run
      return;
    }

    try {
      await this.checkHealth();
    } finally {
      await this.redis.del('lock:health');
    }
  }

  // Wait strategy: All executions must happen
  @Cron('0 * * * *') // Hourly
  async criticalSync() {
    // Use queue instead of skipping
    await this.syncQueue.add('sync', {}, {
      jobId: `sync-${Date.now()}`,
      attempts: 5,
    });
    
    // Queue ensures all jobs execute eventually
    // If one is running, others wait in queue
  }
}
```

**7. Monitor lock duration:**

```typescript
@Injectable()
export class MonitoredLockService {
  @Cron('0 2 * * *')
  async monitoredJob() {
    const lockKey = 'lock:monitored-job';
    const expectedDuration = 600000; // 10 minutes
    const lockTTL = 1800; // 30 minutes (3x expected)

    const startTime = Date.now();
    const acquired = await this.redis.set(lockKey, startTime.toString(), 'EX', lockTTL, 'NX');

    if (!acquired) {
      // Check how long current lock has been held
      const lockValue = await this.redis.get(lockKey);
      const lockAge = Date.now() - parseInt(lockValue);
      
      if (lockAge > expectedDuration * 2) {
        // Lock held too long, alert
        await this.alertService.send({
          severity: 'warning',
          message: `Job lock held for ${lockAge}ms (expected ${expectedDuration}ms)`,
        });
      }
      
      return;
    }

    try {
      await this.performJob();
      
      const duration = Date.now() - startTime;
      if (duration > expectedDuration) {
        this.logger.warn(`Job took ${duration}ms (expected ${expectedDuration}ms)`);
      }
    } finally {
      await this.redis.del(lockKey);
    }
  }
}
```

**Comparison:**

| Approach | Pros | Cons | Use When |
|----------|------|------|----------|
| Redis flag | Simple, fast | No quorum, SPOF | Small deployments |
| Redlock | Production-ready, tested | Complex setup | Production clusters |
| Database flag | No extra infra | Slower, DB bottleneck | Redis not available |
| Bull queue | Built-in, monitoring | Requires Bull setup | Heavy/complex jobs |

**Best practices:**

- **Set appropriate TTL:** 2-3x expected job duration
- **Always use finally:** Release lock even on errors
- **Monitor lock age:** Alert if held too long
- **Idempotent operations:** Safe even if overlap occurs
- **Log lock acquisition:** Track which instance has lock
- **Test lock expiration:** Verify TTL releases lock
- **Use descriptive keys:** `locks:job-name` not just `lock`
- **Handle Redis failures:** Gracefully skip execution if Redis down

</details>


<details>
<summary><strong>43. Should you run scheduled jobs in production on all instances?</strong></summary>

**Answer: It depends on job type**

**❌ Don't run on all instances (causes duplicates):**

- Data cleanup
- Report generation  
- External API sync
- Email notifications
- Database migrations
- Billing operations

**✅ Do run on all instances (instance-specific):**

- Health checks (per instance)
- Local cache refresh
- Metrics collection (per instance)
- Log rotation (instance logs)

**Solutions:**

**1. Dedicated job instance (simple):**

```typescript
// app.module.ts
@Module({
  imports: [
    // Only register scheduler on job instance
    ...(process.env.RUN_JOBS === 'true' ? [ScheduleModule.forRoot()] : []),
  ],
})
export class AppModule {}

// docker-compose.yml
services:
  api-1:
    environment:
      - RUN_JOBS=false  # No jobs
  
  api-2:
    environment:
      - RUN_JOBS=false  # No jobs
  
  jobs:
    environment:
      - RUN_JOBS=true   # Only this instance runs jobs
```

**Pros:**
- Simple configuration
- Clear separation

**Cons:**
- Single point of failure
- No automatic failover

**2. Distributed locks (resilient):**

```typescript
import Redlock from 'redlock';

@Injectable()
export class TasksService {
  private redlock: Redlock;

  constructor() {
    const redis = new Redis(process.env.REDIS_URL);
    this.redlock = new Redlock([redis]);
  }

  // Runs on ALL instances, but only one executes
  @Cron('0 2 * * *')
  async dailyCleanup() {
    let lock;
    try {
      lock = await this.redlock.acquire(['locks:daily-cleanup'], 3600000);
      
      // Only this instance has lock
      this.logger.log(`Instance ${process.env.INSTANCE_ID} acquired lock`);
      await this.performCleanup();
      
    } catch (error) {
      // Another instance has lock, skip
      this.logger.log('Lock held by another instance, skipping');
    } finally {
      if (lock) await lock.release();
    }
  }
}
```

**Pros:**
- Automatic failover (if one instance crashes, another acquires lock)
- All instances have job code

**Cons:**
- Requires Redis
- More complex

**3. Leader election:**

```typescript
import { LeaderElection } from '@nestjs-modules/ioredis-leader';

@Injectable()
export class TasksService {
  private isLeader = false;

  constructor(private leaderElection: LeaderElection) {
    this.setupLeaderElection();
  }

  private async setupLeaderElection() {
    this.leaderElection.on('elected', () => {
      this.isLeader = true;
      this.logger.log('This instance is now the leader');
    });

    this.leaderElection.on('demoted', () => {
      this.isLeader = false;
      this.logger.log('This instance is no longer the leader');
    });
  }

  @Cron('0 * * * *')
  async hourlyTask() {
    if (!this.isLeader) {
      this.logger.log('Not the leader, skipping');
      return;
    }

    // Only leader executes
    await this.performTask();
  }
}
```

**Pros:**
- Automatic leader re-election
- Clean abstraction

**Cons:**
- Additional library
- More complex setup

**4. Cron + Queue hybrid (recommended):**

```typescript
@Injectable()
export class SchedulingService {
  constructor(
    @InjectQueue('tasks') private tasksQueue: Queue,
    @InjectRedis() private redis: Redis,
  ) {}

  // ONE instance adds job to queue (use lock)
  @Cron('0 2 * * *')
  async scheduleCleanup() {
    const lockKey = 'lock:schedule-cleanup';
    const acquired = await this.redis.set(lockKey, '1', 'EX', 60, 'NX');
    
    if (acquired) {
      // This instance won the race
      await this.tasksQueue.add('daily-cleanup', {}, {
        jobId: `cleanup-${new Date().toISOString().split('T')[0]}`,
        attempts: 3,
      });
      
      this.logger.log('Cleanup job queued');
    }
  }
}

// Processor on worker instances (Bull handles coordination)
@Processor('tasks')
export class TasksProcessor {
  @Process('daily-cleanup')
  async handleCleanup(job: Job) {
    // Bull ensures single execution
    await this.performCleanup();
    return { success: true };
  }
}
```

**Pros:**
- Separates scheduling from execution
- Built-in retry, monitoring
- Scalable workers

**Cons:**
- Requires Bull setup

**5. External scheduler (enterprise):**

```typescript
// No cron in app code
// AWS EventBridge → Lambda → API endpoint

// api/jobs.controller.ts
@Controller('jobs')
export class JobsController {
  @Post('daily-cleanup')
  @Header('X-API-Key', process.env.JOBS_API_KEY)
  async triggerCleanup() {
    // Called by external scheduler (AWS EventBridge, Cloud Scheduler)
    await this.cleanupService.performCleanup();
    return { success: true };
  }
}

// AWS EventBridge rule:
// Schedule: cron(0 2 * * ? *)
// Target: API Gateway → POST /jobs/daily-cleanup
```

**Pros:**
- No job code in app
- Managed service
- Scales independently

**Cons:**
- Cloud vendor lock-in
- External dependency

**Instance-specific jobs (run everywhere):**

```typescript
@Injectable()
export class InstanceJobsService {
  // ✅ Run on ALL instances (no lock needed)
  
  @Cron('*/1 * * * *') // Every minute
  async healthCheck() {
    // Each instance checks its own health
    const health = await this.checkInstanceHealth();
    await this.metricsService.recordHealth(
      process.env.INSTANCE_ID,
      health
    );
  }

  @Cron('0 * * * *') // Hourly
  async refreshLocalCache() {
    // Each instance refreshes its own in-memory cache
    await this.cacheService.refresh();
  }

  @Cron('0 0 * * *') // Daily
  async rotateLocalLogs() {
    // Each instance rotates its own log files
    await this.logService.rotate();
  }
}
```

**Decision matrix:**

| Requirement | Solution |
|-------------|----------|
| Simple, can afford SPOF | Dedicated instance |
| Need failover, have Redis | Distributed locks |
| High availability | Leader election |
| Heavy jobs, need monitoring | Cron + Queue |
| Enterprise, managed | External scheduler |
| Instance-specific | Run on all (no lock) |

**Best practices:**

- **Document which jobs run where:** Comment in code
- **Monitor job execution:** Alert if jobs haven't run
- **Health checks:** Ensure job instance is running
- **Test failover:** Kill job instance, verify another takes over
- **Use environment variables:** `RUN_JOBS=true` for configuration
- **Separate concerns:** Scheduling vs execution

</details>

<details>
<summary><strong>44. How do you use distributed locks to ensure single execution in multi-instance setups?</strong></summary>

**Answer: Use Redlock algorithm for production-grade distributed locking**

**Setup:**

```bash
npm install redlock ioredis
npm install --save-dev @types/redlock
```

**1. Create lock service:**

```typescript
// locks/locks.service.ts
import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
import Redlock from 'redlock';
import Redis from 'ioredis';

@Injectable()
export class LocksService implements OnModuleInit, OnModuleDestroy {
  private redlock: Redlock;
  private redis: Redis;

  async onModuleInit() {
    // Connect to Redis (or Redis cluster)
    this.redis = new Redis({
      host: process.env.REDIS_HOST || 'localhost',
      port: parseInt(process.env.REDIS_PORT) || 6379,
      retryStrategy: (times) => Math.min(times * 50, 2000),
    });

    // Create Redlock instance
    this.redlock = new Redlock(
      [this.redis], // For HA, pass multiple Redis clients
      {
        driftFactor: 0.01,
        retryCount: 0, // Don't retry (for scheduled jobs, skip if locked)
        retryDelay: 200,
        retryJitter: 200,
      }
    );

    // Listen to events
    this.redlock.on('error', (error) => {
      console.error('Redlock error:', error);
    });
  }

  async onModuleDestroy() {
    await this.redis.quit();
  }

  /**
   * Acquire distributed lock
   * @param resource Lock key (e.g., 'locks:daily-cleanup')
   * @param ttl Time-to-live in milliseconds (lock auto-expires)
   * @returns Lock object or null if unavailable
   */
  async acquire(resource: string, ttl: number) {
    try {
      return await this.redlock.acquire([resource], ttl);
    } catch (error) {
      // Lock is held by another instance
      if (error.name === 'LockError') {
        return null;
      }
      throw error; // Rethrow if it's not a lock error
    }
  }
}
```

**2. Use lock in scheduled job:**

```typescript
// tasks/tasks.service.ts
import { Injectable, Logger } from '@nestjs/common';
import { Cron } from '@nestjs/schedule';
import { LocksService } from '../locks/locks.service';

@Injectable()
export class TasksService {
  private readonly logger = new Logger(TasksService.name);

  constructor(private locksService: LocksService) {}

  // Runs on ALL instances, but only one executes
  @Cron('0 2 * * *') // Daily at 2 AM
  async dailyCleanup() {
    const lockKey = 'locks:daily-cleanup';
    const lockTTL = 60 * 60 * 1000; // 1 hour (job takes ~30 min)
    
    let lock = null;
    
    try {
      // Try to acquire lock
      lock = await this.locksService.acquire(lockKey, lockTTL);
      
      if (!lock) {
        // Another instance is running this job
        this.logger.log('Lock held by another instance, skipping');
        return;
      }

      // This instance acquired the lock
      this.logger.log(`Acquired lock, instance ${process.env.INSTANCE_ID} executing`);
      
      // Execute job
      await this.performCleanup();
      
      this.logger.log('Cleanup completed successfully');
      
    } catch (error) {
      this.logger.error('Cleanup failed', error);
      throw error;
      
    } finally {
      // Always release lock
      if (lock) {
        try {
          await lock.release();
          this.logger.log('Lock released');
        } catch (releaseError) {
          this.logger.error('Failed to release lock', releaseError);
        }
      }
    }
  }

  private async performCleanup() {
    // Actual cleanup logic
    await this.deleteOldRecords();
    await this.archiveInactiveUsers();
    // ... more cleanup tasks
  }
}
```

**3. Lock TTL calculation:**

```typescript
// Set TTL to 2-3x expected job duration
const estimatedDuration = 30 * 60 * 1000; // 30 minutes
const lockTTL = estimatedDuration * 3;     // 90 minutes (buffer)

// Why? Prevents deadlock if instance crashes:
// - Job starts at 2:00 AM
// - Instance crashes at 2:10 AM
// - Lock auto-expires at 3:30 AM (TTL)
// - Next day at 2:00 AM, job can run again

// Don't set too long (blocks next execution if needed)
// Don't set too short (lock expires while job still running)
```

**4. Lock extension for long jobs:**

```typescript
@Cron('0 3 * * *')
async longRunningJob() {
  const lockKey = 'locks:long-job';
  const lockTTL = 10 * 60 * 1000; // 10 minutes initial
  
  let lock = await this.locksService.acquire(lockKey, lockTTL);
  if (!lock) return;

  try {
    for (let i = 0; i < 100; i++) {
      await this.processBatch(i);
      
      // Extend lock every 5 minutes
      if (i % 10 === 0 && lock) {
        lock = await lock.extend(10 * 60 * 1000);
        this.logger.log('Lock extended');
      }
    }
  } finally {
    if (lock) await lock.release();
  }
}
```

**5. Simple Redis flag alternative (no library):**

```typescript
import { InjectRedis } from '@nestjs-modules/ioredis';
import Redis from 'ioredis';

@Injectable()
export class SimpleLocksService {
  constructor(@InjectRedis() private redis: Redis) {}

  @Cron('0 2 * * *')
  async dailyCleanup() {
    const lockKey = 'lock:daily-cleanup';
    const lockTTL = 3600; // 1 hour in seconds
    
    // SET NX (set if not exists) EX (expiry)
    const acquired = await this.redis.set(
      lockKey,
      process.env.INSTANCE_ID || '1',
      'EX', lockTTL,
      'NX'
    );

    if (!acquired) {
      // Another instance has lock
      this.logger.log('Lock held, skipping');
      return;
    }

    try {
      // This instance acquired lock
      await this.performCleanup();
      
    } finally {
      // Release lock
      await this.redis.del(lockKey);
    }
  }
}
```

**6. Database lock alternative:**

```typescript
// migrations/xxx-create-job-locks.ts
export class CreateJobLocks {
  async up(queryRunner: QueryRunner) {
    await queryRunner.query(`
      CREATE TABLE job_locks (
        job_name VARCHAR(255) PRIMARY KEY,
        is_running BOOLEAN NOT NULL DEFAULT FALSE,
        locked_at TIMESTAMP,
        locked_by VARCHAR(255),
        updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
      )
    `);
  }
}

// tasks.service.ts
@Cron('0 2 * * *')
async dailyCleanup() {
  const lockAcquired = await this.dataSource.transaction(async (manager) => {
    // Atomic update
    const result = await manager.query(`
      UPDATE job_locks
      SET is_running = TRUE,
          locked_at = NOW(),
          locked_by = $1
      WHERE job_name = $2 AND is_running = FALSE
      RETURNING job_name
    `, [process.env.INSTANCE_ID, 'daily-cleanup']);

    return result.length > 0;
  });

  if (!lockAcquired) {
    this.logger.log('Job already running on another instance');
    return;
  }

  try {
    await this.performCleanup();
    
  } finally {
    // Release lock
    await this.dataSource.query(`
      UPDATE job_locks
      SET is_running = FALSE,
          locked_at = NULL,
          locked_by = NULL
      WHERE job_name = $1
    `, ['daily-cleanup']);
  }
}
```

**7. Monitor lock duration:**

```typescript
@Cron('0 2 * * *')
async dailyCleanup() {
  const startTime = Date.now();
  let lock = null;

  try {
    lock = await this.locksService.acquire('locks:cleanup', 60 * 60 * 1000);
    if (!lock) return;

    await this.performCleanup();
    
    // Measure duration
    const duration = Date.now() - startTime;
    await this.metricsService.recordJobDuration('daily-cleanup', duration);
    
    // Alert if exceeding threshold
    if (duration > 30 * 60 * 1000) { // > 30 minutes
      this.logger.warn(`Job took ${duration}ms, approaching lock TTL`);
      await this.alertService.notify('Cleanup job running slow');
    }
    
  } finally {
    if (lock) await lock.release();
  }
}
```

**Comparison:**

| Approach | Pros | Cons | Use Case |
|----------|------|------|----------|
| **Redlock** | Production-grade, atomic, HA-ready | Requires Redis | Enterprise apps |
| **Simple Redis** | Easy, no library | No lock extension | Most apps |
| **Database** | No Redis needed, transactional | DB overhead | Small scale |
| **Bull Queue** | Built-in, retry, monitoring | Queue complexity | Heavy jobs |

**Best practices:**

- **Set TTL to 2-3x job duration:** Prevents deadlock if instance crashes
- **Always release in finally block:** Avoid lock leaks
- **Log lock events:** Acquisition, release, failures (for debugging)
- **Monitor lock metrics:** Wait times, failures, duration
- **Test multi-instance locally:** Docker Compose with 3 instances
- **Handle lock failures gracefully:** Log and skip (don't throw error)
- **Use descriptive lock keys:** `locks:daily-cleanup` not just `cleanup`
- **Consider lock extension:** For jobs > 10 minutes
- **Set up alerts:** If job hasn't run in expected window
- **Prefer Bull queues for heavy jobs:** Better coordination and monitoring

**Testing:**

```typescript
describe('TasksService with locks', () => {
  let service: TasksService;
  let locksService: LocksService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        TasksService,
        {
          provide: LocksService,
          useValue: {
            acquire: jest.fn(),
          },
        },
      ],
    }).compile();

    service = module.get(TasksService);
    locksService = module.get(LocksService);
  });

  it('should execute job when lock acquired', async () => {
    const mockLock = { release: jest.fn() };
    jest.spyOn(locksService, 'acquire').mockResolvedValue(mockLock);

    await service.dailyCleanup();

    expect(locksService.acquire).toHaveBeenCalledWith('locks:daily-cleanup', expect.any(Number));
    expect(mockLock.release).toHaveBeenCalled();
  });

  it('should skip job when lock unavailable', async () => {
    jest.spyOn(locksService, 'acquire').mockResolvedValue(null);

    await service.dailyCleanup();

    // Job skipped, no error thrown
    expect(locksService.acquire).toHaveBeenCalled();
  });
});
```

</details>

<details>
<summary><strong>45. What are the performance implications of having many scheduled jobs?</strong></summary>

**Answer: Multiple performance impacts - memory, CPU, connections, event loop**

**1. Memory overhead:**

```typescript
// Each scheduled job stores metadata in memory
@Cron('0 * * * *')  // Stores: cron expression, callback, context
async hourlyJob() {}

@Interval(5000)     // Stores: interval ID, callback, context
async periodicJob() {}

// Impact:
// - 10 jobs: ~100 KB (negligible)
// - 100 jobs: ~1-2 MB (acceptable)
// - 1000 jobs: ~10-20 MB (noticeable)
// - 10,000 jobs: ~100-200 MB (significant)

// Monitor with:
process.memoryUsage().heapUsed / 1024 / 1024; // MB
```

**2. CPU overhead:**

```typescript
// Cron expressions evaluated every second
@Cron('0 2 * * *')  // Checks every second: "Is it 2 AM?"
async dailyJob() {}

// With 100 jobs:
// - 100 expression evaluations per second
// - Usually fast (<1ms each) but adds up
// - ~100ms CPU time per second at 100 jobs

// Interval jobs fire callbacks:
@Interval(1000)     // Callback fires every second
async everySecond() {
  // Uses CPU every execution
}

// Impact grows with:
// - Number of jobs
// - Frequency (more frequent = more CPU)
// - Job duration (longer jobs = more concurrent executions)
```

**3. Execution overlap (major issue):**

```typescript
// Problem: Job takes 2 minutes, but runs every 1 minute
@Cron('*/1 * * * *')  // Every minute
async slowJob() {
  await this.processLargeDataset(); // Takes 2 minutes
  // Next execution starts before this finishes!
  // Result: 2 instances running concurrently (2x memory/CPU)
}

// Solution: Skip if already running
@Injectable()
export class TasksService {
  private isRunning = false;

  @Cron('*/1 * * * *')
  async slowJob() {
    if (this.isRunning) {
      this.logger.warn('Job still running, skipping this execution');
      return;
    }

    this.isRunning = true;
    try {
      await this.processLargeDataset();
    } finally {
      this.isRunning = false; // Always reset
    }
  }
}

// Advanced: Track per-job state
private runningJobs = new Map<string, boolean>();

@Cron('*/1 * * * *')
async job1() {
  return this.runIfNotRunning('job1', async () => {
    await this.process();
  });
}

private async runIfNotRunning(jobName: string, fn: () => Promise<void>) {
  if (this.runningJobs.get(jobName)) {
    this.logger.warn(`${jobName} still running, skipping`);
    return;
  }

  this.runningJobs.set(jobName, true);
  try {
    await fn();
  } finally {
    this.runningJobs.set(jobName, false);
  }
}
```

**4. Event loop blocking:**

```typescript
// CPU-intensive job blocks event loop
@Cron('0 * * * *')
async heavyComputation() {
  // Synchronous loop blocks Node.js event loop
  for (let i = 0; i < 1000000000; i++) {
    // Complex calculation
  }
  // While running: API requests hang, other jobs delayed
}

// Solution: Use worker threads
import { Worker } from 'worker_threads';

@Cron('0 * * * *')
async heavyComputation() {
  const worker = new Worker('./workers/computation.worker.js');
  
  worker.on('message', (result) => {
    this.logger.log('Computation done:', result);
  });
  
  worker.on('error', (error) => {
    this.logger.error('Worker error:', error);
  });
  
  // Main thread remains responsive
}

// Or: Use Bull queue (better)
@Cron('0 * * * *')
async scheduleHeavyJob() {
  // Just add to queue (non-blocking)
  await this.queue.add('heavy-computation', {});
}
```

**5. Database connection exhaustion:**

```typescript
// Problem: 50 jobs running concurrently, each queries DB
@Cron('*/1 * * * *')
async job1() { await this.db.query(...); } // Connection 1

@Cron('*/1 * * * *')
async job2() { await this.db.query(...); } // Connection 2

// ... 48 more jobs

// If pool size = 20:
// ERROR: "Connection pool exhausted"

// Solution 1: Increase pool size
// typeorm config
TypeOrmModule.forRoot({
  type: 'postgres',
  extra: {
    max: 100, // Increase from default 10
    connectionTimeoutMillis: 5000,
  },
});

// Solution 2: Stagger jobs
@Cron('0 * * * *')   // Job 1 at :00
async job1() {}

@Cron('5 * * * *')   // Job 2 at :05
async job2() {}

@Cron('10 * * * *')  // Job 3 at :10
async job3() {}

// Solution 3: Use Bull queue (controls concurrency)
@Processor('tasks')
export class TasksProcessor {
  @Process({ name: 'db-query', concurrency: 5 }) // Max 5 at once
  async handleQuery(job: Job) {
    await this.db.query(...);
  }
}
```

**6. Mitigation strategies:**

```typescript
// Strategy 1: Stagger schedules (avoid peaks)
// ❌ Bad: All at midnight (50 jobs fire simultaneously)
@Cron('0 0 * * *') async job1() {}
@Cron('0 0 * * *') async job2() {}
// ... 48 more at 00:00

// ✅ Good: Spread throughout hour
@Cron('0 0 * * *')  async job1() {}  // 00:00
@Cron('5 0 * * *')  async job2() {}  // 00:05
@Cron('10 0 * * *') async job3() {}  // 00:10
@Cron('15 0 * * *') async job4() {}  // 00:15

// Strategy 2: Use Bull for >100 jobs
@Injectable()
export class SchedulingService {
  // ONE cron job schedules all others
  @Cron('0 0 * * *')
  async scheduleDailyJobs() {
    // Add 100 jobs to queue (staggered automatically)
    for (const task of this.dailyTasks) {
      await this.queue.add(task.name, task.data, {
        delay: task.delay, // Stagger: 0ms, 5000ms, 10000ms, ...
      });
    }
  }
}

// Workers process with controlled concurrency
@Processor('tasks')
export class TasksProcessor {
  @Process({ concurrency: 10 }) // Only 10 jobs at once
  async process(job: Job) {
    // Actual job logic
  }
}

// Strategy 3: Monitor metrics
@Injectable()
export class MonitoredTasksService {
  @Cron('0 * * * *')
  async hourlyJob() {
    const startTime = Date.now();
    const startMem = process.memoryUsage().heapUsed;

    try {
      await this.doWork();
      
      const duration = Date.now() - startTime;
      const memUsed = process.memoryUsage().heapUsed - startMem;
      
      // Record metrics
      await this.metrics.record('hourly_job', {
        duration,
        memory: memUsed / 1024 / 1024, // MB
      });
      
      // Alert if threshold exceeded
      if (duration > 30000) { // > 30 seconds
        this.logger.warn(`Job took ${duration}ms`);
      }
      
    } catch (error) {
      this.logger.error('Job failed', error);
      await this.alerts.notify('hourly_job_failed');
    }
  }
}
```

**7. When to use alternatives:**

```typescript
// <50 jobs: @Cron is fine
@Cron('0 * * * *') async job1() {}
@Cron('0 2 * * *') async job2() {}

// 50-100 jobs: Still OK, but monitor
// - Stagger schedules
// - Implement skip-if-running
// - Monitor metrics

// >100 jobs: Use Bull
// - Better coordination
// - Built-in retry/monitoring
// - Controlled concurrency
await this.queue.add('task', {});

// >1000 jobs: External scheduler
// - AWS EventBridge → Lambda → API
// - Cloud Scheduler → Cloud Run → API
// - Separate job service
// Removes load from main app
```

**Benchmarks:**

| Job Count | Memory | CPU | DB Connections | Recommendation |
|-----------|--------|-----|----------------|----------------|
| 1-50 | <1 MB | <5% | <10 | ✅ Use @Cron |
| 50-100 | 1-2 MB | 5-10% | 10-20 | ⚠️ @Cron + monitoring |
| 100-500 | 5-20 MB | 10-25% | 20-50 | ⚠️ Consider Bull |
| 500-1000 | 20-50 MB | 25-50% | 50-100 | ❌ Use Bull |
| >1000 | >50 MB | >50% | >100 | ❌ External scheduler |

**Warning signs:**

```bash
# High memory usage
> process.memoryUsage()
{ heapUsed: 200000000 }  # 200 MB (unusual for just jobs)

# Slow API responses
> curl /api/health
# Takes 5+ seconds (event loop blocked by jobs)

# Connection pool errors
ERROR: Connection pool exhausted
All connections are in use

# Jobs missing schedules
# Expected to run at 2:00 AM, but ran at 2:05 AM
# (event loop too busy)

# CPU spikes
# CPU jumps to 100% every hour when jobs fire
```

**Best practices:**

- **Start small:** Test with 5 jobs in dev, gradually increase to production load
- **Use Bull for >100 jobs:** Better coordination and monitoring
- **Stagger schedules:** Avoid all jobs at midnight/top of hour
- **Implement skip-if-running:** Prevents overlap
- **Monitor metrics:** Track duration, memory, CPU, DB connections
- **Use worker threads for CPU-intensive:** Keeps main thread responsive
- **Increase DB pool size:** Match concurrent job count
- **Set up alerts:** If jobs don't run or take too long
- **Test realistic load:** Don't just test with 5 jobs
- **Consider external schedulers:** For >1000 jobs

</details>

