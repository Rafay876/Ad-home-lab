# Active Directory Home Lab

A small Windows domain built from scratch in VirtualBox: a domain controller with AD DS, DNS, DHCP and NAT, a domain-joined Windows 10 client, bulk-created users, and the helpdesk tasks an IT support role does every day (password resets, lockouts, groups, Group Policy).

**Built by Rafay Khan.** I followed Josh Madakor's Active Directory home lab tutorial and added my own admin tasks and troubleshooting notes.

## What this lab covers

- Installing and promoting a Windows Server domain controller (new forest `mydomain.com`)
- DNS, DHCP (172.16.0.100-200) and NAT through Routing and Remote Access
- Organizational Units, a separate admin account and a standard user account
- Bulk user creation with PowerShell
- Joining a Windows 10 client to the domain
- Helpdesk tasks: password reset, disable/enable, account lockout and unlock
- Security groups and Group Policy (lockout policy, blocking Control Panel for the IT OU)
- Troubleshooting real problems I hit along the way

## Lab environment

| Item | Value |
|---|---|
| Hypervisor | Oracle VirtualBox |
| Domain controller | Windows Server 2022 (evaluation), hostname `DC` |
| Client | Windows 10 Pro, hostname `Client1` |
| Domain | `mydomain.com` (NetBIOS `MYDOMAIN`) |
| Internal network | 172.16.0.0/24, DC at 172.16.0.1 |
| DHCP scope | 172.16.0.100 to 172.16.0.200 |
| NAT | Outside: `_INTERNET_` (VirtualBox NAT), inside: `_INTERNAL_` |

## Contents

1. [Architecture](#1-architecture)
2. [VM setup](#2-vm-setup)
3. [Domain controller](#3-domain-controller)
4. [NAT and DHCP](#4-nat-and-dhcp)
5. [Users and OUs](#5-users-and-ous)
6. [Joining the client](#6-joining-the-client)
7. [Admin tasks](#7-admin-tasks)
8. [Troubleshooting](#8-troubleshooting)
9. [What I learned](#9-what-i-learned)

---

## 1. Architecture

<p align="center"><img src="screenshots/01-architecture/01-lab-architecture.png" width="640" alt="Lab architecture (diagram from the tutorial; I built it in VirtualBox)"><br><em>Lab architecture (diagram from the tutorial; I built it in VirtualBox)</em></p>

*Diagram is from the tutorial. It labels the network as VMware and the server as 2019, but I built this in VirtualBox. The DHCP gateway on it is a typo; mine is 172.16.0.1.*

## 2. VM setup

The DC has two adapters (one NAT for internet, one internal network). The client only has the internal network, so it gets everything from the DC.

<p align="center"><img src="screenshots/02-vm-setup/01-client-network-adapter.png" width="640" alt="Client VM: single adapter on the internal network (intnet)"><br><em>Client VM: single adapter on the internal network (intnet)</em></p>

## 3. Domain controller

Installed the AD DS role, then promoted the server to a domain controller in a new forest.

<p align="center"><img src="screenshots/03-dc-config/01-add-roles-adds.png" width="640" alt="Add Roles and Features: Active Directory Domain Services selected"><br><em>Add Roles and Features: Active Directory Domain Services selected</em></p>

<p align="center"><img src="screenshots/03-dc-config/02-adds-install-succeeded.png" width="640" alt="AD DS role installed, ready to promote to a domain controller"><br><em>AD DS role installed, ready to promote to a domain controller</em></p>

<p align="center"><img src="screenshots/03-dc-config/03-post-deployment-notification.png" width="640" alt="Post-deployment configuration notice in Server Manager"><br><em>Post-deployment configuration notice in Server Manager</em></p>

<p align="center"><img src="screenshots/03-dc-config/04-new-forest.png" width="640" alt="Promotion wizard: new forest, root domain mydomain.com"><br><em>Promotion wizard: new forest, root domain mydomain.com</em></p>

<p align="center"><img src="screenshots/03-dc-config/05-review-options.png" width="640" alt="Review options: NetBIOS name MYDOMAIN, DNS server and global catalog enabled"><br><em>Review options: NetBIOS name MYDOMAIN, DNS server and global catalog enabled</em></p>

<p align="center"><img src="screenshots/03-dc-config/06-domain-login-screen.png" width="640" alt="After the reboot the login screen shows MYDOMAIN\Administrator"><br><em>After the reboot the login screen shows MYDOMAIN\Administrator</em></p>

<p align="center"><img src="screenshots/03-dc-config/07-server-manager-adds-dns.png" width="640" alt="Server Manager: AD DS and DNS running"><br><em>Server Manager: AD DS and DNS running</em></p>

## 4. NAT and DHCP

Routing and Remote Access gives the client internet access through the DC. DHCP hands the client its address.

<p align="center"><img src="screenshots/04-nat-dhcp/01-remote-access-role.png" width="640" alt="Remote Access role selected"><br><em>Remote Access role selected</em></p>

<p align="center"><img src="screenshots/04-nat-dhcp/02-remote-access-role-services.png" width="640" alt="Role services: DirectAccess and VPN (RAS) and Routing"><br><em>Role services: DirectAccess and VPN (RAS) and Routing</em></p>

<p align="center"><img src="screenshots/04-nat-dhcp/03-remote-access-install.png" width="640" alt="Remote Access installation on the DC"><br><em>Remote Access installation on the DC</em></p>

<p align="center"><img src="screenshots/04-nat-dhcp/04-rras-console.png" width="640" alt="Routing and Remote Access console"><br><em>Routing and Remote Access console</em></p>

<p align="center"><img src="screenshots/04-nat-dhcp/05-rras-nat-wizard.png" width="640" alt="RRAS setup wizard: Network Address Translation (NAT)"><br><em>RRAS setup wizard: Network Address Translation (NAT)</em></p>

<p align="center"><img src="screenshots/04-nat-dhcp/06-nat-interfaces.png" width="640" alt="NAT interfaces: _INTERNAL_ (172.16.0.1) and _INTERNET_ (10.0.2.15, from DHCP)"><br><em>NAT interfaces: _INTERNAL_ (172.16.0.1) and _INTERNET_ (10.0.2.15, from DHCP)</em></p>

<p align="center"><img src="screenshots/04-nat-dhcp/07-rras-configured.png" width="640" alt="RRAS configured on the DC"><br><em>RRAS configured on the DC</em></p>

<p align="center"><img src="screenshots/04-nat-dhcp/08-dhcp-role.png" width="640" alt="DHCP Server role selected"><br><em>DHCP Server role selected</em></p>

<p align="center"><img src="screenshots/04-nat-dhcp/09-dhcp-install-results.png" width="640" alt="DHCP role installed"><br><em>DHCP role installed</em></p>

<p align="center"><img src="screenshots/04-nat-dhcp/10-dhcp-ip-range.png" width="640" alt="DHCP scope: 172.16.0.100 to 172.16.0.200, /24"><br><em>DHCP scope: 172.16.0.100 to 172.16.0.200, /24</em></p>

<p align="center"><img src="screenshots/04-nat-dhcp/11-dhcp-dns-settings.png" width="640" alt="Scope DNS settings: parent domain mydomain.com, DNS server 172.16.0.1"><br><em>Scope DNS settings: parent domain mydomain.com, DNS server 172.16.0.1</em></p>

<p align="center"><img src="screenshots/04-nat-dhcp/12-dhcp-scope.png" width="640" alt="DHCP console with the scope created"><br><em>DHCP console with the scope created</em></p>

## 5. Users and OUs

Created a separate admin account in its own OU (so day-to-day admin work does not use the built-in Administrator), then bulk-created standard users with a PowerShell script.

<p align="center"><img src="screenshots/05-users-ous/01-aduc-empty-domain.png" width="640" alt="Active Directory Users and Computers on a fresh domain"><br><em>Active Directory Users and Computers on a fresh domain</em></p>

<p align="center"><img src="screenshots/05-users-ous/02-admins-ou.png" width="640" alt="_ADMINS OU created for administrator accounts"><br><em>_ADMINS OU created for administrator accounts</em></p>

<p align="center"><img src="screenshots/05-users-ous/03-new-admin-user.png" width="640" alt="New user: separate admin account (a-Rafay)"><br><em>New user: separate admin account (a-Rafay)</em></p>

<p align="center"><img src="screenshots/05-users-ous/04-new-admin-password.png" width="640" alt="Password options (password never expires, lab only)"><br><em>Password options (password never expires, lab only)</em></p>

<p align="center"><img src="screenshots/05-users-ous/05-new-admin-summary.png" width="640" alt="New object summary"><br><em>New object summary</em></p>

<p align="center"><img src="screenshots/05-users-ous/06-add-to-domain-admins.png" width="640" alt="Adding the account to Domain Admins"><br><em>Adding the account to Domain Admins</em></p>

<p align="center"><img src="screenshots/05-users-ous/07-member-of-domain-admins.png" width="640" alt="Member Of tab: Domain Admins"><br><em>Member Of tab: Domain Admins</em></p>

<p align="center"><img src="screenshots/05-users-ous/08-login-as-admin.png" width="640" alt="Logging in with the new domain admin account"><br><em>Logging in with the new domain admin account</em></p>

<p align="center"><img src="screenshots/05-users-ous/09-powershell-bulk-users.png" width="640" alt="PowerShell script bulk-creating users from names.txt"><br><em>PowerShell script bulk-creating users from names.txt</em></p>

<p align="center"><img src="screenshots/05-users-ous/10-users-ou-populated.png" width="640" alt="_USERS OU populated by the script"><br><em>_USERS OU populated by the script</em></p>

## 6. Joining the client

The client was joined with the domain admin account, then I logged in as a normal domain user.

<p align="center"><img src="screenshots/06-client-join/01-client1-computer-account.png" width="640" alt="CLIENT1 computer account in the Computers container"><br><em>CLIENT1 computer account in the Computers container</em></p>

<p align="center"><img src="screenshots/06-client-join/02-domain-join.png" width="640" alt="Client1 joined to mydomain.com"><br><em>Client1 joined to mydomain.com</em></p>

<p align="center"><img src="screenshots/06-client-join/03-domain-login.png" width="640" alt="Logging in as a standard domain user (MYDOMAIN\dearls)"><br><em>Logging in as a standard domain user (MYDOMAIN\dearls)</em></p>

<p align="center"><img src="screenshots/06-client-join/04-whoami.png" width="640" alt="whoami confirms mydomain\dearls"><br><em>whoami confirms mydomain\dearls</em></p>

<p align="center"><img src="screenshots/06-client-join/05-client-ipconfig.png" width="640" alt="Client got 172.16.0.100 from the DC&#x27;s DHCP (DNS suffix mydomain.com, gateway 172.16.0.1)"><br><em>Client got 172.16.0.100 from the DC&#x27;s DHCP (DNS suffix mydomain.com, gateway 172.16.0.1)</em></p>

## 7. Admin tasks

Each task was done in Active Directory Users and Computers on the DC and verified from the client.

#### 7.1 Password reset

<p align="center"><img src="screenshots/07-admin-tasks/01-password-reset/01-reset-password-dialog.png" width="640" alt="Reset Password dialog in ADUC"><br><em>Reset Password dialog in ADUC</em></p>

<p align="center"><img src="screenshots/07-admin-tasks/01-password-reset/02-password-changed.png" width="640" alt="Confirmation that the password was changed"><br><em>Confirmation that the password was changed</em></p>

#### 7.2 Disable and enable an account

<p align="center"><img src="screenshots/07-admin-tasks/02-disable-enable/01-disable-account.png" width="640" alt="Disable Account from the right-click menu"><br><em>Disable Account from the right-click menu</em></p>

<p align="center"><img src="screenshots/07-admin-tasks/02-disable-enable/02-account-disabled.png" width="640" alt="Account disabled"><br><em>Account disabled</em></p>

<p align="center"><img src="screenshots/07-admin-tasks/02-disable-enable/03-client-login-blocked.png" width="640" alt="Client login refused: account disabled"><br><em>Client login refused: account disabled</em></p>

<p align="center"><img src="screenshots/07-admin-tasks/02-disable-enable/04-enable-account.png" width="640" alt="Enable Account from the right-click menu"><br><em>Enable Account from the right-click menu</em></p>

<p align="center"><img src="screenshots/07-admin-tasks/02-disable-enable/05-account-enabled.png" width="640" alt="Account enabled again"><br><em>Account enabled again</em></p>

#### 7.3 Account lockout and unlock

AD does not lock accounts by default, so I set a lockout policy in the Default Domain Policy first (3 invalid attempts, 1-minute lockout), then triggered it from the client and unlocked the account.

<p align="center"><img src="screenshots/07-admin-tasks/03-lockout-unlock/01-group-policy-management.png" width="640" alt="Group Policy Management: Default Domain Policy"><br><em>Group Policy Management: Default Domain Policy</em></p>

<p align="center"><img src="screenshots/07-admin-tasks/03-lockout-unlock/02-account-policies.png" width="640" alt="Editing Account Policies &gt; Account Lockout Policy"><br><em>Editing Account Policies &gt; Account Lockout Policy</em></p>

<p align="center"><img src="screenshots/07-admin-tasks/03-lockout-unlock/03-lockout-threshold.png" width="640" alt="Account lockout threshold set to 3 invalid attempts"><br><em>Account lockout threshold set to 3 invalid attempts</em></p>

<p align="center"><img src="screenshots/07-admin-tasks/03-lockout-unlock/04-lockout-policy-summary.png" width="640" alt="Lockout policy summary: threshold 3, 1-minute duration and reset"><br><em>Lockout policy summary: threshold 3, 1-minute duration and reset</em></p>

<p align="center"><img src="screenshots/07-admin-tasks/03-lockout-unlock/05-gpupdate.png" width="640" alt="gpupdate /force to apply the policy"><br><em>gpupdate /force to apply the policy</em></p>

<p align="center"><img src="screenshots/07-admin-tasks/03-lockout-unlock/06-client-account-locked-out.png" width="640" alt="Client login after wrong passwords: account locked out"><br><em>Client login after wrong passwords: account locked out</em></p>

<p align="center"><img src="screenshots/07-admin-tasks/03-lockout-unlock/07-unlock-account.png" width="640" alt="Unlock account checkbox on the user&#x27;s Account tab in ADUC"><br><em>Unlock account checkbox on the user&#x27;s Account tab in ADUC</em></p>

#### 7.4 OUs, moving a user and a security group

<p align="center"><img src="screenshots/07-admin-tasks/04-ous-and-groups/01-new-ou.png" width="640" alt="Creating the IT OU"><br><em>Creating the IT OU</em></p>

<p align="center"><img src="screenshots/07-admin-tasks/04-ous-and-groups/02-ous-created.png" width="640" alt="IT, HR and Sales OUs created"><br><em>IT, HR and Sales OUs created</em></p>

<p align="center"><img src="screenshots/07-admin-tasks/04-ous-and-groups/03-user-moved-to-it-ou.png" width="640" alt="User moved into the IT OU"><br><em>User moved into the IT OU</em></p>

<p align="center"><img src="screenshots/07-admin-tasks/04-ous-and-groups/04-new-security-group.png" width="640" alt="New group IT-Staff (Global, Security)"><br><em>New group IT-Staff (Global, Security)</em></p>

<p align="center"><img src="screenshots/07-admin-tasks/04-ous-and-groups/05-add-group-member.png" width="640" alt="Adding dearls to the group"><br><em>Adding dearls to the group</em></p>

<p align="center"><img src="screenshots/07-admin-tasks/04-ous-and-groups/06-group-members.png" width="640" alt="IT-Staff Members tab"><br><em>IT-Staff Members tab</em></p>

#### 7.5 Group Policy

Created a GPO linked to the IT OU that blocks Control Panel, then confirmed it on the client as a user in that OU.

<p align="center"><img src="screenshots/07-admin-tasks/05-group-policy/01-create-gpo-linked-to-it.png" width="640" alt="Creating a GPO linked to the IT OU"><br><em>Creating a GPO linked to the IT OU</em></p>

<p align="center"><img src="screenshots/07-admin-tasks/05-group-policy/02-block-control-panel-enabled.png" width="640" alt="Prohibit access to Control Panel and PC settings: Enabled"><br><em>Prohibit access to Control Panel and PC settings: Enabled</em></p>

<p align="center"><img src="screenshots/07-admin-tasks/05-group-policy/03-client-gpupdate.png" width="640" alt="gpupdate /force on the client as dearls"><br><em>gpupdate /force on the client as dearls</em></p>

<p align="center"><img src="screenshots/07-admin-tasks/05-group-policy/04-control-panel-blocked.png" width="640" alt="Control Panel blocked by the policy on the client"><br><em>Control Panel blocked by the policy on the client</em></p>

## 8. Troubleshooting

### 8.1 PowerShell script would not run

**Symptom:** `1_CREATE_USERS.ps1` was "not recognized as the name of a cmdlet, function, script file".

**Cause:** I was calling it with a path under Downloads, but the files were on the Desktop.

**Fix:** `cd` into the script's folder and run `.\1_CREATE_USERS.ps1` so it also finds `names.txt`.

<p align="center"><img src="screenshots/08-troubleshooting/01-script-wrong-path.png" width="640" alt="PowerShell script run from the wrong path"><br><em>PowerShell script run from the wrong path</em></p>

### 8.2 Client dropped out of the domain

**Symptom:** I accidentally set the client to a workgroup while renaming it. Afterwards the domain option was greyed out and domain logins failed.

**Cause:** the rename was pending a restart (so the domain field is locked), and leaving the domain broke the client's link to its machine account.

**Fix:** restart to apply the rename, move the client to a workgroup, restart again, then rejoin `mydomain.com` using the domain admin account (a standard user cannot join computers), and log in as the standard user afterwards.

<p align="center"><img src="screenshots/08-troubleshooting/02-client-rename-pending-restart.png" width="640" alt="Client: domain option greyed out until the pending rename is restarted"><br><em>Client: domain option greyed out until the pending rename is restarted</em></p>
<p align="center"><img src="screenshots/08-troubleshooting/03-client-leave-domain-workgroup.png" width="640" alt="Client: moving to a workgroup before rejoining the domain cleanly"><br><em>Client: moving to a workgroup before rejoining the domain cleanly</em></p>

## 9. What I learned

- Rename a server **before** promoting it to a domain controller, and never rename it afterwards.
- Take VM snapshots at working points (after the DC is up, after the client joins) so a mistake does not mean a rebuild.
- Use a separate admin account for administration and standard accounts for everyday logins; only an admin account can join computers to the domain.
- Group Policy only applies to the OU it is linked to, so users have to be in the right OU.

## Notes

- All accounts and passwords in this repo are for an isolated lab. The user-creation script uses the tutorial's default password, so do not reuse it anywhere real.
- Credit: Josh Madakor's Active Directory home lab tutorial for the lab design and the user-creation script.
