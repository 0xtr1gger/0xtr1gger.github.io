---
created: 2025-09-17
---

## NFS

>**NFS (Network File System)** is a distributed file system protocol that allows users to access files over a network as if they were stored locally.

- NFS is mostly used in Unix/Linux systems for file sharing. 
- The remote client mounts a shared file system exposed by the server.

- NFS operates on a client-server architecture where the server hosts shared files and directories, while clients access these resources over the network.
	- **NFS server**
		- Hosts the actual files and directories to be shared. It runs NFS server software and manages access permissions.
	- **NFS clients**
		- Connects to the server and mounts the shared directories, integrating them into its own local filesystem tree.

>[!important]+ NFS ports
> NFS uses several ports, depending on its version and configuration:
> - **NFS** primarily operates on **TCP and UDP port `2049`**. It's used by all NFS versions (NFSv2, NFSv3, NFSv4).
> - NFSv2 and NFSv3 also use **TCP or UDP port `111`**. 

### NFS versions

Currently, there're three versions of NFS:

- **NFSv2** (1984)
	- Stateless protocol, uses UDP only.
	- Supports files up to 2 GB in size (32-bit offsets).
	- Each request is independent, the server keeps no client state.
	- No encryption, relies on host-based access control and client-supplied UID/GID.

- **NFSv3** (1995)
	- Stateless, uses UDP or TCP.
	- Supports files more than 2 GB in size (64-offsets).
	- Supports Secure RPC, but still mostly relies on client-supplied credentials. No built-in encryption.

- **NFSv4** (2000)
	- Stateful, uses TCP only.
	- Operates only over TCP port `2049` (no portmapper).
	- The server keeps track of the client state (e.g., open files, locks).
	- Built-in support for strong authentication (Kerberos, LIPKEY), integrity, and encryption.

### RPC

NFS in fundamentally build upon the **[RPC](https://en.wikipedia.org/wiki/Remote_procedure_call) (Remote Procedure Call)** protocol. 

>**[RPC](https://en.wikipedia.org/wiki/Remote_procedure_call) (Remote Procedure Call)** is a stateless, connectionless communication protocol that allows one process (the **client**) to direct another process (the **server**) to execute procedure calls (subroutines) as if the client process had run the calls in its own address space, despite being on a separate physical system.

- NFS is based on the Sun's RPC implementation, the **[Open Network Computing Remote Procedure Call](https://en.wikipedia.org/wiki/Sun_RPC) (ONC-RPC/SUN-RPC)** protocol.
- SUN-RPC uses **TCP and UDP port `111`**.
- RPC messages use **[XDR](https://en.wikipedia.org/wiki/External_Data_Representation) (External Data Representation)** for cross-platform interoperability.

>[!note] XDR addresses fundamental compatibility issues, such as byte order (big-endian), alignment (32-bit), inconsistent data type representations across different architectures, and so on.

## NFS exports

>The act of making file systems accessible over the network is called **exporting**.
- To access files stored in the filesystem the NFS server exports, the client needs to **mount** that filesystem.

>[!important] Filesystems exported by the NFS server are configured in the **`/etc/exports` file** or files in the **`/etc/exports.d` directory**, in the **[NFS server export table](https://man.archlinux.org/man/exports.5)**.

`/etc/exports` can also specify **mount options**:

| **Option**         | **Description**                                                                                                                             |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `rw`               | Read and write permissions.                                                                                                                 |
| `ro`               | Read only permissions.                                                                                                                      |
| `sync`             | Synchronous data transfer. (A bit slower)                                                                                                   |
| `async`            | Asynchronous data transfer. (A bit faster)                                                                                                  |
| `secure`           | Ports above `1024` will not be used.                                                                                                        |
| `insecure`         | Ports above `1024` will be used.                                                                                                            |
| `no_subtree_check` | This option disables the checking of subdirectory trees.                                                                                    |
| `root_squash`      | Assigns all permissions to files of root UID/GID 0 to the UID/GID of anonymous, which prevents `root` from accessing files on an NFS mount. |
| `nohide`           | If another file system was mounted below an exported directory, this directory is exported by its own exports entry.                        |
| `no_root_squash`   | All files created by root are kept with the UID/GID 0.                                                                                      |

## Enumeration

### Nmap

- NFS enumeration with version detection and default scripts:

```bash
sudo nmap 192.168.1.11 -p111,2049 -sV -sC
```

- NFS enumeration with NFS Nmap scripts:

```bash
sudo nmap 192.168.1.11 -p111,2049 -sV --script nfs*
```

>[!interesting]+ `rpcinfo` 
>The [`rcpinfo`](https://nmap.org/nsedoc/scripts/rpcinfo.html) NSE script connects to portmapper and retrieves a list of all registered programs, including the PRC program number, support versions, port number and protocol, and program name.

- Run the `rcpinfo` NSE script to enumerate RPC programs on the target:

```bash
sudo nmap 192.168.1.11 --script 
```

>[!example]+ Example: `rpcinfo`
> 
> ```bash
> sudo nmap 192.168.1.11 -p111,2049 -sV --script rpcinfo
> ```
> 
> ```bash
> Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-09-20 09:25 CDT
> Nmap scan report for 192.168.1.11
> Host is up (0.083s latency).
> 
> PORT     STATE SERVICE VERSION
> 111/tcp  open  rpcbind 2-4 (RPC #100000)
> | rpcinfo: 
> |   program version    port/proto  service
> |   100003  3           2049/udp   nfs
> |   100003  3           2049/udp6  nfs
> |   100003  3,4         2049/tcp6  nfs
> |   100021  1,3,4      37483/udp6  nlockmgr
> |   100021  1,3,4      37575/tcp   nlockmgr
> |   100021  1,3,4      42955/tcp6  nlockmgr
> |   100021  1,3,4      44033/udp   nlockmgr
> |   100227  3           2049/tcp6  nfs_acl
> |   100227  3           2049/udp   nfs_acl
> |_  100227  3           2049/udp6  nfs_acl
> 2049/tcp open  nfs     3-4 (RPC #100003)
> 
> Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
> Nmap done: 1 IP address (1 host up) scanned in 7.04 seconds
> ```

### `showmount`

The [`showmount`](https://man.archlinux.org/man/showmount.8.en) command is used to display mount information for an NFS server. 

- Show exports on an NFS server:

```bash
showmount -e 192.168.1.11
```

>[!example]+ Example: `showmount -e`
>```bash
>showmount -e 192.168.1.11
> ```
> ```bash
> Export list for 192.168.1.11:
> /var/nfs      192.168.0.0/24
> /mnt/nfsshare 192.168.0.0/24
> ```

| Option                | Description                                                                   |
| --------------------- | ----------------------------------------------------------------------------- |
| `-a`,` --all`         | List client hostname or IP address and mounted directory (`host:dir` format). |
| `-d`, `--directories` | List only directories mounted by some client.                                 |
| `-e`, `--exports`     | Show the list of exports on an NFS server.                                    |
| `-v`, `--version`     | Display version and exit.                                                     |
| `--no-headers`        | Suppress descriptive headings from the output.                                |

### Mounting NFS shares

- To mount an NFS share, you can use a standard `mount` command:

```bash
sudo mount -t nfs 192.162.1.11:/ ./nfs -o nolock
```

>[!example]+ Example: mounting NFS shares
> ```bash
> sudo mount -f nfs 192.162.1.11:/ ./nfs -o nolock
> ```
> 
> ```bash
> cd nfs && tree
> ```
> 
> ```bash
> ├── nfs
>     ├── mnt
>     │   └── nfsshare
>     │       └── flag.txt
>     └── var
>         └── nfs
>             └── flag.txt
> ```

Once the share is mounted, you can browse it just like your local files and directories.
## References and further reading

- [`2049 - Pentesting NFS Service`](https://book.hacktricks.wiki/en/network-services-pentesting/nfs-service-pentesting.html)
- [`NFS — Arch Wiki`](https://wiki.archlinux.org/title/NFS)
- [`Network File System — Wikipedia`](https://en.wikipedia.org/wiki/Network_File_System)
- [`Remote procedure call — Wikipedia`](https://en.wikipedia.org/wiki/Remote_procedure_call)
- [`Sun RPC — Wikipedia`](https://en.wikipedia.org/wiki/Sun_RPC)
- [`Network File System (NFS): Overview & Setup — ninjaOne`](https://www.ninjaone.com/blog/network-file-system-nfs/)
- [`Linux Hacking Case Studies Part 2: NFS — NetSPI`](https://www.netspi.com/blog/technical-blog/network-pentesting/linux-hacking-case-studies-part-2-nfs/)
- [`NFS Pentesting Best Practices — secybr`](https://secybr.com/posts/nfs-pentesting-best-practicies/)