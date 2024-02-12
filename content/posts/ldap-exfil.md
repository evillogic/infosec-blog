---
title: An LDAP Exfil Technique
date: 2020-05-06
draft: false
description: Using native LDAP w/ Active Directory (AD) to obfuscate data exfiltration
---

Recently I’ve been looking a lot at data exfiltration techniques in general, especially techniques that allow you to bypass network access controls and escape from PCI environments.

I recently discovered a little trick that you can use to traverse network boundaries and obfuscate the source of your data via LDAP, which means that as long as the environment uses active directory and there’s a device (such as a primary domain controller) that has the ability to cross network boundaries, you can leverage this technique to use any Windows box as a proxy between two boxes you control.

The idea goes like this — almost all windows boxes expose their IPC$ share over SMB, which doesn’t actually allow access to files but instead call specific methods from that machine. Among other things, this allows us to query a domain for a user. Not just the domain that the machine is joined to, but any domain. Simply put, we kindly ask the target to query a fake domain (set up by the attacker) to query for a specific user containing our data to be exfiltrated.

The mechanics of this are as follows:  
1. Query a target for User: Data and Domain: ATTACKER.COM  
2. Target issues a DNS SRV request for _ldap._tcp.SCF._sites.dc._msdcs.ATTACKER.COM  
3. If ATTACKER.COM replied correctly, Target issues a CLDAP request to port 389 containing the data we have specified.

There’s a second, bonus technique here that we can use this to obfuscate the source of our DNS exfiltration, as the requests will look like they’re coming from our target.

I’ve successfully tested this technique and works well. The only issues arise when UDP traffic is blocked, as the request will come out as a UDP CLDAP packet, or when the DNS has issues. I might post my PoC code for replying properly with a SRV record if there’s interest. 

This was heavily inspired by [Reino Mostert’s post a couple years back about Null Session attacks](https://web.archive.org/web/20220520135811/https://sensepost.com/blog/2018/a-new-look-at-null-sessions-and-user-enumeration/). This attack is possible where Null Session attacks are possible, but also since this is an exfiltration technique we can assume we’ve compromised domain credentials and perform the technique with those instead, which would make this technique blend in a little bit more and opens up the scope.

Another quick note is that this tecnique could be used for two-way communications since it allows for a response by the domain controller.