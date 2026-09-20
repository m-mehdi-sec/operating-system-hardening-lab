# Operating System Hardening Lab

> **Assess → Harden → Verify → Monitor**

A practical operating system security project focused on assessing, hardening, validating, and monitoring Windows and Linux systems using established security baselines and security monitoring controls.

The lab covers three operating systems:

- **Windows 11**
- **Windows Server 2025**
- **Ubuntu Server 24.04**

The project combines **Microsoft Security Baselines**, **CIS guidance**, **Ubuntu Security Guide (USG)**, Windows auditing, Sysmon, and Wazuh to demonstrate a structured operating system hardening lifecycle.

Rather than treating hardening as a one-time configuration task, the project follows a repeatable security engineering workflow:

**Assess → Harden → Verify → Monitor**

---

## Project Objectives

The project was designed to:

- establish a measurable security baseline before making changes
- identify security configuration gaps and unnecessary attack surface
- apply established and role-appropriate security baselines
- preserve required system functionality during hardening
- verify the effective security state after implementation
- compare security posture before and after hardening
- document remaining deviations instead of forcing artificial compliance
- improve Windows security visibility through Audit Policy and Sysmon
- validate selected security events through Wazuh
- demonstrate the relationship between hardening, verification, and continuous monitoring

The objective was not to reach an artificial **100% compliance score**, but to apply security controls in a controlled, measurable, and operationally safe way.

---

## Lab Architecture

| System | Role | IP Address | Security Focus |
|---|---|---:|---|
| Windows 11 | Security workstation | `192.168.10.20` | Microsoft Security Baseline, Audit Policy, Sysmon |
| Windows Server 2025 | Domain Controller / DNS | `192.168.10.10` | Microsoft Domain Controller Security Baseline, GPO |
| Ubuntu Server 24.04 | Hardened Linux server | `192.168.10.30` | CIS Level 1 Server, USG |
| Ubuntu Server 24.04 | Wazuh server | `192.168.10.40` | Centralized security monitoring |
| OPNsense | Gateway / Firewall | `192.168.10.1` | Lab network gateway |

The environment runs in **Hyper-V** on an isolated internal lab network.

Windows Server 2025 provides **Active Directory Domain Services (AD DS)** and **DNS** for the `lab.local` domain.

---

## Security Methodology

The same four-stage methodology was used throughout the project.

### 1. Assess

The existing security state was reviewed before major configuration changes were made.

The assessment included areas such as:

- operating system and system role
- users and administrative privileges
- firewall configuration
- endpoint protection
- Audit Policy
- authentication settings
- running services
- listening ports
- remote administration
- security baseline deviations

Windows systems were assessed using **Microsoft Security Compliance Toolkit and Policy Analyzer**.

Ubuntu Server was assessed using **Ubuntu Security Guide against the CIS Level 1 Server profile**.

This established measurable pre-hardening baselines.

### 2. Harden

Security controls were applied using established vendor and industry guidance rather than arbitrary configuration changes.

The main hardening frameworks were:

- **Microsoft Security Baselines**
- **Microsoft Security Compliance Toolkit**
- **Group Policy / LGPO**
- **CIS Level 1 Server**
- **Ubuntu Security Guide**

Rollback mechanisms were created before major changes so that systems could be recovered if security controls affected required functionality.

### 3. Verify

Hardening was followed by both security and functional validation.

The objective was to answer two questions:

**Did the security configuration improve?**

and:

**Does the system still perform its required role?**

Effective configuration was therefore reassessed after hardening, while networking, authentication, administration, and system-specific services were tested separately.

### 4. Monitor

The final phase extended the hardened environment with security telemetry.

Windows Audit Policy and Sysmon provided endpoint visibility, while Wazuh was used as a smaller centralized monitoring and detection layer.

This demonstrated that operating system security does not end when a baseline has been applied.

---

# Windows 11

Windows 11 was assessed against the applicable **Microsoft Security Baseline** before major security changes were introduced.

Policy Analyzer identified differences between Microsoft's recommended configuration and the system's Effective State.

The assessment included security areas such as:

- Audit Policy
- authentication
- Microsoft Defender
- Windows Firewall
- local security policy
- administrative privileges
- services
- network exposure

Rollback capability was established before applying the baseline.

After hardening, the workstation was restarted and tested to ensure that:

- Windows booted normally
- user authentication remained functional
- network connectivity remained available
- administrative functionality was preserved
- Microsoft Defender remained operational
- Windows Firewall remained enabled
- required lab connectivity continued working

Policy Analyzer was then used again to compare the hardened system against the selected Microsoft baseline.

### Result

Only:

**3 of 425 analyzed policy entries**

remained different from the selected baseline.

The remaining differences were documented and evaluated rather than changed blindly.

This demonstrated that a baseline deviation is not automatically equivalent to a vulnerability and should be evaluated in the context of system role and operational requirements.

---

# Windows Server 2025

Windows Server 2025 was configured as a **Domain Controller and DNS server** for:

`lab.local`

Because security requirements differ depending on server role, the hardening baseline was selected specifically for a Domain Controller.

The implementation used:

- Microsoft Windows Server 2025 Security Baseline
- Domain Controller baseline
- Domain Security baseline
- Security Compliance Toolkit
- Policy Analyzer
- Group Policy
- LGPO

Initial assessment identified deviations involving areas such as:

- Audit Policy
- UAC
- SMB
- NTLM / LSA
- anonymous access
- remote administration

After baseline implementation, several security-relevant controls aligned with Microsoft's recommended configuration, including:

- Process Creation auditing
- Directory Service Changes auditing
- Sensitive Privilege Use auditing
- stronger UAC configuration
- restricted anonymous access
- hardened UNC paths
- stronger NTLM session security
- WinRM restrictions

### Functional Validation

Hardening a Domain Controller requires more than verifying security settings.

The server's infrastructure role was therefore tested after implementation.

Validation included:

- Active Directory Domain Services
- DNS
- Netlogon
- Domain Controller discovery
- SYSVOL
- NETLOGON
- Group Policy
- domain administration
- Domain Controller health checks

The tests confirmed that the server remained operational as a Domain Controller after the security baseline was applied.

---

# Ubuntu Server 24.04

Ubuntu Server 24.04 was assessed using **Ubuntu Security Guide (USG)** against the:

**CIS Level 1 Server**

profile.

The initial compliance assessment produced:

| Metric | Before Hardening |
|---|---:|
| PASS | 246 |
| FAIL | 95 |
| Compliance | 74.77% |

The assessment identified security controls requiring improvement across areas including:

- file integrity monitoring
- sudo auditing
- authentication
- password security
- kernel configuration
- firewall configuration
- SSH security

A rollback point was created before remediation.

The CIS Level 1 Server profile was then applied through USG.

### Functional Validation

After remediation and reboot, the server was tested to confirm that:

- SSH remained accessible
- administrative login remained available
- sudo remained functional
- networking remained operational
- the expected IP configuration remained intact
- SSH continued listening on TCP/22
- AppArmor remained active
- no failed systemd units were present

The CIS audit was then repeated using the same profile.

### Before vs After

| Metric | Before | After |
|---|---:|---:|
| PASS | 246 | 345 |
| FAIL | 95 | 6 |
| Compliance | 74.77% | 94.20% |

Compliance improved by approximately:

**19.43 percentage points**

while required administrative functionality remained operational.

Six controls remained non-compliant and were documented instead of being changed solely to increase the compliance percentage.

This demonstrated an important distinction between **compliance and risk-based security decisions**.

---

# Hardening Results

Although each operating system required different tools and controls, the same security engineering methodology was maintained.

| Area | Windows 11 | Windows Server 2025 | Ubuntu Server 24.04 |
|---|---|---|---|
| Security baseline | Microsoft Security Baseline | Microsoft Domain Controller Baseline | CIS Level 1 Server |
| Assessment | Policy Analyzer | Policy Analyzer | USG Audit |
| Implementation | SCT / LGPO | GPO / SCT / LGPO | USG |
| Verification | Effective State | Effective State + DC validation | USG Audit |
| Rollback | Hyper-V + LGPO | Hyper-V + LGPO | Hyper-V checkpoint |
| Monitoring | Audit Policy + Sysmon | Windows Security Auditing | Wazuh integration |

The common workflow remained:

**Baseline → Assessment → Controlled Implementation → Functional Validation → Security Verification**

---

# Monitoring & Detection Validation

Hardening establishes a stronger security configuration, but it does not provide continuous visibility into activity occurring afterward.

A smaller monitoring layer was therefore added using:

- **Windows Audit Policy**
- **Sysmon**
- **Wazuh**

The objective was not to build another dedicated SIEM project.

Instead, monitoring was used to validate that security-relevant activity on the hardened environment could be collected, investigated, and detected.

---

## Windows Audit Policy

Windows auditing was reviewed after baseline implementation.

Relevant audit categories included:

- Logon
- Account Lockout
- Credential Validation
- Process Creation
- File Share
- Sensitive Privilege Use
- Audit Policy Change
- User Account Management

Standard Windows Security auditing provided visibility into activity such as process creation and authentication events.

---

## Sysmon

Sysmon was used on Windows 11 to provide richer endpoint telemetry.

The final tuned configuration retained security-relevant telemetry including:

| Event ID | Telemetry |
|---:|---|
| 1 | Process Creation |
| 3 | Network Connection |
| 5 | Process Termination |
| 6 | Driver Load |
| 22 | DNS Query |

Controlled activity was generated to verify process, network, discovery, and DNS telemetry.

Account Discovery activity was successfully identified through Sysmon process events and Wazuh detection rules.

---

## Sysmon Telemetry Tuning

The initial Sysmon configuration intentionally collected broad telemetry.

Analysis showed that several event categories generated large volumes of low-value events, particularly:

- Registry activity
- Image / DLL loading
- File creation

The volume contributed to Wazuh agent buffer pressure and warnings that events could be lost.

Instead of simply increasing processing capacity, the telemetry source was reviewed and tuned.

High-volume categories were removed from the lab configuration while retaining telemetry considered most useful for the project's detection objectives.

After tuning:

- excessive Registry telemetry stopped
- ImageLoad noise was reduced
- unnecessary FileCreate telemetry was removed
- Wazuh agent pressure decreased
- important process telemetry remained available

Detection functionality was then tested again.

Account Discovery detection continued working after the configuration was tuned.

This demonstrated an important monitoring principle:

> **More telemetry is not automatically better telemetry.**

Effective security monitoring requires balancing visibility, signal quality, noise, and processing capacity.

---

# Wazuh Validation

Wazuh was used as a lightweight centralized monitoring and validation layer.

Two Windows endpoints were connected:

- **Win11-Sysmon**
- **WinSrv2025-DC**

Several controlled security scenarios were used to verify event collection and detection.

---

## Account Discovery

Controlled discovery commands generated Sysmon process telemetry on Windows 11.

Wazuh identified the activity using discovery-related detection rules.

This validated the telemetry path:

**Process Execution → Sysmon → Wazuh Agent → Wazuh Manager → Detection**

---

## Failed Authentication

A controlled authentication attempt using a nonexistent account was generated against the Domain Controller.

Windows generated:

**Event ID 4625 — An account failed to log on**

Wazuh detected the event as:

**Rule 60122 — Logon Failure - Unknown user or bad password**

The event preserved useful investigation context including:

- source workstation
- source IP address
- target username
- authentication mechanism
- logon type

This demonstrated how Windows authentication auditing can provide useful context for centralized investigation.

---

## File Integrity Monitoring

Wazuh File Integrity Monitoring was tested against a monitored directory on Windows 11.

A controlled test file went through the following lifecycle:

**Create → Modify → Modify → Delete**

Wazuh detected the filesystem changes and generated corresponding FIM alerts.

This demonstrated how unauthorized or unexpected file changes could be centrally observed.

---

## Windows Log Clearing

A controlled Windows Application log clearing operation was performed on the Domain Controller after the log was backed up.

Wazuh detected:

**A Windows log file was cleared**

Log clearing is security relevant because attackers may attempt to remove evidence during or after malicious activity.

In this lab, the activity was generated deliberately to validate monitoring coverage.

---

# Assess → Harden → Verify → Monitor

The complete project can be summarized as four connected security phases.

| Phase | Purpose |
|---|---|
| **Assess** | Understand the existing security state and identify deviations |
| **Harden** | Apply established and role-appropriate security controls |
| **Verify** | Confirm both security improvements and continued functionality |
| **Monitor** | Maintain visibility into security-relevant activity after hardening |

These phases should not be treated independently.

A hardened system that is not verified may be incorrectly configured.

A hardened system without monitoring may provide limited visibility into later activity.

A monitored system without a controlled baseline may produce large amounts of telemetry from an inconsistent security configuration.

Together, the four phases provide a more complete operating system security lifecycle.

---

## Screenshots

### Windows 11 - Baseline Assessment and Hardening

The initial Policy Analyzer comparison identified multiple deviations between the effective Windows 11 configuration and the Microsoft Security Baseline, including missing audit settings.

![Windows 11 Policy Analyzer - Before Hardening](images/windows-11/01-win11-policy-analyzer-before.png)

After applying the Microsoft Security Baseline and verifying the effective configuration, only three deviations remained for further analysis.

![Windows 11 Policy Analyzer - After Hardening](images/windows-11/02-win11-policy-analyzer-after.png)

---

### Windows Server 2025 - Domain Controller Hardening

The initial Policy Analyzer assessment identified several differences between the effective Domain Controller configuration and the Windows Server 2025 security baseline.

![Windows Server 2025 Policy Analyzer - Before Hardening](images/windows-server/03-server2025-policy-analyzer-before.png)

After hardening, key security auditing categories such as Process Creation, Directory Service Changes, Sensitive Privilege Use, Credential Validation, and File Share auditing were enabled.

![Windows Server 2025 Audit Policy - After Hardening](images/windows-server/04-server2025-audit-policy-after.png)

Post-hardening verification confirmed that Active Directory Domain Services, DNS, and Netlogon remained operational and that the `lab.local` domain continued to resolve correctly.

![Windows Server 2025 Domain Controller - Functional Verification](images/windows-server/05-server2025-dc-verification.png)

---

### Ubuntu Server 24.04 - CIS Level 1 Hardening

The initial Ubuntu Security Guide audit against the CIS Level 1 Server profile produced 246 passed rules, 95 failed rules, and a compliance score of 74.77%.

![Ubuntu CIS Level 1 - Before Hardening](images/ubuntu/06-ubuntu-cis-before.png)

After remediation and verification, the system reached 345 passed rules with only 6 remaining failures, increasing the compliance score to 94.20%.

![Ubuntu CIS Level 1 - After Hardening](images/ubuntu/07-ubuntu-cis-after.png)

---

### Monitoring and Detection Validation

Wazuh was used as a monitoring and validation layer after hardening. Windows 11 and Windows Server 2025 were connected as active endpoints.

![Wazuh - Active Windows Endpoints](images/monitoring/08-wazuh-active-agents.png)

Sysmon telemetry from Windows 11 was successfully processed by Wazuh, detecting account discovery activity executed through PowerShell and mapping the activity to MITRE ATT&CK techniques T1087 and T1059.001.

![Wazuh - Account Discovery Detection](images/monitoring/09-wazuh-account-discovery.png)

A controlled failed network authentication attempt against the Domain Controller generated Windows Event ID 4625. The event preserved useful investigation context including the target account, logon type, failure reason, and source address.

![Wazuh - Failed Logon Detection](images/monitoring/10-wazuh-failed-logon.png)

---

# Key Findings

### Security baselines provide consistency, not absolute security

Microsoft Security Baselines and CIS profiles provide strong reference configurations, but system role and operational requirements must still be considered.

### Verification is as important as implementation

Successfully applying a baseline does not prove that the resulting system is both secure and functional.

Effective configuration and required functionality must be tested afterward.

### Remaining deviations require analysis

A non-compliant setting is not automatically a vulnerability.

Some deviations may require manual remediation, tailoring, risk acceptance, or further investigation.

### System roles affect hardening decisions

A Domain Controller cannot be treated like a workstation or generic server.

Security controls must reflect the purpose and dependencies of the system being protected.

### Visibility is part of operating system security

Audit Policy and Sysmon demonstrated how hardening can improve the telemetry available for detection and investigation.

### Monitoring requires tuning

The Sysmon exercise demonstrated that excessive telemetry can create operational problems.

Useful security monitoring requires balancing detection coverage with signal quality and processing capacity.

---

# Lessons Learned

This project reinforced several practical security engineering principles:

- assess systems before changing them
- understand the role of the system being hardened
- create rollback mechanisms before major security changes
- use established security baselines instead of arbitrary settings
- measure security posture before and after implementation
- verify required functionality after hardening
- investigate remaining deviations rather than chasing artificial compliance
- combine preventive controls with security telemetry
- tune monitoring based on detection value rather than event volume
- treat operating system hardening as an ongoing lifecycle rather than a one-time task

---

# Skills Demonstrated

### Operating System Security

- Windows 11 hardening
- Windows Server 2025 hardening
- Ubuntu Server hardening
- attack surface reduction
- authentication security
- host firewall validation
- security configuration analysis

### Security Baselines & Compliance

- Microsoft Security Baselines
- Microsoft Security Compliance Toolkit
- Policy Analyzer
- LGPO
- Group Policy
- CIS Benchmarks
- Ubuntu Security Guide
- compliance assessment
- before-and-after security validation
- configuration drift concepts

### Windows Security

- Active Directory
- Domain Controller security
- Windows Audit Policy
- Microsoft Defender
- Windows Firewall
- UAC
- SMB hardening
- NTLM / LSA security
- WinRM security
- AppLocker
- Sysmon

### Linux Security

- CIS Level 1 Server
- USG auditing and remediation
- SSH security
- AppArmor
- AIDE
- PAM
- sudo security
- nftables
- systemd validation

### Blue Team & Monitoring

- Wazuh
- Windows Event Logs
- Sysmon telemetry
- authentication monitoring
- process monitoring
- File Integrity Monitoring
- security event investigation
- telemetry tuning
- detection validation

---

# Repository Structure

    operating-system-hardening-lab/
    │
    ├── README.md
    ├── documentation.md
    │
    ├── configs/
    │   └── sysmon/
    │       └── sysmon-soc.xml
    │
    ├── images/
    │   ├── architecture/
    │   ├── windows-11/
    │   ├── windows-server/
    │   ├── ubuntu/
    │   └── monitoring/
    │
    └── LICENSE

---

# Documentation

Detailed technical documentation, implementation steps, commands, validation results, and troubleshooting are available here:

[Lab Documentation](docs/lab-documentation.md)

---

# Author

**Muhammad Mehdi**

IT Security Developer Student
