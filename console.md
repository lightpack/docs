# Console: Lightpack CLI Assistant

Lightpack ships with developer-focused command-line assistant named **console**. It’s simple, powerful, and built for rapid PHP application development, automation, and extension.

---

## Quick Example

To generate a new controller class:

```cli
php console create:controller Product
```

---

## Available Commands

console CLI provides a comprehensive set of commands for code generation, database management, job processing, application utilities, and more. Each command is described below with usage examples and options.

### Code Generators

#### create:config

Create a new `config` file in your project's **config** by copying from the example template.

```cli
php console create:config hello
```

You can force replace existing config file having same name using `--force` flag.

```cli
php console create:config hello --force
```

#### create:env
Create a new `.env` file in your project root by copying from the example template.

```cli
php console create:env
```

#### create:event
Generate a new event class in `app/Events`.

```cli
php console create:event UserRegistered
```

#### create:model
Generate a new model class in `app/Models`.

```cli
php console create:model Product
```
- `--table=products` : Set the table name.
- `--key=product_id` : Set the primary key name.

#### create:filter
Generate a new filter class in `app/Filters`.

```cli
php console create:filter InputFilter
```

#### create:command
Generate a new command class in `app/Commands`.

```cli
php console create:command HelloCommand
```

#### create:provider
Generate a new provider class in `app/Providers`.

```cli
php console create:provider MailProvider
```
- `--instance` : Create a provider that calls the container's factory method.

#### create:controller
Generate a new controller class in `app/Controllers`.

```cli
php console create:controller ProductController
```

#### create:migration
Generate a new migration file in `database/migrations`.

```cli
php console create:migration create_table_users
```

#### create:job
Generate a new job class in `app/Jobs`.

```cli
php console create:job SendEmailJob
```

#### create:mail
Generate a new mail class in `app/Mails`.

```cli
php console create:mail WelcomeMail
```

#### create:seeder
Generate a new seeder class in `database/seeders`.

```cli
php console create:seeder UserSeeder
```

#### create:transformer
Generate a new transformer class in `app/Transformers`.

```cli
php console create:transformer UserTransformer
```

#### create:request
Generate a new request class in `app/Requests`.

```cli
php console create:request RegisterRequest
```

---

### File Storage

#### link:storage
Create a symbolic link from `public/uploads` to `storage/uploads/public`.

```cli
php console link:storage
```

#### unlink:storage
Remove the symbolic link created by `link:storage`.

```cli
php console unlink:storage
```

---

### Database & Migrations

#### migrate:up
Run all pending database migrations (MySQL/MariaDB only).

```cli
php console migrate:up
```
- Prompts for confirmation in production.

#### migrate:down
Rollback database migrations.

```cli
php console migrate:down --all
php console migrate:down --steps=2
```
- `--all` : Rollback all migrations.
- `--steps=N` : Rollback N batches.

#### db:seed
Run the `DatabaseSeeder` to seed your database.

```cli
php console db:seed
```
- Prompts for confirmation before running.

---

### Jobs & Scheduling

#### process:jobs
Run the background job worker to process queued jobs.

```cli
php console process:jobs
```
- `--sleep=N` : Polling interval in seconds (default: 5).
- `--queue=emails,notifications` : Comma-separated list of queues (default: `default`).
- `--cooldown=N` : Exit after this many seconds of inactivity (default: run forever).

#### schedule:events
Run all scheduled tasks (for cron integration).

```cli
php console schedule:events
```

---

### Application Utilities

#### app:key
Generate and set a new `APP_KEY` in your `.env` file.

```cli
php console app:key
```

#### app:serve
Start the PHP built-in server using `APP_URL` from `.env` as the host.

```cli
php console app:serve
```

**You can specify the port to run:**

```cli
php console app:serve 8001
```

---

### Hot Reload & Watch

#### watch
Watch files or directories for changes and run a shell command (hot reload for development).

```cli
php console watch --path=app,config --ext=php,json --run="vendor/bin/phpunit"
```
- `--path=app,config` : Comma-separated list of directories/files to watch (required).
- `--ext=php,json` : Comma-separated file extensions to filter (optional).
- `--run=command` : Shell command to execute on change (optional).
- `--help` : Show detailed usage.

---

## Interactive Prompts & Output

Lightpack console CLI provides a rich set of APIs for interactive command-line UX:

### Output Formatting

Use the `Lightpack\Console\Output()` class for styled output:

```php
$output = new Output();

$output->info('Information');
$output->success('Success!');
$output->error('An error occurred');
$output->warning('This is a warning');
$output->line('Plain text');
$output->pad('Left', 'Right', 30, '.'); // Left................. Right
$output->infoLabel('INFO'); // Colored label
```

### Interactive Prompts

Use the `Lightpack\Console\Prompt` class for user input:

```php
$prompt = new Prompt();

$name = $prompt->ask('What is your name?');
$password = $prompt->secret('Enter password:');
$agree = $prompt->confirm('Do you agree?', true); // [Y/n]
$email = $prompt->askWithValidation('Email:', fn($v) => filter_var($v, FILTER_VALIDATE_EMAIL));
$choice = $prompt->choice('Pick one:', ['a' => 'Apple', 'b' => 'Banana']);
```

---

## Custom Commands & Registration

### Creating a Command

1. Generate a new command class:

   ```cli
   php console create:command MyCommand
   ```

2. Implement the `run(array $arguments = [])` method in your class.

3. Register your command in `boot/commands.php`:

   ```php
   return [
       'my:command' => App\Commands\MyCommand::class,
   ];
   ```

4. Run your command:

   ```cli
   php console my:command arg1 --flag=value
   ```

**Note:** All arguments are passed as an array to `run()`.

---