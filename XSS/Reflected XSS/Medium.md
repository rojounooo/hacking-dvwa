# Reflected XSS - Medium

---

## Attack Steps

1. Set security level to medium
    - http://localhost/dvwa/security.php

2. Navigate to XSS (Reflected)

3. Enter the payload into the form
    ```html 
    <SCRipt>alert(1)</SCRipt>
    ```

4. Submit and refresh the page

5. An alert box should show up
---

## Vulnerable Code Analysis

### File:
`C:\xampp\htdocs\DVWA\vulnerabilities\xss_r\source\medium.php`

#### Code
```php
<?php

header ("X-XSS-Protection: 0");

// Is there any input?
if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
	// Get input
	$name = str_replace( '<script>', '', $_GET[ 'name' ] );

	// Feedback for end user
	$html .= "<pre>Hello {$name}</pre>";
}

?>
```

## Explanation

- There is basic input sanitsation of the `<script>` tag, replacing it with an empty string 
    ```php 
    $name = str_replace( '<script>', '', $_GET[ 'name' ] );
    ```
    - This only checks for the tag in lowercase, so a case insensitive tag such as `<SCRipt>` can be used to bypass it
    - It is also vulnerable to other tags such as `<img>`

- There is no output encoding using functions like htmlspecialchars() so the browser will interpret all HTML/JavaScript as code rather than data.

- Causes: 
    - Insufficient sanitisation 
    - Lack of output encoding