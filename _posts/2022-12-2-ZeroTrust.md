---
layout: post
title:  "What exactly is the Zero Trust model?"
tags:
  - Zero Trust
---
"Zero Trust" is definitely a buzzword that gets thrown around a lot, usually by people/companies that want to sell you something, but what makes an architecture "Zero Trust"? Let's find out!

#### <u>How things were before Zero Trust</u>
Historically, enterprise resources (servers, applications, etc.) were located on-premises, within a company-controlled network, and any external access was mediated through edge firewalls / VPN gateways. In this model, it was easy to delineate a clear and strong perimeter around where company resources to be protected were, and the "bad outside world" to protect them from. To draw a parallel from times gone by, all the company resources were located within a castle (company's on-premise network) with fortified walls, towers, and a moat denoting a clear perimeter and protecting those resources from the outside world.

Even then, **there were issues with that model, the main one being *assumed trust:*** being located within the company's network granted implicit trust. The issue with this is that if an adversary gets a foothold in an organization's network, it allows for easy lateral movement; another issue is that it fails to address insider risk (employees doing bad things). So, if you think about it, it contravenes the critically important concept of *defense in depth* (multiple layers of security you have to get through to get to protected assets) and also doesn't help with the equally important concepts of *need to know* and *least privilege*, as you get a significant level of access to company resources from the inherent trust of simply being located within the company's network.

The trends of remote work, BYOD, and the move to the cloud have made the notion of a *clear* perimeter that could be drawn around enterprise resources obsolete: employees can be onsite, but can also be located anywhere in the world, they may use a company issued laptop, or their own laptop, phone or tablet, and workloads / applications can be running on servers on the company's network, or on one of many public cloud platforms. 
#### <u>So, what is Zero Trust?</u>
**Maybe the best way to describe Zero Trust is that it's not about not trusting anything, but about not having any *assumed* trust, meaning trust has to be earned and doesn't solely derive from location (within the company's network vs. outside)**. With Zero Trust, the only thing that's assumed is compromise, i.e. all network are assumed to be hostile. As a result, each and every single session / transaction has to be encrypted, authenticated, and authorized. We are thus moving away from perimeter-based security to a model focused on the security of individual transactions. Instead, **all users, devices and workflows (access to resources) have to be explicitly authenticated and authorized.**

The Zero Trust model was popularized by Google, that achieved the first at-scale implementation in 2009 through their BeyondCorp project. Several articles were penned by Google staff in 2014 in Usenix.org's ";login:" magazine, describing the solution, its benefits, and implementation journey. NIST also produced one of their "special publications", namely SP 800-207, in 2020, describing the concept in a formal way.
#### <u>What are the core tenets of the Zero Trust security model?</u>
**The Zero Trust security model does away with implicit trust granted by location (physical or logical), to move to a model where the environment (network) is always considered to be hostile (potentially compromised), regardless of location, and therefore trust needs to be dynamically and continually evaluated.
This means that each session / transaction / access request (based on desired granularity) has to be authenticated and authorized, and all communications encrypted.**

Zero Trust is a process, or journey, so it's more about concepts to be applied, rather than a particular target or destination. Nevertheless, it needs to take into account the three key elements below for each individual access decision:
1. **Trust in the *user***: trust in the user is achieved through a security token (ideally an SSO token) representing their validated identity, with identities being a set of attributes (name, team, entitlements, etc.) in a central identity provider. Multi-factor authentication is required. That trust is not static, but dynamic: it takes into account the context (external factors), and may challenge the user to re-authenticate based on factors such as geolocation, time of access, etc. as enforced by dynamic policies detecting anomalies from learned "normal" behavior, as well as by set rules (e.g. connection from "risky" countries)
2. **Trust in the *device***: not only do users have to be authenticated, the devices they use also do. This necessitates a strong asset management process in place, with a central system inventorying all devices (company or employee owned) and storing information (attributes) about them. Device authentication is usually achieved through the issuance of device certificates, necessitating TPMs on devices for their secure storage. Here as well, context is taken into account by dynamic policies when it comes to authorizing a device: ownership of the device (company vs. employee owned), OS version, patch level, security baseline, etc. Information such as OS version, patch level, security baseline as well as other, is provided by and necessitates a continuous diagnostic and mitigation (CDM) system whose role is to collect and provide such information to the access authorization process (it also has the ability to push down / enforce policies to the devices)
3. **Individual *resource access policy***: each resource should have its own access policy reflecting "need to know" and "least privilege" principles as it relates to granting access to it (application, data, etc.), to allow for access enforcement that is as granular as possible. Those policies should also set minimum trust requirements for both the user requesting the access, and their device. It could be through a set of hard (non-negotiable) requirements, or based on a score (different number of points attributed to each user and device validated security property)

Having talked about the 3 key considerations that need to be looked at for each access decision, we also need to talk about systems that allow for those access decisions to be made, as well as enforce them:
- **Policy decision point (PDP)**: this logical component (can be implemented in several different ways) is in charge of receiving / accessing all the relevant information about the user, their device, the resource access policy, as well as contextual information from external systems (CDM, activity logs, etc.) and using it to make its decision as to whether access should be granted or denied
- **Policy enforcement point (PEP)**: this logical component is responsible for allowing the access / connection to be made from the user device to the desired resource, monitoring it, and potentially terminating it should conditions change. The PEP relies on the PDP to provide the decision as to whether access should be granted or not, and networks need to be architected so that access to the desired resources is not possible but through the PEP: the PEP acts as a gatekeeper (choke point) for communication paths
- **Systems / components providing information and context to the PDP**: with the Zero Trust model, trust is to be dynamically and continually evaluated, making the context around each access request a key piece of the access decision puzzle. Such information needs to be provided by specific systems / components to the PDP. Below are some of those systems:
	- Continuous diagnostics and mitigation (CDM) system: system in charge of, as previously described, collecting and providing data on user endpoints such as OS version, patch level, whether or not the required security baseline is applied (EDR, etc.)
	- Industry compliance system: system that acts as a repository of all the policy rules an enterprise develops to ensure compliance
	- Threat intelligence feeds: internal or external feeds about newly discovered vulnerabilities, malware, etc. and trending attacks
	- Network and systems activity logs: the enterprise's own log aggregation and correlation solution (e.g. Splunk) to correlate the current access request to previous ones in an attempt to identify patterns indicative of an on-going attack
	- Resource access policies: the fine-grained access policies for each resource
	- Enterprise PKI: the enterprise public key infrastructure responsible for issuing, maintaining and revoking certificates that securely identify devices, users, resources, etc.
	- IdP systems: identity provider systems that maintain a database of users, their attributes, entitlements, etc. as well as authenticate users
	- Security Information and Event Management (SIEM) system: to collect security-centric information for later analysis

Below is a diagram showing how all those components interact with each other.

![ZeroTrust_diagram](/assets/images/ZeroTrust_diagram.png)

#### <u>How did Google implement Zero Trust?</u>

Here's a quick summary on how Google, through their BeyondCorp project, implemented Zero Trust.

First of all, they decided to remove any inherent trust from their own (corporate) internal network, and to treat it like any external unprivileged network by only providing basic network infrastructure services such as (DHCP, DNS, NTP), but no special inherent access to their applications (no firewall rules allowing access, nothing). They even took it further by making the decision of exposing their internal applications to the internet, showing that they don't consider their internal network to be any more secure than the internet, and that they are confident enough in their Zero Trust implementation to put it to the test by having it internet facing.

When it comes to the three pillars of Zero Trust &ndash;  trust in the user, trust in the device, individual fine-grained resource access policies &ndash; here's how Google implemented those:
1. Trust in the user: their identity provider is 100% driven by HR processes through automation. The HR system is used as the single source of truth of active employees (automated provisioning / deprovisioning of users), group memberships / entitlements are  automatically derived from the team / job function as per the HR system, etc. They also use an SSO system and MFA
2. Trust in the device: Google relied on a very strong asset management system, with a meta-inventory database holding relevant security information about every device, and tracking all changes to it, for the purpose of making access decisions. Each device was also issued a certificate for device authentication, held securely in their TPM
3. Individual fine-grained resource access policies: there is no access to resources but through an access proxy that gets thorough and current information about the user, their device, ascertains their authentication (SSO token for user and device certificate), and uses all that information as input to make access decisions. Access control decisions are made on a *per-request basis* (no long-lived session), in order to react quickly should conditions change, and access policies for each resource are fine-grained (i.e. only users from a specific team, meeting a set of security rules or a minimum security score, using a particular type of device, also meeting a set of rules of minimum security score, can access a particular resource). Those access proxies also ensure all communications are encrypted, protect from DDOS, ensure load balancing, etc. 