---
layout: post
title: OwnCloud in a container
summary: ... or how to dump Dropbox
categories: docker owncloud
tags: owncloud docker volume nginx fpm
---

> *UPDATE:*
> As [dinkel][100] pointed out in a [GitHub issue][101] `PHP` only allowed file uploads of maximum 2MB by the web interface. To work around this restriction `PHP.ini` had to be updated.

Quite some time, and especially since the NSA disclosures, I wanted to run my own file syncing service. So when I lately toyed around with [DigitalOcean][1], [coreos][2] and [docker][3] the idea got prominent again and I set out to use all these hipster tools to install and run my own [OwnCloud][4].

This is no introduction to the tools but rather a guide on how to install OwnCloud in the given environment. So some knowledge is expected.

## General considerations

OwnCloud is an online file hosting, syncing and whatever service. There are some commercial vendors but as far as I know it is meant to be run privately.

Uploaded files will be stored in the local file system whereas user information and other metadata will be stored in a database.

OwnCloud can use [SQLite][5] but it is recommended to use a *real* database in a multi user setup. There is already a nice [PostgreSQL][6] docker container available, so the decision to use this particular database is quite easy. Lets call the `postgres` container in the following `owncloud-postgres`.

To serve the OwnCloud files a webserver with `PHP` support is required. In our case `nginx` will do the work. The official `nginx` docker container doesn't provide `PHP` support and it is not so easy to configure it afterwords. I spent quite some time trying to get it to work but in the end I used my own Ubuntu container, `owncloud-nginx`.

In a docker environment files will normally be stored in separate volume containers to persist the file system changes. In our setup we need two of these, one to store the uploaded files and one to store the database. Lets call them `owncloud-data` and `owncloud-postgres-data`.

All containers together will form the following structure.
![docker-owncloud containers](/images/docker-owncloud.png)

In the next sections we will discuss how to setup these individual containers and how to put everything together.

## owncloud-data

The `owncloud-data` container holds the uploaded files. OwnCloud saves files in the `data` sub directory. Furthermore configuration changes are stored in the `config` sub directory.

To persist all changes the `owncloud-data` container needs two volume declarations, `/var/www/owncloud/data` and `/var/www/owncloud/config`. Because OwnCloud is very picky regarding the directory permissions a separate Dockerfile is needed in which all permissions are set to `0770` and the user to the later used `www-data`.

{% highlight nasm %}
# OwnCloud - Data container

FROM busybox

MAINTAINER Norbert Schneider  <mail@herr-norbert.de>

RUN mkdir -p /var/www/owncloud/data
RUN chmod -R 0777 /var/www/owncloud/data
RUN chown -R www-data:www-data /var/www/owncloud/data
VOLUME ["/var/www/owncloud/data"]

RUN mkdir -p /var/www/owncloud/config
RUN chmod -R 0777 /var/www/owncloud/config
RUN chown -R www-data:www-data /var/www/owncloud/config
VOLUME ["/var/www/owncloud/config"]

CMD echo "Data container only (/data, /config)"
{% endhighlight %}

To create the container copy the statements above in a file called `Dockerfile` and run the following command in the same directory.

{% highlight nasm %}
docker -t owncloud-data .
{% endhighlight %}

This creates a new image with the name `owncloud-data` from which you can start a container with this command:

{% highlight nasm %}
docker run --name owncloud-data owncloud-data
{% endhighlight %}

Docker just prints the `echo` statement and exits immediately but to do so it also creates the volume container `owncloud-data`.

## owncloud-nginx

To serve the OwnCloud application a webserver is needed. Like I said earlier I had some problems with the official `nginx` container so I created my own.

{% highlight nasm %}
# OwnCloud -  nginx

FROM ubuntu

MAINTAINER Norbert Schneider  <mail@herr-norbert.de>

# Install stuff - supervisor, nginx and php dependencies

RUN apt-get update
RUN apt-get install -y supervisor nginx php5 php5-gd php-xml-parser php5-intl php5-json php5-mcrypt php5-fpm php-apc php5-imap php5-mcrypt php5-curl php5-imagick php5-pgsql php5-mysql php5-sqlite smbclient curl libcurl3 bzip2
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Install OwnCloud

WORKDIR /var/www
RUN curl https://download.owncloud.org/community/owncloud-7.0.2.tar.bz2 | tar jx -C /var/www/
RUN chown -R www-data:www-data /var/www

# Configure nginx

ADD ssl.crt /etc/ssl/nginx/ssl.crt
ADD ssl.key /etc/ssl/nginx/ssl.key
ADD nginx.conf /etc/nginx/nginx.conf

# Configure php

RUN sed -i -e "s/^upload_max_filesize\s*=\s*2M/upload_max_filesize = 16G/" /etc/php5/fpm/php.ini
RUN sed -i -e "s/^post_max_size\s*=\s*8M/post_max_size = 16G/" /etc/php5/fpm/php.ini
RUN sed -i -e "s/^output_buffering\s*=\s*4096/output_buffering = 0/" /etc/php5/fpm/php.ini

# Configure php-fpm

ADD www.conf /etc/php5/fpm/pool.d/www.conf

# Configure supervisor

COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

EXPOSE 443

# Start everything

CMD supervisord
{% endhighlight %}

Let us go through the instructions. First `supervisor`, `nginx` and some `PHP` dependencies are installed.

After that `curl` downloads the OwnCloud tar and extracts the content into `/var/www`.

Then a self-signed `SSL` key is added to the image. Just take a look at [one][7] of the many tutorials if you need to generate one.

In the next step `nginx` and `PHP` configuration files are added. The files are basically the provided ones from the [OwnCloud documentation][8] and the whole content can be found on [GitHub][9]. They ensure that `nginx` uses `SSL` and can talk to the `PHP` service.

In the last part `supervisor` is configured and set as command for docker to run on startup of the container. Docker runs _only_ one command, so if one only starts `nginx` the `PHP` service will not be available (this obvious constraint took me some time to figure out). To overcome this *limitation* `supervisor` is used to start both `nginx` and the `PHP` service.

To build an image from this `Dockerfile` use the following command:

{% highlight nasm %}
docker build -t owncloud-nginx .
{% endhighlight %}

This image contains everything needed to run OwnCloud but as mentioned earlier the use of a full-blown database is recommended. So read on ...

## owncloud-postgres-data

The official [PostgreSQL container][10] uses the volume `/var/lib/postgresql/data` to store the database files. To create a separate volume container simply run the following command:

{% highlight nasm  %}
docker run --name owncloud-postgres-data -v /var/lib/postgresql/data busybox echo PostgreSQL data-only container
{% endhighlight %}

## owncloud-postgres

The `PostgreSQL` docker container is very easy to use and exposes an already installed default database schema on port `5432`. To start the database run the following command:

{% highlight nasm %}
docker run --name owncloud-postgres --volumes-from owncloud-postgres-data postgres
{% endhighlight %}

This will pull the official container, run it with the local name  `owncloud-postgres` and use the volume container `owncloud-postgres-data` to store the actual database files.

## Putting everything together

Now everything is created and can be put together:

{% highlight  nasm  %}
docker run --name owncloud-nginx -p 443:443 --volumes-from owncloud-data --link owncloud-postgres:postgres owncloud-nginx
{% endhighlight %}

This will run the `owncloud-nginx` image, with the same name as container name, create a link to the `owncloud-postgres` container, use the `owncloud-data` container to store files and expose `nginx` on part `443`.

Afterwards the OwnCloud configuration should be available at `http://localhost:443`. To finalize the installation add an admin user on the configuration site and configure the database access to use `nginx` on host `nginx` (name of the docker link between the container) with username `nginx` without a password.

## Next steps

After everything is up and running you should think about updates, backups and maintenance. I recommend something like [`systemd`][11] to automatically restart the containers in the case of a crash or system restart and to have a look at the [docker volume description][12] to find out about backups.

## Conclusion

It is not so hard to run your own OwnCloud especially when key infrastructure components are provided in a preconfigured container.


[1]: https://www.digitalocean.com
[2]: https://coreos.com
[3]: https://docker.com
[4]: https://owncloud.org
[5]: https://sqlite.org
[6]: http://www.postgresql.org
[7]: https://www.digitalocean.com/community/tutorials/how-to-create-a-ssl-certificate-on-apache-on-arch-linux
[8]: http://doc.owncloud.org/server/7.0/admin_manual/installation/configuration_nginx.html
[9]:https://github.com/bertschneider/docker-owncloud
[10]: https://registry.hub.docker.com/_/postgres/
[11]: https://coreos.com/using-coreos/systemd/
[12]: https://docs.docker.com/userguide/dockervolumes/#volume-def
[100]: https://github.com/dinkel
[101]: https://github.com/bertschneider/docker-owncloud/issues/1
