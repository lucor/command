command
=======

External command runner / executor for PHP.  This is an object oriented, robust replacement for `exec`, `shell_exec`, the backtick operator and the like.

~~Taken from http://pollinimini.net/blog/php-command-runner/ and hosted here for maintenance, improvement and use with Packagist~~

This package is diverging heavily from the original project, and it will continue to diverge.

## Running Commands

At its simplest form, you can execute commands like this:

```php
$cmd = Command::factory('ls')->run();
```

### Adding Arguments and Options

Here we are safely adding arguments:

```php
use kamermans\Command\Command;

$cmd = Command::factory('/usr/bin/svn');
$cmd->option('--username', 'drslump')
    ->option('-r', 'HEAD')
    ->option('log')
    ->argument('http://code.google.com/drslump/trunk');
    ->run();

echo $cmd->getStdOut();
```

### Using a Callback for Incremental Updates
Normally all command output is buffered and once the command completes you can access it.  By using a callback, the output is buffered until the desired number of bytes is received (see `Command::setReadBuffer(int $bytes)`), then it is passed to your callback function:

```php
use kamermans\Command\Command;

$cmd = Command::factory('ls');
$cmd->setCallback(function($pipe, $data) {
        // Gets run for every 4096 bytes
        echo $data;
    })
    ->setReadBuffer(4096)
    ->setDirectory('/tmp')
    ->option('-l')
    ->run();
```

Alternately, you can set the second argument for `Command::run(string $stdin, bool $lines)` to `true` to execute your callback once for every line of output:

```php
use kamermans\Command\Command;

$cmd = Command::factory('ls');
$cmd->setCallback(function($pipe, $data){
        // Gets run for each line of output
        echo $data;
    })
    ->setDirectory('/tmp')
    ->option('-l')
    ->run(null, true);
```

Some more features:
 - `StdIn` data can be provided to the process as a parameter to `run()`
 - Set environment variables for the process with `setEnv()`
 - Second argument to `option()` and argument to `argument()` are automatically escaped.
 - Options separator is white space by default, it can be changed by manually setting it as third argument to `option()` or setting a new default with `setOptionSeparator()`.
 - The `proc_open` wrapper is exposed as a static method for your convenience `Command::exec()`
