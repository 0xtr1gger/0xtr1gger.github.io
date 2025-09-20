---
created: 2025-05-18
---
## Hydra

>**[THC Hydra](https://github.com/vanhauser-thc/thc-hydra)** is a fast network login brute-force tool developed by van Hauser and The Hackers Choice (THC) group. 

- Hydra supports attacks against numerous protocols and services, such as SSH, FTP, LDAP, SMTP, VNC, and even databases like MySQL.
- Hydra uses **parallel connections** to perform multiple login attempts simultaneously, which speeds up the attacks significantly.
- Despite its power, the tool is relatively easy to use.

```bash
hydra [LOGIN_OPTIONS] [PASSWORD_OPTIONS] [ATTACK_OPTIONS] [SERVICE_OPTIONS]
```

| Option                           | Description                                                                                                          |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `-l LOGIN`<br>or<br>`-L FILE`    | Specifies a single username (`-l`) or a file with a list of usernames (`-L`) to try.                                 |
| `-p PASSWORD`<br>or<br>`-P FILE` | Specifies a single password (`-p`) or a file with a list of passwords (`-L`) to try.                                 |
| `-t THREADS`                     | Defines the number of parallel threads to run (default `16`).                                                        |
| `-f`                             | Fast mode: stops the attack after first successful login.                                                            |
| `-s PORT`                        | Specifies a non-default port for the target service (the default is determined by protocol or service under attack). |
| `-v`, `-V`                       | Verbose output.                                                                                                      |
| `-M`                             | Specifies a file with a list of target hosts.                                                                        |
| `-W`                             | Sets a delay (in seconds) between connection attempts.                                                               |
| `-w`                             | Defines the maximum wait time (in seconds) for responses (default `32`).                                             |
| `-c`                             | Defines the wait time in seconds per login attempt over all threads (`-t 1` is recommended).                         |
| `-4`/`-6`                        | Prefer IPv4 (default) or IPv6 addresses.                                                                             |
| `-I`                             | Ignore an existing restore file (don't wait 10 seconds).                                                             |
| `-d`                             | Debug mode.                                                                                                          |
|                                  |                                                                                                                      |

- `/usr/share/wordlists/rockyou.txt`
- `/usr/share/wordlists/SecLists/Usernames/top-usernames-shortlist.txt`
https://man.archlinux.org/man/hydra.1.en
## Hydra services
### SSH

```bash
hydra -L usernames.txt -P passwords.txt ssh://192.168.1.11
```

- To target multiple servers:

```bash
hydra -L usernames.txt -P passwords.txt -M targets.txt ssh 
```

- To use verbose output;

```bash
hydra -L usernames.txt -P passwords.txt -V -M targets.txt ssh 
```
### FTP

```bash
hydra -L usernames.txt -P passwords.txt ftp://192.168.1.11
```

- To test FTP login on a non-standard port:

```bash
hydra -L usernames.txt -P passwords.txt -s 2121 -V 192.168.1.11 ftp
```
### HTTP

### `POST` forms

```bash
hydra -L usernames.txt -P passwords.txt example.com http-post-form "/login.php:username=^USER^&password=^PASS^:F=INnvalid credentials"
```

```bash
hydra -l admin -P passwords.txt example.com http-post-form "/login:userma,e=^USER^&pass=^PASS^:S=302" # 302 HTTP status code indicates success
```

>[!note] Here, Nmap will consider a login successful if the target responds with the `302` status code.

```bash
hydra -l admin -P passwords.txt example.com http-post-form "/:username=^USER^&pass=^PASS^:S=Dashboard" # the application display "Dashboard" on successful login
```

>[!note] 
>- `F=` specifies a failure condition; `S=` specifies a success condition.
>- Hydra dynamically replaces placeholders in the HTTP form (`^USER^` and `^PASS^`) with values from the wordlists you specified.

### Basic HTTP authentication (HTTP `GET`)

- Basic HTTP authentication (HTTP `GET`):

```bash
hydra -L usernames.txt -P passwords.txt example.com http-get
```

```bash
hydra -L ./usernames.txt -P ./10k* 83.136.250.201 -s 56345 http-get -I 
```

```bash
hydra -L ./usernames.txt -P /usr/share/wordlists/seclists/Passwords/2020-200_most_used_passwords.txt 83.136.250.201 -s 56345 http-get -I
```

### SMTP

```bash
hydra -L usernames.txt -P passwords.txt smtp://mail.example.com
```
### POP3

```bash
hydra -L usernames.txt -P passwords.txt pop3://mail.example.com
hydra -l user@example.com -P passwords.txt pop3://mail.example.com
```
### IMAP

```bash
hydra -L usernames.txt -P passwords.txt imap://mail.example.com
hydra -l user@example.com -P passwords.txt imap://mail.example.com
```
### MySQL

```bash
hydra -L usernames.txt -P passwords.txt mysql://192.168.1.11
```
### VNC

```bash
hydra -L usernames.txt -P passwords.txt vnc://192.168.1.11
```

### RDP

w
- To perform simple brute-force attack rather than a dictionary attack:

```bash
hydra -l administrator -x 6:8:abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789 192.168.1.11 rdp
```

>[!note]
>The above command generates and test passwords ranging from 6 to 8 characters and using the specified characterset.

## References

- https://man.archlinux.org/man/hydra.1.en
- https://github.com/lyudaio/cheatsheets/blob/main/security/tools/hydra.md
- https://github.com/vanhauser-thc/thc-hydra