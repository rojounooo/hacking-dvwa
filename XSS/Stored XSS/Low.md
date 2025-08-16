# Stored XSS - Low

---

## Attack Steps

1. Open the target page in the browser: 
    - http://localhost/dvwa/vulnerabilities/xss_s/

2. Sign the guestbook with a script

    ```sql 
    Name: hacker 
    Message: <script> alert("Hello") </script>
    ```

3. Refresh the page 
    - An alert saying "Hello" should show up

---
## Vulnerable Code Analysis 

### File: 
`C:\xampp\htdocs\DVWA\vulnerabilities\xss_s\source\low.php`

#### Key Vulnerability Points:

```php 
<?php

if( isset( $_POST[ 'btnSign' ] ) ) {
    // Get input
    $message = trim( $_POST[ 'mtxMessage' ] );
    $name    = trim( $_POST[ 'txtName' ] );

    // Sanitize message input
    $message = stripslashes( $message );
    $message = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $message ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Sanitize name input
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

    - It applies `stripslashes()` and `mysql_real_escape_string()` to sanitize for SQL injection, but does not encode HTML characters before storing in or displaying from the database.

    - Since there is no output encoding like `htmlspecialchars()` when the comment is displayed, an attacker can input malicious HTML or JavaScript like:

    ```html
    <script>alert(1)</script>
    ```

    - This input is stored as-is in the database and rendered directly into the page later. When any user visits the page, the malicious script executes in their browser.

- Causes: 

    - No input validation or filtering for dangerous characters other than SQL characters. 
    - No output encoding like `htmlspecialchars()` when displaying stored content
    - User input is trusted HTML when rendered


