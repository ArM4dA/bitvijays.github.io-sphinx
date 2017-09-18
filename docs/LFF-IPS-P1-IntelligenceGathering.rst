***************************************************************
Infrastructure PenTest Series : Part 1 - Intelligence Gathering
***************************************************************

This post (always Work in Progress) would list the technical steps which might be important while doing the information gathering of an organization when we only know the company name or it’s domain name such as example.com.

**Thanks to Vulnhub-ctf team, bonsaiviking, recrudesce, Rajesh and Tanoy**

Suppose, we have to do a external/ internal penetration test of a big organization with DMZ, Data centers, Telecom network etc.

Scenarios
=========

Mostly, there are only two scenarios either we are outside/ inside the organization.

Outside - External
------------------

If we are outside or doing external pentest. Probably, we need to figure out the attack surface area first

Example:

What are the

* Different domain/ subdomains present? (like example.com -- domain; ftp.example.com -- subdomain)
* Different IP Address/ Network ranges/ ASN Number assigned?
* Different Services/ Ports running on those IP Addresses?
* Email addresses or People working in the organization?
* What are the different Operating Systems/ Software used?
* Any breaches which happened in the organization? 

Possibly, we would be able to compromise a user credential or vulnerable service running and get inside the internal network of the organization.

Inside - Internal
-----------------

When we are inside the organization (Let's say physically), probably, posing as internal employee (already have access to the internal network) or external consultant (with no internal network access as of now). Let's explore what are the options we have as external consultant sitting in a conference room.

Wired LAN
^^^^^^^^^

If there's a LAN cable around and we plug it in our computer, there might be few possibilities

* DHCP (Dynamic Host Configuration Protocol) is enabled and your machine is provided with an IP Address.
* DHCP is disabled; however, LAN cable is working. In this case, probably, we could sniff the network and figure out the near-by IP Address, netmask and default gateway.
* Network Access Control is enabled, then probably we would need to search for a Printer attached to network (clone it's MAC address and try) or IP Phones or any Hub.
* LAN port is disabled (Here you can't much! Can you?).

Wireless LAN
^^^^^^^^^^^^

* Check for Open/ Guest Wi-Fi - If you are connected somehow, probably try to internal network ranges. Most probably, the organization would have segregated the network well. However, sometimes DNS Name can be resolved.
* Check if any WEP/ WPA2 networks are present. If so, probably, can try to crack them.

Once you are inside, we need to find answer to the above questions (Outside - External section).

.. Note:: We can do fingerprinting from both Internal/ External of the organization

Responder/ Inveigh
^^^^^^^^^^^^^^^^^^

Once, you are inside, probably the first thing would be utilize **Responder** or **Inveigh** in the **Analyze mode**.

* `Responder <https://github.com/SpiderLabs/Responder>`_ : Responder is a LLMNR, NBT-NS and MDNS poisoner, with built-in HTTP/ SMB/ MSSQL/ FTP/ LDAP rogue authentication server supporting NTLMv1/ NTLMv2/ LMv2, Extended Security NTLMSSP and Basic HTTP authentication. It is very important to understand LLMNR, NBT-NS. Understand the basics by reading the `Local Network Attacks: LLMNR and NBT-NS Poisoning <https://www.sternsecurity.com/blog/local-network-attacks-llmnr-and-nbt-ns-poisoning>`_, `What is LLMNR & WPAD and How to Abuse Them During Pentest ? <https://pentest.blog/what-is-llmnr-wpad-and-how-to-abuse-them-during-pentest/>`_  and `How to get Windows to give you credentials through LLMNR <https://www.pentestpartners.com/security-blog/how-to-get-windows-to-give-you-credentials-through-llmnr/>`_

 Basically it's like this

 * Let's say a user wants to access a file server named "NAS001" by \\\\NAS001, however, mistakenly types in \\\\NAS01.  
 * The query goes to the DNS server to resolve the IP of NAS01. However, as it's not a valid hostname, DNS Server responds to the user saying that it doesn’t know that host.
 * The user then broadcasts on the local network if anyone knows who is \\\\NAS01
 * The attacker (if on the same network) seeing the opportunity says "I am NAS01 here is my IP Address"
 * The user believes the attacker and sends its own username and NTMLv2 hash to the attacker.
 * The attacker gathers all the hashes and then crack the hash to discover the password offline.

 Recently, the Responder also have the functionality of Multi-relay, which allows you to relay the NTLMv1/2 authentication to a specific target and possibly execute the code (during a successful attack) on the target node. NotSoSecure has written a detailed blog on `Pwning with Responder – A Pentester’s Guide <https://www.notsosecure.com/pwning-with-responder-a-pentesters-guide/>`_ 

 Similar to Python Responder, there's inveigh

* `Inveigh <https://github.com/Kevin-Robertson/Inveigh>`_ is a PowerShell LLMNR/ mDNS/ NBNS spoofer and man-in-the-middle tool designed to assist penetration testers/ red teamers that find themselves limited to a Windows system.

NTLM/ NTLMv1/v2 / Net-NTLMv1/v2
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

*Probably, We should cover this in Exploitation. However as we have mentioned Responder/ Inveigh here, it makes sense to include this*

* NTLM: NTLM hashes are stored in the Security Account Manager (SAM) database and in Domain Controller's NTDS.dit database. 

 ::

  aad3b435b51404eeaad3b435b51404ee:e19ccf75ee54e06b06a5907af13cef42

 The LM hash is the one before the semicolon and the NT hash is the one after the semicolon. Starting with Windows Vista and Windows Server 2008, by default, only the NT hash is stored.

* NTLMv1/v2 / Net-NTLMv1/v2 : Net-NTLM hashes are used for network authentication (they are derived from a challenge/response algorithm and are based on the user's NT hash). Here's an example of a Net-NTLMv2 (a.k.a NTLMv2) hash 

 ::

  admin::N46iSNekpT:08ca45b7d7ea58ee:88dcbe4446168966a153a0064958dac6:5c7830315c7830310000000000000b45c67103d07d7b95acd12ffa11230e0000000052920b85f78d013c31cdb3b92f5d765c783030 

From a pentesting perspective:

* You CAN perform Pass-The-Hash attacks with NTLM hashes.
* You CANNOT perform Pass-The-Hash attacks with Net-NTLM hashes

The above has been taken from `Practical guide to NTLM Relaying in 2017 (A.K.A getting a foothold in under 5 minutes) <https://byt3bl33d3r.github.io/practical-guide-to-ntlm-relaying-in-2017-aka-getting-a-foothold-in-under-5-minutes.html>`_ He has explained it very well and also showed how to own the network using relaying the hashes from Responder to get a system shell. Another good blog to understand this is `SMB Relay Demystified and NTLMv2 Pwnage with Python <https://pen-testing.sans.org/blog/pen-testing/2013/04/25/smb-relay-demystified-and-ntlmv2-pwnage-with-python>`_


Fingerprinting
==============

We can either do **Passive fingerprinting** (method to learn more about the enemy, without them knowing it) or **Active fingerprinting** (process of transmitting packets to a remote host and analyzing corresponding replies). **Passive fingerprinting** and **Active fingerprinting** can be done by using various methods such as

+------------------------------------------------+--------------------------------------+
|         Passive Fingerprinting                 |       Active Fingerprinting          |
+================================================+======================================+
| - whois                                        | - Finding DNS, MX, AAAA, A           |
+------------------------------------------------+--------------------------------------+
| - ASN Number                                   | - DNS Zone Transfer                  |
+------------------------------------------------+--------------------------------------+
| - Enumeration with Domain Name                 | - SRV Records                        |
+------------------------------------------------+--------------------------------------+
| - Publicly available scans of IP Addresses     | - Port Scanning                      |
+------------------------------------------------+--------------------------------------+
| - Reverse DNS Lookup using External Websites   |                                      |
+------------------------------------------------+--------------------------------------+

Do you remember, We need to find answers to 

+---------------------------------------------------------------+-------------------------------------------------------+
|     Questions (What are the)                                  | Answer                                                |
+===============================================================+=======================================================+
| Different domain/ subdomains present?                         | whois, DNS-MX/AAAA/A/SRV, Enumeration with Domain Name|
+---------------------------------------------------------------+-------------------------------------------------------+
| Different IP Address/ Network ranges/ ASN Number assigned?    | DNS, ASN-Number, DNS-Zone-Transfer                    |
+---------------------------------------------------------------+-------------------------------------------------------+
| Different Services/ Ports running on those IP Addresses?      | Public Scans of IP/ Port Scanning                     |
+---------------------------------------------------------------+-------------------------------------------------------+
| Email addresses or People working in the organization?        | harvestor, LinkedIn                                   |
+---------------------------------------------------------------+-------------------------------------------------------+
| What are the different Operating Systems/ Software used?      | FOCA                                                  |
+---------------------------------------------------------------+-------------------------------------------------------+
| Any breaches which happened in the organization?              |                                                       |
+---------------------------------------------------------------+-------------------------------------------------------+

The active and passive fingerprinting would help us to get those answers!

Passive Fingerprinting:
=======================

Whois
-----
Whois provide information about the registered users or assignees of an Internet resource, such as Domain name, an IP address block, or an autonomous system. 

whois command acts differently for ip address and domain name.

* In Domain name, it just provides registrar name etc.
* In IP address, it provides the net-block, ASN Number etc.

::

  whois <Domain Name/ IP Address>  
  -H Do not display the legal disclaimers some registries like to show you.                                
      
Googling for

:: 

  "Registrant Organization" inurl: domaintools


also helps for to search for new domains registered by the same organization. "Registrant Organization" is present in the output of whois. This technique was used by person who compromised FinFisher in the `writeup <http://pastebin.com/raw/cRYvK4jb>`__.

ASN Number
----------

We could find AS Number that participates in Border Gateway Protocol (BGP) used by particular organization which could further inform about the IP address ranges used by the organization. ASN Number could be found by using Team CMRU whois service

:: 
    
  whois -h whois.cymru.com " -v 216.90.108.31"                         |
      
If you want to do bulk queries refer @ `IP-ASN-Mapping-Team-CYMRU <http://www.team-cymru.org/IP-ASN-mapping.html>`_

Hurricane Electric Internet Services also provide a website `BGPToolkit <http://bgp.he.net>`__ which provides your IP Address ASN or search function by Name, IP address etc. It also provides AS Peers which might help in gathering more information about the company in terms of its neighbors.

.. Todo ::  Commandline checking of subnet and making whois query efficient.

Recon-ng
^^^^^^^^^^^

* use recon/domains-hosts/bing\_domain\_web : Harvests hosts from Bing.com by using the site search operator.
* use recon/domains-hosts/google\_site\_web : Harvests hosts from google.com by using the site search operator.
* use recon/domains-hosts/brute\_hosts : Brute forces host names using DNS.
* use recon/hosts-hosts/resolve : Resolves the IP address for a host.
* use reporting/csv : Creates a CSV file containing the specified harvested data.

Jason Haddix has created a dynamic resource script for sub-domain discovery which is available `here <https://github.com/jhaddix/domain>`__. Simply put the domain name and it runs the necessary modules, creates a new workspace and save the report.
         
.. Todo :: Check API option too, why google\_site\_web is failing, add a module to add ASN Info and Location Info too.
        

The Harvester
^^^^^^^^^^^^^

The harvester provides a email address, virtual hosts, different domains, shodan results for the domain. Provides really good results, especially if you combine with shodan results as it may provide server versions and what's OS is running on the IP address.

:: 

  Usage: theharvester options      
     -d: Domain to search or company name                          
     -b: data source: google, googleCSE, bing, bingapi, pgp        
                      linkedin, google-profiles, people123, jigsaw,
                      twitter, googleplus, all
     -v: Verify host name via dns resolution and search for virtual hosts                              |
     -f: Save the results into an HTML and XML file 
     -c: Perform a DNS brute force for the domain name             
     -t: Perform a DNS TLD expansion discovery
     -e: Use this DNS server   
     -h: use SHODAN database to query discovered hosts             |
         

.. Todo :: Combine these results with recon-ng and DNS Dumpsters and create one csv with all results.


Enumeration with Domain Name (e.g. example.com) using external websites
-----------------------------------------------------------------------

If you have domain name you could use

DNS Dumpster API
^^^^^^^^^^^^^^^^

We can utilize DNS Dumpster API to know the various sub-domain related to that domain.
:: 
       
  curl -s http://api.hackertarget.com/hostsearch/?q=example.com > hostsearch    

and the various dns queries by

:: 

  curl -s http://api.hackertarget.com/dnslookup/?q=example.com > dnslookup      

Google search operators
^^^^^^^^^^^^^^^^^^^^^^^^

* **site**: Get results from certain sites or domains.
* **filetype:suffix**: Limits results to pages whose names end in suffix. The suffix is anything following the last period in the file name of the web page. For example: filetype:pdf
* **allinurl/ inurl**: Restricts results to those containing all the query terms you specify in the URL. For example, [ allinurl: google faq ] will return only documents that contain the words “google” and “faq” in the URL, such as “www.google.com/help/faq.html”.
* **allintitle/ intitle**:Restricts results to those containing all the query terms you specify in the title.

Three good places to refer are `Search Operators <https://support.google.com/websearch/answer/2466433>`__, `Advanced Operators <https://sites.google.com/site/gwebsearcheducation/advanced-operators>`__ and `Google Hacking Database <https://www.exploit-db.com/google-hacking-database/>`__.

Other Tools
^^^^^^^^^^^

* `Mcafee Site Digger <http://www.mcafee.com/in/downloads/free-tools/sitedigger.aspx>`__ searches Google’s cache to look for vulnerabilities, errors, configuration issues,proprietary information, and interesting security nuggets on web sites.
* `SearchDiggityv3 <http://www.bishopfox.com/resources/tools/google-hacking-diggity/attack-tools/>`__ is Bishop Fox’s MS Windows GUI application that serves as a front-end to the most recent versions of our Diggity tools: GoogleDiggity, BingDiggity, Bing, LinkFromDomainDiggity, CodeSearchDiggity, DLPDiggity, FlashDiggity, MalwareDiggity, PortScanDiggity, SHODANDiggity, BingBinaryMalwareSearch, and NotInMyBackYard Diggity.


Publicly available scans of IP Addresses
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* `Exfiltrated <https://exfiltrated.com/>`__ provides the scans from the 2012 Internet Census. It would provide the IP address and the port number running at the time of scan in the year 2012.
* `Shodan <https://www.shodan.io/>`__: provides the same results may be with recent scans. You need to be logged-in. Shodan CLI is available at `Shodan Command-Line Interface <https://cli.shodan.io/>`__

Shodan Queries 

:: 

  title   : Search the content scraped from the HTML tag
  html    : Search the full HTML content of the returned page
  product : Search the name of the software or product identified in the banner
  net     : Search a given netblock (example: 204.51.94.79/18)
  version : Search the version of the product
  port    : Search for a specific port or ports
  os      : Search for a specific operating system name
  country : Search for results in a given country (2-letter code)
  city    : Search for results in a given city

.. Todo :: Learn how to access Shodan with API

* `Censys <https://censys.io/>`__: Censys is a search engine that allows computer scientists to ask questions about the devices and networks that compose the Internet. Driven by Internet-wide scanning, Censys lets researchers find specific hosts and create aggregate reports on how devices, websites, and certificates are configured and deployed. A good feature is the Query metadata which tells the number of Http, https and other protocols found in the IP network range.

 Censys.io queries
   
 :: 

  ip:192.168.0.0/24 -- CIDR notation

           
Reverse DNS Lookup using External Websites
------------------------------------------

Even after doing the above, sometimes we miss few of the domain name. Example: Recently, In  one of our engagement, the domain name was example.com and the asn netblock was 192.168.0.0/24. We did recon-ng, theharvester, DNS reverse-lookup via nmap. Still, we missed few of the websites hosted on same netblock but with different domain such as example.in. We can find such entries by using ReverseIP lookup by

DomainTools Reverse IP Lookup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
`Reverse IP Lookup by Domaintools <http://reverseip.domaintools.com>`__: Domain name search tool that allows a wildcard search, monitoring of WHOIS record changes and history caching, as well as Reverse IP queries.

PassiveTotal
^^^^^^^^^^^^
`Passive Total <https://www.passivetotal.org/>`__ : A threat-analysis platform created for analysts, by analysts.

Server-Sniff
^^^^^^^^^^^^

`Server Sniff <http://serversniff.net.ipaddress.com/>`__ : A website providing IP Lookup, Reverse IP services.

Robtex
^^^^^^
`Robtex <https://www.robtex.com/>`__ : Robtex is one of the world's largest network tools. At robtex.com, you will find everything you need to know about domains, DNS, IP, Routes, Autonomous Systems, etc. There's a nmap nse `http-robtex-reverse-ip <https://nmap.org/nsedoc/scripts/http-robtex-reverse-ip.html>`__ which can be used to find the domain/ website hosted on that ip.

::
 
  nmap --script http-robtex-reverse-ip --script-args http-robtex-reverse-ip.host='XX.XX.78.214'
  Starting Nmap 7.01 ( https://nmap.org ) at 2016-04-20 21:39 IST
  Pre-scan script results:
  | http-robtex-reverse-ip: 
  |   xxxxxxindian.com
  |_  www.xxxxxindian.com

         
Active Fingerprinting
=====================

Most probably by now we have gathered all the public available information without interacting with client infrastructure. Next, we can use **DNS enumeration** to gather more information about the client. The below information could gather externally as well as internally. However, amount of information gathered from internal network would definitely be more than when done externally.

Finding DNS, MX, AAAA, A using
------------------------------
      
host
^^^^

:: 
 
  host <domain> <optional_name_server>
  host -t ns <domain>           -- Name Servers
  host -t a <domain>            -- Address
  host -t aaaa <domain>         -- AAAA record points a domain or subdomain to an IPv6 address
  host -t mx <domain>           -- Mail Servers   
  host -t soa <domain>          -- Start of Authority
  host <IP>                     -- Reverse Lookup

Example:

::
 
  host -t ns zonetransfer.me
  zonetransfer.me name server nsztm1.digi.ninja.
  zonetransfer.me name server nsztm2.digi.ninja.

nslookup
^^^^^^^^

::

  nslookup - <optional_name_server>
  set type=mx
  set type=ns

DNS Zone Transfer: Using
--------------------------

host
^^^^

:: 

  host -l <Domain Name> <DNS Server>

Try zonetransfer using host for zonetransfer.me using their name servers.

Dig
^^^^
        
:: 
  
  dig axfr <domain_name> @nameserver

Try zonetransfer using dig for zonetransfer.me using their name servers.
        
dnsrecon
^^^^^^^^

:: 
         
  dnsrecon -d <domain> -t axfr  

dnsrecon could also be used for other purposes such as finding nameservers, mailserver, forward reverse lookup

:: 

  -d, --domain      <domain>          Domain to Target for enumeration.
  -r, --range       <range>           IP Range for reverse look-up brute force in formats (first-last) or in (range/bitmask).
  -n, --name_server <name>            Domain server to use, if none is given the SOA of the target will be used

DNSEnum
^^^^^^^

DNS Enumeration tool

:: 

  dnsenum <domain>

SRV Records
^^^^^^^^^^^

Service record (SRV record) is a specification of data in the Domain Name System defining the location, i.e. the hostname and port number, of servers for specified services. An SRV record has the form:

* **Retrieving an SRV record:**

 :: 

   $ dig _sip._tls.example.com SRV

   $ host -t SRV _sip._tls.example.com

   $ nslookup -querytype=srv _sip._tls.example.com

   $ nslookup
    > set querytype=srv
    > _sip._tls.example.com

* **Usage:** 

 SRV records are used by the below standardized communication protocols.

 :: 

   Teamspeak 3 (since version 3.0.8 - Neither priority nor weight is taken into consideration. The client appears to choose an SRV record at random for a connection attempt.[1])
   Minecraft (since version 1.3.1, _minecraft._tcp)
   CalDAV and CardDAV
   Client SMTP Authorization
   DNS Service Discovery (DNS-SD)
   IMPS
   Kerberos
   LDAP
   Puppet
   SIP
   XMPP
   Mail submission, Post Office Protocol, and Internet Message Access Protocol
   Libravatar uses SRV records to locate avatar image servers
   Microsoft Lync
   Citrix Receiver

 Checkout the brute\_srv function in the dnsrecon tool script to get familiar with the different SRV names and services.


Internal Infrastructure Mapping
================================

All the steps in 2.a which are DNS related recon could also be performed in the internal penetration testing provided we have the access to the internal DNS Server. After, we have gathered all the information from DNS enumeration, still we haven't enumerated internal infrastructure. We apply the below methods to enumerate further.

Internal Network Range Identification
-------------------------------------

In many instances, we are provided or expected to find vulnerabilities in a 10.0.0.0/8 network which would contain around 16 million IP Addresses. Scanning 16 million IP address in a considerable time is difficult. In which case, we need faster and targeted result. So, how do we find out the ranges?

Ping Gateway IP Addresses
^^^^^^^^^^^^^^^^^^^^^^^^^

Let's say internally, we got an IP address 192.168.56.101 netmask 255.255.255.0 with a default gateway of 192.168.56.1. It is a high probability that rest of network rangers would have been defined as /24 CIDR. In that case, a ping scan to 192.168.*.1 with a watch on the TTL would possibly reveal what are the other network ranges.

::

 nmap -sn -v -PE 192.168.*.1

DNS Enumeration
^^^^^^^^^^^^^^^^
   
If you are connected to a internal dns server, you may query it with

::

  dig -t any <domainname>

             
which should result in outputting different name servers, mail servers, A, AAAA, SOA records which would possibly give you a inner scenario how the network has been designed as there can be different nameservers, domain controllers for different locations, internal departments etc.
         
.. Todo :: Convert dig output directly into hostname, ip address format.
       
  
Internal Portal Links
^^^^^^^^^^^^^^^^^^^^^

Most of the organizations have one internal portals which serves has a one-stop links to every possible portal link. This could also result in some internal range exposure.
         
.. Todo :: Write the script for grep and printing host and IP address and combine it with DNS Enumeration.
      
Reverse DNS Lookup
^^^^^^^^^^^^^^^^^^^

Nmap provides a List scan option which does the reverse lookup. It provides the hostnames of the IP Address

:: 

  nmap -sL 10.0.0.0/8

It can also be used with the below options:

::
 
  --randomize-hosts  : make the scans less obvious to various network monitoring systems
  --dns-servers server1,server2 : By default, it would use the dns servers which are listed in resolve.conf (if you haven't used --system-dns option). We can also list custom servers using these options.


Identifying Alive IP Addresses
------------------------------

Nmap by default provides a -sn Ping scan option. The default host discovery done with -sn consists of an ICMP echo request, TCP SYN to port 443, TCP ACK to port 80, and an ICMP timestamp request by default. This works as if ICMP echo request is blocked, nmap would know if a host is alive if it receives any response from port 443 or 80 or timestamp reply.
   
Let's see what the nmap does when do a ping scan.

:: 
      
  nmap -sn -n 10.0.0.230
  #My IP is 10.0.0.1
        
It is very important to mention that -n option (No DNS resolution) should be used going forward as we have already did DNS resolution while using List scan. Since DNS can be slow even with Nmap's built-in parallel stub resolver, this option can slash scanning times. TCP Dump output is presented here. As both the IP addresses are in the same subnet, nmap would use ARP Ping scan to find the alive IP Address.

:: 

  22:11:27.292054 ARP, Request who-has 10.0.0.230 (Broadcast) tell 10.0.0.1, length 28
  22:11:27.361100 ARP, Reply 10.0.0.230 is-at 8c:64:22:3b:2b:2d (oui Unknown), length 28 
 		 
However, this behavior can be changed using --disable-arp-ping  
     
:: 

  nmap -sn 10.0.0.230 --disable-arp-ping

TCPdump output is as below One ICMP Echo Request, SYN to Port 443, ACK to Port 80 and a time stamp request.

:: 

  22:14:02.742180 IP 10.0.0.1 > 10.0.0.230: ICMP echo request, id 45066, seq 0, length 8
  22:14:02.742222 IP 10.0.0.1.59246 > 10.0.0.230.https: Flags [S], seq 3994420539, win 1024, options [mss 1460], length 0
  22:14:02.742234 IP 10.0.0.1.59246 > 10.0.0.230.http: Flags [.], ack 3994420539, win 1024, length 0
  22:14:02.742241 IP 10.0.0.1 > 10.0.0.230: ICMP time stamp query id 38635 seq 0, length 20
  22:14:02.801243 IP 10.0.0.230 > 10.0.0.1: ICMP echo reply, id 45066, seq 0, length 8
  22:14:02.801930 IP 10.0.0.230.https > 10.0.0.1.59246: Flags [R.], seq 0, ack 3994420540, win 0, length 0
  22:14:02.805083 IP 10.0.0.230.http > 10.0.0.1.59246: Flags [R], seq 3994420539, win 0, length 0
  22:14:02.805930 IP 10.0.0.230 > 10.0.0.1: ICMP time stamp reply id 38635 seq 0: org 00:00:00.000, recv 16:40:52.731, xmit 16:40:52.731, length 20


If you use --reason option, nmap would tell why it thinks the host is alive. In the below case (received echo-reply).

:: 

  Nmap scan report for 10.0.0.230
  Host is up, received echo-reply (0.073s latency).
      
If we only want to send ICMP Ping query (as if the host replies to it, the other three packets (SYN 443, ACK 80 and Timestamp) are extra burden. (I may be wrong here). We can use

::

  nmap -n -sn -PE --disable-arp-ping 10.0.0.230 

TCP Dump output:

:: 
 
  22:30:20.768525 IP 10.0.0.1 > 10.0.0.230: ICMP echo request, id 39366, seq 0, length 8
  22:30:20.826098 IP 10.0.0.230 > 10.0.0.1: ICMP echo reply, id 39366, seq 0, length 8

Please note, this ICMP scan would miss all the host which are alive but the firewall is dropping the ICMP echo request packet. However, if you want to find more hosts, it would be advisable to separate the list of IPs which responded to ICMP from the IP address scan range and run the scan again may be with SYN to 443 and ACK to 80 using PA, PS options.
      
Please also note Nmap's ICMP ping, by default, sends zero data as part of the ping. Nmap typically pings the host via icmp if the user has root privileges, and uses a tcp-ping otherwise. This is easily detected by the Snort IDS Rule 1-469 `SID 1-469 <https://www.snort.org/rule_docs/1-469>`__.

This could be evaded by using

:: 

  --data <hex string> (Append custom binary data to sent packets)
  --data-string <string> (Append custom string to sent packets)
  --data-length <number> (Append random data to sent packets)

Please note that you should use these options only on ICMP Echo Request for IDS Evasion as the data gets appended to every packet (ex. port scan packets). Designing the ideal combinations of probes as suggested in the Nmap Book is

::
     
  -PE -PA -PS 21,22,23,25,80,113,31339 -PA 80,113,443,10042
   Adding --source-port 53 might also help

The above combination would find more hosts than just the ping scan, however it also gonna cost a decent amount of time. Normal Time vs. Accuracy trade off.

Port Scanning
--------------
      
Once you have the list of IP Addresses which are alive, we can do port scan on them. Nmap provides multiple options such as

:: 

  -sS TCP SYN Stealth : Half Open SYN Scan : Nmap sends the SYN packet, Server would send SYN/ACK, System would send RST.
  -sT TCP Connect Scan : Nmap uses system to send the SYN scan : Connect full TCP Handshake
  -sU UDP Scan 
  -sA ACK Scan : Ack scan is generally used to map out firewall rule sets. Whether firewall is stateful or not.

Please note p0f recognizes Nmap's SYN scan because of the TCP Options such as TCP window size a multiple of 1024, and only the MSS option supported with a value of 1460 (Check the tcpdump output of Ping scan above, SYN Packet). Recently, a IRC user was getting filtered port while using SYN Scan whereas was getting OPN ports which using telnet or TCP Connect Scan. Also, A patch to allow a user to override the TCP Window size in SYN scan was just posted to the `Nmap DevelopmentList <http://seclists.org/nmap-dev/2015/q3/52>`__. 

By default, nmap scans the 1000 most popular ports of each protocol (gathered by scanning million of IP address). Scanning 1000 ports in an unknown environment with 16 million IP Address could be challenging. Nmap also provides -F Fast scan option which scans the 100 most common ports in each protocol. Otherwise it also provides --top-ports to specify an arbitrary number of ports. So, How do we know what are the ports scanned with --top-ports option. This could be found by

:: 
 
  nmap -sT -oG - -v | grep '^# Ports'

or 
  
:: 

  nmap localhost -F -oX - | grep '^<scaninfo'

Nmap needs an nmap-services file with frequency information in order to know which ports are the most common. See the section called `Well Known Port List: nmap-services <http://seclists.org/nmap-dev/2015/q3/52>`__ : for more information about port frequencies. We could provide ports to nmap by using -p option also, for example

:: 
 
  -p 22 : Scan single port
  -p 22,25,80 : Scan multiple ports with comma separated values. If -sS is specified TCP ports would be scanned. If -sU UDP Scan is specified, UDP Ports would be scanned.
  -p80-85, 443, 8000-8005 : Scan port with ranges.
  -p- : Scan all the ports excluding 0.
  -pT:21,22,25,U:53,111,161 : Scan TCP 21,22,25 and UDP Ports 53,111,161. -sU must also be specified.
  -p http* : wild cards may be used for ports with similar names. This would match nine ports including 80,280,443,591,593,8000,8008,8080,8443.

Port scanning via **netcat**: Netcat might not be the best tool to use for port scanning, but can be used quickly. netcat scans TCP ports by default, but we can perform UDP scans as well.
      
For a TCP scan, the format is

::
      
  nc -vvn -z xxx.xxx.xxx.xxx startport-endport
     -z flag is Zero-I/O mode (used for scanning)  
     -vv will provide verbose information about the results
     -n flag allows to skip the DNS lookup

For a UDP Port Scan, we need to add -u flag which makes the format

:: 
   
  nc -vvn -u -z xxx.xxx.xxx.xxx startport-endport


If we have windows machine without nmap, we can use `PSnmap <https://www.powershellgallery.com/packages/PSnmap/>`_


Identifying service versions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Ideally, we can use -sV to probe the ports to find the version running. When performing a version scan (-sV), Nmap sends a series of probes, each of which is assigned a rarity value correctly identified. However, high intensity scans takes longer. The intensity must be between 0 and 9. The default is 7.
      
Ideally, to avoid the IDS Detection, we should avoid using -sV option. However, we can keep the noise less by using --version intensity by which we can control the number of probes sent to determine the service. Setting this option to 0 will send only the Null probe (connect and wait for banner) and any probes that have been specifically listed as pertaining to the scanned port in nmap-service-probes. The other options available are below:

:: 

  --version-light (Enable light mode) : Alias for --version-intensity 2.
  --version-all (Try every single probe) : An alias for --version-intensity 9
  --version-trace (Trace version scan activity) : Print debugging information.
      
Also, when -sV is specified apart from the probes, all the scripts in the `Version <https://nmap.org/nsedoc/categories/version.html>`__ category are executed. These scripts could be prevented from running by removing them from the script.db catalog or by building Nmap without NSE support (./configure --without-liblua). However, if --version-intensity option is less than 7, those scripts won't be executed (I might be a little wrong here).
 
So our scan would become approx

:: 

  nmap <IP_Address_Range> -n --top-ports <number>/-p <Custom Port List> -sV --version-intensity 0/ (No -sV)


Performance
^^^^^^^^^^^
      
So, How can we improve the performance of our nmap scan, so that result could be achieved faster. However, as always we will have Time Vs Accuracy Trade off.
      
:: 
  
  -T<0-5>: Set timing template (higher is faster)
  --min-rtt-timeout/max-rtt-timeout/initial-rtt-timeout <time>: Specifies probe round trip time.
  --max-retries <tries>: Caps number of port scan probe retransmissions.
  --host-timeout <time>: Give up on target after this long
  --scan-delay/--max-scan-delay <time>: Adjust delay between probes
  --min-rate <number>: Send packets no slower than <number> per second
  --max-rate <number>: Send packets no faster than <number> per second
      
T0, T1, T2 is specifically for IDS Evasion. T3 is the default. We can set max-retries to a lower value such as 2. Currently it's 10 for T0, T1, T2, T3; 6 for T4 and 2 for T5.
     
Nmap Scripts
^^^^^^^^^^^^^
 
As bonsaiviking says in `They See Me Scanning Part 2 <http://blog.bonsaiviking.com/2015/07/they-see-me-scannin-part-2.html>`__ If you are wild enough to try NSE scripts against an IDS-protected target, you should know how to read Lua, since the script sources are the final authority on what data is sent. But if you're just looking to get a little better at blending in, these tips should help:

* Use --script-args-file to pass script arguments to Nmap from a file. This will keep your command line clean and make it harder to accidentally miss one of the options you choose
* Obviously avoid dos, intrusive, and exploit category scripts.
* Use scripts by name instead of by category, so that you know exactly what will be run.
* Thoroughly read the documentation for each script you intend to use. Set http.useragent to something believable that blends in. Currently, The HTTP scripts all use a User-Agent header that identifies as "Nmap Scripting Engine."

Output Options
^^^^^^^^^^^^^^^

:: 
 
  -oN/-oX/-oS/-oG <file>: Output scan in normal, XML, s|<rIpt kIddi3, and Grepable format, respectively, to t.
  -oA <basename>: Output in the three major formats at once
  --reason: Display the reason a port is in a particular state
  --open: Only show open (or possibly open) ports
  --packet-trace: Show all packets sent and received
  --resume <filename>: Resume an aborted scan : Filename should be .nmap or .gnmap

At this point, it's good to find what are the most common ports open in the scan we just performed by

:: 

  grep "^[0-9]\+" <nmap file .nmap extension> | grep "\ open\ " | sort | uniq -c | sort -rn | awk '{print "\""$1"\",\""$2"\",\""$3"\",\""$4"\",\""$5" "$6" "$7" "$8" "$9" "$10" "$11" "$12" "$13"\""}' > test.csv

Exploring the Network Further
------------------------------

By now, we would have information about what ports are open and possibly what services are running on them. Further, we need to explore the various options by which we can get more information.
       
Gathering Screenshots for http* services
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There are four ways (in my knowledge to do this)

* **http-screenshot NSE**: Nmap has a NSE script `http-screenshot <https://github.com/SpiderLabs/Nmap-Tools/blob/master/NSE/http-screenshot.nse>`__ This could be executed while running nmap. It uses wkhtml2image tool in the script. Sometimes, you may find that running this script takes a long time. It might be a good idea to gather the http\* running IP, Port and provide this information to wkhtml2image directly via scripting. You do have to install wkhtml2image and test with disable javascript and other options available.

* **httpscreenshot** from breenmachine: `httpscreenshot <https://github.com/breenmachine/httpscreenshot>`__ is a tool for grabbing screenshots and HTML of large numbers of websites. The goal is for it to be both thorough and fast which can sometimes oppose each other.

* **Eyewitness** from Chris Truncer: `EyeWitness <https://github.com/ChrisTruncer/EyeWitness>`__ is designed to take screenshots of websites, provide some server header info, and identify default credentials if possible.

* Another method is to use `html2image <https://code.google.com/p/java-html2image/>`__ which is a simple Java library converts plain HTML markup to image and provides client-side image-map using html element.

* **RAWR: Rapid Assessment of Web Resources**: `RAWR <https://bitbucket.org/al14s/rawr/wiki/Home>`__ provides with a customizable CSV containing ordered information gathered for each host, with a field for making notes/etc.; An elegant, searchable, JQuery-driven HTML report that shows screenshots, diagrams, and other information. A report on relevant security headers. In short, it provides a landscape of your webapplications. It takes input from multiple formats such as Nmap, Nessus, OpenVAS etc.
      
Information Gathering for http* Services
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* `WhatWeb <http://www.morningstarsecurity.com/research/whatweb>`__ recognises web technologies including content management systems (CMS), blogging platforms, statistic/analytics packages, JavaScript libraries, web servers, and embedded device. `Tellmeweb <https://www.aldeid.com/wiki/Tellmeweb>`__ is a ruby script to read Nmap Gnmap file and run whatweb on all of them. A `WhatWeb Result Parser <https://github.com/stevecoward/whatweb-parser>`__ also has been written which converts the results to CSV format. More information about advance usage can be found `Whatweb Advance Usage <https://github.com/urbanadventurer/WhatWeb/wiki/Advanced-Usage>`__.
      
* `Wapplyzer <http://wappalyzer.com>`__ is a Firefox plug-in. There are four ways (in my knowledge to do this)be loaded on browser. It works completely at the browser level and gives results in the form of icons.
* `W3Tech <http://w3techs.com/>`__ is another Chrome plug-in which provides information about the usage of various types technologies on the web. It tells the web technologies based on the crawling it has done. So example.com, x1.example.com, x2.example.com will show the same technologies as the domain is same (which is not correct).
      
* `ChromeSnifferPlus <https://github.com/justjavac/ChromeSnifferPlus>`__ is another chrome extension to sniff about the different web-technologies used by the website.
      
* `BuiltWith <http://builtwith.com/>`__ is another website which provides a good amount of information about the different technologies used by website.

NetBIOS Service
^^^^^^^^^^^^^^^^

Netbios listens on TCP Port 139, 445 and UDP Port 137. How do we machines on which these three ports or a combination are open and feed that IP information to nbtscan and enum4linux. We can do this by using grep such as

:: 

  grep -E "^Host.*[ ]137/open/udp" <Nmap .gnmap file>     : Grep 137 UDP Ports to run nbtscan
  grep -E "^Host.*[ ]139/open/tcp" <Nmap .gnmap file>     
  #If we want that tcp port 139 and 445 both must be open
  grep -E "^Host.*[ ]139/open/tcp" <Nmap .gnmap file> | grep -E "^Host.*[ ]445/open/tcp"  	 	 <Nmap .gnmap file> : Grep TCP 135 and 445 port to run enum4linux
  #If we want that tcp port 139 or 445 must be open
  grep -E "^Host.*[ ]139/open/tcp|[ ]443/open/tcp" <Nmap .gnmap file>

NBTSCAN
^^^^^^^^

:: 
  
  nbtscan
      -v        Verbose output. Print all names received from each host.
      -f filename     Take IP addresses to scan from file "filename"

      
enum4linux
^^^^^^^^^^^^
A Linux alternative to enum.exe for enumerating data from Windows and Samba hosts. It is is basically a wrapper around the Samba tools smbclient, rpclient, net and nmblookup. A very good usage guide is `enum4linx <https://labs.portcullis.co.uk/tools/enum4linux/>`__

         
SNMP Enumeration
^^^^^^^^^^^^^^^^^

For SNMP Enumeration, UDP Port 161 should be open. If the port 161 is open we can use

* **snmpcheck:**

 :: 
    
  snmpcheck -t <IP address>
       -c : SNMP community; default is public
       -v : SNMP version (1,2); default is 1
       -w : detect write access (separate action by enumeration)

* **snmpwalk:**

It also allows us to interact with the SNMP version 3. It also allows to extract particular nodes of a MIB tree.

 :: 
 
  snmpwalk -­c public ­‐v1 <IP Address>  : Enumerating  the  Entire  MIB  Tree
  snmpwalk -­c public ­‐v1 <IP Address>  <MIB Tree Number> : Enumerate particular node
      -v 1|2c|3     specifies SNMP version to use
      -c COMMUNITY      set the community string


* **OneSixtyOne:**

onesixtyone allows you to brute force the community strings, you could onesixty one tool

Attack Surface Area - Reconnaissance Tools
==========================================

Aquatone: A tool for domain flyovers
------------------------------------

`Aquatone <https://github.com/michenriksen/aquatone>`_ is a set of tools for performing reconnaissance on domain names. It can discover subdomains on a given domain by using open sources as well as the more common subdomain dictionary brute force approach. After subdomain discovery, AQUATONE can then scan the hosts for common web ports and HTTP headers, HTML bodies and screenshots can be gathered and consolidated into a report for easy analysis of the attack surface. Detailed blog at `AQUATONE: A tool for domain flyovers <http://michenriksen.com/blog/aquatone-tool-for-domain-flyovers/>`_

DataSploit
----------

`Datasploit <https://github.com/DataSploit/datasploit>`_ Tool to perform various OSINT techniques, aggregate all the raw data, and give data in multiple formats.

Overview:

* Performs OSINT on a domain / email / username / phone and find out information from different sources.
* Correlates and collaborate the results, show them in a consolidated manner.
* Tries to find out credentials, api-keys, tokens, subdomains, domain history, legacy portals, etc. related to the target.
* Use specific script / launch automated OSINT for consolidated data.
* Performs Active Scans on collected data.
* Generates HTML, JSON reports along with text files.

Spiderfoot
----------

`SpiderFoot <http://www.spiderfoot.net/>`_ is an open source intelligence automation tool. Its goal is to automate the process of gathering intelligence about a given target, which may be an IP address, domain name, hostname or network subnet. SpiderFoot can be used offensively, i.e. as part of a black-box penetration test to gather information about the target or defensively to identify what information your organization is freely providing for attackers to use against you.

Intrigue.io
-----------

`Intrigue <https://github.com/intrigueio/intrigue-core>`_ makes it easy to discover information about attack surface connected to the Internet. Intrigue utilizes common sources of OSINT via “tasks” to create “entities”. Each discovered entity can be used to discover more information, either automatically or manually.

Appendix-I : Interesting Stories
================================

Initial Compromise
-------------------

* `Apache and Java Information Disclosures Lead to Shells <http://threat.tevora.com/apache-and-java-information-disclosures-lead-to-shells/>`_ : Richard De La Cruz talks about a recent Red-Team engagement, where they discovered a series of information disclosures on a site allowing the team to go from zero access to full compromise in a matter of hours.

 * Information disclosures in Apache HTTP servers with mod_status enabled allowed our team to discover.jar files, hosted on the site.
 * Static values within exposed .jar files allowed our team to extract the client’s code signing certificate and sign malicious Java executables as the client.
 * These malicious .jar files were used in a successful social engineering campaign against the client.

.. disqus::
