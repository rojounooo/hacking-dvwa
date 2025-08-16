# Reflected XSS - Medium

---

## Attack Steps

--- 

## Vulnerable Code Analysis

### File
`C:\xampp\htdocs\DVWA\vulnerabilities\xss_r\source\medium.php`

#### Code
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