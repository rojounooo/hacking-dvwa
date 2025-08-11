# Command Injection - Low 

## Attack Steps 

1. Navigate to the DVWA Command Injection page:
    
    - http://<ip address>/dvwa/vulnerabilities/exec
 
2. In the input field, enter the following payload:  

    ```bash
    8.8.8.8|ls
    ```


3. Click the **Submit** button.

4. The output should display the ping results followed by the directory listing from the server, for example:  

    ```bash 
    help
    index.php
    source
    ```



## Vulnerable Code Analysis

### File: `[RELATIVE FILE PATH]`

#### Key Vulnerability Points:

```php
 <?php

if( isset( $_POST[ 'submit' ] ) ) {

    $target = $_REQUEST[ 'ip' ];

    // Determine OS and execute the ping command.
    if (stristr(php_uname('s'), 'Windows NT')) { 
    
        $cmd = shell_exec( 'ping  ' . $target );
        echo '<pre>'.$cmd.'</pre>';
        
    } else { 
    
        $cmd = shell_exec( 'ping  -c 3 ' . $target );
        echo '<pre>'.$cmd.'</pre>';
        
    }
    
}
?> 
```

### Explanation
- The code takes direct user input `$target = $_REQUEST['ip'];` and passes it straight into a shell command without any validation or sanitisation.  
- The value of `$target` is concatenated into the `ping` command string inside `shell_exec()`, e.g.:  
    ```php
    shell_exec('ping -c 3 ' . $target);
    ```  
- This can be exploited using a payload such as:  
    ```
    127.0.0.1 && whoami
    ```  
    - The command executed by the server becomes:  
      ```
      ping -c 3 127.0.0.1 && whoami
      ```  
    - After the ping runs, `whoami` will also execute, revealing the username of the process running PHP.  
- Causes:  
    - No input validation or sanitisation on `$target`, so arbitrary commands can be injected.  
    - Direct insertion of user input into a shell command allows attackers to append extra commands using operators like `&&`, `;`, or `|`.  
    - Not using safe functions like `escapeshellarg()` or input whitelisting means special characters are interpreted by the shell.  
