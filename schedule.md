# Lightpack Scheduler

Lightpack provides support for scheduling jobs and commands in your applications. The scheduler lets you automate recurring tasks—like sending emails, cleaning up data, or running custom scripts—using familiar cron expressions, a fluent API, and full testability.

**Scheduling is essential for:**

- Automated maintenance (cleanup, backups, pruning)
- Sending periodic notifications or emails
- Running reports or analytics
- Integrating with external services at intervals
- Any recurring or timed workflow

At the heart of the scheduler is the **cron expression**—a 5-part string describing *when* a task should run:

```
* * * * *
| | | | |
| | | | +----- Day of week (0-6, Sunday=0)
| | | +------- Month (1-12)
| | +--------- Day of month (1-31)
| +----------- Hour (0-23)
+------------- Minute (0-59)
```

- `* * * * *` = every minute
- `0 * * * *` = every hour, at minute 0
- `0 0 * * *` = every day at midnight

You can use ranges (`1-5`), steps (`*/15`), lists (`1,15,30`), and wildcards (`*`).

---

## Running the Scheduler

To actually execute your scheduled jobs and commands, you need to run the scheduler at regular intervals. Lightpack provides a built-in console command for this purpose:

```sh
php console schedule:events
```

This command is handled by the `ScheduleEvents` class and will execute all jobs and commands that are due at the current time.

**Production Setup:**

In the production environment, you should set up a system **cron** job to call this command every minute:

```cron
* * * * * cd /path/to/lightpack && php console schedule:events >> /dev/null 2>&1
```

This ensures your scheduled events are checked and executed on time.

---

## How It Works

The scheduler works in three steps:

1. **Define schedules** in `boot/schedules.php` - You register jobs and commands with their timing
2. **Cron checks every minute** - System cron runs `schedule:events` command every minute
3. **Scheduler executes due events** - Only events that are due at that moment are executed

### Job vs Command

**Jobs** are dispatched to your queue system (async):
```php
schedule()->job(SendEmailJob::class)->daily();
// Dispatches to queue → processed by worker → non-blocking
```

**Commands** run immediately (sync):
```php
schedule()->command(BackupCommand::class)->daily();
// Executes directly → blocks until complete → returns output
```

**Use jobs for:** Email sending, API calls, long-running tasks  
**Use commands for:** Database cleanup, file operations, quick scripts

---

## Registering Schedules

Create or edit `boot/schedules.php` in your project root:

```php
<?php

// boot/schedules.php

// Schedule a job to run every day at midnight
schedule()->job(SendDailyReport::class)->daily();

// Schedule a command to run every hour
schedule()->command(CleanupCommand::class)->hourly();

// Schedule a command with arguments
schedule()->command(BackupCommand::class, ['--force' => true])->daily()->at('02:00');

// Schedule a job every 15 minutes
schedule()->job(SyncDataJob::class)->everyMinutes(15);
```

**Complete Setup Example:**

```php
<?php

// boot/schedules.php

use App\Jobs\SendDailyReport;
use App\Jobs\CleanupTempFiles;
use App\Jobs\SyncExternalData;
use App\Console\Commands\BackupDatabase;
use App\Console\Commands\GenerateReports;

// Send daily report at 9 AM
schedule()->job(SendDailyReport::class)
    ->daily()
    ->at('09:00');

// Cleanup temp files every hour
schedule()->job(CleanupTempFiles::class)
    ->hourly();

// Sync data every 30 minutes
schedule()->job(SyncExternalData::class)
    ->everyMinutes(30);

// Backup database every day at 2 AM
schedule()->command(BackupDatabase::class)
    ->daily()
    ->at('02:00');

// Generate weekly reports on Mondays at 8 AM
schedule()->command(GenerateReports::class, ['--type' => 'weekly'])
    ->mondays()
    ->at('08:00');
```

## Setting When Events Run

**Using Cron Expressions Directly**

```php
schedule()->job(BackupJob::class)->cron('0 3 * * *'); // Every day at 3:00 AM
```

**Using Fluent Helpers**

- `daily()` — every day at midnight
- `hourly()` — every hour at minute 0
- `weekly()` — every Sunday at midnight
- `monthly()` — first day of month at midnight
- `everyMinutes(15)` — every 15 minutes
- `mondays()`, `tuesdays()`, ..., `sundays()` — at midnight on that day
- `at('17:30')` — run at a specific time (combine with day helpers)

## Examples

Every Friday at 5:00 PM

```php
schedule()->job(SendNewsletter::class)->fridays()->at('17:00');
```

15th of each month at 9:00 AM

```php
schedule()->job(SendNewsletter::class)->monthlyOn(15, '09:00');
```

Run a Job Every 15 Minutes

```php
schedule()->job(SyncJob::class)->everyMinutes(15);
```

Run a Command Every Monday at 9:15 AM

```php
schedule()->command(WeeklySummary::class)->mondays()->at('9:15');
```

Run on the 1st and 15th of Each Month at 8:00 AM

```php
schedule()->job(BiMonthlyJob::class)->cron('0 8 1,15 * *');
```

---

## Important Notes

### Timezone
All scheduled times use your server's timezone. Make sure your server timezone is configured correctly:

```sh
date  # Check current server time
```

### Overlapping Jobs
If a job takes longer than its schedule interval, multiple instances will be dispatched. For example:

```php
schedule()->job(LongRunningJob::class)->everyMinutes(5);
// If job takes 10 minutes, scheduler dispatches it again at 5 min mark
// Result: 2+ instances running simultaneously
```

**To prevent overlaps, following are few guidelines:**

1. **Make jobs faster** - Break large jobs into smaller chunks
2. **Increase interval** - Schedule less frequently than job duration
3. **Add job locking** - Use Lightpack's built-in lock utility to prevent overlaps:

```php
public function run() {
    // Try to acquire lock (expires after 600 seconds)
    if (lock()->acquire('long-running-job', 600) === false) {
        return; // Already running, skip this execution
    }
    
    try {
        // Your job logic here
    } finally {
        lock()->release('long-running-job');
    }
}
```

See [Mutex Lock documentation](/mutex-lock.md) for more details.

### Testing Schedules
You can test if events are scheduled correctly:

```php
$event = schedule()->job(MyJob::class)->daily()->at('09:00');

// Check if due at specific time
$isDue = $event->isDueAt(new \DateTime('2024-01-15 09:00:00'));

// Get next run time
$nextRun = $event->nextDueAt();

// Get previous run time
$previousRun = $event->previousDueAt();
```

---