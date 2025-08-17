# Brute Force - Medium 

--- 

## Attack Steps 

1. Download password list 

    - https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/darkweb2017_top-100.txt

2. Open burp suite
    - Temporary Project
    - Use Burp Default

3. Navigate to proxy
    - Set intercept to off
    - Open browser

4. Set security level to medium
    - http://localhost/dvwa/security.php

5. Navigate to Brute Force

6. Turn intercept back on

7. Enter credentials and login

    ```bash 
    Username: admin 
    Password: anything 
    ```

    - Press login

8. Send request to Intruder 

9. Click Clear § in positions

10. Add §

    - Highlight password value 
    - Click Add §

    - Value should go from 
    ```html 
    password=anything
    ```
    
    - To
    ```html 
    password=§anything§
    ```

11. Set attack type 

    - Sniper attack as only brute forcing password 

12. Set Payload 
 
    - Simple list 
    - Load the downloaded password list

13. Start attack and look for a password with a different response length 

    - E.g. the correct password will have a Response of 40 but incorrect passwords will have a length of over 2000

14. Try logging in with the correct password 

--- 

## Vulnerable Code Analysis 

### File: 
`C:\xampp\htdocs\DVWA\vulnerabilities\brute\source\medium.php`

#### Code:

```php 
<?php

if( isset( $_GET[ 'Login' ] ) ) {
	// Sanitise username input
	$user = $_GET[ 'username' ];
	$user = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $user ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

	// Sanitise password input
	$pass = $_GET[ 'password' ];
	$pass = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
	$pass = md5( $pass );

	// Check the database
	$query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";
	$result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

	if( $result && mysqli_num_rows( $result ) == 1 ) {
		// Get users details
		$row    = mysqli_fetch_assoc( $result );
		$avatar = $row["avatar"];

		// Login successful
		$html .= "<p>Welcome to the password protected area {$user}</p>";
		$html .= "<img src=\"{$avatar}\" />";
	}
	else {
		// Login failed
		sleep( 2 );
		$html .= "<pre><br />Username and/or password incorrect.</pre>";
	}

	((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?>
```

## Explanation
- The login form accepts user input via GET parameters:
	```php 
	$user = $_GET['username'];
	$pass = $_GET['password'];
	```
- The password is hashed using MD5 before being checked against the database:
	```php 
	$pass = md5($pass);
	```

- There is input sanitisation to prevent SQL injections
    ```php 
     ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    ```

- This is vulnerable to brute force attacks because:
	- There is some rate limiting 
        ```php
        sleep( 2 )
        ```
        - However this just makes the brute force attack slower
    - No CAPTCHA or account lockout mechanism
	- The server responds with noticeably different content lengths for valid vs. invalid credentials, which can be detected using Burp Suite's Intruder tool.
	- The use of MD5 hashing does not prevent brute force attacks—it simply means the attacker must hash each password before sending it.

- Causes: 
	- No proper brute force mitigation only basic rate limiting 
	- MD5 is insufficient 
    
