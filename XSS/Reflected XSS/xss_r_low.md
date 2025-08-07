# Reflected XSS - Low 

--- 

## Attack Steps 

1. Open target page in browser 
    - http://<ip address>/dvwa/vulnerabilities/xss_r/

2. Enter XSS Payload 

    ```html 
    <script>alert(1)</script> # Replace 1 with your message
    ```

3. Refresh screen

    - An alert should pop up with your message

--- 

## Vulnerable Code Analysis 

### File: 
`/var/www/dvwa/vulnerabilities/xss_r/source/low`

#### Key Vulnerability Points:

```php 
    <?php

    if(!array_key_exists ("name", $_GET) || $_GET['name'] == NULL || $_GET['name'] == ''){

    $isempty = true;

    } else {
        
    echo '<pre>';
    echo 'Hello ' . $_GET['name'];
    echo '</pre>';
    
    }

    ?> 
```
 
### Explanation

- The code takes direct user input from the **GET** request:

    ```php
    echo 'Hello ' . $_GET['name'];
    ```
    - It performs **no input validation**, filtering, or sanitization of the `name` parameter.
    - The value of `$_GET['name']` is **directly inserted into the HTML response** without any output encoding like `htmlspecialchars()`.
    - As a result, if an attacker submits a malicious payload like:

    ```html
    <script>alert(1)</script>
    ```

    - The script is **reflected immediately** in the page and executed by the browser as JavaScript.

- **Causes:**
    -  No input validation or filtering of dangerous characters.
    -  No output encoding using functions like `htmlspecialchars()` to neutralize HTML or JavaScript.
    -  The application assumes that user input is safe to render as HTML, which allows attackers to inject executable scripts.

