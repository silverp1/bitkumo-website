---
layout: post
title: "Blog More in 2016 with Ghost"
category: blog
tags:
  - blogging
  - ghost
permalink: /blog/:title/
author: levlaz
---

Happy New Year from the entire Bitkumo team! It seems that a lot of people set a resolution to blog more each year, this year you can make it happen easily by setting up [Ghost](https://ghost.org) on Bitkumo. Earlier this week we wrote about how we [publish our site and blog using Jekyll](https://bitkumo.com/blog/continuously-deploying-with-jekyll-and-circleci/). Jekyll is awesome, but if you want a more traditional CMS we would highly recommend ghost. Ghost is a no frills blogging platform that allows you to create beautiful blogs using Markdown. This guide will help you get off on the right foot in 2016. 

We will be setting up Ghost on a 512MB Bitkumo Server running Ubuntu 14.04 LTS. 

## Preparing your Server to run Ghost  

Ghost is powered by Node.JS so before we can get started with Ghost we need to install Node. Log into your Ubuntu server via terminal and issue the following commands to install npm, node.js, the unzip utility, and the nginx web server. 

    apt-get install nginx tmux npm nodejs nodejs-legacy unzip 

## Installing Ghost 

Now that we have Node installed we can start the ghost installation. First we will create a new directory for Ghost and then enter this new directory. 

    mkdir -p /var/www/ghost 
    cd /var/www/ghost 

Next we will download and unzip the latest version of Ghost. 
    
    curl -L https://ghost.org/zip/ghost-latest.zip -o ghost.zip
    unzip ghost.zip 

Lastly, we we will install Ghost. 

    npm install --production

## Start Ghost 

Now that ghost is installed we can start it up. Open up a new tmux session so that Ghost keeps running even if our connection dies and then start up ghost. (If your server does die, simply SSH back in and type in `tmux attach` to get back to the session)

    tmux 
    npm start --production

This will start up Ghost on 127.0.0.1:2368, we can not set up nginx to proxy requests to this address so that you can view ghost from the outside world. 

## Configuring Nginx 

First we will create a new nginx configuration for our new site. 

    vim /etc/nginx/sites-available/ghost_blog 

The contents of this file should look something like this: 

```bash
server {
    listen 80;
    server_name $MY_GHOST_BLOG.com www.$MY_GHOST_BLOG.com;

    location / {
        proxy_pass http://127.0.0.1:2368;
        proxy_set_header    Host            $host;
        proxy_set_header    X-Real-IP       $remote_addr;
        proxy_set_header    X-Forwarded-for $remote_addr;
        port_in_redirect off;
        proxy_redirect   https://127.0.0.1:2368  /;
        proxy_connect_timeout 300;
    }
}
```

Save this file after changing $MY_GHOST_BLOG.com to your own site name, then we will create a new symlink to make this configuration active and restart nginx. 

    rm /etc/nginx/sites-enabled/default
    ln -s /etc/nginx/sites-available/ghost_blog /etc/nginx/sites-enabled/ghost_blog 
    service nginx restart 

Now you should be able to go to `http://$YOUR_IP` and see your shiny new Ghost blog. Go to `http://$YOUR_IP/ghost` to set up a new admin account and get started! 

<img src="/images/blog/ghost.png" alt="Ghost Screenshot">

For more information on what to do next check out [how to use ghost](http://support.ghost.org/how-to-use-ghost/). If you have any questions please let us know in the comments below. We are looking forward to reading your blogs in 2016! 

*Looking for 0% nonsense cloud hosting? [Get started in the Bitkumo Cloud Today!](https://app.bitkumo.com/auth/register)*
