# Cross Site Request Forgery - Low 

## Attack Steps: 

1. Navigate to target page: 
    - http://<ip address>/dvwa/vulnerabilities/csrf

2. Edit the url to the following: 
    - http://<ip address>>/dvwa/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change#

3. Logout 

4. Login with the new password we set 

## Vulnerable Code Analysis 

### File Path:
`/var/www/dvwa/vulnerabilities/csrf/source/low`

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


