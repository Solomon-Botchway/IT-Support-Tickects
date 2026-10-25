# IT-Support-Tickects
# SoloTech Active Directory & IT Support Lab

> A practical Windows Server, Active Directory, file services, endpoint support, monitoring and security lab built to demonstrate IT Support skills and a foundation for cybersecurity.

## Overview

This project simulates IT Support work for **SoloTech**, a fictional company running the `solotech.local` Active Directory domain. The lab grew from a domain controller and one client into a multi machine environment with a file server, Linux domain member, Group Policy, central event collection, network shares, printers and an internal intranet.

The tickets below record the issue, investigation, action, verification and learning. User names and business data are fictional lab data. Passwords are intentionally excluded.

## Environment

| System | Purpose | Main role |
|---|---|---|
| `DC01-DOMAIN SERVER` | Windows Server domain controller | AD DS, DNS, Group Policy, Windows Event Collector, IIS intranet |
| `FS01-FILE SERVER` | Dedicated file server | Finance share, NTFS permissions, user home folders |
| `PC01-USERS` | Windows domain client | User logon, GPO testing, mapped drives, printers and event source |
| `PC02-LINUX` | Linux domain member | Linux joined to `solotech.local` |
| `PC03-NETWORKING` | Networking practice VM | Network troubleshooting and lab expansion |
| Hyper-V host | Virtualisation platform | VM management, networking and checkpoints |

**Domain:** `solotech.local`  
**DC / DNS:** `DC01` (`192.168.10.10`)  
**Current mapped drives:** Finance `F:` from FS01 and individual home `H:` drives from `\\FS01\userhomes$\username`

~~~mermaid
flowchart LR
  U[Finance users] --> PC01[PC01 Windows client]
  PC01 -->|AD authentication DNS GPO| DC01[DC01<br/>AD DS DNS GPO WEC IIS]
  PC01 -->|F Finance drive<br/>H Home drive| FS01[FS01 file server]
  PC01 -->|Forwarded security events| DC01
  Linux[PC02 Linux] -->|Domain joined| DC01
  DC01 --> Intranet[intranet.solotech.local]
~~~

## Skills demonstrated

- Active Directory users, groups, OUs and computer accounts
- Password resets, account unlocks and recurring lockout investigation
- Group Policy configuration and troubleshooting with `gpupdate`, `gpresult` and RSOP
- Shared folders, NTFS permissions, mapped drives and least privilege
- Printer deployment and user targeting
- Software deployment concepts, AppLocker troubleshooting and execution control
- DNS, IIS and client resolver cache troubleshooting
- Windows Event Forwarding, remote Event Viewer and audit policy
- Domain trust repair and Hyper-V recovery awareness
- Scheduled task investigation and stored credential remediation
- Phishing click triage, endpoint isolation and monitoring
- Windows and Linux domain integration

## Recommended screenshot evidence

Create a `screenshots/` folder beside this file and add sanitized screenshots.

| Suggested file | Evidence |
|---|---|
| `hyper-v-lab.png` | Hyper-V Manager with DC01, FS01, PC01, PC02 Linux and PC03 |
| `event-forwarding-gpo.png` | Event Forwarding GPO linked to the domain and Workstations OU |
| `fs01-mapped-drives.png` | Finance `F:` drive and user home `H:` drive |
| `intranet-dns.png` | `intranet.solotech.local` loading by name |
| `wef-forwarded-events.png` | Forwarded Events on DC01 |
| `finance-permissions.png` | Finance, Projects and Payroll group / NTFS design |

Do not publish passwords, real employee information, license keys or unredacted security logs.

---

# Ticket portfolio

## Ticket 001 — Account lockout after password change

**Category:** Account access / authentication  
**User:** Sarah Mitchell (`smitchell`)  
**Priority:** High

### Issue

Sarah reported that her domain account was locked. The account was enabled and the employee status was active. She had changed her password the previous afternoon and had signed in successfully afterward.

### Investigation

1. Verified the caller’s identity before discussing the account.
2. Checked Caps Lock, network connection and the sign in method.
3. Confirmed Sarah was using password sign in rather than PIN or an incorrect sign in tile.
4. Asked for the exact error and whether she was using the new password.
5. Established that she had temporarily forgotten the password change and entered the old password repeatedly.

### Resolution and verification

- Unlocked the account in Active Directory Users and Computers.
- Asked Sarah to sign in with the new password.
- Confirmed successful sign in and recorded the password change as the cause.

**Skill shown:** identity verification, user communication, account unlock and ticket notes.

---

## Ticket 002 — Repeated account lockout caused by stored credentials

**Category:** Authentication / recurring incident  
**User:** Sarah Mitchell (`smitchell`)  
**Priority:** High

### Issue

Sarah’s account locked again after it had been unlocked. She was using the correct password on her main PC, indicating that another device or application was still submitting the old password.

### Investigation

1. Asked about a work phone, home laptop, secondary PC, Outlook, Teams, OneDrive, Edge, printer authentication and SSO services.
2. Explained that changing a domain password does not update every stored credential automatically.
3. Used an isolation test: Sarah placed her work phone in airplane mode.
4. Unlocked the account and monitored it for about 20 minutes.
5. No new lockout occurred while the phone was isolated.

### Root cause

> Old credentials in mobile work applications were repeatedly attempting to authenticate after the password change.

### Resolution and verification

- Kept the phone disconnected during investigation.
- Had Sarah sign out of work accounts in Outlook, Teams and Edge, then sign in again with the current password.
- Advised her to update other work devices before using them.
- Confirmed no further lockout occurred during monitoring.

**Learning:** an unlock is only a temporary action if an application, mobile device, scheduled task or second workstation continues using the old password.

---

## Ticket 003 — New employee onboarding: David Carter

**Category:** New starter / access provisioning  
**User:** Daniel Okafor (`david.carter`)  
**Department:** Finance  
**Role:** Finance Assistant  
**Manager:** Priya Shah  
**Start date:** 28 September 2026

### Request validation

Before creating the account, I confirmed that Daniel was a permanent employee, the required start date, department, manager, location and requested access level. He required standard Finance access, not Payroll access.

### Implementation

1. Created `dokafor` in the Finance OU using Active Directory Users and Computers.
2. Set a temporary password and required a password change at first sign in.
3. Recorded job title, department, manager and office location.
4. Added the user to `Domain Users` and `Finance-Team`.
5. Did not add the user to `Finance-Team-Payroll`.
6. Verified the Finance drive and Finance printer assignment.

### Access outcome

| Resource | Result |
|---|---|
| General Finance files | Allowed through `Finance-Team` |
| Finance drive | Allowed |
| `FINANCE-PRINTER` | Allowed |
| Payroll folder | Denied |
| Other department resources | Not deployed / denied |

No Microsoft 365 or mail server existed in the lab at this stage, so the account was an AD authentication account, not a live mailbox.

---

## Ticket 004 — Finance drive missing for Daniel

**Category:** Group Policy / mapped drive  
**User:** Daniel Okafor  
**Priority:** Medium

### Issue

Daniel could see the Finance printer but the Finance drive was missing from **This PC**.

### Investigation

1. Confirmed that the expected drive should be visible under **This PC**.
2. Checked the drive mapping GPO and its visibility setting.
3. Checked Daniel’s membership of `Finance-Team`.
4. Ran:

~~~powershell
gpupdate /force
gpresult /r
~~~

5. Found the Finance drive GPO was denied by Security Filtering because it targeted `Finance-Team-Payroll` instead of `Finance-Team`.

### Resolution and verification

- Corrected the GPO filtering so `Finance-Team` had **Read** and **Apply group policy**.
- Refreshed policy and confirmed `F:` mapped.
- Confirmed Daniel still could not open Payroll.

**Learning:** correct group membership is not enough. The GPO itself must also target the correct group. The original drive mapping existed before the later migration to FS01; the current Finance drive is hosted on FS01.

---

## Ticket 005 — Finance Procedures desktop shortcut does not open

**Category:** Group Policy Preferences / user environment  
**User:** David Carter  
**Priority:** Medium

### Issue

David’s Finance Procedures shortcut was missing. After it appeared, the target would not open from the shortcut even though the location opened directly.

### Investigation and resolution

1. Checked `gpresult /r` and found the Finance desktop settings GPO was filtered to `Finance-Team-Payroll`.
2. Corrected the filtering to `Finance-Team`.
3. Created / updated the User Configuration → Preferences → Windows Settings → Shortcuts item.
4. Verified the target folder and NTFS permissions worked directly.
5. Enabled **Run in logged-on user’s security context (user policy option)** on the **Common** tab.
6. Used `F:\Procedures` as the shortcut target rather than exposing the UNC path.
7. Refreshed policy and confirmed the shortcut opened successfully.

**Learning:** a mapped drive provides a cleaner user experience, but security still depends on AD groups and NTFS permissions.

---

## Ticket 006 — Finance Projects access without Payroll access

**Category:** File access / least privilege  
**User:** David Carter  
**Priority:** Medium

### Request

David required read and edit access to Finance Projects but did not need confidential Payroll data.

### Resolution

1. Verified David’s identity and manager approval.
2. Confirmed that `Finance-Team-Payroll` would grant broader access than requested.
3. Created the `Finance-Projects` security group.
4. Added David to `Finance-Projects`.
5. Granted the group **Modify** NTFS permission on the Finance Projects folder.
6. Confirmed David could access Finance Projects and remained denied from Payroll.

**Learning:** purpose specific groups are more secure and easier to audit than direct permissions or adding a user to a broader group.

---

## Ticket 007 — Finance PDF application deployment and access control

**Category:** Software deployment / application access  
**Priority:** Medium

### Issue

A PDF reader MSI was deployed by Group Policy to `Authenticated Users`, so it installed on every computer receiving the computer policy. The lab had one shared PC used by several departments.

### Investigation

- Reviewed the GPO and identified `Authenticated Users` as too broad.
- Recognised that a Computer Configuration MSI is installed per **device**, not per user.
- Tested AppLocker in Audit Only and Enforce mode.
- Learned that incomplete allow rules and broad deny rules can block Windows components and Settings, making an incomplete AppLocker design unsuitable for a live shared PC.

### Resolution

- Corrected deployment targeting for Finance.
- Used NTFS permissions on the installed application folder to manage launch access on the shared PC.
- Granted Finance users **Read & Execute**.
- Retained `SYSTEM`, `Administrators` and `TrustedInstaller`.
- Removed broad execution access where appropriate.
- Avoided explicit Deny rules because Deny takes precedence and Finance users are also members of `Domain Users`.

### Verification

- A Finance user could run the PDF application.
- A non Finance user received Access Denied.

**Learning:** installing software and controlling who may use it are separate requirements. In a larger organisation, Intune, Configuration Manager, WDAC or carefully designed AppLocker policies would be considered.

---

## Ticket 008 — Internal intranet unavailable by name

**Category:** DNS / IIS / web access  
**Priority:** Medium

### Implementation

1. Installed IIS on DC01:

~~~powershell
Install-WindowsFeature Web-Server -IncludeManagementTools
~~~

2. Built the SoloTech Finance intranet page at `C:\inetpub\wwwroot\index.html`.
3. Created the DNS record:

~~~text
intranet.solotech.local → 192.168.10.10
~~~

4. Tested the site by IP address and by `http://intranet.solotech.local`.

### Incident and resolution

The record was deliberately changed to the wrong client IP for troubleshooting practice, then restored. The hostname still failed because the client held a stale resolver cache.

~~~cmd
nslookup intranet.solotech.local
ipconfig /flushdns
~~~

After flushing DNS, the hostname resolved to DC01 and the intranet loaded.

**Learning:** if a service works by IP address but fails by name, validate the DNS record, the client DNS server and the client resolver cache before troubleshooting IIS or the browser.

---

## Ticket 009 — Domain trust relationship failure on PC01

**Category:** Active Directory / workstation authentication  
**Priority:** High

### Issue

PC01 displayed:

> The trust relationship between this workstation and the primary domain failed.

The local Administrator password was unavailable during the incident.

### Investigation

1. Checked that the client name remained `PC01`.
2. Verified the domain was `solotech.local`.
3. Confirmed the PC01 computer object existed in Active Directory.
4. Confirmed network reachability to DC01 with `ping 192.168.10.10`.
5. Tested the secure channel:

~~~powershell
Test-ComputerSecureChannel -Verbose
~~~

The initial result was `False`. An early repair attempt reported **The server is not operational**, which showed that network and DC reachability had to be repaired first.

### Lab recovery and resolution

- Reset the PC01 computer account during the investigation.
- Used a controlled Hyper-V network adapter disconnection as a lab recovery method to access stored domain credentials where available, then reconnected it for the repair.
- Repaired the secure channel once DC connectivity was available:

~~~powershell
Test-ComputerSecureChannel -Repair -Credential (Get-Credential) -Verbose
~~~

The command returned `True` and reported that the secure channel with `solotech.local` was successfully repaired.

### Verification and learning

- Confirmed the secure channel passed and domain sign in worked again.
- Learned to check DNS, network reachability, time, computer object and secure channel before unjoining and rejoining a computer.

---

## Ticket 010 — Employee offboarding workflow and retention preparation

**Category:** Offboarding / identity lifecycle  
**Employee:** Emma Hughes (`ehughes`)  
**Department:** Finance

### Request

Emma had left the organisation. HR and line manager approval were confirmed. Her access included Finance, Finance Projects and Payroll.

### Workflow implemented

1. Verified requester identity and recorded HR and manager approval.
2. Recorded the leaving time and required account disable timing.
3. Disabled the AD account to end interactive access.
4. Removed, or planned immediate removal from, sensitive access groups beginning with `Finance-Team-Payroll`.
5. Preserved business data instead of deleting it immediately.
6. Built the FS01 user home folder design, for example:

~~~text
\\FS01\userhomes$\Emma.Hughes
~~~

7. Set a 90 day retention plan and planned manager approved handover of Finance Projects work to Priya.

### Current outcome

FS01 now hosts user home folders and mapped `H:` drives, allowing realistic onboarding and offboarding data handling. The former employee’s account should remain disabled throughout the approved retention period and be reviewed before deletion.

**Learning:** offboarding includes approval, timed disablement, access removal, data retention, ownership transfer and documentation.

---

## Ticket 011 — Repeated failed logons caused by an outdated scheduled task password

**Category:** Authentication monitoring / scheduled task  
**User:** David Carter  
**Priority:** High

### Issue

David was not attempting to sign in, but failed authentication events appeared at regular intervals. The timing suggested an automated source rather than an interactive error.

### Investigation

1. Reviewed PC01 Security logs and forwarded events on DC01.
2. Identified repeated failed logons at about 15 minute intervals.
3. Checked Task Scheduler Library and found `Finance Budget Backup`.
4. Identified that it ran as `SOLOTECH\dcarter` and used credentials from before David changed his password.

### Resolution

- Disabled the task temporarily to stop the authentication attempts.
- Kept David’s current password private. IT did not request or view it.
- David updated the stored credentials through the approved task properties workflow.
- Re enabled the task and ran it manually.
- Confirmed the report copy completed.

### Verification

- Monitored for more than one task interval.
- Confirmed no further failed logons appeared.

**Learning:** regularly timed failed logons point to scheduled tasks, services, scripts, mapped drives or saved credentials before they point to a user mistake or password spraying.

---

## Ticket 012 — Phishing link clicked, no credentials entered

**Category:** Security incident response  
**User:** David Carter  
**Priority:** High

### Issue

David received a suspicious email impersonating Finance support with an urgent Payroll message. The sender and URL did not match the legitimate `intranet.solotech.local` site. David clicked the link but did not enter a password.

### Containment

- In the lab, isolated the VM by disconnecting its Hyper-V network adapter.
- In an enterprise environment, equivalent containment could be an EDR isolate action, NAC, switch port shutdown or quarantine VLAN according to the incident response procedure.

### Investigation

1. Recorded sender, subject, time and URL.
2. Checked browser downloads and extensions added after the email.
3. Reviewed Downloads, `%TEMP%` and `C:\Windows\Temp`.
4. Checked Windows Security / Protection History.
5. Ran a full Microsoft Defender scan.
6. Reviewed running processes, startup items and scheduled tasks.
7. Reviewed PC01 and forwarded security events for unusual authentication activity.

### Resolution and verification

- No suspicious download, extension, Defender detection or follow on activity was found.
- Monitored for a business day without suspicious events.
- Returned the endpoint to the network after checks were complete.
- No password reset was required because credentials were not submitted and there was no evidence of compromise.
- Recommended preserving the email, blocking the sender / domain / URL and checking for other recipients.

**Learning:** a click does not prove compromise, but it requires evidence based containment and investigation.

---

# Monitoring implementation

## Windows Event Forwarding

Centralised logging was added after the password lockout investigations showed that a client often contains richer authentication detail than the domain controller view alone.

| Component | Configuration |
|---|---|
| Collector | DC01 running Windows Event Collector (`Wecsvc`) and WinRM |
| Source | PC01 |
| Subscription | Source initiated |
| Scope | Domain Computers |
| Events | User logon and audit failure events relevant to support investigations |
| Review location | Event Viewer → Forwarded Events on DC01 |

### Supporting work

- Enabled Windows Event Collector and WinRM on DC01.
- Created and linked the Event Forwarding GPO to the relevant domain / Workstations OU scope.
- Enabled advanced audit policy for Logon Success and Failure on PC01.
- Verified audit policy with:

~~~powershell
auditpol /get /subcategory:"Logon"
~~~

- Used `gpupdate /force` and restarted PC01 when policy changes were not active.

### Troubleshooting completed

- Investigated historical Event Forwarding Plugin errors 102, 104 and 105.
- Corrected the subscription target to Domain Computers.
- Confirmed the subscription runtime became **Active** and observed Event ID 100 on the source.
- Found Event Viewer on DC01 could appear stale; closing and reopening it displayed the accumulated forwarded events.
- Rechecked GPO, service state, firewall and audit policy after a VM rollback.

## Direct remote Event Viewer access

Remote Event Log Management firewall access was also configured so DC01 can view PC01 event logs directly.

| Method | Use |
|---|---|
| Windows Event Forwarding | Central review of selected events without signing in to the endpoint |
| Remote Event Viewer | Deeper review of the full log from a specific endpoint |

---

# Access control model

| Group | Purpose |
|---|---|
| `Finance-Team` | General Finance documents, Finance drive and Finance printer |
| `Finance-Team-Payroll` | Confidential Payroll folder only |
| `Finance-Projects` | Finance Projects folder only |
| `Domain Users` | Standard domain membership |

This design applies least privilege: general Finance access does not grant Payroll access, and Finance Projects access is managed separately.

# Key commands used

~~~powershell
# Refresh and verify Group Policy
gpupdate /force
gpresult /r

# Test DNS and clear stale client records
nslookup intranet.solotech.local
ipconfig /flushdns

# Check domain controller reachability
ping 192.168.10.10

# Test and repair a domain secure channel
Test-ComputerSecureChannel -Verbose
Test-ComputerSecureChannel -Repair -Credential (Get-Credential) -Verbose

# Confirm audit policy
auditpol /get /subcategory:"Logon"

# Confirm collector services
Get-Service Wecsvc, WinRM
~~~

# Lessons learned

1. Verify identity and approvals before changing access.
2. Use event timing as evidence when investigating authentication failures.
3. Check GPO scope and security filtering, not only the policy settings.
4. Separate software installation from a user’s right to run an application.
5. Use security groups rather than direct user permissions.
6. Check DNS early when an application works by IP but fails by name.
7. Validate network and DC reachability before repairing a trust relationship.
8. Verify the user outcome and monitor for recurrence after each fix.

# Next development goals

- Add Microsoft 365 / Exchange Online identity and mailbox administration.
- Add a ticketing workflow with SLAs, status updates and knowledge base articles.
- Practice Intune or Configuration Manager concepts.
- Expand EDR style incident response and evidence preservation.
- Test backup, restore and disaster recovery for AD and FS01.
- Add vulnerability assessment and remediation projects.

# Public GitHub publishing checklist

- [ ] Add sanitized screenshots to `screenshots/`.
- [ ] Remove passwords, recovery codes, license keys and personal data from all evidence.
- [ ] Remove public IPs, real company names and real email addresses.
- [ ] Clearly label names and data as fictional lab content.

*Last updated: October 2026*

