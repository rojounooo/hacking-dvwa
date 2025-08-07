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