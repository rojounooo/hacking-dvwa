# DVWA Testing Documentation

This repository contains my walkthrough and notes while testing Damn Vulnerable Web Application, a deliberately insecure PHP app for learning about web vulnerabilities.

---

## 🎯 Goal
To understand how each vulnerability behaves at different security levels: **Low**, **Medium**, **High**, and **Impossible**.

---

## 📂 Structure

The documentation is organized by vulnerability, e.g. SQL Injection, Brute Force and File Upload. Each directory will contain a vulnerabilty_overview.md, explaining what the attack it and how it is exploited as well as an analysis of the insecure code. It will also include my methodology for the different levels:

- **Low** – Minimal or no protection.
- **Medium** – Basic input validation or filters.
- **High** – More sophisticated protections.

---

## 🛠 Tools Used
- XAMPP
- DVWA 
- WSL
- Netcat
- Burp Suite 

---

## ✅ Progress Tracker

| Vulnerability     | Low | Medium | High |
|-------------------|-----|--------|------|
| SQL Injection     | ✅  | ⬜     | ⬜  |
| Reflected XSS     | ✅  | ⬜     | ⬜  |
| Stored XSS        | ✅  | ⬜     | ⬜  |
| Command Injection | ✅  | ✅     | ✅  |
| File Upload       | ✅  | ✅     | ✅  |
| CSRF              | ✅  | ⬜     | ⬜  |
| Brute Force       | ✅  | ⬜     | ⬜  |

--- 

## ⚠️ Disclaimer

This project is for **educational purposes only**. Do not attempt to exploit systems you don’t own or have explicit permission to test.

