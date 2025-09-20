---
created: 2025-09-15
---
## FTP 

>**FTP (File Transfer Protocol)** is an industry standard protocol used to transfer files over a network, originally developed in the 1970s.

>[!note] FTP was first standardized in 1971.

>[!important] By default, FTP listens on TCP port `21` for control connections and uses TCP port `20` for data connections in active mode.

- FTP operates on a client-server model where clients connect to servers to upload, download, and manage files and directories. 
- The protocol supports **authentication with a username and password**. 

>[!critical] All FTP data is transmitted in plain text.
>All FTP communications, including authentication credentials and file content, are transmitted in **plain text** without encryption. This makes FTP vulnerable to eavesdropping attacks.

>[!tip]
>For greater security, you can also use one these two protocols:
>- **FTPS (FTP over SSL/TLS, or FTP Secure)** — upgrade to FTP 
>- **SFTP (SSH File Transfer Protocol)** — new protocol

FTP is more complex that TFTP. Not only it allows for file transfers, but also navigation across file directories and directory management, adding/removing directories, listing files, etc. The client sends **FTP commands** to the server to perform these functions.

>[!note] See [List of FTP commands — Wikipedia](https://en.wikipedia.org/wiki/List_of_FTP_commands)

### FTP connections

FTP uses two types of connections:

- **FTP control connection** (TCP port `21`)
	- Used for sending **FTP commands** and receiving server responses.

>[!note] See [List of FTP server return codes — Wikipedia](https://en.wikipedia.org/wiki/List_of_FTP_server_return_codes)

- **FTP data connection** (TCP port `20`)
	- Used for actual **file transfers** and directory listings.
### FTP modes

FTP can operate in two modes:
- **Active mode**
- **Passive mode**
#### Active mode

1. An **FTP client** opens a **control connection** to the FTP server's **TCP port `21`** to send FTP commands. 
2. The **FTP client** starts listening on an **arbitrary port** for a **data connection**. It then sends the **`PORT` command** to the server to tell the port it's listening on.
3. The **FTP server** initiates a **data connection** from its **TCP port `20`** to the client's specified port.
4. Once the data connection is established, data (files, directory listings, etc.) can be transferred from the server to client or vice versa.
5.  After the transfer is complete, the data connection is closed.


>[!important] The **client listens** for incoming TCP connections from the server on a client-specified port. The server *actively* connects back to the client for data transfers.

>[!warning] One of the biggest disadvantages of the active mode is that the client not always can listen for incoming connections because of firewalls and NAT, which often restrict outside devices to initiate connections.
#### Passive mode

1. An **FTP client** opens a **control connection** to the FTP server's **TCP port `21`** to send FTP commands. 
2. The client requests the server to listen for a data connection by sending the `PASV` command.
3. The FTP server responds with an IP address and a port number for the client to connect to (dynamic port, usually above `1023`).
4. The client then initiates a TCP connection **from its side to the server's specified port**.
5. Once the data connection is established, data (files, directory listings, etc.) can be transferred from the server to client or vice versa.
6. After the transfer is complete, the data connection is closed.

>[!important] The **server listens** on a dynamically allocated port, and the **client connects** to the server for data transfer.

>[!note]
>Passive mode overcomes the problem with firewalls and NAT: no need for the client to listen for incoming connections at all. Today, the passive mode is used much more often than active, because of the widespread implementation of firewalls.

>[!important] In the passive FTP mode, the TCP port `20` is not used at all.

## TFTP

>**Trivial File Transfer Protocol (TFTP)** is an industry-standard protocol for file transfers, first standardized in 1981. TFTP is simpler than FTP and runs over UDP.

TFTP was first standardized in 1981. It's named *Trivial* because it's simple and has only basic features compared to FTP.

>[!important] TFTP servers listen on **UDP port `69`**.

- **No authentication**: servers will respond to **all TFTP requests**.
- **No encryption**: all data is sent in **plain text**.
- Best used in a controlled environment to transfer small files quickly.
- TFTP uses UDP, which is connectionless and doesn't provide reliability features. However, TFTP has built-in reliability (acknowledgments and retransmissions) within the protocol itself.

>[!note] TFTP reliability
>- Every TFTP data message is **acknowledged**.
>	- If the client is transferring a file to the server, the server will send Ack messages in response.
>	- If the server is transferring a file to the client, the client will send Ack messages.
>- TFTP clients and servers use **timers**: if an expected acknowledgment isn't received in time, the waiting device will re-send its previous message.
>
>TFTP uses **lock-step communication**: the client and server alternately send a message and then wait for a reply; retransmissions are sent as needed.
### TFTP file transfers

TFTP file transfers have three phases:
1. **Connection**
	- TFTP client sends a request to the server, and the server responds back, initializing the connection.

2. **Data Transfer**
	- The client and server exchange TFTP messages. One sends data, and the other sends acknowledgments. 

3. **Connection termination**
	 - After the last data message has been sent, a final acknowledgment is sent to terminate the connection.

### TFTP TID

>When the client sends the first message to the server, the destination port is UDP `69` and the source is a random ephemeral port. In TFTP, this random port is called a **Transfer Identifier (TID)**, and is used to identify the data transfer.

- The server then also selects a random TID to use as the source port when it replies, not `69`.
- When the client sends the next message, the destination port will be the server's TID, not `69`.