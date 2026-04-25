# Console: Lightpack CLI Assistant

Lightpack ships with a developer-focused command-line assistant named **console**. It's simple, powerful, and built for rapid PHP application development, automation, and extension.

---

## Quick Example

To generate a new controller class:

```cli
php console create:controller Product
```

---

## Available Commands

The console CLI provides a comprehensive set of commands for code generation, database management, job processing, application utilities, and more. Each command is described below with usage examples and options.

### Code Generators

#### create:config

Create a new `config` file in your project's **config** directory by copying from the example template.

```cli
php console create:config hello
```

You can force replace an existing config file with the same name using the `--force` flag.

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
- `--tenant` : Generate a tenant-aware model extending `TenantModel`.

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

#### create:tool
Generate a new AI tool class in `app/Tools`.

```cli
php console create:tool SearchProducts
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

#### jobs:run
Run the background job worker to process queued jobs.

```cli
php console jobs:run
```
- `--sleep=N` : Polling interval in seconds (default: 5).
- `--queue=emails,notifications` : Comma-separated list of queues (default: `default`).
- `--cooldown=N` : Exit after this many seconds of inactivity (default: run forever).

#### jobs:retry
Retry failed jobs that have exhausted all retry attempts.

```cli
# Retry all failed jobs
php console jobs:retry

# Retry a specific failed job by ID
php console jobs:retry 123
php console jobs:retry job_abc123xyz

# Retry all failed jobs from a specific queue
php console jobs:retry --queue=emails
```

**Options:**
- `--queue=<name>` : Retry only failed jobs from the specified queue.

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
Start the PHP built-in server on `127.0.0.1`.

```cli
php console app:serve
```

**Specify a custom port** (default: 8000):

```cli
php console app:serve --port=3000
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

The Lightpack console CLI provides a rich set of APIs for interactive command-line UX:

### Output Formatting

When extending `Command`, the `Output` utility is available via `$this->output`:

```php
$this->output->info('Information');
$this->output->success('Success!');
$this->output->error('An error occurred');
$this->output->warning('This is a warning');
$this->output->line('Plain text');
$this->output->pad('Left', 'Right', 30, '.'); // Left................. Right
$this->output->infoLabel('INFO'); // Colored label
```

You can also instantiate `Output` directly when needed outside a command:

```php
use Lightpack\Console\Output;

$output = new Output();
$output->success('Done!');
```

### Interactive Prompts

When extending `Command`, the `Prompt` utility is available via `$this->prompt`:

```php
$name = $this->prompt->ask('What is your name?');
$password = $this->prompt->secret('Enter password:');
$agree = $this->prompt->confirm('Do you agree?', true); // [Y/n]
$email = $this->prompt->askWithValidation('Email:', fn($v) => filter_var($v, FILTER_VALIDATE_EMAIL));
$choice = $this->prompt->choice('Pick one:', ['a' => 'Apple', 'b' => 'Banana']);
```

You can also instantiate `Prompt` directly when needed outside a command:

```php
use Lightpack\Console\Prompt;

$prompt = new Prompt();
$name = $prompt->ask('What is your name?');
```

---

## Custom Commands & Registration

### Creating a Command

1. Generate a new command class:

   ```cli
   php console create:command MyCommand
   ```

2. Implement the `run()` method in your class.

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

---

### Parsing Command Arguments

Lightpack provides the `Args` helper class for clean argument parsing:

```php
use Lightpack\Console\Command;

class MyCommand extends Command
{
    public function run()
    {
        // Get positional arguments
        $name = $this->args->first();              // First positional arg
        $first = $this->args->argument(0);         // First positional arg (0-indexed)
        $second = $this->args->argument(1);        // Second positional arg (0-indexed)
        $all = $this->args->positional();          // All positional args

        // Get options with values
        $table = $this->args->get('table');        // --table=users
        $key = $this->args->get('key', 'id');      // --key=id (with default)

        // Check for flags
        if ($this->args->has('force')) {           // --force
            $this->output->warning('Force mode enabled');
        }

        // Access raw arguments
        $raw = $this->args->all();                 // Original array
    }
}
```

#### Argument Syntax

**Positional arguments** (no dashes):
```cli
php console my:command User Product
```

**Options with values** (double dash with equals):
```cli
php console my:command --table=users --key=id
```

**Flags** (double dash, no value):
```cli
php console my:command --force --verbose
```

**Mixed usage**:
```cli
php console create:model User --table=users --key=id
```

**Note:** Only long options (`--option`) are supported. Short flags (`-o`) are treated as positional arguments.

---