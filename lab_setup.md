# 🛠️ How to Set Up DVWA on a Windows Machine


---

# 🧭 Table of Contents
- [XAMPP & DVWA](#xampp--dvwa)
- [Burp Suite](#burp-suite)
- [WSL & Netcat](#wsl--netcat)

--- 
# XAMPP & DVWA

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

---

# Burp Suite 

# 🕵️‍♂️ How to Install Burp Suite on Windows

## 1. Download Burp Suite
    - Visit the [official PortSwigger website](https://portswigger.net/burp).
    - Choose either:
        - **Burp Suite Community Edition** (free)
        - **Burp Suite Professional** (paid)
    - Click **Download** and select the **Windows installer (.exe)**.

## 2. Run the Installer
    - Locate the downloaded `.exe` file (usually in your Downloads folder).
    - Double-click to launch the installer.
    - Follow the installation wizard:
        - Accept the license agreement.
        - Choose an installation directory (default is fine).
        - Click **Install**.

## 3. Launch Burp Suite
    - After installation, launch Burp Suite from the Start Menu or desktop shortcut.
    - On first launch:
        - Choose **Temporary Project** or **New Project**.
        - Select **Use Burp defaults**.
        - Click **Start Burp**.

## 4. Configure Browser Proxy (Optional but Recommended)
    - Burp Suite listens on `127.0.0.1:8080` by default.
    - To intercept traffic, configure your browser to use this proxy:
        - In Firefox:  
            - Go to `Settings > Network Settings > Manual proxy configuration`.
            - Set HTTP Proxy to `127.0.0.1` and Port to `8080`.
        - In Chrome:  
            - Use an extension like [FoxyProxy](https://getfoxyproxy.org/) to manage proxy settings.

## 5. Install Burp Certificate (for HTTPS Interception)
- In your browser, go to:
    - http://burpsuite
- Download the **CA Certificate**.
- Import it into your browser's certificate store as a **Trusted Root Certificate**.

    ### 🌐 Chrome (Windows)
    1. Export Burp's CA Certificate
        - Open Burp Suite.
        - Go to:
            Proxy > Options > Import / export CA certificate > Export certificate in DER format
        - Save the file as `burpCA.der` (or similar).
    
    2. Open Windows Certificate Manager
        - Press `Win + R`, type:
            certmgr.msc
        - Hit Enter

    3. Import the Certificate
        - In the left pane, expand:
            Trusted Root Certification Authorities > Certificates
        - Right-click **Certificates** > **All Tasks** > **Import**.
        - Follow the wizard:
        - Browse to your `burpCA.der` file.
        - Choose **Place all certificates in the following store**:  
            `Trusted Root Certification Authorities`
        - Finish and confirm the import.

    4. Restart Chrome
        - Close and reopen Chrome to apply the changes.

    ### 🦊 Firefox (Uses Its Own Certificate Store)

        1. Export Burp's CA Certificate
            - Same as Chrome: export the certificate in DER format from Burp Suite.

        2. Open Firefox Certificate Settings
            - In Firefox, go to:
                Settings > Privacy & Security > Certificates > View Certificates
            - Click **Authorities** tab.
        
        3. Import the Certificate
            - Click **Import**.
            - Select your `burpCA.der` file.
            - Check:
                - ✅ "Trust this CA to identify websites"
            - Click **OK** to finish.

        4. Restart Firefox
            - Close and reopen Firefox to apply the changes.

---

# WSL & Netcat 

1. Install WSL via PowerShell
    ```powershell
    wsl --install --version 1
    ```
    - Version 1 doesn't require virtualisation to be enabled 

2. Inside WSL
    
    ```bash
    sudo apt update 
    sudo apt install netcat -y 
    ```

