# Awesome Windows Domain Hardening [![Awesome](https://cdn.rawgit.com/sindresorhus/awesome/d7305f38d29fed78fa85652e3a63e154dd8e8829/media/badge.svg)](https://github.com/PaulSec/Windows-domain-hardening)

A curated list of awesome Security Hardening techniques for Windows.

Created by [gepeto42](https://twitter.com/gepeto42) and [PaulWebSec](https://twitter.com/PaulWebSec) but highly inspired from [PyroTek3](https://twitter.com/PyroTek3) research!


### Summary

This document summarizes the information related to Pyrotek and Harmj0y's DerbyCon talk called "111 Attacking EvilCorp Anatomy of a Corporate Hack". Video and slides are available below. 

It also incorporates hardening techniques necessary to prevent other attacks, including techniques discussed by [gepeto42](https://twitter.com/gepeto42) and [joeynoname](https://twitter.com/joeynoname) during their [THOTCON 0x7 talk](https://evil.plumbing/).

Something's missing? Create a Pull Request and add it.

### Initial foothold

- No hardening effort should come at the expense of upgrading operating systems.
- Deploy [EMET](https://support.microsoft.com/en-us/help/2458544/the-enhanced-mitigation-experience-toolkit) to Workstations (End of line in July 2018 - Consider keeping EMET for Windows 7 but prioritize upgrades to Windows 10 and Edge). 
- Use [AppLocker](https://technet.microsoft.com/en-us/library/dd759117(v=ws.11).aspx) to block exec content from running in user locations (home dir, profile path, temp, etc).
- Manage PowerShell execution via Applocker or constrained language mode.
- Enable [PowerShell logging](https://www.fireeye.com/blog/threat-research/2016/02/greater_visibilityt.html) (v3+) & command process logging.
- [Block Office macros](https://blogs.technet.microsoft.com/mmpc/2016/03/22/new-feature-in-office-2016-can-block-macros-and-help-prevent-infection/) (Windows & Mac) on content downloaded from the Internet.
- Deploy security tooling that monitors for suspicious behavior. Consider using [WEF](https://blogs.technet.microsoft.com/jepayne/2015/11/23/monitoring-what-matters-windows-event-forwarding-for-everyone-even-if-you-already-have-a-siem/) to forward only interesting events to your SIEM or logging system.
- Limit capability by blocking/restricting attachments via email/download:
	-  Executables extensions:
	-  (ade, adp, ani, bas, bat, chm, cmd, com, cpl,
crt, hlp, ht, hta, inf, ins, isp, job, js, jse, lnk, mda, mdb,
mde, mdz, msc, msi, msp, mst, pcd, pif, reg, scr, sct, shs,
url, vb, vbe, vbs, wsc, wsf, wsh, exe, pif, etc.)
	- Office files that support macros (docm, xlsm, pptm, etc.)
-  Change default program for anything that opens with Windows scripting to notepad (test first!)
	- bat, js, jse, vbe, vbs, wsf, wsh, etc.

### Reconnaissance

- Deploy Windows 10 and limit local group enumeration.
- Limit workstation to workstation communication.
- Increase security on sensitive [GPO](https://msdn.microsoft.com/en-us/library/bb742376.aspx)s.
-  Evaluate deployment of behavior analytics [(Microsoft ATA)](https://www.microsoft.com/en-us/cloud-platform/advanced-threat-analytics).

### Lateral Movement

-  Configure GPO to prevent local accounts from network authentication [(KB2871997)](https://support.microsoft.com/en-us/help/2871997/microsoft-security-advisory-update-to-improve-credentials-protection-and-management-may-13,-2014).
- Ensure local administrator account passwords are automatically changed [(Microsoft LAPS)](https://www.microsoft.com/en-us/download/details.aspx?id=46899) & remove extra local admin accounts.
- Limit workstation to workstation communication [(Windows Firewall)](https://technet.microsoft.com/en-us/network/bb545423.aspx).
  - Test psexec with good credentials between two workstations. If it works, you have a lateral movement problem.

### Privilege Escalation

- Remove files with passwords in SYSVOL [(including GPP)](https://adsecurity.org/?p=2288).
- Ensure admins don’t log onto untrusted systems (regular workstations) by configuring **DENY** user right assignments with GPOs.
- Provide Privileged Access Workstations or [PAWs](https://technet.microsoft.com/en-us/windows-server-docs/security/securing-privileged-access/privileged-access-workstations) for all highly privileged work. Those should never have access to the Internet.
- Use Managed Service Accounts for SAs when possible [(gMSA)](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)
- For systems that do not support Managed Service Accounts, deploy a [Fine-Grained Password Policy](https://technet.microsoft.com/en-us/library/cc770842(v=ws.10).aspx) to ensure the passwords are >32 characters.
- Ensure all computers are talking NTLMv2 & Kerberos, deny [LM/NTLMv1](https://support.microsoft.com/en-us/help/2793313/security-guidance-for-ntlmv1-and-lm-network-authentication).

### Protect Administration Credentials
 
- Ensure all admins only log onto approved admin workstations & servers. (See PAW in Privilege Escalation section)
- Ensure all built-in groups but Administrator are denied from logging on to Domain Controllers user User Right Assignments. By default, Backup operators, Account operators can login to Domain Controllers, which is dangerous.
- Add all admin accounts to [Protected Users group](https://technet.microsoft.com/en-us/library/dn466518%28v=ws.11%29.aspx) (requires Windows 2012 R2 DCs).
- Admin workstations & servers:
	- Control & limit access to admin workstations & servers.
	- Remove NetBIOS over TCP/IP
	- Disable [LLMNR](https://en.wikipedia.org/wiki/Link-Local_Multicast_Name_Resolution).
	- Disable [WPAD](https://en.wikipedia.org/wiki/Web_Proxy_Auto-Discovery_Protocol).
 
### Strengthen/Remove Legacy

- Audit/Restrict NTLM.
- Enforce [LDAP signing](https://technet.microsoft.com/en-us/library/dd941832%28v=ws.10%29.aspx).
- Enable [SMB signing](https://blogs.technet.microsoft.com/josebda/2010/12/01/the-basics-of-smb-signing-covering-both-smb1-and-smb2/) (& encryption where poss.).
- Disable WPAD & LLMNR & work to disable NetBIOS.
- Windows 10, remove:
	- SMB 1.0/CIFS
	- Windows PowerShell 2.0
- Use [shims](https://technet.microsoft.com/en-ca/library/dd837644(v=ws.10).aspx) to enable old applications that require admin privileges to work by believing they have them.
 
### Tools

- [Responder](https://github.com/lgandx/Responder) - A LLMNR, NBT-NS and MDNS poisoner
- [BloodHound](https://github.com/BloodHoundAD/BloodHound) - Six Degrees of Domain Admin
- [AD Control Path](https://github.com/ANSSI-FR/AD-control-paths) - Active Directory Control Paths auditing and graphing tools  
- [PowerSploit](https://github.com/PowerShellMafia/PowerSploit/) - A PowerShell Post-Exploitation Framework
- [PowerView](https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon) - Situational Awareness PowerShell framework
- [Empire](https://github.com/EmpireProject/Empire) - PowerShell and Python post-exploitation agent
- [Mimikatz](https://github.com/gentilkiwi/mimikatz) - Utility to extract plaintexts passwords, hash, PIN code and kerberos tickets from memory but also perform pass-the-hash, pass-the-ticket or build Golden tickets
- [Tools Cheatsheets](https://github.com/HarmJ0y/CheatSheets) - (Beacon, PowerView, PowerUp, Empire, ...)
- [UACME](https://github.com/hfiref0x/UACME) - Defeating Windows User Account Control
- [Windows System Internals](https://technet.microsoft.com/en-us/sysinternals/bb545021.aspx) - (Including Sysmon etc.)
- [Hardentools](https://github.com/securitywithoutborders/hardentools) - Collection of simple utilities designed to disable a number of "features" exposed by Windows
- [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) - A swiss army knife for pentesting Windows/Active Directory environments

### Videos

- [BSides DC 2016 - PowerShell Security: Defending the Enterprise from the Latest Attack Platform](https://www.youtube.com/watch?v=_8yBjg7bRLo&feature=youtu.be&t=106)
- [Six Degrees of Domain Admin... - Andy Robbins, Will Schroeder, Rohan Vazarkar](https://www.youtube.com/watch?v=lxd2rerVsLo)
- [111 Attacking EvilCorp Anatomy of a Corporate Hack](https://www.youtube.com/watch?v=nJSMJyRNvlM&feature=youtu.be&t=16)
- [Red vs Blue: Modern Active Directory Attacks & Defense](https://www.youtube.com/watch?v=rknpKIxT7NM)
- [Offensive Active Directory with Powershell](https://www.youtube.com/watch?v=cXWtu-qalSs)
- [Advanced Incident Detection and Threat Hunting using Sysmon and Splunk](https://www.youtube.com/watch?v=vv_VXntQTpE)
- [Real Solutions From Real Incidents: Save Money and Your Job!](https://www.youtube.com/watch?v=313J20uPbcw)

### Slides

- [How to go from Responding to Hunting with Sysinternals Sysmon](https://onedrive.live.com/view.aspx?resid=D026B4699190F1E6!2843&ithint=file%2cpptx&app=PowerPoint&authkey=!AMvCRTKB_V1J5ow)
- [111 Attacking EvilCorp Anatomy of a Corporate Hack](https://adsecurity.org/wp-content/uploads/2016/09/DerbyCon6-2016-AttackingEvilCorp-Anatomy-of-a-Corporate-Hack-Presented.pdf)
- [Real Solutions From Real Incidents: Save Money and Your Job!](https://evil.plumbing/Current-version-June.pdf)

### Additional resources

- [ADSecurity](https://adsecurity.org/)
- [Harmj0y's blog](http://blog.harmj0y.net/)
- [Sysmon SecuriTay's configuration file](https://github.com/SwiftOnSecurity/sysmon-config) - template with default high-quality event tracing
- [Explaining and adapting Tay’s Sysmon configuration](https://medium.com/@lennartkoopmann/explaining-and-adapting-tays-sysmon-configuration-27d9719a89a8#.mi0rmwn1v)
- [Use of PSExec](https://www.toshellandback.com/2017/02/11/psexec/)
