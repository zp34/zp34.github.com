---
layout: post
title: "Installing nginx in Mac OS X Mountain Lion with homebrew"
date: 2012-10-10 03:45
comments: true
categories: [nginx, mac os x, homebrew]
---


## Install with brew

Use `brew` to install the `nginx` with command:

{% codeblock lang:bash %}
$ brew install nginx
{% endcodeblock %}

After install run:

{% codeblock lang:bash %}
$ sudo nginx
{% endcodeblock %}

## Testing

Test it by going to URL:

    http://localhost:8080

## Configuration

The default place of `nginx.conf` on Mac after installing with `brew` is:

    /usr/local/etc/nginx/nginx.conf

### Changing the default port

The nginx default port is 8080, we shall change it to 80. First stop the nginx server if it is running by:

{% codeblock lang:bash %}
$ sudo nginx -s stop
{% endcodeblock %}

Then open `nginx.conf` with:

{% codeblock lang:bash %}
$ vim /usr/local/etc/nginx/nginx.conf
{% endcodeblock %}

and change the:

{% codeblock /usr/local/etc/nginx/nginx.conf lang:nginx %}
server {
    listen       8080;
    server_name  localhost;

    #access_log  logs/host.access.log  main;

    location / {
        root   html;
        index  index.html index.htm;
    }
{% endcodeblock %}

to:

{% codeblock /usr/local/etc/nginx/nginx.conf lang:nginx %}
server {
    listen       80;
    server_name  localhost;

    #access_log  logs/host.access.log  main;

    location / {
        root   html;
        index  index.html index.htm;
    }
{% endcodeblock %}

Save configuration and start nginx by

{% codeblock lang:bash %}
$ sudo nginx
{% endcodeblock %}

## Changing the path of defualt web location

The nginx html folder (brew install only) is by the defult in:

    /usr/local/Cellar/nginx/1.2.3/html

**Note**: change `1.2.3` to your nginx version.

The defualt path configuration:

{% codeblock /usr/local/etc/nginx/nginx.conf lang:nginx %}
server {
    listen       80;
    server_name  localhost;

    #access_log  logs/host.access.log  main;

    location / {
        root   html;
        index  index.html index.htm;
    }
{% endcodeblock %}

To let say `Users/xajler/www`:

{% codeblock /usr/local/etc/nginx/nginx.conf lang:nginx %}
server {
    listen       80;
    server_name  localhost;

    #access_log  logs/host.access.log  main;

    location / {
        root   /Users/xajler/www;
        index  index.html index.htm;
    }
{% endcodeblock %}

After change stop and start nginix server and nginx is now serving pages from your custom folder!
