---
author: "Velan Salis"
title: "Nextcloud to Good Ol' Dav"
date: 2022-05-01T07:22:41+05:30
description: ""
categories: []
tags: []
slug: "nextcloud-to-good-ol-dav"
draft: false
---
We need standards, not tools

The above line really resonates with me now. It was a while ago when I saw this lurking in my Mastodon feed and was lost with a scroll. Unfortunately, I'm not able to find the person who said it, but whatever is said, is true. It's true because standards remove the Lock-In that is created by a tool and opens a person's horizons to a new beyond. So today I thought I would write about a web standard that made my life quite easier and that is CalDAV / CardDAV / WebDAV.

## But, Nextcloud?
Initially, I thought that Nextcloud was a good option to have my contacts/calendars and files synced up. Nextcloud is an amazing option, but it comes with its limitations. It could viably become a good option for me if it hadn't become a lock-in in itself.

Nextcloud provides all the above-mentioned tools. But, it is also required to run a Nextcloud instance (which sometimes becomes painstakingly difficult to figure out if something goes wrong), or pay for a cloud provider that runs a managed Nextcloud instance. Which would be expensive just for a simple backup of files, calendar and contacts.

I started with their free tier plan with one of the cloud providers. However, the response times became inconsistent sometimes. They aren't to blame. I was using their service for free, so they had no moral obligation to be accountable towards me. So I went in search for a better, lightweight option that could give me the same convenience with more effectiveness.

## Enter the – “DAV”
My perpetual search for convenience and effectiveness led me to DAV. I knew about DAV. But I was using it under the hood of Nextcloud, which provided the DAV service as an extension. And it was a no-brainer to run these services independently for better usability and convenience. But then I thought, better late than never and jumped right into running these on top of Docker.

I already own a self-hosted server. So all I had to do was, run the services on top of Docker and expose these endpoints to the outside world so that I can use them from my other devices. So I zero'd in onto the two Dockerhub images, they are,
* **bytemark/webdav** for WebDAV, which runs a small Apache server with WebDAV extension. Extremely lightweight and blazing fast.
* **ckulka/baikal** for CalDAV and CardDAV, which runs a fully managed Caldav and CardDAV server. Baikal is quite popular and has a good community to help just in case something goes wrong.

And it was done. My contacts are backed up. My files are backed up. All with just two lightweight Docker images quietly doing their thing.

## Closing Thoughts
The reason we need standards is that, when integrating the existing CalDAV,  CardDAV and WebDAV with my phone and desktop, I didn't have to run a new software to let them know how they need to parse the incoming data. Since it's a standard, it is already embedded into the system and the input is expected to come in such format.

If the standards are widely implemented across all the tools, it's not as hard to make it interoperable, and it also lets the person decide how they want to experience the internet without a lot of overhead. We not just live in the world where there are lesser standard implementing tools, but we live in the world where most of the code that we run in our machines are not open sourced. Therefore, there is no way to verify its integrity. And that has become a new normal.

Hopefully, there comes a time when we get to freely chat with people in Signal from Telegram (also other chat platforms) and vice versa and the lock-ins are removed completely. Hopefully, there comes a time when the person gets to decide the kind of experience they want to get from the internet. And hopefully there comes a time when I get to write about this in a happier tone. :)

Thanks for reading, have a nice day.