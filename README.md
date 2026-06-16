![Status](https://img.shields.io/badge/status-complete-brightgreen)

## Privileged_Access_Management_PAM_Devolutions_Lab

A cybersecurity lab that demonstrates the deployment and configuration of a Privileged Access Management environment using Devolutions Server, Active Directory, Remote Desktop Manager, Devolutions Gateway, MFA, checkout approval, password rotation, session recording, SQL backup and local workgroup account management.

---

## Project Summary

This project simulates a Privileged Access Management environment built with Devolutions Server. The goal of the lab was to understand how privileged accounts can be stored, controlled, checked out, used for remote sessions and rotated after use.

In the first part of the lab, I connected Devolutions Server with Active Directory, created a PAM vault, configured a Domain User Provider, added a managed privileged domain account, configured heartbeat, enabled checkout/check-in, added approval for privileged access requests and tested password rotation through Remote Desktop Manager. 

In the second part of the lab, I extended the setup by adding a Windows 10 Pro workgroup VM that was not joined to the Active Directory domain. I configured WinRM, TrustedHosts, a Windows User Provider and a local managed account so Devolutions Server PAM could rotate the password of a local Windows user.

The project also includes MFA for Devolutions Server users, Devolutions Gateway as an additional controlled access layer with session recording and SQL database backup for the Devolutions Server environment.
