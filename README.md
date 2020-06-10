plemp
===

> :exclamation: **This project is BETA. Don't run it on a production server unless you know what you are doing. Please report any security concerns or other suggestions by submitting an issue.**

> :information_source: **I'm not a programmer, nor a sys/net-admin. If there are bugs, please report them and I'll do my best to fix them. This project is made just for fun and learning.**


## What is plemp ?

### Info

plemp (Perfect Linux (e)Nginx MYSQL & PHP), is what I call the 'ideal' webserver for basic webapplications. It uses Docker and is easy to deploy. The project was meant to be a teaching experience cause I wanted to know more about Docker. It has several security and pagespeed enhancements out of the box. Is it really perfect ? No, of course not, especially right now since it's in beta. I cannot possibly take all requirements of everybody into consideration, so it might not be suitable for everyone.

plemp is a network of 3 docker containers.

1. Nginx 1.19.0
2. MYSQL 8
3. PHP7.4-FPM

The docker-compose file let's you deploy a turn-key webserver. All you need to do is edit the virtual host file and define your domain name.

### Speed
#### ngx_pagespeed
The Nginx application is compiled with [ngx_pagespeed](https://github.com/apache/incubator-pagespeed-ngx). The configuration and presets are already applied. Straight away you are caching, optimizing html/css/js and images. If you are using a CMS like WordPress, this will save you from installing such plugins. It's all handled at the server level, before it even reaches WordPress. If you want to see or edit the options, view `nginx/files/conf.d/pagespeed.conf`

#### FastCGI Caching

Nginx is caching the static content like images, videos, css and js files. Nginx is know for it's incredibly fast caching mechanism.

### Security

#### ModSecurity
The Nginx application is compiled with [ModSecurity](https://github.com/SpiderLabs/ModSecurity). ModSecurity act as a WAF (WebApplication Firewall), and it will secure your webapplication from malicious visitors. This too will make security plugins redundant (e.g.: WordFence for WordPress).

#### OWASP Core Rule Set
The ModSecurity module is extended with the [Core Rule Set](https://github.com/SpiderLabs/owasp-modsecurity-crs/) created by OWASP. It will prevent popular attacks like Cross Site Scriptin, SQL Injection and Cross Site Request Forgery. Other attack vectors are Local/Remote File Inclusion, Remote Code Execution and Session Fixation. Read their GitHub page for more information.

> :exclamation:Some poorly coded themes/plugins/extensions for your CMS may trigger false positives.
Test your application before moving to production !


#### Secure headers

    server_tokens off;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Xss-Protection "1; mode=block" always;

## Installation
We're assuming your already have Docker installed on your server.

1. Clone this repository and enter it.
2. Edit the .env file for MYSQL credentials.
3. Rename and edit the virtual hosts file in `./nginx/files/sites-enabled/example.com`, and replace all instanced of 'example.com' with your own domain.
4. Rename instances of 'example.com' in the `./nginx/Dockerfile`.
5. run `docker-compose up -d`.

The server should now be listening on port http/80.

## Roadmap

- Add Let's Encrypt automation
- Add Amplify to monitor Nginx
- Add MYSQL Tuner for database optimization
- Enable http2

###### tags: `nginx` `php` `php-fpm` `mysql` `security` `pagespeed` `cms` `wordpress` `drupal` `magento` `woocommerce` `modsecurity` `lemp` `linux` `webserver` `docker` `docker-compose` `webapplication`
