---
author: "Velan Salis"
title: "How I self-hosted Pixelfed on my tiny server with Docker"
date: 2022-08-07T05:10:48+05:30
description: ""
categories: []
tags: []
slug: "how-i-hosted-pixelfed-on-my-selfhosted-setup-with-docker"
draft: false
---

Pixelfed! An ethical alternative to Instagram. Ethical because this decentralized network is run by enthusiastic users as moderators instead of the "Grow at all costs" corporation, which will be compelled to use your data against your own goodwill if that churns enough profits for them.

The best thing about Pixelfed is that, it's [open source](https://github.com/pixelfed/), and it's [self-hostable](https://github.com/pixelfed/pixelfed/blob/dev/docker-compose.yml). If you have some spare space on your servers, you could spin up a Pixelfed server and interact with the rest of the [Fediverse](https://www.youtube.com/watch?v=S57uhCQBEk0). Isn't that cool? I recently spun up a [Pixelfed instance on my server,](https://pixelfed.lhin.space) and this is an explanatory post about the steps required to host an instance for yourself on your server. Let's dive in, shall we?

## Before we could dive in...

> **Note:** The instructions mentioned in this post concerns with Pixelfed version [0.11.3](https://github.com/pixelfed/pixelfed/releases/tag/v0.11.3) with additional commits upto [352500f](https://github.com/pixelfed/pixelfed/commit/352500fdcee0188cc3ab239939182b43cd194ab9) . I can't guarentee the instructions will ever work for the uncoming releases. Feel free to get in touch with me at [Mastodon](https://mastodon.lhin.space/@0xvms) or [Email](mailto:hi@0xvms.com) if necessary.

## Environment Setup

Assuming that you have installed [docker](https://docs.docker.com/engine/install/) and [docker-compose](https://docs.docker.com/compose/install/) on your server, the next thing to note is to set up an [.env](https://github.com/pixelfed/pixelfed/blob/dev/.env.docker) file for docker-compose of Pixelfed. My .env file for my self-hosted instance is as follows

```
    APP_NAME="Pixelfed"
    APP_ENV=production
    APP_KEY=
    APP_DEBUG=false
    APP_URL=https://pixelfed.lhin.space
    
    APP_DOMAIN="pixelfed.lhin.space"
    ADMIN_DOMAIN="pixelfed.lhin.space"
    SESSION_DOMAIN="pixelfed.lhin.space"
    TRUST_PROXIES="*"
    
    LOG_CHANNEL=stack
    
    DB_CONNECTION=mysql
    DB_HOST=mariadb
    DB_PORT=3306
    DB_DATABASE=somedatabase # Change this
    DB_USERNAME=someusername # Change this
    DB_PASSWORD=somepassword # Change this
    
    PUID=1000
    PGID=1000
    MYSQL_DATABASE=somedatabase 		# Change this
    MYSQL_USER=someusername 			# Change this
    MYSQL_PASSWORD=somepassword			# Change this
    MYSQL_ROOT_PASSWORD=somepassword	# Change this
    
    BROADCAST_DRIVER=redis
    CACHE_DRIVER=redis
    SESSION_DRIVER=redis
    QUEUE_DRIVER=redis
    
    REDIS_SCHEME=tcp
    REDIS_HOST=redis
    REDIS_PASSWORD=null
    REDIS_PORT=6379
    
    # For this setup, you could add any email config as your email provider.
    MAIL_DRIVER=smtp
    MAIL_HOST=smtp.mail.host
    MAIL_PORT=587
    MAIL_USERNAME=mailbox@server.name
    MAIL_PASSWORD=somepassword
    MAIL_ENCRYPTION=tls
    MAIL_FROM_ADDRESS=mailbox@server.name
    MAIL_FROM_NAME=Pixelfed
    
    OPEN_REGISTRATION=false
    ENFORCE_EMAIL_VERIFICATION=true
    PF_MAX_USERS=1000
    
    MAX_PHOTO_SIZE=15000
    MAX_CAPTION_LENGTH=150
    MAX_ALBUM_LENGTH=4
    
    ACTIVITY_PUB=true
    AP_REMOTE_FOLLOW=true
    AP_SHAREDINBOX=true
    AP_INBOX=true
    AP_OUTBOX=true
    ATOM_FEEDS=true
    NODEINFO=true
    WEBFINGER=true
    
    HORIZON_DARKMODE=true
    HORIZON_EMBED=true
    INSTANCE_CONTACT_EMAIL=admin@server.name
    OAUTH_ENABLED=true
    ENABLE_CONFIG_CACHE=false
```

.env.pixelfed
- Create a file `.env.pixelfed` in your server directory.
- Copy the contents from the above to the `.env.pixelfed` file. 

Make sure to replace the values mentioned under *change this *with the respective values that the database and mail server complies with. Once this .env file is all set and done, we move on to docker-compose setup.

## Docker Setup

The following is my docker-compose.yml file. I'm using the [zknt/pixelfed](https://hub.docker.com/r/zknt/pixelfed) image from the Dockerhub, since it's the most downloaded and actively maintained image of Pixelfed, yet. 
```
    version: '3'
    
    networks:
      internal:
        internal: true
      web:
        driver: bridge
        external: true
    
    volumes:
      pixelfed_redis:
        external: true
      pixelfed_mariadb:
        external: true
      pixelfed_storage:
        external: true
      pixelfed_bootstrap:
        external: true
    
    services:
      app:
        container_name: pixelfed-app
        image: zknt/pixelfed:2022-08-06
        restart: unless-stopped
        env_file:
          - .env.pixelfed
        volumes:
          - pixelfed_storage:/var/www/storage
          - pixelfed_bootstrap:/var/www/bootstrap
          - ./.env.pixelfed:/var/www/.env
        networks:
          - web
          - internal
        depends_on:
          - mariadb
          - redis
        labels:
          - traefik.enable=true
          - traefik.http.routers.pixelfed.rule=Host(`pixelfed.lhin.space`)
          - traefik.http.routers.pixelfed.tls=true
          - traefik.http.routers.pixelfed.service=pixelfed
          - traefik.http.routers.pixelfed.tls.certresolver=lets-encrypt
          - traefik.http.services.pixelfed.loadbalancer.server.port=80
    
      worker:
        container_name: pixelfed-worker
        image: zknt/pixelfed:2022-08-04
        restart: unless-stopped
        env_file:
          - .env.pixelfed
        volumes:
          - pixelfed_storage:/var/www/storage
          - pixelfed_bootstrap:/var/www/bootstrap
          - ./.env.pixelfed:/var/www/.env
        networks:
          - web
          - internal
        depends_on:
          - mariadb
          - redis
        entrypoint: /worker-entrypoint.sh
        healthcheck:
          test: php artisan horizon:status | grep running
          interval: 60s
          timeout: 5s
          retries: 1
    
      mariadb:
        image: ghcr.io/linuxserver/mariadb:alpine-version-10.5.9-r0
        container_name: pixelfed-mariadb
        restart: always
        networks:
          - internal
        env_file:
          - .env.pixelfed
        volumes:
          - pixelfed_mariadb:/config:rw
    
      redis:
        image: redis:5-alpine
        container_name: pixelfed-redis
        restart: unless-stopped
        env_file:
          - .env.pixelfed
        volumes:
          - pixelfed_redis:/data
        networks:
          - internal
```
docker-compose.yml
Copy the following contents to a `docker-compose.yml` file and keep it in the same directory as the `.env.pixelfed` file. Change the contents as it pleases you. This is all you need to set up a Pixelfed instance on your server.

> **Note:** Since I'm using traefik to manage my docker images, I tend to have a `web` network that binds with the traefik docker image. You could use other reverse proxies by removing the `labels` in the `app` service and by adding `ports: - 80:8080`. In this you are redirecting the traffic coming to port 8080 to the docker port 80. [More info here](https://docs.docker.com/compose/networking/)

## Pre-requisites

In order to run Pixelfed, we need to create external docker volumes first. You could also use [bind mounts](https://docs.docker.com/storage/bind-mounts/) to local directory. But I tend to use volumes so that I don't have to fiddle around with permission issues. The following commands will create four volumes.

    docker volume create pixelfed_bootstrap
    docker volume create pixelfed_storage
    docker volume create pixelfed_mariadb
    docker volume create pixelfed_redis

create-volume.sh
## Voilà!

Now for the climax, you need to run `docker-compose up -d` (?)

Well, not so fast. You need to do so in steps so that the MariaDB and the Redis is set up properly. 

- First, you need to run `docker-compose up -d mariadb redis`. This will set up the MariaDB and Redis instance. You could also do `docker-compose logs -f` to check the logs and see what's cooking in there.
- Once that's done, you should run `docker-compose up -d app`. This will  spin up the app instance and run all the database migrations that are required. 
- Next, You need to run `docker-compose up -d worker`. This will spin up the `worker` instance which will be responsible for Remote fetching account avatars and other ActivityPub stuff.

Once this is done, you could check the logs using `docker-compose logs -f` to see if there are any errors in the installation. If not, we move on to the next step.

## Server Setup

Firstly, you need to create an app key. Run `docker-compose exec app php artisan key:generate` to generate an app key. This key will be added to your `.env.pixelfed` file in `base64` format. 

Now that the instance is set up properly, we need to create an admin user. You could do so by running `docker-compose exec app php artisan user:create`. Then answer the prompts given on the screen, and you are done. 

## Troubleshooting

- **Empty user profiles:** For a brief moment, I had an issue of getting an empty user profile when remote account is being fetched. It was fixed by running `docker-compose exec app php artisan passport:install` on my server as mentioned in [this issue](https://github.com/pixelfed/pixelfed/issues/3549).

## Closing Thoughts

I'm very optimistic about a future where decentralized social media like, Pixelfed, Mastodon and the entirety of Fediverse is a thing. I hope to see this pick up steam among the netizens, and also hope to see the service itself becomes accessible to anyone and everyone. For that to happen, this is my small contribution to anyone who wants to spin up a server for themselves. Feel free to [contact me](mailto:hi@0xvms.com) for any assistance. 

Have a nice day :)
