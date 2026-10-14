# Windows Server 2022 Active Directory Home Lab

## Project Overview

This project documents a hands-on IT support and Windows Server administration lab built using **VMware**, Windows Server 2022, and a Windows 11 client. It focuses on Active Directory Domain Services (AD DS), DNS, domain authentication, user and group management, Organizational Units (OUs), and Group Policy.

The lab is developed through practical exercises, PowerShell verification, troubleshooting, and documentation. The objective is to build practical skills relevant to entry-level IT support, desktop support, and junior Windows systems administration roles.

## Objectives

- Validate Active Directory Domain Services and domain controller health.
- Verify DNS resolution for Active Directory.
- Create and manage domain users and security groups.
- Test domain-user authentication from a Windows client.
- Organize users using Organizational Units.
- Create, configure, and test Group Policy Objects.
- Verify configuration and authentication using PowerShell and Windows command-line tools.
- Document technical procedures, test results, and troubleshooting findings.

## Lab Environment

| Component | Configuration |
|---|---|
| Virtualisation platform | VMware |
| Server operating system | Windows Server 2022 |
| Client operating system | Windows 11 |
| Domain DNS name | `test.local` |
| NetBIOS domain name | `TEST` |
| Domain controller | `WIN-KA4BD6SPOCE` |
| Client computer | `WIN11-DEMO` |
| Directory service | Active Directory Domain Services (AD DS) |
| DNS service | Windows Server DNS |
| Administration tools | Server Manager, Active Directory Users and Computers, Group Policy Management, PowerShell, Command Prompt |

## Stage 1: Initial Environment Checks

### Tasks performed

- Confirmed that Active Directory Domain Services was installed.
- Checked the server's domain membership using PowerShell.
- Confirmed that the server reported `PartOfDomain = True`.
- Confirmed that a Windows client virtual machine was available for domain testing.

### Verification command

```powershell
Get-CimInstance Win32_ComputerSystem |
    Select-Object Name, Domain, PartOfDomain
```

### Result

The server reported that it was part of a domain. This established the starting point for subsequent Active Directory and DNS checks.

## Stage 2: Active Directory and DNS Health Checks

### 2.1 Verified essential services

Checked the following services on the domain controller:

- **NTDS:** Active Directory Domain Services.
- **DNS:** Domain Name System.
- **Netlogon:** Supports domain authentication and related domain operations.
- **KDC:** Kerberos Key Distribution Center, used for Kerberos authentication.

**Command:**

```powershell
Get-Service NTDS, DNS, Netlogon, KDC
```

**Result:** All four services were reported as running.

### 2.2 Queried the Active Directory domain

**Command:**

```powershell
Get-ADDomain
```

**Result:** The command returned domain information successfully.

### 2.3 Ran domain controller diagnostics

**Command:**

```powershell
dcdiag
```

**Result:** The diagnostic results were reported as healthy during the initial checks.

### 2.4 Tested DNS resolution from the client

Performed DNS lookups for the domain and Active Directory LDAP service records from the Windows client.

Active Directory uses DNS service records to help domain-joined computers locate services provided by domain controllers.

**Result:** The domain and LDAP SRV lookups succeeded. The client was already joined to the domain, so rejoining it was unnecessary.

## Stage 3: Active Directory User and Security Group Management

### 3.1 Created a security group

Created the following security group in Active Directory:

`IT-Support`

**Configuration:**
- Group type: Security
- Group scope: Global

The group provides a way to organize users by support responsibilities and practise group-based access management.

### 3.2 Created a domain user

Created the following domain account:

`peter.nwachineke`

The account was subsequently confirmed to be enabled.

**Verification command:**

```powershell
Get-ADUser -Identity "peter.nwachineke" |
    Format-List Name, SamAccountName, DistinguishedName, Enabled
```

### 3.3 Added the user to the security group

Added `peter.nwachineke` to the `IT-Support` security group.

### 3.4 Verified group membership

**Commands:**

```powershell
Get-ADUser peter.nwachineke -Properties MemberOf |
    Select-Object SamAccountName, MemberOf

Get-ADGroupMember IT-Support
```

**Result:** PowerShell confirmed the user's group membership.

## Stage 4: Domain-User Authentication

### Objective

Verify that an Active Directory domain account can authenticate successfully on a domain-joined Windows client.

### Environment

- Domain: `test.local`
- NetBIOS domain name: `TEST`
- Domain user: `peter.nwachineke`
- Domain controller reported by Windows: `WIN-KA4BD6SPOCE`
- Client computer: `WIN11-DEMO`

### Verification commands

```cmd
whoami
echo %LOGONSERVER%
```

### Observed results

The `whoami` command returned:

```text
test\peter.nwachineke
```

The `echo %LOGONSERVER%` command returned:

```text
\\WIN-KA4BD6SPOCE
```

### Outcome

Successful. Windows identified the signed-in account as `TEST\peter.nwachineke` and reported the logon server for the session.

### Skills practised

- Domain-user authentication
- Windows command-line verification
- Domain identity troubleshooting
- Basic Windows domain administration

## Stage 5: Group Policy Inspection

### Objective

Inspect the existing Group Policy configuration before creating a separate policy for the test user.

### 5.1 Listed existing Group Policy Objects

**Command:**

```powershell
Get-GPO -All |
    Select-Object DisplayName, GpoStatus
```

### Observed GPOs

| Group Policy Object | Reported status |
|---|---|
| Default Domain Policy | AllSettingsEnabled |
| Default Domain Controllers Policy | AllSettingsEnabled |
| Deploy Softwares | AllSettingsEnabled |

### 5.2 Inspected Group Policy inheritance

Checked the GPO links at the domain level and on the Domain Controllers OU.

**Commands:**

```powershell
Get-GPInheritance -Target "DC=test,DC=local"

Get-GPInheritance -Target "OU=Domain Controllers,DC=test,DC=local"
```

**Results:**
- The domain-level output listed `Default Domain Policy` and `Deploy Softwares`.
- The Domain Controllers OU output listed `Default Domain Controllers Policy`.
- The `Deploy Softwares` policy was left unchanged.

### 5.3 Verified the user account location

**Command:**

```powershell
Get-ADUser -Identity "peter.nwachineke" |
    Format-List Name, SamAccountName, DistinguishedName, Enabled
```

The account was initially located in the default `Users` container:

`CN=Peter Junior Nwachineke,CN=Users,DC=test,DC=local`

The account was confirmed as enabled.

## Stage 6: Created a Dedicated Organizational Unit

### Objective

Organize the test user in a dedicated OU to practise targeted Group Policy management.

### Tasks performed

- Created an Organizational Unit named `IT-Support` in the `test.local` domain.
- Moved the `peter.nwachineke` account from the default `Users` container into the new OU.
- Verified the updated distinguished name using PowerShell.

**Verification command:**

```powershell
Get-ADUser -Identity "peter.nwachineke" |
    Select-Object Name, SamAccountName, DistinguishedName
```

**Expected distinguished name:**

```text
CN=Peter Junior Nwachineke,OU=IT-Support,DC=test,DC=local
```

**Result:** The user was reported to have the expected distinguished name after the move.

### Skills practised

- Organizational Unit management
- Active Directory object organization
- PowerShell verification
- Preparing user objects for targeted Group Policy configuration

## Stage 7: Created and Tested a User Group Policy Object

### Objective

Create a separate user-level Group Policy Object and test whether a setting can launch a program at sign-in.

### 7.1 Created and linked the GPO

Created a new Group Policy Object named:

`IT-Support - User Desktop Policy`

The GPO was created and linked to the `IT-Support` OU.

**Verification command:**

```powershell
Get-GPO -Name "IT-Support - User Desktop Policy" |
    Select-Object DisplayName, GpoStatus
```

**Result:** PowerShell returned the GPO name and reported its status as `AllSettingsEnabled`.

### 7.2 Configured the logon setting

In Group Policy Management Editor, navigated to:

`User Configuration → Policies → Administrative Templates → System → Logon`

Configured the following policy:

**Run these programs at user logon**

The policy was enabled, with the following program entered:

```text
notepad.exe
```

### 7.3 Tested the policy on the Windows client

Signed in to the Windows client as `TEST\peter.nwachineke`.

**Observed result:** Notepad opened a few minutes after login.

This is consistent with the configured logon policy. The final `gpresult /r` confirmation that explicitly lists `IT-Support - User Desktop Policy` under applied user GPOs remains to be recorded.

### Skills practised

- Group Policy Management
- User-level policy configuration
- OU-based policy targeting
- Windows client testing
- Verification and documentation of policy behavior

### Cleanup

After capturing the evidence, return the `Run these programs at user logon` setting to **Not Configured** and refresh Group Policy on the client. This prevents Notepad from continuing to launch at every sign-in.

Record cleanup as complete only after performing and checking it.

## Evidence and Screenshots

Store genuine screenshots from the VMware lab in the repository.

Suggested structure:

```text
windows-server-it-support-lab/
├── README.md
├── docs/
│   ├── active-directory/
│   │   └── README.md
│   ├── dns/
│   │   └── README.md
│   ├── printers/
│   │   └── README.md
│   ├── windows-troubleshooting/
│   │   └── README.md
│   └── sop/
│       └── sop-template.md
└── screenshots/
    ├── active-directory/
    ├── dns/
    └── windows-troubleshooting/
```

Suggested screenshots:

- `screenshots/active-directory/domain-health-check.png`
- `screenshots/active-directory/user-group-membership.png`
- `screenshots/active-directory/domain-user-login.png`
- `screenshots/active-directory/it-support-ou.png`
- `screenshots/active-directory/it-support-gpo-settings.png`
- `screenshots/active-directory/it-support-gpo-applied.png`
- `screenshots/active-directory/notepad-logon-test.png`


## Skills Demonstrated So Far

- Windows Server 2022 administration
- Active Directory Domain Services
- DNS service and SRV record validation
- Domain-user and security group management
- Organizational Unit management
- Group Policy Object creation and configuration
- Domain-user authentication
- PowerShell and Command Prompt verification
- Basic troubleshooting and technical documentation

## Current Project Status

The lab has progressed through initial domain checks, Active Directory and DNS health checks, user and security group management, domain authentication, OU creation, and a practical user Group Policy test.

The test user successfully authenticated to the domain, and Notepad opened after login following configuration of the user logon policy. Explicit `gpresult /r` confirmation of the custom GPO and cleanup of the temporary Notepad setting remain to be recorded.

This is a learning project built in a VMware environment. Results are documented based on the checks performed, with remaining verification and cleanup tasks identified separately.
