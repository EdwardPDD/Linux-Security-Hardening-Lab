# Linux-Security-Hardening-Lab

## Project Overview

This project documents a Linux security hardening exercise completed in the XP Cyber Range. The objective was to review Lynis audit findings on two Linux servers, Prod-Joomla and Fileshare, and implement the recommended security changes.

The lab involved strengthening password policies, configuring PAM authentication controls, securing an Apache web server, installing Fail2Ban, and enabling process accounting. Each system required different remediation steps based on its operating system and existing configuration.

## Scenario

A security audit was performed on the organization's Linux systems using Lynis. The audit identified several security weaknesses that required remediation on the Prod-Joomla and Fileshare servers.

The goal was to review the audit findings, determine the appropriate configuration changes, implement the fixes, and verify that the systems reached the required security state.

## Lab Environment

- XP Cyber Range
- Prod-Joomla Linux Server
- Fileshare Linux Server
- Lynis Security Auditing Tool
- Linux Command Line
- PAM (Pluggable Authentication Modules)
- Apache
- Fail2Ban
- Process Accounting

 ## Security Objectives

The Lynis audit identified several security recommendations across the Prod-Joomla and Fileshare servers. The main objectives of the lab were to:

- Strengthen password policies using PAM.
- Set the minimum password length to at least 10 characters.
- Set the minimum password age to at least 3 days without changing the maximum password age.
- Protect the Apache web server against DoS and brute-force attacks.
- Protect Apache against Slowloris attacks.
- Add protection against common web application attacks.
- Install and configure Fail2Ban.
- Enable process accounting on the Fileshare server.

## Audit Findings

The security recommendations were provided through Lynis audit reports located at:

`/home/playerone/audit.log`

The Prod-Joomla server required password-policy changes, Apache security modules, and Fail2Ban. The Fileshare server required password-policy changes, Fail2Ban, and process accounting.

The two systems did not have identical configurations, so some remediation steps had to be adjusted based on the packages and PAM modules available on each server.

## Prod-Joomla Remediation

### Password Policy Hardening

The Lynis audit identified weaknesses in the server's password configuration. I installed a PAM password-strength module and updated the password policy to require a minimum password length of 10 characters.

I also updated `/etc/login.defs` and changed the minimum password age from:

`PASS_MIN_DAYS 0`

to:

`PASS_MIN_DAYS 3`

The maximum password age was intentionally left unchanged based on the audit instructions.

The PAM password configuration was updated in:

`/etc/pam.d/common-password`

Cracklib was enabled for password-strength checking with a minimum password length of 10 characters.

### Apache Security Hardening

The Prod-Joomla server also required additional protections for the Apache web server. I used APT to identify and install the recommended security modules:

- `mod_evasive` — protection against excessive requests and DoS/brute-force behavior
- `mod_qos` — additional controls against resource-exhaustion and Slowloris-style attacks
- `mod_security` — web application firewall functionality for detecting malicious HTTP requests

### Fail2Ban

Fail2Ban was installed to provide additional protection against repeated authentication failures. The package monitors authentication activity and can temporarily block hosts responsible for repeated failed login attempts.

### Prod-Joomla Verification

After completing the remediation, all five Prod-Joomla security checks reached the **Desired State**.

## Fileshare Remediation

### Password Policy Hardening

The Fileshare server required similar password-policy changes, but its PAM configuration was different from Prod-Joomla.

I initially attempted to install Cracklib using:

`sudo apt install libpam-cracklib`

However, APT reported that `libpam-cracklib` had no installation candidate on this system. Instead of using Cracklib, I worked with the existing `pam_pwquality` module available on Fileshare.

The PAM password configuration was located at:

`/etc/pam.d/common-password`

The `pam_pwquality` configuration was used to enforce the required password-strength policy, including a minimum password length of 10 characters.

I also updated `/etc/login.defs` so the minimum password age was set to:

`PASS_MIN_DAYS 3`

As with Prod-Joomla, the maximum password age was intentionally left unchanged.

### Fail2Ban

Fail2Ban was installed on the Fileshare server to provide protection against repeated authentication failures.

### Process Accounting

The Lynis audit also identified that process accounting needed to be enabled on Fileshare. I enabled process accounting so system activity could be recorded for auditing and security monitoring purposes.

### Fileshare Verification

After completing the password-policy changes, installing Fail2Ban, and enabling process accounting, the Fileshare security checks reached the **Desired State**.

## Troubleshooting

One of the main challenges during the lab was that the two Linux servers did not support the same password-strength configuration.

On Prod-Joomla, I installed `libpam-cracklib` and configured Cracklib through `/etc/pam.d/common-password`. During the configuration process, I found that both Cracklib and `pam_pwquality` were present, which created overlapping password-strength configurations. I removed the unnecessary `pam_pwquality` packages and configured the system to use Cracklib.

On Fileshare, attempting to install `libpam-cracklib` returned an error stating that the package had no installation candidate. I reviewed the existing PAM configuration and found that Fileshare already used `pam_pwquality`. Instead of forcing the same configuration used on Prod-Joomla, I modified the existing `pam_pwquality` configuration to meet the password-policy requirements.

This required using different PAM implementations on each server while still achieving the same security objective.
