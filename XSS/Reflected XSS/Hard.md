# Reflected XSS - Hard

--- 

## Attack Steps

---

## Vulnerable Code Analysis

### File:

#### Code

```php
<?php

header ("X-XSS-Protection: 0");

// Is there any input?
if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
	// Get input
	$name = preg_replace( '/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i', '', $_GET[ 'name' ] );

	// Feedback for end user
	$html .= "<pre>Hello {$name}</pre>";
}

?>
```

## Explanation

- The script uses regex to filter for case insensitive `<script>` tags

```php
$name = preg_replace( '/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i', '', $_GET[ 'name' ] );
```

- There is no html encoding or filtering for any other tags so it is vulnerable to payloads using `<img>`

- Causes:
    - Lack of filtering for multiple tag types
    - No output encoding using htmlspecialchars()