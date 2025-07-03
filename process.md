# Process Documentation

Lightpack's Process component provides a clean interface for executing and managing system processes. It offers features like process execution, output streaming, timeout management, and error handling.

## Basic Usage

### Simple Command Execution

```php
use Lightpack\Utils\Process;

$process = new Process();

// Execute a simple command
$process->execute('ls -la');

// Get command output
echo $process->getOutput();

// Check for errors
if ($process->failed()) {
    echo $process->getError();
    echo 'Exit code: ' . $process->getExitCode();
}
```

### Array Command Syntax

```php
$process = new Process();

// Execute command with array arguments (safer)
$process->execute(['git', 'clone', 'https://github.com/user/repo.git']);

// Check execution status
if ($process->failed()) {
    throw new RuntimeException('Git clone failed: ' . $process->getError());
}
```

## Process Configuration

### Working Directory

```php
$process = new Process();

// Set working directory
$process->setDirectory('/path/to/directory')
    ->execute('git status');

// Get current working directory
$workDir = $process->getDirectory();
```

### Timeout Configuration

```php
$process = new Process();

// Set timeout in seconds (default: 60)
$process->setTimeout(120)
    ->execute('long-running-command');

// Handle timeout exception
try {
    $process->setTimeout(5)
        ->execute('sleep 10');
} catch (RuntimeException $e) {
    echo 'Process timed out';
}
```

## Output Handling

### Basic Output Retrieval

```php
$process = new Process();
$process->execute('echo "Hello World"');

// Get command output
$output = $process->getOutput();  // stdout
$error = $process->getError();    // stderr
$code = $process->getExitCode();  // exit code
```

### Streaming Output

```php
$process = new Process();

// Process output line by line
$process->execute('long-running-command', function(string $line, string $type) {
    if ($type === 'stdout') {
        echo "Output: $line";
    } else {
        echo "Error: $line";
    }
});
```

### Progress Monitoring

```php
$process = new Process();

// Monitor command progress
$process->execute('tar -czf archive.tar.gz directory/', function($line, $type) {
    if ($type === 'stdout') {
        // Update progress bar or log
    }
});
```

## Error Handling

### Basic Error Handling

```php
$process = new Process();

try {
    $process->execute('invalid-command');
} catch (RuntimeException $e) {
    echo 'Process failed: ' . $e->getMessage();
    echo 'Error output: ' . $process->getError();
    echo 'Exit code: ' . $process->getExitCode();
}
```

### Status Checking

```php
$process = new Process();
$process->execute('command');

// Check process status
if ($process->isRunning()) {
    echo 'Process is still running';
}

if ($process->failed()) {
    echo 'Process failed with exit code: ' . $process->getExitCode();
    echo 'Error output: ' . $process->getError();
}
```

## Practical Examples

These examples provide more insights into using process utility along with best practices. You should read them to gain a better understanding of practical use case scenarios of process utility.

1. **Command Safety**
   ```php
   class CommandExecutor
   {
       public function runCommand(array $command): string
       {
           $process = new Process();

           // Always use array syntax for command arguments
           $process->execute($command);
           
           if ($process->failed()) {
               throw new RuntimeException(
                   "Command failed: " . $process->getError()
               );
           }
           
           return $process->getOutput();
       }
   }
   ```

2. **Resource Management**
   ```php
   class LongRunningTask
   {
       public function execute()
       {
           $process = new Process();
           
           // Set appropriate timeout
           $process->setTimeout(3600);  // 1 hour
           
           // Use callback for memory efficiency
           $process->execute('large-data-processor', function($line, $type) {
               if ($type === 'stdout') {
                   $this->processLine($line);
               } else {
                   logger()->error($line);
               }
           });
       }
   }
   ```

3. **Environment Setup**
   ```php
   class DeploymentProcess
   {
       public function deploy()
       {
           $process = new Process();
           
           // Set working directory
           $process->setDirectory('/path/to/project')
               ->execute(['git', 'pull', 'origin', 'main']);
           
           if ($process->failed()) {
               $this->handleDeploymentFailure($process->getError());
               return;
           }
           
           $this->runPostDeployTasks();
       }
   }
   ```

4. **Output Processing**
   ```php
   class LogProcessor
   {
       public function processLogs(string $logFile)
       {
           $process = new Process();
           
           // Process large log files efficiently
           $process->execute(['tail', '-f', $logFile], function($line, $type) {
               if ($type === 'stdout') {
                   $this->parseLine($line);
               }
           });
       }
       
       private function parseLine(string $line)
       {
           // Process each line
       }
   }
   ```

5. **Error Recovery**
   ```php
   class ResilientProcess
   {
       public function executeWithRetry(array $command, int $maxAttempts = 3)
       {
           $process = new Process();
           $attempts = 0;
           
           while ($attempts < $maxAttempts) {
               try {
                   $process->execute($command);
                   
                   if (!$process->failed()) {
                       return $process->getOutput();
                   }
               } catch (RuntimeException $e) {
                   $attempts++;
                   if ($attempts === $maxAttempts) {
                       throw $e;
                   }
                   sleep(pow(2, $attempts));  // Exponential backoff
               }
           }
       }
   }
   ```
