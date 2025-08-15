# Brute Force - Low 

---

## Attack Steps

1. **Open DVWA Brute Force page**  
   Navigate to: http://localhost/dvwa/vulnerabilities/brute/

2. **Enter SQL injection payload in login form**  

    ```sql
    Username: admin' or '1'='1 -- 
    Password: anything
    ```
    - Any password value will work because the injection bypasses the check.

3. **Submit and observe result**  
You should see:
Welcome to the password protected area admin' or '1'='1 --


## Notes
- This vulnerability is in the **Brute Force** module, but we’re exploiting it using **SQL Injection**.
- Tools like Hydra or Burp Suite may produce false positives here because the response length doesn’t change.
- Reviewing the source code reveals the root cause.

---

## Vulnerable Code Analysis 

### File: 
`C:\xampp\htdocs\DVWA\vulnerabilities\brute\source\low.php`

#### Key Vulnerability Points:

```php 
<?php

if( isset( $_GET[ 'Login' ] ) ) {
	// Get username
	$user = $_GET[ 'username' ];

	// Get password
	$pass = $_GET[ 'password' ];
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
		$html .= "<pre><br />Username and/or password incorrect.</pre>";
	}

	((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?>
```

### Explanation 
- The code is taking direct user input `$user = $_GET['username'];/$pass = $_GET['password'];`. The password is hashed using MD5 `$pass = md5($pass);` before the username and password hash is inserted into the SQL query.

- This can be exploited using the payload ' or '1'=1 --: 
    - The query will become "SELECT * FROM `users` WHERE user=' ' or '1'=1 -- ".
    - The ' ' will be ignored and the boolean value will always be true. 
    - The `--` will comment out the password section 

- Causes: 
    - No input validation or sanitisation on $user or $pass so users can input anything
    - Direct insertion of user input into queries means queries can be modified
    - Not using prepared statements, prepared statements treat user input only as data rather than SQL code so even if SQL keywords are included they won't modify the queries  