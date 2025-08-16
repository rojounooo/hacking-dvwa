# Stored XSS - Low

---

## Attack Steps

1. Open burp suite
    - Temporary Project 
    - Use Burp Default

2. Navigate to proxy
    - Set intercept to off 
    - Open browser

3. Set security level to medium 
    - http://localhost/dvwa/security.php

4. Navigate to XSS (Stored)

5. Turn intercept back on 

6. Sign guestbook 
    ```bash 
    Name: Hecker 
    Message:: XSS Found
    ```

7. Edit the intercepted request 
    - Change name from:
        ```bash 
        hecker
        ```
    - To:
        ```bash 
        <SCRipt>alert(1)</SCRipt>

8. Turn off intercept 

9. Forward request

10. Refresh the page 

    - After refreshing an alert will show up

---
## Vulnerable Code Analysis 

### File: 
`C:\xampp\htdocs\DVWA\vulnerabilities\xss_s\source\medium.php`

#### Key Vulnerability Points:

```php 
<?php

if( isset( $_POST[ 'btnSign' ] ) ) {
	// Get input
	$message = trim( $_POST[ 'mtxMessage' ] );
	$name    = trim( $_POST[ 'txtName' ] );

	// Sanitize message input
	$message = strip_tags( addslashes( $message ) );
	$message = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $message ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
	$message = htmlspecialchars( $message );

	// Sanitize name input
	$name = str_replace( '<script>', '', $name );
	$name = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $name ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

	// Update database
	$query  = "INSERT INTO guestbook ( comment, name ) VALUES ( '$message', '$name' );";
	$result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

	//mysql_close();
}

?>
```

### Explanation 

- The code takes direct user input from the POST request:

    ```php
    $message = trim($_POST['mtxMessage']);
    $name    = trim($_POST['txtName']);
    ```

    - It applies `stripslashes()` and `mysql_real_escape_string()` to sanitize for SQL injection
    - The use of htmlspecialchars(), encodes the output to prevent browser from interpreting HTML/JavaScript as code

- Name isn't fully sanitised 

    ```php
    $name = str_replace( '<script>', '', $name );
    ```

    - Only sanitises "script" exactly so it is vulnerable to case insensitive tags like `<SCRipt></SCRipt>` or using unicode characters such as `"İ”`
    - Also vulnerable to other tags such as `<img>`

- Causes: 

    - No consistent input validation 
    - No output encoding for $name


