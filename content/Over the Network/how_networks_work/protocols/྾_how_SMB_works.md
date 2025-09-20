---
created: 2025-09-16
---
## SMB

>**[SMB](https://en.wikipedia.org/wiki/Server_Message_Block) (Server Message Block)** is a network communication protocol used to share files, printers, serial ports, and other resources between devices on a network.

- SMB is mostly used in **Windows environments**, but also supported by other OS like Linux (via [Samba](https://en.wikipedia.org/wiki/Samba_(software))).
- SMB operates on a **client-server model** where clients send requests to servers to access shared resources.

>[!important] SMB primarily uses **TCP port `445`**.
>- Modern SMB runs directly over **TCP port `445`** directly.
>- Older versions uses [NetBIOS](https://en.wikipedia.org/wiki/NetBIOS) over TCP or UDP ports `137`-`139`.

>[!note] SMB functions as an **application-layer protocol**.

>[!important]+ Direct SMB vs. SMB over NetBIOS
>
>- **SMB over TCP (port `445`)**
> 	- Packets are **directly encapsulated in TCP** without any intermediate layers.
> 	- SMB traffic is preceded with a **4-byte header** (first byte is always `0x00`, next 3 bytes indicate data length).
> 	- Doesn't use **NetBIOS**.
> 
>-  **SMB over NetBIOS session service (port `139`)**
> 	- SMB traffic is **encapsulated within NetBIOS sessions**.
> 	- Requires **three-way NetBIOS handshake** before SMB communication.
> 	- Uses **NetBIOS name resolution** instead of DNS.
> 	- **Vulnerable to various attacks** targeting NetBIOS protocols.
> 	- **Deprecated** but still supported for backward compatibility.
> 

>[!interesting] SMB can also be used for inter-process communication (IPC) over a network.

>[!note]
>See [[྾_SMB]] for attacks against SMB servers.
## SMB session establishment

## SMB shares

SMB introduces a concept of **shares**.

>An **SMB share** is a file, directory, or printer on an SMB server that can be accessed by SMB clients over a network.
- Each share can have specific permissions and access controls. 

## SMB authentication

SMB supports two levels of authentication:

- **User-level authentication**
	- The user must provide their username and password when attempting to access a share on a server. 
	- When authenticated, the user can access all shares on that server (except those additionally protected with share-level authentication).

- **Share-level authentication**
	- Access to a share is protected by a share-specific password.
	- No usernames are required.
	- Mostly used on legacy systems.

>[!important] Under both authentication levels, the password is sent encrypted. 

As an underlying authentication mechanism, SMB primarily uses:
- [NTLM](https://en.wikipedia.org/wiki/NTLM) (NT LAN Manager; a challenge-response protocol, vulnerable to replay and downgrade attacks).
- [Kerberos](https://en.wikipedia.org/wiki/Kerberos_\(protocol\)) (cryptographically secure, ticket-based protocol preferred in Active Directory environments).

>[!important] SMB can be configuration not to require authentication at all. This is often called a **null session**, similar to FTP's anonymous session.

>[!note]+ SMB session lifecycle
>1. **Connection establishment:** An SMB client initiates a TCP connection to the server (typically port `445`). The client and server negotiate the dialect (SMB version) and session parameters.
>2. **Authentication:** The client provides authentication credentials, and the server verifies then using Kerberos or NTLM.
>3. **Resource access:** The client sends requests (open, read, write, close), and the server responds accordingly, verifying access controls.
>4. **Session termination:** The client closes the session. 

>[!note]+ SMB clients
>Below are some tools you can use to interact with SMB servers:
>- [`smbclient`](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html)
>- [`CrackMapExec`](https://github.com/byt3bl33d3r/CrackMapExec)
>- [`SMBMap`](https://github.com/ShawnDEvans/smbmap)
>- [`Impacket`](https://github.com/fortra/impacket)

## SMB versions

>[!important]+ Overview of SMB versions
> 
> - **SMBv1 (SMB (1.0)**
> 	- The original SMB protocol; suffered from significant security weaknesses.
> 	- Inherently insecure design with multiple known vulnerabilities. 
> 	- **No encryption** support for data in transit.
> 	- Exploited by major malware including [WannaCry](https://en.wikipedia.org/wiki/WannaCry_ransomware_attack) ransomware. 
> 	- Deprecated by Microsoft.
> 
> - **SMBv2 (SMB 2.0/2.1)**
> 	- Introduced with **Windows Vista and Server 2008**; addresses many SMBv1 limitations.
> 	- Redesigned protocol architecture; reduced complexity.
> 	- Improved performance and reliability.
> 	- Better hashing algorithms.
> 	- HMAC SHA-256 for message signing.
> 
> - **SMBv3 (SMB 3.0/3.1/3.1.1)**
> 	- The most security SMB version; introduced with **Windows 8 and Server 2012**.
> 	- **End-to-end encryption** support.
> 	- AES-CCM encryption algorithm.
> 	- AES-CMAC algorithm for integrity checks.
> 	- Security dialect negotiation (prevents downgrade attacks).
> 	- SMB multichannel for improved performance.
> 	- AES-256-GCM encryption in newer versions (Windows 11/Server 2022).
## SMB workgroups and domains

SMB can operate within two primary network organization models:
- **Workgroups**
- **Domains**

>[!important]+ Wordgroups
>**Workgroups** represent a **peer-to-peer network model** where each computer maintains its own user accounts and security policies.
> - **Decentralized authentication:** each machine validates its own users.
> - **Local user accounts** stored on individual computers.
> - **Limited scalability**; suitable for small networks (typically under 20 computers).
> - NTLM authentication only; Kerberos is not supported in workgroup mode.

>[!important]+ Domains
>**Domains** implement a **centralized security model** using **Active Directory**.
>- A **Domain Controller (DC)** acts as a centralized server and runs Active Directory.
>- **Centralized authentication:** all login requests are processed by DC.
>- **Unified password policies** and lockout settings.
>- **Company-wide security policies** through **GPOs (Group Policy Objects)**.
>- **Hierarchical organization** that supports organizational units (OUs).
>- **Kerberos authentication** by default.

