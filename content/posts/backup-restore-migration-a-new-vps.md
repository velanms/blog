---
author: "Velan Salis"
title: "Backup, Restore, Migration - a New VPS"
date: 2022-04-17T09:40:53+05:30
description: "Outlining my experiences moving from in-house selfhosted setup to a new remote VPS"
categories: []
tags: []
slug: "backup-restore-migration-a-new-vps"
draft: false
---
If you had been following my blog for some time, you would know that I host most of my services on a self-hosted VPS. For those who are unfamiliar, it's a computer located in a remote part of the world still enabling me to rent it, access it and host things on it so that me and other people can access it from any part of the world. Sounds revolutionary and exhilarating, can't deny.

However, the VPS that I had owned earlier had a massive storage of 2 TB, for which I had to make a tradeoff with the processing speed to make it “budge” into my budget (pun intended). Why 2 TB you might ask; well, That was to store numerous photos and videos that my family and I had clicked overtime which were aggregated by a program called Photoprism. But that tradeoff didn't seem worth the buck, since I was aggravated by the slowness in processing, and it also became obnoxious at times.

So I made a decision to migrate my setup to a new VPS.

## But, Photos and Videos?
Ahhh, Wait! I know you would ask that. I moved all my photos and videos (a hefty 45 GB!) to a paid hosted media management service called ente.io. Ente is built on top of open source and peer reviewed architecture with a healthy and growing community. Therefore, they are entirely accountable to its end users.

And to top it all off, they also offer end-to-end encryption to all the hosted media. I can be rest assured to know that no machine is scraping my images to figure out what sort of sandwich I ate this morning! Ente costs ₹299 / month for a 100 GB (the plan I took up amongst others), which I feel is sufficient for my needs. This is a price that I pay for a healthy space for my ideas and memories. And I believe it's worth it. :)

## A fun backup and restore routine!

I'm just going to copy and paste two lines here, which were so elegant that the whole backup and restore across two VPS's were a breeze. _[Credits: GitHub, Docker]_
* Backup: `docker run --rm --volumes-from CONTAINER -v $(pwd):/backup busybox tar cvfz /backup/backup.tar CONTAINERPATH`
 
* Restore: `docker run --rm --volumes-from CONTAINER -v $(pwd):/backup busybox bash -c \"cd CONTAINERPATH && tar xvf /backup/backup.tar --strip 1\`

This is the command that will save me a lot of hours eventually as I add more and more containers to my setup. This will also help me safely upgrade and downgrade my setup, make tarball backups (automation maybe?) as I go. A fun process, not gonna lie!

## Closing Thoughts

Moving to a new VPS reduced my expenses to 75% from the original expenses. All of this while making no compromises on the performance side of things. This makes me wonder if photos and videos storage in a personal VPS is as good as it's been glorified.

In my opinion, a personal VPS should always be inclined towards better performance with limited storage (as a tradeoff). If storage is really required, I believe a new VPS dedicated solely on storage with limited performance (as a tradeoff) can be rented and bridged with the personal VPS via FTP. That would cost less than those two combined. That's just my two cents.

Okay bye!