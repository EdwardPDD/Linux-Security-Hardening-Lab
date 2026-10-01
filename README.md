# Linux Security Hardening Lab

## Project Overview

This project documents a Linux security hardening exercise completed in the XP Cyber Range. The objective was to review Lynis audit findings on two Linux servers, Prod-Joomla and Fileshare, and implement the recommended security changes.

The lab involved strengthening password policies, configuring PAM authentication controls, securing an Apache web server, installing Fail2Ban, and enabling process accounting. Each system required different remediation steps based on its operating system and existing configuration.

## Scenario

A security audit was performed on the organization's Linux systems using Lynis. The audit identified several security weaknesses that required remediation on the Prod-Joomla and Fileshare servers.

The goal was to review the audit findings, determine the appropriate configuration changes, implement the fixes, and verify that the systems reached the required security state.

### Initial Security State

At the beginning of the challenge, all eight security checks were in an **Undesirable State**. These checks covered the Apache security controls and authentication hardening on Prod-Joomla, along with authentication hardening, Fail2Ban, and process accounting on Fileshare.

![Initial challenge checks showing Undesirable State](screenshots/Screenshot%202026-10-01%20104553.png)

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

### Network Topology

The lab environment separated the two target systems across different network segments. Prod-Joomla (`172.16.10.100`) was located in the screened subnet, while Fileshare (`172.16.30.100`) was located in the production subnet.

![Lab Network Topology](screenshots/Screenshot%202026-10-01%20104723.png)

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

### Lynis Audit Findings

The Lynis audit identified several security recommendations for Prod-Joomla, including stronger password controls, additional Apache security protections, and Fail2Ban.

![Prod-Joomla Lynis Audit Findings](screenshots/Screenshot%202026-10-01%20105340.png)

### Password Policy Hardening

The Lynis audit identified weaknesses in the server's password configuration. I installed a PAM password-strength module and updated the password policy to require a minimum password length of 10 characters.

I also updated `/etc/login.defs` and changed the minimum password age from:

`PASS_MIN_DAYS 0`

to:

`PASS_MIN_DAYS 3`

The original configuration showed `PASS_MIN_DAYS` set to `0` before remediation.

![Prod-Joomla original password aging configuration](screenshots/Screenshot%202026-10-01%20110623.png)

The maximum password age was intentionally left unchanged based on the audit instructions.

The PAM password configuration was updated in:

`/etc/pam.d/common-password`

Cracklib was enabled for password-strength checking with a minimum password length of 10 characters.

### Apache Security Hardening

The Prod-Joomla server also required additional protections for the Apache web server. I used APT to identify and install the recommended security modules:

- `mod_evasive` — protection against excessive requests and DoS/brute-force behavior
- `mod_qos` — additional controls against resource-exhaustion and Slowloris-style attacks
- `mod_security` — web application firewall functionality for detecting malicious HTTP requests

I installed the Apache QoS and ModSecurity modules using APT. The screenshot below shows `libapache2-mod-qos` successfully installed and enabled, followed by the installation of `libapache2-modsecurity`.

![Prod-Joomla Apache security module installation](screenshots/Screenshot%202026-10-01%20111956.png)

### Fail2Ban

Before installation, I used `apt search fail2ban` to verify that the Fail2Ban package was available in the configured repositories.

![Prod-Joomla Fail2Ban package search](screenshots/Screenshot%202026-10-01%20112220.png)

Fail2Ban was installed to provide additional protection against repeated authentication failures. The package monitors authentication activity and can temporarily block hosts responsible for repeated failed login attempts.

### Prod-Joomla Verification

After completing the remediation, all five Prod-Joomla security checks reached the **Desired State**.

## Fileshare Remediation

### Lynis Audit Findings

The Fileshare Lynis audit identified recommendations related to password strength, password aging, Fail2Ban, and process accounting.

![Fileshare Lynis Audit Findings](screenshots/Screenshot%202026-10-01%20120329.png)

### Password Policy Hardening

The Fileshare server required similar password-policy changes, but its PAM configuration was different from Prod-Joomla.

I initially attempted to install Cracklib using `sudo apt install libpam-cracklib`, but APT reported that the package had no installation candidate on Fileshare.

![Fileshare Cracklib package unavailable](screenshots/Screenshot%202026-10-01%20120616.png)

Since Cracklib was unavailable, I reviewed `/etc/pam.d/common-password` and confirmed that Fileshare was already configured to use `pam_pwquality` for password-strength checking.

![Fileshare pam_pwquality configuration](screenshots/Screenshot%202026-10-01%20122216.png)

I continued using the existing `pam_pwquality` module as the password-strength control on Fileshare.

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

One of the main challenges during the lab was that the two Linux servers did not support the same password-strength configuration. On Prod-Joomla, the PAM configuration identified that `pam_pwquality` and Cracklib password-strength checking could not be enabled at the same time.

![Prod-Joomla incompatible PAM profiles](screenshots/Screenshot%202026-10-01%20113515.png)

I checked the installed PAM packages and confirmed that both Cracklib and `pam_pwquality` were present on Prod-Joomla.

![Prod-Joomla installed PAM packages](screenshots/Screenshot%202026-10-01%20113715.png)

To resolve the conflict, I removed the `pam_pwquality` packages from Prod-Joomla and verified that Cracklib remained installed. I then located the PAM password configuration file at `/etc/pam.d/common-password`.

![Prod-Joomla PAM conflict resolution](screenshots/Screenshot%202026-10-01%20114918.png)

The final PAM configuration used Cracklib with `retry=3`, `minlen=10`, and `difok=3` to enforce the password-strength requirements.

![Prod-Joomla final Cracklib password policy](screenshots/Screenshot%202026-10-01%20115643.png)

Fileshare required a different approach because Cracklib was unavailable from its configured repositories. Rather than forcing the same configuration on both servers, I used the PAM password-strength module available on each system while still meeting the required security objectives.

## Final Results

After completing the remediation on both servers, I ran the challenge checks to verify the configurations. All eight security checks reached the **Desired State**.

![All security checks at Desired State](screenshots/Screenshot%202026-10-01%20122955.png)

The final submission confirmed a **Full Check Pass (8/8)** for the Engineer's Audit Advice challenge.

![Final challenge submission - 8 of 8](screenshots/Screenshot%202026-10-01%20123825.png)

## Skills Demonstrated

- Linux security hardening
- Lynis audit remediation
- PAM password policy configuration
- Linux password aging controls
- Apache security hardening
- Fail2Ban
- Process accounting
- Linux package management with APT
- Linux configuration file management
- Troubleshooting PAM module conflicts
- Security control validation
