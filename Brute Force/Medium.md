# Brute Force - Medium 

--- 

## Attack Steps 

1. Download password list 

    - https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/darkweb2017_top-100.txt

2. Open burp suite
    - Temporary Project
    - Use Burp Default

3. Navigate to proxy
    - Set intercept to off
    - Open browser

4. Set security level to medium
    - http://localhost/dvwa/security.php

5. Navigate to Brute Force

6. Turn intercept back on

7. Enter credentials and login

    ```bash 
    Username: admin 
    Password: anything 
    ```

    - Press login

8. Send request to Intruder 

9. Click Clear § in positions

10. Add §

    - Highlight password value 
    - Click Add §

    - Value should go from 
    ```html 
    password=anything
    ```
    
    - To
    ```html 
    password=§anything§
    ```

11. Set attack type 

    - Sniper attack as only brute forcing password 

12. Set Payload 
 
    - Simple list 
    - Load the downloaded password list

13. Start attack and look for a password with a different response length 

    - E.g. the correct password will have a Response of 40 but incorrect passwords will have a length of over 2000

14. Try logging in with the correct password 

--- 

## Vulnerable Code Analysis 

    
