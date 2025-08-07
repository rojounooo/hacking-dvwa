# Stored XSS - Low

---

## Attack Steps

1. Open the target page in the browser: 
    - http://<ip address>/dvwa/vulnerabilities/xss_s/

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
`/var/www/dvwa/vulnerabilities/xss_s/source/low`

#### Key Vulnerability Points:

```php 
    <?php

    if(isset($_POST['btnSign']))
    {

    $message = trim($_POST['mtxMessage']);
    $name    = trim($_POST['txtName']);
   
     // Sanitize message input
     $message = stripslashes($message);
     $message = mysql_real_escape_string($message);
   
     // Sanitize name input
    $name = mysql_real_escape_string($name);
  
     $query = "INSERT INTO guestbook (comment,name) VALUES ('$message','$name');";
   
    $result = mysql_query($query) or die('<pre>' . mysql_error() . '</pre>' );
   
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


