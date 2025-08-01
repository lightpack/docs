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

In thr production environment, you should set up a system **cron** job to call this command every minute:

```cron
* * * * * php /path/to/lightpack schedule:run
```

This ensures your scheduled events are checked and executed on time.

---

## Defining Schedules

You should define all the schedules in `schedules/schedules.php` file.

```php
// Schedule a job to run every day at midnight
schedule()->job(SendDailyReport::class)->daily();

// Schedule a command to run every hour
schedule()->command(CleanupCommand::class)->hourly();

// Schedules a command with arguments
schedule()->command(MyCommandClass::class, ['--force' => true]); 
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
schedule()->job(SendNewsletter::class)->monthlyOn(15, '09:00')
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