# Stored XSS - Hard 

--- 

## Attack Steps 

1. Open burp suite
    - Temporary Project 
    - Use Burp Default

2. Navigate to proxy
    - Set intercept to off 
    -  Open browser

3. Set security level to medium 
    - http://localhost/dvwa/security.php

4. Navigate to XSS (Stored)

5. Turn intercept back on 

6. Sign guestbook 
    ```bash 
    Name: Hecker 
    Message:: XSS Found
    ```

7. Edit the intercepted request 
    - Change name from:
        ```bash 
        hecker
        ```
    - To:
        ```bash 
        <img src=0 onerror=alert(1)>

8. Turn off intercept 

9. Forward request

10. Refresh the page 

    - After refreshing an alert will show up

---