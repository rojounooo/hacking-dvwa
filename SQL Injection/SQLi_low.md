# SQL Injection - Easy 

---

## Attack Steps 

1. Open the target page in browser 
    - http://localhost/dvwa/vulnerabilities/sqli

2. Enter an id number e.g. 1

3. Observe the url
    - It becomes http://localhost/dvwa/vulnerabilities/sqliid=1&Submit=Submit#

4. Inject payload 
    - Change the URL to http://localhost/dvwa/vulnerabilities/sqli/?id=1' or '1'=1&Submit=Submit

5. Observe the list of names

---

## Vulnerable Code Analysis

### File:
`C:\xampp\htdocs\DVWA\vulnerabilities\sqli\source\low.php`

#### Key Vulnerability Points:

```php
<?php

if( isset( $_REQUEST[ 'Submit' ] ) ) {
	// Get input
	$id = $_REQUEST[ 'id' ];

	switch ($_DVWA['SQLI_DB']) {
		case MYSQL:
			// Check database
			$query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
			$result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

			// Get results
			while( $row = mysqli_fetch_assoc( $result ) ) {
				// Get values
				$first = $row["first_name"];
				$last  = $row["last_name"];

				// Feedback for end user
				$html .= "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
			}

			mysqli_close($GLOBALS["___mysqli_ston"]);
			break;
		case SQLITE:
			global $sqlite_db_connection;

			#$sqlite_db_connection = new SQLite3($_DVWA['SQLITE_DB']);
			#$sqlite_db_connection->enableExceptions(true);

			$query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
			#print $query;
			try {
				$results = $sqlite_db_connection->query($query);
			} catch (Exception $e) {
				echo 'Caught exception: ' . $e->getMessage();
				exit();
			}

			if ($results) {
				while ($row = $results->fetchArray()) {
					// Get values
					$first = $row["first_name"];
					$last  = $row["last_name"];

					// Feedback for end user
					$html .= "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
				}
			} else {
				echo "Error in fetch ".$sqlite_db->lastErrorMsg();
			}
			break;
	} 
}

?>
```

### Explanation: 
- The code is taking direct user input `$id = $_GET['id'];` and inserting it into the sql query `WHERE user_id = '$id'"`.

- This can be exploited using the payload 1' or '1'=1: 
    - The query will become SELECT first_name, last_name FROM users WHERE user_id = ' ' or 1=1".
    - The ' '  is ignored because it is empty but the or statement will always result in true. 
    - Consequently all first and last names will be output.

- Causes: 
    - No input validation or sanitisation on $id so users can input anything 
    - Direct insertion of user input into queries means queries can be modified
    - Not using prepared statements, prepared statements treat user input only as data rather than SQL code so even if SQL keywords are included they won't modify the queries 


