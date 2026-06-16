![Status](https://img.shields.io/badge/status-complete-brightgreen)

## Privileged_Access_Management_PAM_Devolutions_Lab

A cybersecurity lab that demonstrates the deployment and configuration of a Privileged Access Management environment using Devolutions Server, Active Directory, Remote Desktop Manager, Devolutions Gateway, MFA, checkout approval, password rotation, session recording, SQL backup and local workgroup account management.

---

## Project Summary

This project simulates a Privileged Access Management environment built with Devolutions Server. The goal of the lab was to understand how privileged accounts can be stored, controlled, checked out, used for remote sessions and rotated after use.

In the first part of the lab, I connected Devolutions Server with Active Directory, created a PAM vault, configured a Domain User Provider, added a managed privileged domain account, configured heartbeat, enabled checkout/check-in, added approval for privileged access requests and tested password rotation through Remote Desktop Manager. 

In the second part of the lab, I extended the setup by adding a Windows 10 Pro workgroup VM that was not joined to the Active Directory domain. I configured WinRM, TrustedHosts, a Windows User Provider and a local managed account so Devolutions Server PAM could rotate the password of a local Windows user.

The project also includes MFA for Devolutions Server users, Devolutions Gateway as an additional controlled access layer with session recording and SQL database backup for the Devolutions Server environment.

---

## Lab Architecture

The lab was built using virtual machines in Hyper-V.

| Machine             | Operating System       | Role                                                                 |
| ------------------- | ---------------------- | -------------------------------------------------------------------- |
| `Domain Controller` | Windows Server 2019       | Active Directory Domain Services, domain users and delegated permissions       |
| `DVLS-SERVER`       | Windows Server 2019        | Devolutions Server, IIS, SQL connection, PAM configuration and backup          |
| `VM-WORK`           | Windows 10 Pro         | Workgroup VM used for local account PAM integration and password rotation      |
| `VM-ADMIN-RDM`            | Windows Server 2019       | Remote Desktop Manager used to open PAM controlled sessions             |

The lab included two main PAM scenarios:

| Scenario | Target Account Type | Provider Used | Main Purpose |
| -------- | ------------------- | ------------- | ------------ |
| Domain PAM | Active Directory domain user | Domain User Provider | Manage and rotate a privileged domain account |
| Workgroup PAM | Local Windows user | Windows User Provider | Manage and rotate a local account on a non domain VM |

For the workgroup VM, two separate connections were used:

| Connection | Purpose |
| ---------- | ------- |
| RDP 3389 | Used by RDM to open the remote desktop session |
| WinRM 5985 | Used by Devolutions Server PAM for heartbeat and password rotation |

The important distinction is that RDM opens the session, while Devolutions Server PAM performs the password rotation.

---

## PAM Flow

The general PAM flow in this lab is:

User → DVLS - RDM → Checkout Request → Approval → Privileged Session → Check-in → Password Rotation

For domain accounts:

DVLS PAM → Domain Provider → Active Directory → Managed Domain Account

For workgroup local accounts:

DVLS PAM → Windows User Provider → WinRM 5985 → Local Windows Account

---
## What I Did

### 1. Devolutions Server Installation and IIS Instance
- Installed Devolutions Server Console.
- Created and configured an IIS instance for Devolutions Server.
- Verified that the Devolutions Server web interface was reachable from the browser.
- Then I was ready to continue the PAM configuration from the DVLS web interface.

IIS was used to host the Devolutions Server web application. The Devolutions Server data was stored separately in the SQL Server database.

### 2. Active Directory User Preparation

  I created two main Active Directory users for the domain PAM scenario: 

  - `pamtest3` = Managed privileged domain account
  - `svr-dvls-pam` = Provider/service account used by DVLS for password rotation.

  The pamtest3 account was the privileged account that was added to the PAM vault. This is the account that users check out and use for remote access. The svr-dvls-pam account was used as the provider account. This account was given the required delegated permissions in Active Directory so Devolutions Server could rotate   the password of the managed account.

### 3. Delegated Permissions in Active Directory

- Assigned delegated permissions to the provider account.
- Allowed the provider account to manage or reset the password of the managed account.
- Confirmed the relationship between the accounts:
              Managed Account: `DOMAIN\pamtest3`
              Provider Account: `DOMAIN\svr-dvls-pam`

### 4. Domain Authentication in Devolutions Server

- Configured Domain Authentication in Devolutions Server.
- Connected DVLS with the Active Directory domain.
- Allowed DVLS to recognize domain users and groups.
- Verified that Active Directory users could be used inside DVLS.

This allowed users from Active Directory to log in to the Devolutions Server web interface instead of relying only on local DVLS users.

### 5. Logging in to DVLS with an Active Directory User

After configuring Domain Authentication, I tested logging in to the DVLS web interface using an Active Directory user: `aksi3`.

Using Active Directory users makes access management cleaner because authentication can be centralized through the domain. This shows that DVLS can use the existing domain identity system. My example login: `BLUEVALUE\aski3`

### 6. MFA for Devolutions Server Users

- Enabled MFA for selected DVLS users.
- Configured TOTP authentication.
- Tested the login flow through the Devolutions Server web interface and Verified that users use both their credentials and the MFA code to access DVLS.

MFA adds another security layer because access to Devolutions Server is not protected only by username and password.

### 7. PAM Domain Provider Configuration

- Configured and connected the PAM provider with the delegated provider/service account: `svr-dvls-pam`.
- Used the provider for domain account management, heartbeat and password rotation.

The PAM provider is the component that allows Devolutions Server to perform privileged account management operations against Active Directory accounts.

### 8. PAM Vault Creation

- Created a PAM vault.
- Used the PAM vault to store and manage privileged accounts.
- Assigned the vault to the correct Domain Provider.
- Defined the administrator of the vault (`aski3` and the default `dvls-admin`).

A PAM vault was required to manage privileged accounts with password rotation.

### 9. Adding the Managed Domain Account to the PAM Vault

- Added the domain user `pamtest3` to the PAM vault.
- Tested the connection with heartbeat to confirm the successful integration of the privileged account.

After the account was added, Devolutions Server became responsible for managing its password lifecycle. Heartbeat was used to confirm that the managed account was healthy and that DVLS could validate the credential.
