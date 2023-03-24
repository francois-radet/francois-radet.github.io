---
layout: post
title:  "An overview of authentication and authentication support protocols on Windows systems"
tags:
  - Microsoft protocols
  - Authentication
---
This is an overview of the authentication process on Microsoft systems, and the protocols supporting it. A lot of that content is straight from Microsoft's own Open Specifications documents (technical specs), the goal here is to provide a useful summary of documents that can be dozens if not hundreds of pages long.

This post does not focus on the authentication protocols themselves (Kerberos, NTLM, etc.), but on the overall architecture:
- APIs: GSSAPI, SASL
- Support protocols used both by NTLM and Kerberos: Netlogon, APDS, SPNEGO
- Network vs. interactive logon

### <u>GSS authentication (GSSAPI)</u>
- The GSS-API is an application programming interface standard that **insulates application communication protocols and authentication protocols**
- The primary purpose of GSS-API is to **abstract the commonalities of different authentication protocols and to hide their implementation details**
- A related purpose is to **disentangle application communication protocols from authentication protocols**

- **For the initial generation of client-server computing, <u>applications and authentication protocols were tightly coupled. Authentication was hardwired into each application or into each security module</u>**, and both were closely tied to the operating system and the communications transport layer
- **This application-specific design increased development and maintenance costs and impeded interoperability** between applications that were running on the same or different communications networks
- **In the 1990s, a new paradigm decoupled application protocols from authentication protocols. This approach, which became the <u>Generic Security Service Application Programming Interface (GSS-API)</u>, simplified the interactions between the application protocols and authentication protocols**
- The Generic Security Services (GSS) style or GSS model underlies most currently implemented authentication protocols that interact directly with application protocols.
In the GSS style or model, the authentication protocol produces opaque messages that are known as security tokens. The application protocol is responsible for security token exchange between sender and receiver, but does not parse or interpret the security tokens
Client-side requests and server responses usually drive GSS authentication.
![GSSAPI_diagram](/assets/images/GSSAPI_diagram.png)
- **When authentication is complete, session-specific security services are available. <u>The application can then invoke the authentication protocol to sign or encrypt the messages that are sent as part of the application protocol</u>**. These operations are performed in much the same way, where the application can indicate which portion of the message is to be encrypted, and then include a per-message security token
By signing and/or encrypting the messages, the application obtains privacy, resists message tampering, and detects dropped, suppressed, or replayed messages
- **In Windows, the Security Support Provider Interface (SSPI) is the implementation of the GSS-style authentication model**. <u>SSPI is a Windows-specific API implementation that provides the means for connected network applications to call one of several **security support providers (SSPs)** to establish authenticated connections and to exchange data securely over those connections</u>
- **SSP is the implementation of an authentication protocol as a dynamic link library (DLL)**. SSPI is the Windows equivalent of GSS-API, and the two sets of APIs are on-the-wire compatible
 
#### SASL and GSSAPI
- **SASL** stands for Simple Authentication and Security Layer; it's a **framework that allows developers to implement different authentication mechanisms, and allows clients and servers to negotiate a mutually acceptable mechanism** for each connection (rather than hard-coding or pre-configuring them)
- GSSAPI &ndash; itself another framework for developing and implementing various authentication mechanisms &ndash; is usually made available as one of the mechanisms that SASL can use

### <u>Interactive vs. network logon</u>
The authentication services protocols provide authentication services through the following methods:
- **Interactive logon** authentication
- **Network logon** authentication (also called non-interactive authentication)
 
#### Interactive logon
- Interactive logon **relies on user interface components, such as a dialog box, to collect data**
- **A user can interactively logon to a computer in one of two ways**:
	- **Locally**, when the user has direct physical (or virtual, i.e. console) access to the computer
	- **Remotely**, through Terminal Services, in which case the logon is further qualified as **remote interactive**
- If the user credentials consist of a user name and password pair, the domain logon authentication process first tries the Kerberos Authentication Protocol ([MS-KILE]). If Kerberos fails, the authentication process falls back to the NTLM pass-through mechanism ([MS-APDS]). For smart card logons in which the credentials contain X.509 certificates, the domain logon process uses the Public Key Cryptography for Initial Authentication (PKINIT) in the Kerberos Protocol ([MS-PKCA]).
 
#### Network logon
- **Network logon authentication is used <u>only</u> after interactive logon authentication has taken place** 
- Network logon **does not rely on user interface components**, such as a dialog box, **to collect data**
-  Instead, **previously established credentials <u>or another method to collect credentials</u> is used**
-  This process confirms the user's identity to any network service that the user attempts to access
-  This process is typically invisible to the user, unless the user is required to provide alternate credentials

As depicted the following diagram, network logon authentication is performed when an application uses underlying authentication protocol packages through the GSS-API layer to establish a secure network connection. 
Network logon authentication is the mechanism at work when a user connects to multiple machines on a network. For example, if an application needs to open a secure folder on a remote machine and the application user is already logged on to a domain user account, the application does not require the user to supply logon data again. Instead, the application requests network logon authentication by using the GSS-API layer to pass the previously established security information to underlying security support providers (SSPs).

![Network_logon_diagram](/assets/images/Network_logon_diagram.png)

The communication between the client and server applications can occur over application communication protocols that are LAN-oriented (SMB, RPC, etc.) or Internet-oriented (HTTP, POP3, LDAP, etc.).

### <u>Authentication protocol architecture</u>

The following diagram depicts the whole suite of authentication protocols and authentication support protocols, as well as their interactions.

![Microsoft_auth_architecture_diagram](/assets/images/Microsoft_auth_architecture_diagram.png)

### <u>SPNEGO</u>
- When Microsoft adopted the Kerberos protocol for Windows and moved away from NT LAN Manager (NTLM), the decision required a substantial change for a number of protocols. This process is still ongoing today... 
- Rather than repeat the process when circumstances required a new or additional security protocol, Microsoft chose to insert a protocol, SPNEGO, to allow security protocol selection and extension
- Applications should use the SPNEGO protocol, instead of explicitly using an authentication method, so as not to require application changes when new/preferable authentication methods are introduced

### <u>NetLogon</u>
- The Netlogon protocol is an RPC interface that is **used for secure communication between machines in a domain and domain controllers (both domain members and DCs)**
- The communication is secured by using a shared session key computed between the client and the DC that is engaged in the secure communication. <u>The session key is computed by using a preconfigured shared secret that is known to the client and the DC (machine password)</u>
When the Netlogon service that runs on a client computer connects to the Netlogon service on a domain controller to authenticate a user, the Netlogon services challenge each other to determine whether they both have a valid computer account. This allows a secure communication channel to be established for logon purposes
- The Netlogon protocol is **used to maintain domain relationships**:
	-  **from the <u>members</u> of a domain to the <u>domain controller</u>**
	-  **<u>among DCs</u> for a domain**
	-  **and <u>between DCs across domains</u>**
- This RPC interface is used to discover and manage these relationships
- The Netlogon protocol is an older protocol that has been through multiple revisions and expansions. As a result, some of the methods are used only in LAN Manager environments, and new structures and methods have been introduced to support new functionality

![Netlogon_diagram](/assets/images/Netlogon_diagram.png)

#### Uses
- **Pass-through authentication (<u>through APDS</u>)** on domain-based networks
	- For <u>any</u> protocol that wants to leverage that secure channel, **usually NTLM**
	- <u>Can also be used across domains</u>, if the user and the resource server are not in the same domain (the DC in the resource domain will use its secure channel with the DC in the user domain to perform the pass-through authentication)
	- After the authentication is complete, **the Netlogin protocol also returns authorization information**
- **Kerberos PAC validation (<u>through APDS</u>)**
- The Netlogon protocol is <u>also used to replicate the database for backup domain controllers (probably not used anymore and replaced by DRS?)</u>
- **Secure channel maintenance**: to allow for the machine passwords to be rotated on a recurring basis, which is key to ensure the session keys &ndash; derived from those shared secrets &ndash; are secure
- Domain trust services: to get a list of trusts
- Message protection services: some applications might need to authenticate their messages sent to and received from a DC (Windows Time Service, W32Time, is one of them). The Netlogon Remote Protocol provides services to such applications via methods for computing a cryptographic digest of the message by using the machine account or trust password as the cryptographic key
- Administrative services: administrators might need to control or query the behavior related to Netlogon operations. For example, an administrator might want to force a change of the machine account password, or might want to reset the secure channel to a particular DC in the domain. Netlogon provides such administrative services via methods for querying and controlling the server

### <u>APDS (Authentication Protocol Domain Support)</u>
- **Runs over Netlogon**
- Authentication protocols such as NTLM, Kerberos, SSL/TLS, and Digest authentication are used by a variety of higher-layer protocols to provide security services
**The Authentication Protocol Domain Support Protocol specifies the communication between the server and the domain controller for each of the protocols**
- Each of the protocols has a specific exchange with the domain controller as follows:
	- **Authenticating the client**: NTLM and digest
	- **Obtaining authorization information**, such as group memberships: NTLM, digest, and SSL/TLS
	- **Verifying the authorization information**: verify the Privilege Attribute Certificate (**PAC**) for **Kerberos**

