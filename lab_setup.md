# 🛠️ How to Set Up DVWA on a Windows Machine

## 1. Install XAMPP
- Download XAMPP from the [official Apache Friends website](https://www.apachefriends.org/index.html).
- Run the installer and follow the setup wizard.
- Install it to the default location (usually `C:\xampp`).

## 2. Setup XAMPP
- Launch the XAMPP Control Panel.
- Start the following services:
  - **Apache**
  - **MySQL**
- Ensure both services are running (green indicators).

## 3. Download DVWA Zip
- Go to the [DVWA GitHub repository](https://github.com/digininja/DVWA).
- Click **Code > Download ZIP**.
- Save the ZIP file to your computer.

## 4. Extract to `htdocs`
- Extract the contents of the DVWA ZIP file.
- Move the extracted folder (usually named `DVWA-master`) to:
    - C:\xampp\htdocs\
- Rename the folder to `dvwa` for simplicity:
    - C:\xampp\htdocs\dvwa


## 5. Enable phpMyAdmin
- Open your browser and go to:
    - http://localhost/phpmyadmin

- Create a new database named `dvwa`.
- You can leave the collation as default.

## 6. Copy and Edit `config.inc.php`
- Navigate to:
    - C:\xampp\htdocs\dvwa\config
    - Copy `config.inc.php.dist` and rename it to `config.inc.php`.
    - Open `config.inc.php` in a text editor and update the following lines:
    ```php
    $_DVWA = array();
    $_DVWA['db_server']   = '127.0.0.1';
    $_DVWA['db_database'] = 'dvwa';
    $_DVWA['db_user']     = 'root';
    $_DVWA['db_password'] = '';       // default XAMPP MySQL root password
    $_DVWA['db_port']     = '3306';
    ```
    - Save the file.

7. Finish setup.php

    - In your browser, go to:
        `http://localhost/dvwa/setup.php`
    
    - Click the button to create the database and setup the application.

    - Once complete, log in using the default credentials:
        ```
        Username: admin
        Password: password
        ```