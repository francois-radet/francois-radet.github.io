---
layout: post
title:  "Exchange Online's attribution and authentication process"
tags:
  - Microsoft 365
  - Exchange Online
---
In this post, I explain Exchange Online's attribution and authentication process. Most of the content comes from Microsoft's documentation, so this is
my attempt at summarizing and simplifying it.

#### <u>Message attribution (originating vs. incoming)</u>

Attribution can be found in the **X-MS-Exchange-Organization-MessageDirectionality** header (present most of the time) and in detailed/extended 
email traces (always there).

<u>Office 365 first tries to attribute a message as originating, then as incoming.</u>

**High-level logic:**
- **Originating**: 
	- emails sent from an organization's exchange online mailbox
	- **or**, emails received from an organization's "on-prem" (Your org) inbound connector **with** a MailFrom domain matching one of that 
organization's accepted domains
- **Incoming**:
	-  emails received by O365 for which the RcptTo domain matches an organization's accepted domain
	-  if the email is received via a Partner type inbound connecter, then it is attributed to that connector,  otherwise it is attributed 
to the "default inbound connector"

**Detailed logic:**
- First, gather the following:
	- Connecting IP
	- Client certificate presented, if any. If so, look at subject name and SAN
	- MailFrom domain
	- RcptTo domain
- Originating:	
	- If a client certificate is presented, use the subject name and SAN to go through **all** O365 tenants, looking for "on-prem" (Your org) inbound 
connectors using such a certificate for client authentication, and then try to **narrow it down** to one O365 tenant by looking for **the one** that has 
the **most-specific match** between the certificate domain information and the list of accepted domains. If it yields a result, the email is attributed as 
originating
	- **If** the previous step didn't yield a result (no certificate used, or no "your org" inbound connector match, or no accepted domain match), 
**then** look at the connecting IP, and look for **all** O365 tenants with an "on-prem" (Your org) inbound connector using that IP to authenticate it, 
and then **narrow it down** to **the one** tenant that has the MailFrom domain as one of its accepted domains. If it yields a results, the email is 
attributed as originating
- Incoming:
	-  If the email couldn't be attributed as originating, then try to attribute it as incoming
	-  Go through all O365 tenants, looking for one for which the RcptTo domain is listed in its list of accepted domains
		- If one is found, we look for a Partner type inbound connector in that tenant matching the connection (through client certifcate 
or connecting IP authentication)
			-    if one is found, the email is attributed to that connector
			-    otherwise it's attributed to the "default inbound connector"
		- If none are found, then the email is rejected 

#### <u>Message authentication:</u>

Authentication information can be found in the **X-MS-Exchange-Organization-AuthAs** and **X-MS-Exchange-Organization-AuthSource** headers 
(for an organization's tenant current inspection), as well as in the **X-MS-Exchange-CrossTenant-AuthAs** and **X-MS-Exchange-CrossTenant-AuthSource** 
headers (for an organization's tenant previous inspection, when it first came to the tenant, which can happen if for example the organization uses 
a separate email security solution for inspection, that then sends it back to the tenant after its processing).

- **Internal**: messages are considered Internal **if they are authenticated** in some manner
	- Authenticated mailbox on-prem or Exchange Online (StoreDriver)
	- Authenticated mailbox using SMTP client submission (SMTP AUTH)
	- Received through an on-prem receive connector with ExternalAuthoritative (Externally Secured) permission enabled
	- Came into Exchange Online via an inbound connector with TreatMessagesAsInternal set to “true” and the sender is an accepted domain.
- **External**: messages are considered External if they are received through an **anonymous** source
	- Internet
	- SMTP relay (receive connector without ExternalAuthoritative)
	- Submitted by Pickup directory

**Message authentication (internal vs anonymous/external) is important because it's used in many important decisions**:
- **Internal**:
	- **Bypass spam filters, spoof verdict, phish controls and anti-impersonation controls for inbound email**. Internal email going outbound to 
On-Prem is still scanned by EOP.
	- **Bypass ATP SafeLink Policy** if EnableForInternalSenders is set to $False 
	- Allowed to send to distribution lists where RequireSenderAuthenticationEnabled is set to $True by default
	- By default, Internal messages are processed by resource mailboxes and the Resource Booking Attendant
	- In certain circumstances, may trigger transport rule condition “Is received from: Inside the organization”	
- **Anonymous**:
	- **Scanned by spam filters**
	- Will be rejected to distribution lists that require authentication
	- Will not be processed by the Resource Booking Attendant (ProcessExternalMeetingMessages must be set to $True)
	- Display name of the sender doesn’t resolve to an object in the Global Address List
	- Images won’t automatically downloaded in an email
	- Office documents open in “Protected View” mode
	
**Are originating messages always marked as Internal ? No, not necessarily. Likewise, a non-authenticated message received from external sources may 
be marked as Internal.**

#### <u>References:</u>

[Office 365 Message Attribution](https://techcommunity.microsoft.com/t5/exchange-team-blog/office-365-message-attribution/ba-p/749143)<br>
[Demystifying and troubleshooting hybrid mail flow: when is a message internal?](https://techcommunity.microsoft.com/t5/exchange-team-blog/
demystifying-and-troubleshooting-hybrid-mail-flow-when-is-a/ba-p/1420838)

