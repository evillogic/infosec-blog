---
title: Cracking Legacy Passwords
date: 2024-03-14
draft: false
description: How to crack MySQL323 (Half MD5) and SHA1 on AWS efficiently
---

The advent of cloud computing and ephemeral compute has to be one of the greatest things that ever happened to password cracking. Gone are the days of needing to buy an insane GPU rig, spend many to hours build it out, and have it be obsolete by the end of the year.

~~Unfortunately, the state of password cracking also hasn't changed very much in many years. Hashcat is still the king, rainbow tables are still the most effective method of looking things up, and to my knowledge there are no projects that exist to make deployment, cracking, and infra termination all happen in a single step.~~

Rainbow tables are still hard to come by and require torrenting as well, even now that bandwidth rates exceeding 25 Gigabit are readily available  on the cloud and 10 Gigabit fiber is currently being deployed to consumers around the globe. Free, readily available rainbow table sizes cap out around 1 TB from what I have seen, but somehow still require torrenting.

I recently needed to crack some MySQL passwords, and I found that torrenting these rainbow tables was not working on AWS (I can't imagine why /s). Curiously, it also didn't work on my home network though. Luckily, my teammate was able to find that all of the rainbow tables from freerainbowtables.com are also available for direct download, although the site doesn't advertise it. You can view them all [here](http://freerainbowtables.mirror.garr.it/rainbow/RTI2/).

## Modern Cracking Tools

Coalfire released [npk](https://github.com/c6fc/npk) in 2019, which is massively helpful for automated deployment of cloud resources to crack passwords. This is a very cool project that aims to automate the process of spinning up resources to crack passwords and shut them down in order to never need to buy a GPU or risk running expensive cloud resources for too long.

[Hashtopolis](https://github.com/hashtopolis/server) is the de-facto standard for cracking infrastructure now, which allows management of many cracking jobs through a convenient web application. It supports multiple users, has an API, and allows management of cracking clusers as well.

## Cracking SHA1 with Rainbow Tables

The rainbow tables are split into several parts, and those parts are split into several parts as well. I can only assume this is done for torrenting purposes. You can download them like this once you've found the table you want.

```bash
wget --no-proxy -r -np -nH -R "index.html*,*.md5sums" http://freerainbowtables.mirror.garr.it/rainbow/RTI2/mysqlsha1/mysqlsha1_numeric%231-12_0/
```

RainbowCrack seems to be best answer to actually using these tables, however for MySQL/SHA1 the algorithm was not implemented, so we had to resort to the older [rcracki_mt](https://sourceforge.net/projects/rcracki/files/rcracki_mt/) program to run the hashes through these tables. This is pretty slow, but still much cheaper and more accessible than running a p3.16xlarge instance on AWS.

Running rcracki_mt with 128 threads on the p3.16xlarge instance (since I already had it running hashcat, and the CPU was not being utilized), took 1-2 seconds per hash to precompute some lookup value. Precomputation of the hash occurs once per table, which translates to 4 times since the table I was using (numeric 1-12) was split into 4 parts. This meant that for the some 400 hashes I was cracking, it took about a little less than an hour. I'm not sure how long it would take for a more useful table, such as all characters 1-8 length, but it doesn't seem as fast as I would expect.

These programs used to be open source, but don't appear to be any longer. The github repository for rcracki_mt only contain the last release version of the tool now, and RainbowCrack appears to be intentionally closed source. While rcracki_mt supposedly supports GPU usage, it segfaulted when we tried to use it.

With all of the drawbacks to using these rainbow table tools, I would be surprised if you were convinced after reading this blogpost to go try and use them. However, I submit to you that hashcat even running on this very old, very fast hash type, with a very strong 8 Tesla V100 cards at its fastest settings, took around 30 hours to complete the 8 digit keyspace with all characters.

The total cost for running the p3.x16large instance for 30 hours is $720. Would you rather do that, or just download some rainbow tables?

If you are going to use hashcrack, it's not currently worth it to try to brute force anything. Even with today's GPUs, rainbow tables are still the clear winner when you can use them for passwords like this. On the same instance, Hashcrack was capable of incredible feats, running the entire [15GB Crackstation Wordlist](https://crackstation.net/crackstation-wordlist-password-cracking-dictionary.htm) through the [Dive rule](https://github.com/hashcat/hashcat/blob/master/rules/dive.rule) (790 KB of rules) under an hour. This is insane speed, and resulted in cracking many passwords the rainbow table could not, but is still unable to match the coverage and efficiency of the rainbow tables we were able to download. 

If you'd like to try this yourself, I recommend taking a look at [InfoSec Irvin's post](https://www.infosecirvin.info/blog/rainbowtables.html) about using rainbow tables. It was very helpful to us to actually get started. If the wget command doesn't work for you, [here is a script](/scripts/download-sha1-rt.sh) to download the 1TB mysql table.

I'm disappointed that there's not a clear, open source solution in this space, especially when NTLM, SHA1, and MD5 hashes will be around for awhile. This seems like a really good opportunity for a solid open source project in 2024.

## Cracking Half MD5 with TobTu's Collider

We also ended up finding some very old Half MD5 hashes. These were used by old versions of MySQL, and are generally referred to as MySQL323 hashes.

Hashcat was very fast at cracking these, but surprisingly not as fast as [Tobtu's Collider](https://tobtu.com/mysql323.php).

It's hard to definitively compare the two, since it seems Tobtu's collider uses a technique specific to half md5 to increase the hash rate, the tool took about 10 minutes to identify collisions for all of our ~80 MySQL323 hashes. Hashcat's rate for these hashes was about 1.3 TH (TerraHashes), but the collider was running at a rate of over 2 PH (PetaHashes). The collider increased in speed as it ran, capping out at around 200PH.

I ran it with about 30000 MiB of memory and 128 threads, without using the GPU. Initially, it sat there for a while, presumably generating 30GB of lookup table in RAM. Afterwards though, it was able to search for collisions all the way into the 12 digit character space. It also only used printable characters, which is something I found hashcat did not. This was very helpful, as all of the passwords it generated would not require debugging and confirmation of it it would work in practice.

I was also able to use the results it output as a wordlist into Hashcat, and Hashcat verified that the output for all hashes indeed resulted in a match for the hashes with its algorithm. I highly recommend this as a method to crack this hash type, and I would love to see how fast it could go if someone were make it Cuda compatible.
