# Cross Site Request Forgery - Low 

## Attack Steps: 

1. Navigate to target page: 
    - http://localhost/dvwa/vulnerabilities/csrf

2. Edit the url to the following: 
    - http://localhost/dvwa/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change#

3. Logout 

4. Login with the new password we set 

## Vulnerable Code Analysis 

### File Path:
`C:\xampp\htdocs\DVWA\vulnerabilities\csrf\source\low.php`

#### Key Vulnerability Points 

```php 
<?php

if( isset( $_GET[ 'Change' ] ) ) {
	// Get input
	$pass_new  = $_GET[ 'password_new' ];
	$pass_conf = $_GET[ 'password_conf' ];

	// Do the passwords match?
	if( $pass_new == $pass_conf ) {
		// They do!
		$pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
		$pass_new = md5( $pass_new );

		// Update the database
		$current_user = dvwaCurrentUser();
		$insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . $current_user . "';";
		$result = mysqli_query($GLOBALS["___mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

		// Feedback for the user
		$html .= "<pre>Password Changed.</pre>";
	}
	else {
		// Issue with passwords matching
		$html .= "<pre>Passwords did not match.</pre>";
	}

	((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?>
```
### Explanation

- The code changes the logged‑in user’s password if the `Change` parameter is present in a **GET** request:
  ```php
  if( isset( $_GET[ 'Change' ] ) ) { ... }
  ```

- It retrieves password_new and password_conf from the query string:

	```php
	$pass_new  = $_GET['password_new'];
	$pass_conf = $_GET['password_conf'];
	```
	- and updates the password if they match — no further verification is performed.

- The application does not require:

	- Any CSRF token
	- Checking the request’s origin or referrer This means any webpage can cause a logged‑in user’s browser to send the malicious request.

	- Example exploit URL on Windows/XAMPP:

	```html 
	http://localhost/dvwa/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change#
	```
	- Visiting this while logged in will change the current account’s password to hacked without user consent.

- Causes:

	- No CSRF protection mechanism in place
	- Accepting sensitive actions via GET instead of POST
	- Trusting session state (dvwaCurrentUser()) alone to determine authorisation
	- No user confirmation step before updating the password



