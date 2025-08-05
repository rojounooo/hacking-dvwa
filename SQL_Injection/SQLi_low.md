# SQL Injection - Easy 

---

## Attack Steps 

1. Open the target page in browser 

2. Enter an id number e.g. 1

3. Observe the url
    - It becomes http://<ip address>/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit#

4. Inject payload 
    - Change the URL to http://<ip address>/dvwa/vulnerabilities/sqli/?id=1' or '1'=1&Submit=Submit

5. Observe the list of names

---

## Vulnerable Code Analysis

### File: `/var/www/dvwa/vulnerabilities/sqli/source/low`

#### Key Vulnerability Points:

```php
    $id = $_GET['id'];

    $getid = "SELECT first_name, last_name FROM users WHERE user_id = '$id'"; 
```

Explanation: 
- The code is taking direct user input `$id = $_GET['id'];` and inserting it into the sql query `WHERE user_id = '$id'"`.

- This can be exploited using the payload 1' or '1'=1: 
    - The query will become SELECT first_name, last_name FROM users WHERE user_id = ' ' or 1=1".
    - The ' '  is ignored because it is empty but the or statement will always result in true. 
    - Consequently all first and last names will be output.

- Causes: 
    - No input validation or sanitisation on $id so users can input anything 
    - Direct insertion of user input into queries means queries can be modified
    - Not using 
