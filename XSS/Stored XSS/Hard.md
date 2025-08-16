# Stored XSS - Hard 

--- 

## Attack Steps 

1. Open burp suite
    - Temporary Project 
    - Use Burp Default

2. Navigate to proxy
    - Set intercept to off 
    -  Open browser

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
        <img src=0 onerror=alert(1)>

8. Turn off intercept 

9. Forward request

10. Refresh the page 

    - After refreshing an alert will show up

---
## Vulnerable Code Analysis 

### File: 
`C:\xampp\htdocs\DVWA\vulnerabilities\xss_s\source\hard.php`

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
	$name = preg_replace( '/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i', '', $name );
	$name = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $name ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

	// Update database
	$query  = "INSERT INTO guestbook ( comment, name ) VALUES ( '$message', '$name' );";
	$result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

	//mysql_close();
}

?>
```
