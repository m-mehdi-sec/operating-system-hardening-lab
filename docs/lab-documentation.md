# Technical Documentation

Detailed implementation notes, commands, validation results, and troubleshooting for the Operating System Hardening Lab.

[← Back to project overview](../README.md)

This document contains the technical implementation, commands, validation steps, and selected results from the Operating System Hardening Lab.

---

## 1. Windows 11 Hardening

### 1.1 Initial Security Assessment

Windows 11 was assessed before applying the Microsoft Security Baseline.

The initial review covered:

- operating system version
- local administrative accounts
- Microsoft Defender
- Windows Firewall
- running services
- listening ports
- Windows Update status
- Audit Policy
- effective security policy

Lab configuration:

| Property | Value |
|---|---|
| System | Windows 11 Pro |
| IP address | `192.168.10.20/24` |
| Role | Security workstation |
| Domain membership | Standalone |
| Virtualization | Hyper-V |

Microsoft Security Compliance Toolkit and Policy Analyzer were used to compare the system's Effective State against the applicable Microsoft Security Baseline.

The initial comparison identified several policy differences, including security-relevant auditing gaps.

Examples included:

- Credential Validation
- Process Creation
- Account Lockout
- File Share
- Sensitive Privilege Use

---

### 1.2 Rollback Preparation

Rollback capability was established before applying the baseline.

A Hyper-V checkpoint was created to provide full VM recovery.

The existing Local Group Policy configuration was also backed up using LGPO.

This provided:

- VM-level rollback through Hyper-V
- policy-level rollback through LGPO

---

### 1.3 Baseline Implementation

The Windows 11 security configuration was implemented using:

- Microsoft Security Baseline
- Security Compliance Toolkit
- Policy Analyzer
- LGPO
- Local Group Policy

Because the workstation was not joined to Active Directory, the baseline configuration appropriate for a standalone system was used.

After applying the baseline, Group Policy was refreshed:

    gpupdate /force

The workstation was then restarted.

---

### 1.4 Microsoft Defender Verification

Microsoft Defender was checked after baseline implementation.

The relevant protection components remained enabled:

    Get-MpComputerStatus | Select-Object AntivirusEnabled,RealTimeProtectionEnabled,BehaviorMonitorEnabled,AntispywareEnabled

Verified protections included:

- Antivirus
- Real-Time Protection
- Behavior Monitoring
- Antispyware

This confirmed that endpoint protection remained operational after hardening.

---

### 1.5 Windows Firewall Verification

Firewall status was verified across all profiles:

    Get-NetFirewallProfile | Select-Object Name,Enabled

The following profiles remained enabled:

- Domain
- Private
- Public

---

### 1.6 Services and Network Exposure

Running services and listening ports were reviewed as part of the host assessment.

Relevant services included:

- Microsoft Defender
- Windows Firewall
- Sysmon
- Tenable Nessus

Listening services were reviewed to distinguish required lab functionality from unnecessary exposure.

The workstation retained the network services required by the lab.

---

### 1.7 Functional Validation

The system was tested after baseline implementation.

Validation confirmed:

- Windows started normally
- local authentication worked
- network connectivity remained available
- administrative access remained available
- Microsoft Defender remained operational
- Windows Firewall remained enabled
- required security tools remained available
- SSH connectivity to the Ubuntu environment remained functional

This was important because the hardening process was not considered successful solely because the policies had been applied.

---

### 1.8 Post-Hardening Policy Analysis

Policy Analyzer was run again against the hardened Effective State.

The final comparison showed:

**3 remaining differences out of 425 analyzed policy entries.**

The remaining differences were documented rather than automatically modified.

---

## 2. Windows Server 2025 Hardening

### 2.1 Server Role and Configuration

Windows Server 2025 was configured as a Domain Controller and DNS server.

| Property | Value |
|---|---|
| Operating System | Windows Server 2025 |
| Hostname | `WIN-SRV2025` |
| IP address | `192.168.10.10/24` |
| Domain | `lab.local` |
| Server role | Domain Controller |
| Roles | AD DS, DNS |
| Virtualization | Hyper-V Generation 2 |

The server role was identified before baseline selection because Microsoft provides different security baselines for different server roles.

---

### 2.2 Rollback Preparation

A Hyper-V checkpoint was created before applying the baseline:

    PRE-BASELINE-WS2025

A Local Group Policy backup was also created using LGPO:

    cd C:\BaselineLab\Tools\LGPO

    mkdir C:\BaselineLab\Backups\Server-PreBaseline

    .\LGPO.exe /b C:\BaselineLab\Backups\Server-PreBaseline /n "Server2025 Pre Baseline"

This provided both VM-level and policy-level recovery.

---

### 2.3 Microsoft Security Baseline

The server was hardened using:

**Microsoft Windows Server 2025 Security Baseline - 2602**

The implementation used:

- Security Compliance Toolkit
- Policy Analyzer
- LGPO
- Group Policy

The relevant role-specific policies were:

- Windows Server 2025 Domain Controller
- Windows Server 2025 Domain Security

The Domain Controller policy was linked to the Domain Controllers OU.

The Domain Security policy was linked at the `lab.local` domain level.

Member Server policies were not used because the server did not have that role.

---

### 2.4 Pre-Hardening Policy Analysis

Policy Analyzer was used to compare the Microsoft baseline with the server's Effective State.

The analysis covered:

- Authentication
- Audit Policy
- Windows Firewall
- administrative privileges
- SMB
- Security Options
- Microsoft Defender
- Remote Administration

Selected pre-hardening findings included:

| Control | Baseline | Before |
|---|---|---|
| Process Creation | Success | No Auditing |
| Directory Service Changes | Success | No Auditing |
| Sensitive Privilege Use | Success + Failure | No Auditing |
| RestrictAnonymous | `1` | `0` |
| ConsentPromptBehaviorAdmin | `2` | `5` |
| NTLM Session Security | Stronger requirement | Lower value |

The findings established the security state before baseline implementation.

---

### 2.5 Group Policy Implementation

The Domain Controller and Domain Security baselines were imported and applied through Group Policy.

Group Policy was refreshed:

    gpupdate /force

The server was then restarted.

Applied policies were verified using:

    gpresult /r

The resulting policy set included:

- Default Domain Controllers Policy
- Microsoft Windows Server 2025 Domain Controller baseline
- Default Domain Policy
- Microsoft Windows Server 2025 Domain Security baseline

---

### 2.6 Core Domain Controller Services

After restart, the required services were verified:

    Get-Service NTDS,DNS,Netlogon

The following services reported:

    Running

- NTDS
- DNS
- Netlogon

---

### 2.7 DNS Validation

DNS resolution for the Active Directory domain was tested:

    Resolve-DnsName lab.local

Expected resolution:

    lab.local → 192.168.10.10

DNS continued functioning after baseline implementation.

---

### 2.8 Domain Controller Discovery

Domain Controller discovery was tested using:

    nltest /dsgetdc:lab.local

The server was correctly identified as:

    \\WIN-SRV2025.lab.local

with:

    192.168.10.10

---

### 2.9 SYSVOL and NETLOGON

Domain Controller shares were verified using:

    net share

The required shares remained available:

- `SYSVOL`
- `NETLOGON`

This confirmed that essential domain resources remained accessible after hardening.

---

### 2.10 Active Directory and DNS Administration

Active Directory Users and Computers was opened after baseline implementation.

The `lab.local` domain remained administratively accessible.

DNS Manager was also verified.

The server continued hosting:

- `_msdcs.lab.local`
- `lab.local`

Both DNS zones remained operational.

---

### 2.11 Domain Controller Health

Domain Controller health was checked using:

    dcdiag

Important tests passed, including:

- Connectivity
- Advertising
- SysVolCheck
- NetLogons
- Replications
- Services
- LocatorCheck
- partition tests

DFSREvent and SystemLog reported warnings.

The warnings were documented because subsequent testing confirmed that:

- AD DS remained operational
- DNS remained operational
- SYSVOL remained available
- NETLOGON remained available

---

### 2.12 Post-Hardening Audit Policy

After baseline implementation, several security-relevant audit categories were active, including:

- Credential Validation
- Kerberos Authentication Service
- Process Creation
- Directory Service Changes
- Group Membership
- File Share
- Sensitive Privilege Use

This increased visibility into authentication, administrative activity, and Active Directory changes.

---

### 2.13 UAC

Relevant UAC settings after hardening included:

    ConsentPromptBehaviorAdmin = 2

    ConsentPromptBehaviorUser = 0

These values aligned with the selected baseline.

---

### 2.14 SMB and Hardened UNC Paths

SMB-related protections were applied through the baseline.

Hardened UNC paths included:

    \\*\NETLOGON

    \\*\SYSVOL

Restrictions on insecure guest authentication were also applied.

---

### 2.15 NTLM and LSA

Relevant post-hardening settings included:

    LmCompatibilityLevel = 5

    RestrictAnonymous = 1

    RestrictAnonymousSAM = 1

    NTLMMinClientSec = 537395200

    NTLMMinServerSec = 537395200

These settings provided a more restrictive authentication configuration than the initial state.

---

### 2.16 WinRM

Remote administration controls were hardened through policy.

Restrictions included:

- Basic authentication
- Digest authentication
- unencrypted traffic

---

### 2.17 AppLocker Behavior

A functional change was observed after baseline implementation.

When Microsoft Edge attempted to open a PDF file, execution was blocked with:

    This app has been blocked by your system administrator.

Policy Analyzer showed that AppLocker-related controls had been applied.

The behavior was documented as a functional effect of the hardened configuration.

---

### 2.18 Microsoft Defender

Microsoft Defender remained active after hardening.

Verified protections included:

- Antivirus
- Real-Time Protection
- Behavior Monitoring
- Antispyware

One remaining deviation was identified:

    PUAProtection = 0

The selected baseline expected:

    PUAProtection = 1

The deviation was documented for further analysis rather than changed without evaluating its policy source and impact.

---

## 3. Ubuntu Server 24.04 Hardening

### 3.1 Initial Configuration

The Ubuntu hardening target used:

| Property | Value |
|---|---|
| Operating System | Ubuntu Server 24.04 LTS |
| IP address | `192.168.10.30/24` |
| Security profile | CIS Level 1 Server |
| Assessment tool | Ubuntu Security Guide |

The server was assessed before automated remediation.

---

### 3.2 SSH Configuration

SSH was treated as a required administrative dependency.

Relevant configuration included:

    Port 22
    PermitRootLogin no
    PubkeyAuthentication yes
    PasswordAuthentication no
    UsePAM yes

SSH access from Windows 11 had to remain functional throughout the hardening process.

---

### 3.3 Initial CIS Audit

Ubuntu Security Guide was used to audit the system against the CIS Level 1 Server profile.

The initial result was:

| Result | Count |
|---|---:|
| PASS | 246 |
| FAIL | 95 |
| Compliance | 74.77% |

Findings included controls related to:

- AIDE
- sudo auditing
- password quality
- password history
- authentication lockout
- kernel configuration
- firewall configuration

---

### 3.4 Recovery Preparation

A Hyper-V checkpoint was created before automated CIS remediation.

The rollback point protected against changes that could affect:

- SSH
- authentication
- networking
- sudo
- system startup

---

### 3.5 CIS Remediation

The CIS Level 1 Server profile was applied using Ubuntu Security Guide.

The server was restarted after remediation.

Functional validation was performed before the compliance result was evaluated.

---

### 3.6 SSH Verification

SSH connectivity from Windows 11 was tested after remediation.

Remote administrative access remained operational.

The SSH service continued listening on:

    TCP/22

---

### 3.7 Network Verification

The server retained its expected network configuration:

    192.168.10.30/24

Required network connectivity remained available.

---

### 3.8 Privileged Administration

sudo access was tested after remediation.

Administrative commands continued functioning correctly.

---

### 3.9 AppArmor

AppArmor remained active after the CIS profile was applied.

This confirmed that mandatory access control remained available after remediation.

---

### 3.10 systemd Validation

Failed services were checked after reboot.

No failed systemd units were present during final validation.

---

### 3.11 Final CIS Audit

The same CIS Level 1 Server profile was audited again after remediation.

| Result | Before | After |
|---|---:|---:|
| PASS | 246 | 345 |
| FAIL | 95 | 6 |
| Compliance | 74.77% | 94.20% |

The compliance result improved by approximately:

**19.43 percentage points**

while SSH, networking, sudo, and system services remained functional.

---

### 3.12 Remaining CIS Findings

Six controls remained non-compliant.

Documented examples included:

- `/tmp` was not configured as a separate partition
- GRUB did not use bootloader password protection
- nftables did not use a Default Deny policy
- SSH access was not explicitly restricted to selected users

The remaining findings were left documented rather than modified solely to increase the compliance score.

---

## 4. Windows Audit Policy

### 4.1 Audit Configuration

Windows auditing was reviewed using:

    auditpol /get /category:*

Relevant active categories included:

- System Integrity
- Logon
- Account Lockout
- Special Logon
- File Share
- Detailed File Share
- Removable Storage
- Sensitive Privilege Use
- Process Creation
- Audit Policy Change
- Authentication Policy Change
- User Account Management
- Credential Validation

---

### 4.2 Process Creation Auditing

Windows Security Event ID `4688` was used to examine native process auditing.

A process creation event provided information such as:

- executable
- user
- parent process
- command line
- elevation context

This provided a native Windows telemetry source before Sysmon enrichment was examined.

---

## 5. Sysmon

### 5.1 Service Verification

Sysmon was installed on Windows 11 as:

`Sysmon64`

The service was checked using:

    Get-Service | Where-Object {$_.Name -like "Sysmon*" -or $_.DisplayName -like "*Sysmon*"}

Service configuration was also verified:

    Get-CimInstance Win32_Service -Filter "Name='Sysmon64'" | Select-Object State,StartMode,PathName

The service was:

- Running
- Automatic

Sysmon events were written to:

`Microsoft-Windows-Sysmon/Operational`

---

### 5.2 Process Telemetry

Sysmon Event ID `1` was compared with native Windows Security Event ID `4688`.

Sysmon provided additional process context including:

- Process GUID
- hashes
- file metadata
- parent image
- parent command line

This additional context was useful for investigation and correlation.

---

### 5.3 Initial Telemetry Scope

The initial Sysmon configuration was intentionally broad to provide detailed endpoint telemetry and establish which event categories generated the most activity.

The initial lab configuration collected:

- Event ID 1 — Process Creation
- Event ID 3 — Network Connection
- Event ID 5 — Process Termination
- Event ID 6 — Driver Load
- Event ID 7 — Image Load
- Event ID 11 — File Create
- Event IDs 12–14 — Registry Activity
- Event ID 22 — DNS Query

This broad collection provided extensive visibility, but it also generated telemetry that was not equally valuable for the detection scenarios used in the lab.

---

### 5.4 Sysmon Telemetry Analysis and Tuning

The Sysmon Operational log was analyzed before tuning to determine which event categories generated the highest volume.

A sample of 500 recent events showed:

| Event ID | Event Type | Count |
|---:|---|---:|
| 12 | Registry Event | 330 |
| 13 | Registry Event | 84 |
| 7 | Image Load | 57 |
| 11 | File Create | 25 |
| 1 | Process Create | 2 |
| 5 | Process Terminate | 2 |

Registry activity, Image Load, and File Create events dominated the sample.

The Wazuh agent subsequently reported:

    Agent buffer at 90 %.

followed by:

    Agent buffer is full: Events may be lost.

and:

    Agent buffer is flooded: Producing too many events.

The buffer later recovered:

    Agent buffer is under 70 %. Working properly again.

The event volume demonstrated that collecting more telemetry did not automatically improve monitoring quality. The configuration therefore needed to be tuned so that useful security visibility could be preserved without continuously forwarding unnecessary high-volume events.

The following high-volume categories were removed from the final configuration:

- Image Load
- File Create
- Registry Event

The following telemetry was retained:

- Process Creation
- Network Connection
- Process Termination
- Driver Load
- DNS Query

The tuning workflow was:

**Broad Collection → Measure → Identify Noise → Tune → Re-test Detection**

---

### 5.5 Sysmon Configuration Files

Both the initial and final configurations are retained to document the tuning process.

- [Pre-Tuning Sysmon Configuration](../configs/sysmon/sysmon-pre-tuning.xml) — initial broad configuration used to analyze event volume and identify unnecessary telemetry.
- [Tuned Sysmon Configuration](../configs/sysmon/sysmon-soc.xml) — final configuration used after telemetry analysis and detection validation.

The tuned configuration was applied using:

    C:\Windows\Sysmon64.exe -c C:\Sysmon\sysmon-soc.xml

Sysmon reported:

    Configuration file validated.
    Configuration updated.

---

### 5.6 Post-Tuning Verification

Events generated after the configuration change were sampled again.

The resulting event IDs were:

| Count | Event ID |
|---:|---:|
| 3 | 5 |
| 2 | 3 |
| 2 | 1 |

No new Event ID 7, 11, 12, 13, or 14 events appeared in the sample.

This confirmed that the high-volume telemetry had been removed from the active configuration.

---

### 5.7 Detection Validation After Tuning

Controlled activity was generated after tuning:

    whoami
    net user
    Test-NetConnection 192.168.10.10 -Port 53

Wazuh continued receiving process telemetry from the Windows 11 agent.

The `net user` activity produced discovery detections based on Sysmon Event ID `1`.

This confirmed that the tuning did not remove the process telemetry required for the selected detection scenario.

---

### 5.8 DNS Query Verification

DNS activity was generated using:

    nslookup example.com
    nslookup microsoft.com

Sysmon Event ID `22` events were verified locally using:

    Get-WinEvent -FilterHashtable @{
        LogName='Microsoft-Windows-Sysmon/Operational'
        Id=22
        StartTime=(Get-Date).AddMinutes(-5)
    } -ErrorAction SilentlyContinue |
    Select-Object -First 5 TimeCreated, Id, Message

Event ID `22` was successfully generated.

---

## 6. Wazuh

### 6.1 Wazuh Server

A separate Ubuntu Server 24.04 system was used for Wazuh.

| Property | Value |
|---|---|
| Hostname | `wazuh-server` |
| IP address | `192.168.10.40` |
| Operating System | Ubuntu Server 24.04 LTS |
| Deployment | All-in-one |
| Wazuh version | 4.14.7 |

The Wazuh dashboard was accessed over HTTPS.

---

### 6.2 Windows Agents

Two Windows systems were enrolled:

| Agent | IP Address | Role |
|---|---:|---|
| `Win11-Sysmon` | `192.168.10.20` | Windows workstation |
| `WinSrv2025-DC` | `192.168.10.10` | Domain Controller |

Both agents reached an active state.

---

### 6.3 Sysmon Event Collection

The Windows 11 Wazuh agent monitored:

`Microsoft-Windows-Sysmon/Operational`

The agent log confirmed:

    Analyzing event log: 'Microsoft-Windows-Sysmon/Operational'.

This verified that the Wazuh agent was reading the Sysmon event channel.

---

### 6.4 Account Discovery Detection

Controlled discovery activity was generated on Windows 11:

    whoami
    net user
    net localgroup administrators

Sysmon recorded process creation activity.

Wazuh generated discovery detections including:

**Rule 92031**

`Discovery activity executed`

and:

**Rule 92033**

`Discovery activity spawned via powershell execution`

The detections were based on Sysmon Event ID `1`.

---

### 6.5 Failed Authentication Detection

A controlled network authentication attempt was generated against the Domain Controller using a nonexistent account:

`SOC-Test-User`

Windows Server generated:

**Event ID 4625 — An account failed to log on**

Relevant event data included:

| Field | Value |
|---|---|
| Target User | `SOC-Test-User` |
| Target Domain | `lab` |
| Logon Type | `3` |
| Authentication Package | `NTLM` |
| Source Workstation | `DESKTOP-TH2MB3N` |
| Source IP | `192.168.10.20` |

Wazuh generated:

**Rule 60122**

`Logon Failure - Unknown user or bad password`

This confirmed that failed network authentication against the Domain Controller was visible centrally.

---

### 6.6 File Integrity Monitoring

A monitored test directory was available on Windows 11:

`C:\WazuhLab`

A controlled file lifecycle was generated:

    New-Item -Path "C:\WazuhLab\fim-test.txt" -ItemType File -Force

    Set-Content -Path "C:\WazuhLab\fim-test.txt" -Value "SOC Blue Team FIM test"

    Add-Content -Path "C:\WazuhLab\fim-test.txt" -Value "File modified"

    Remove-Item "C:\WazuhLab\fim-test.txt"

Wazuh recorded:

| Activity | Rule |
|---|---:|
| File added | 554 |
| Integrity checksum changed | 550 |
| Integrity checksum changed | 550 |
| File deleted | 553 |

The complete sequence was:

**Create → Modify → Modify → Delete**

---

### 6.7 Windows Log Clearing Detection

A controlled log-management operation was performed on Windows Server 2025.

The Application log was backed up before clearing:

    wevtutil cl Application /bu:C:\Windows\Temp\ApplicationBackup.evtx

The Security log was not cleared.

Wazuh detected:

**Event ID 104**

`A Windows log file was cleared`

with:

**Wazuh Rule 63104**

This confirmed centralized visibility into Windows log-clearing activity.

---

## 7. Troubleshooting and Tuning

### 7.1 Wazuh Agent Manager Address

During the initial Windows 11 agent setup, the agent attempted to connect to an incorrect manager address:

`192.168.10.150`

The configuration was corrected to:

`192.168.10.40`

Connectivity was then confirmed in the agent log:

    Connected to the server ([192.168.10.40]:1514/tcp).

The agent was re-enrolled as:

`Win11-Sysmon`

and reached an active state.

---

### 7.2 Excessive Sysmon Telemetry

The initial broad Sysmon configuration generated large volumes of:

- Registry events
- ImageLoad events
- FileCreate events

This caused Wazuh agent buffer pressure.

The configuration was reduced to the telemetry required for the lab, and the selected detection scenario was tested again afterward.

This preserved useful detection visibility while reducing unnecessary event volume.

---

### 7.3 Wazuh Event Visibility

Not every locally generated event appeared as a dedicated alert in Wazuh Threat Hunting.

For example, Sysmon DNS Query Event ID `22` was verified directly in the local Sysmon Operational log even when no corresponding Threat Hunting result was produced.

This distinction was important when validating the pipeline:

- local event generation
- event collection
- alert generation

were treated as separate stages rather than assuming that every collected event must result in a Wazuh alert.

---

## 8. Final Technical State

At the end of the implementation:

| Component | Status |
|---|---|
| Windows 11 Microsoft baseline | Applied and verified |
| Windows 11 remaining baseline differences | 3 / 425 |
| Windows Server 2025 DC baseline | Applied and verified |
| AD DS | Operational |
| DNS | Operational |
| SYSVOL / NETLOGON | Operational |
| Ubuntu CIS Level 1 Server | Applied |
| Ubuntu CIS compliance | 94.20% |
| Ubuntu remaining CIS failures | 6 |
| Windows Audit Policy | Verified |
| Sysmon | Active and tuned |
| Sysmon DNS telemetry | Verified |
| Wazuh Windows agents | Active |
| Account Discovery detection | Verified |
| Failed authentication detection | Verified |
| File Integrity Monitoring | Verified |
| Windows log-clearing detection | Verified |
| Post-tuning detection | Verified |
