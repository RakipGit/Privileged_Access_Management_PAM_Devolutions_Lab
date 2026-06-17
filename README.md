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

### 10. Checkout, Check-in and Password Rotation

- Configured checkout for the managed account and its password rotation.
- Enabled rotation after check in of the new password when the user logs out of the machine that he used PAM with the privileged account.
- Verified that the old password was no longer valid after rotation.

The checkout/check-in process works as follows:

a. The user requests access to the managed account.

b. The account is checked out for temporary use.

c. The user opens the remote session.

d. After the session is finished, the account is checked in.

e. DVLS rotates the password according to the configured policy.

f. The new password is stored securely in the vault.

This reduces the risk of static or reused privileged passwords.

### 11. Checkout Approval Workflow

- Configured approval for checkout requests.
- Assigned an approver: `supervisor1`.
- Tested the approval flow from Remote Desktop Manager (RDM) and the DVLS web interface.

When a user requests access to the privileged account, the request is not automatically approved. An approver can review the request and either approve or deny it before the user gets access.

This adds an additional control point before privileged access is granted.

### 12. Remote Desktop Manager 

- Installed Remote Desktop Manager.
- Created a Devolutions Server data source to connect RDM to the DVLS server.
- Verified that vaults and entries from DVLS appeared in RDM.

Remote Desktop Manager was used as the main client application for opening remote sessions.

Through RDM, users can:

- Open RDP sessions ny using the privileged accounts credentials stored in DVLS.
- Access only the entries they are allowed to see.
- Request checkout for privileged accounts.
- Open controlled remote sessions without manually typing the privileged password, just straight logging in.
- Use and configure the DVLS vault structure directly from the RDM interface.

### 13. Testing Domain PAM Through an RDP Entry

To test the PAM setup, I created a new RDP Windows entry in Remote Desktop Manager. The RDP entry pointed to a Windows VM and was configured to use as loggin the managed privileged account `pamtest3`.

The flow was:

a) Open Remote Desktop Manager.
b) Connect to the DVLS data source.
c) Select the PAM vault.
d) Open the RDP entry.
e) Request checkout for the privileged account.
f) Wait for approval.
g) Open the RDP session as the managed privileged user.
h) Log off from the session.
j) Check in the account.
k) Verify password rotation from DVLS logs.

### 14. Workgroup VM PAM Integration

After completing the domain based PAM configuration, I extended the lab by adding a Windows 10 Pro workgroup VM that was not joined to the Active Directory domain. The goal was to test whether Devolutions Server PAM could also manage and rotate the password of a local Windows account.

For this scenario, I used two local accounts on the workgroup VM:

- `dvls-mgmt` = Local administrator account used by DVLS for management operations.
- `poppi` = Local managed account added to the PAM vault.

The dvls-mgmt account was used by Devolutions Server through the Windows User Provider and the poppi account was the local privileged account whose password was rotated by PAM.

### 15. Workgroup VM Communication Flow

For the workgroup VM, two separate connections were used:

| Connection | Purpose |
| ---------- | ------- |
| RDP 3389 | Used by RDM to open the remote desktop session |
| WinRM 5985 | Used by Devolutions Server PAM for the password rotation |

Remote Desktop Manager does not rotate the password. RDM only uses the current password stored in PAM to open the session. Devolutions Server PAM performs the actual password rotation. RDP was used only to open the remote desktop session while WinRM was used by Devolutions Server PAM to perform heartbeat checks and rotate the local user password.

### 16. Creating Local Users on the Workgroup VM

On the workgroup VM, I created two local users.

The first user was the local management account:

```powershell
net user dvls-mgmt Password /add<br>
net localgroup administrators dvls-mgmt /add
```

The second user was the local managed account:

```powershell
net user poppi Password /add<br>
net localgroup administrators poppi /add
```

The `dvls-mgmt` account remains the local administrator account used by DVLS for management operations. The `poppi` account is the managed local account whose password is rotated by PAM.

### 17. Enabling RDP on the Workgroup VM

- Enabled Remote Desktop on the workgroup VM.
- Allowed RDP through the firewall.
- Confirmed that the VM could accept RDP connections on port 3389.

The RDP connection was later used by Remote Desktop Manager to open the session as the managed local account.

### 18. Enabling WinRM on the Workgroup VM

To allow Devolutions Server PAM to manage the local user password, WinRM had to be enabled on the workgroup VM.<br>

On the workgroup VM, I ran: 

```powershell
powershell Enable-PSRemoting -Force
```

Then I enabled the firewall rule for Windows Remote Management: 

```powershell
Enable-NetFirewallRule -DisplayGroup "Windows Remote Management
```

### 19. Adding the Workgroup VM to TrustedHosts

To allow WinRM communication, I added the workgroup VM IP address to TrustedHosts on the Devolutions Server machine.

On the Devolutions Server VM, I ran:

```powershell
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "192.168.10.x" -Force
``` 
Then I verified the TrustedHosts configuration: 
```powershell
Get-Item WSMan:\localhost\Client\TrustedHosts
```

This allowed the Devolutions Server VM: `dvls-server` to trust the workgroup VM for WinRM-based management.

### 20. Testing WinRM Connectivity

From the Devolutions Server machine, I tested whether port 5985 was reachable: 

```powershell
Test-NetConnection 192.168.10.x -Port 5985
```
The result was: 

```powershell
TcpTestSucceeded : True
```
I also tested a remote PowerShell session:
```powershell
Enter-PSSession -ComputerName 192.168.10.x -Credential WORKGROUPVM\dvls-mgmt
```
This confirmed that the Devolutions Server machine could reach and authenticate to the workgroup VM through WinRM.


### 21. Creating a Windows User Provider in Devolutions Server

After confirming WinRM connectivity, I created a Windows User Provider in Devolutions Server:  `dvls-mgmt` .

The path was:

Administration → Privileged Access → Providers → Add → Windows User

After saving the provider, I tested the connection and confirmed that it was successful. 



### 22. Adding the Local Workgroup User to the PAM Vault

- Created PAM vault for the workgroup and linked the account to the Windows User Provider : `dvls-mgmt` .
- Added the local user poppi as a PAM Windows User with its password. 
- Ran heartbeat to verify that the account was accessible.

After the first successful rotation, the original password was no longer used. DVLS became responsible for storing and managing the current password.


### 23. Testing Workgroup PAM Through RDM

After adding the local account to the PAM vault, I created an RDP entry in Remote Desktop Manager for the workgroup VM.

The testing flow was:

a. The user opens the workgroup VM entry in RDM.

b. RDM sends a checkout request to Devolutions Server.

c. The approver approves the request.

d. RDM retrieves the current password from DVLS.

e. RDM opens an RDP connection to the workgroup VM (port:3389).

f. The session opens as the local user `poppi`.

g. After logoff/check-in, Devolutions Server rotates the password (port:5985).

h. The old password is no longer valid.

This proved that Devolutions Server PAM can manage not only Active Directory accounts but also local users on standalone workgroup Windows machines.

### 24. Devolutions Gateway

Devolutions Gateway was included as an additional access component. The Gateway can be used as an intermediate connection point between Remote Desktop Manager / Devolutions Server and target systems. 

Instead of connecting directly to a target VM or server, the session can pass through the Gateway: 

RDM / DVLS → Devolutions Gateway → Target VM / Server / Machine

The Gateway does not replace PAM. It complements PAM by providing a controlled path for remote connections.

### 25. Session Recording

Devolutions Gateway can also support session recording when configured. Session recording allows privileged remote sessions to be recorded for audit and review purposes. This improves accountability because administrators can later review what happened during a privileged session.

The general idea is:

a) User opens session from RDM.

b) DVLS checks permissions and PAM access.

c) Connection passes through Gateway.

d) Session recording captures the privileged session.

e) User connects to the target system.

### 26. Devolutions Server Backup

I created a backup of the Devolutions Server SQL database using SQL Server Management Studio.

This is important because the DVLS database contains important configuration and operational data, such as: 
- Vaults
- Entries
- Users
- Permissions
- PAM configuration
- Provider configuration
- Audit and history data

For a more complete recovery plan, the Devolutions Server application files and configuration should also be considered in addition to the SQL database backup.

---
## Screenshots

![Active Directory Lab Architecture](images/LAB-ARCHITECTURE.png)

<details>
<summary>🔎 View Full Lab Walkthrough (Screenshots)</summary>

### 1. Devolutions Server Console and IIS Instance

![Server Console](images/IIS1.png)
![Server Console](images/IISDB.png)
![Server Console](images/DVLS-CONSOLE.png)

### 2. Domain Authentication Configuration

![Domain Authentication](images/Auth.png)
![Domain Authentication](images/Auth2.png)

### 3. Logging in to DVLS with Active Directory User

![Domain User Integration into DVLS Server](images/User1.png)
![Domain User Integration into DVLS Server](images/User2.png)

### 4. MFA Configuration

![MFA](images/MFA2.png)
![MFA](images/MFA3.png)
![MFA](images/MFA4.png)

![MFA](images/MFA1.png)
![MFA](images/MFA5.png)

![MFA](images/LOG1.png)
![MFA](images/LOG2.png)
![MFA](images/LOG3.png)



### 5. PAM Domain Provider

![Provider](images/PRO1.png)
![Provider](images/PRO2.png)
![Provider](images/PRO3.png)
![Provider](images/PRO4.png)
![Provider](images/PRO5.png)

### 6. PAM Vault Creation

![Vault Creation](images/VAULT1.png)
![Vault Creation](images/VAULT2.png)
![Vault Creation](images/VAULT3.png)
![Vault Creation](images/VAULT4.png)
![Vault Creation](images/VAULT5.png)

### 7. Managed Domain Account

![Adding privileged account](images/ADDV1.png)
![Adding privileged account](images/ADDV2.png)
![Adding privileged account](images/ADDV3.png)
![Adding privileged account](images/ADDV4.png)

### 8. Heartbeat Validation

![Heartbeat](images/HEARTBEAT1.png)
![Heartbeat](images/HEARTBEAT2.png)

### 9. Checkout and Password Rotation Policy

![Policy](images/CHECK4.png)
![Policy](images/CHECK1.png)
![Policy](images/CHECK2.png)
![Policy](images/CHECK3.png)

</details>

---

## Tools & Technologies
- Hyper-V for VMs: Windows Server and Windows 10 Pro
- Devolutions Server
- Devolutions Server Console
- Devolutions Gateway
- Devolutions Remote Desktop Manager
- Devolutions Gateway with Session Recording
- PAM Vaults for Password Rotation
- SQL Server Management Studio for the Devolutions Server Backup
- Active Directory Users and Computers
- Assigning Group delegated permissions
- Opening ports 3389 and 5985
- PowerShell

## Security Concepts Demonstrated

- Privileged Access Management
- Centralized privileged account control
- Active Directory integration
- Domain authentication
- Managed privileged accounts
- Provider account integation
- Delegated permissions in ADUC
- Password rotation in DVLS 
- Heartbeat validation DVLS Server
- Checkout and check in workflow
- Approval based privileged access
- Multi Factor Authentication
- Remote session management with DVLS RDP
- Workgroup local account management
- WinRM local user remote management
- Powershell configurations
- Session recording
- Audit trail and accountability by viewing logs into DVLS Server
- Backup and recovery planning

## Key Powershell Commands Used

Workgroup VM Local User Creation 

```powershell
net user dvls-mgmt Password /add
net localgroup administrators dvls-mgmt /add
```
and 

```powershell
net user poppi Password /add
net localgroup administrators poppi /add
```

Enable WinRM on Workgroup VM

```powershell
Enable-PSRemoting -Force
Enable-NetFirewallRule -DisplayGroup "Windows Remote Management"
```

Add Workgroup VM to TrustedHosts

```powershell
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "192.168.10.x" -Force
Get-Item WSMan:\localhost\Client\TrustedHosts
```
Test WinRM Connectivity

```powershell
Test-NetConnection 192.168.10.x -Port 5985
Enter-PSSession -ComputerName 192.168.10.x -Credential WORKGROUPVM\dvls-mgmt
```

---

## Insights & Lessons Learned

- Building this lab helped me understand how Devolutions Server PAM works in practice.
- Configuring Devolutions Server PAM with Active Directory showed how Devolutions can manage privileged domain accounts through a PAM vault and provider.
- Creating a provider account demonstrated why delegated permissions are required for Devolutions Server PAM to rotate Active Directory account passwords.
- Heartbeat validation helped me understand how Devolutions Server PAM checks whether the stored credential is still valid and synchronized with the real account password.
- Checkout and check in showed how Devolutions Server PAM can provide temporary privileged access instead of exposing static credentials to users.
- Password rotation demonstrated how Devolutions Server PAM reduces the risk of reused or permanently known privileged passwords.
- Approval workflows showed how Devolutions Server PAM can require human approval before privileged access is granted.
- MFA improved the security of the Devolutions Server web interface by adding a second authentication factor.
- Remote Desktop Manager showed how Devolutions Server PAM can be used in practice to open controlled sessions without manually typing privileged passwords.
- The workgroup VM scenario helped me understand how Devolutions Server PAM can also manage local users on non domain Windows machines.
- Configuring WinRM and TrustedHosts showed the difference between domain-based PAM management and workgroup-based PAM management.
- Devolutions Gateway and session recording introduced the idea of controlled access paths and session accountability inside the Devolutions ecosystem.
- Creating a SQL backup showed the importance of protecting the Devolutions Server database because it stores vaults, entries, permissions, PAM configuration, and audit related data.

---

## Copyright Notice

All content and visuals in this repository are original and may not be reused without permission.


## Rakip 

ICT Engineering | Cybersecurity & Network Security

---

