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

