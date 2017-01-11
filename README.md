# Supported tags and respective `Dockerfile` links

-   [`latest-es-es`, `latest` (*mainline/jessie/Dockerfile*)](https://github.com/joserprieto/docker-nginx-i18n/blob/master/mainline/jessie/Dockerfile)
-   [`stable-es-es`, `stable`, (*stable/jessie/Dockerfile*)](https://github.com/joserprieto/docker-nginx-i18n/blob/master/stable/jessie/Dockerfile)


For more information about the Official nginx Image, its history, please see [the relevant manifest file (`library/nginx`)](https://github.com/docker-library/official-images/blob/master/library/nginx). This official nginx image is updated via [pull requests to the `docker-library/official-images` GitHub repo](https://github.com/docker-library/official-images/pulls?q=label%3Alibrary%2Fnginx).

For detailed information about the virtual/transfer sizes and individual layers of each of the above supported tags, please see the [`repos/nginx/tag-details.md` file](https://github.com/docker-library/repo-info/blob/master/repos/nginx/tag-details.md) in [the `docker-library/repo-info` GitHub repo](https://github.com/docker-library/repo-info).


# What is Nginx?

Nginx (pronounced "engine-x") is an open source reverse proxy server for HTTP, HTTPS, SMTP, POP3, and IMAP protocols, as well as a load balancer, HTTP cache, and a web server (origin server). The nginx project started with a strong focus on high concurrency, high performance and low memory usage. It is licensed under the 2-clause BSD-like license and it runs on Linux, BSD variants, Mac OS X, Solaris, AIX, HP-UX, as well as on other *nix flavors. It also has a proof of concept port for Microsoft Windows.

> [wikipedia.org/wiki/Nginx](https://en.wikipedia.org/wiki/Nginx)

![logo](./logo.png)

# About this image

This image is based on the nginx Official Image:

> [nginx Official Image](https://hub.docker.com/_/nginx/)
> [nginx Official Image Repository](https://github.com/nginxinc/docker-nginx)

But add extra support for configure timezone and locales before built the image, with the arguments in the `docker build` command (`--build-arg`), and `ARG` in the `Dockerfile`.

Default values for the timezone and locale are:

```bash
Timezone:   "Europe/Madrid"
Locale:     es_ES.UTF-8
```



# How to use this image

First, we explain the configuration of the locale and timezone; after that, we put the same content that in the official image, because is the same kind of use.

**IMPORTANT NOTE**: this image only works with Debian Based nginx images, because the configuration of the timezone and locales, is apt / dpkg based.

## Build a localized image.

To build a localized image, this is, a Debian Image with locale and timezone correctly defined, we have to use the `--build-arg` parameter of the `docker build` command.

An example:

```bash
docker build --build-arg TIMEZONE="Europe/France" --build-arg LOCALE_LANG_COUNTRY="fr_FR" .
```

## Locales

The locales are generated with the snippet described in Debian Official Image, and [used in PostgreSQL Official Image](https://github.com/docker-library/postgres/blob/69bc540ecfffecce72d49fa7e4a46680350037f9/9.6/Dockerfile#L21-L24):

```dockerfile
RUN apt-get update && apt-get install -y locales && rm -rf /var/lib/apt/lists/* \
    && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
ENV LANG en_US.utf8
```

We only add a few arguments for the build:

```dockerfile
ARG LOCALE_LANG_COUNTRY="es_ES"
ARG LOCALE_CODIFICATION="UTF-8"
ARG LOCALE_CODIFICATION_ENV="utf8"
```

And execute the build as:


```dockerfile
    echo "=> Configuring and installing locale (${LOCALE_LANG_COUNTRY}.${LOCALE_CODIFICATION}):" && \
        DEBIAN_FRONTEND=noninteractive apt-get install -y locales && \
        rm -rf /var/lib/apt/lists/* && \
        localedef -i ${LOCALE_LANG_COUNTRY} -c -f ${LOCALE_CODIFICATION} -A /usr/share/locale/locale.alias ${LOCALE_LANG_COUNTRY}.${LOCALE_CODIFICATION}
```

## Timezone

We use a snippet provided by [Oscar](https://oscarmlage.com/) (Thanks!):

```dockerfile
    echo "=> Configuring and installing timezone:" && \
        echo "Europe/Madrid" > /etc/timezone && \
        dpkg-reconfigure -f noninteractive tzdata && \
```

Only added an ARG for the build:

```dockerfile
ARG TIMEZONE="Europe/Madrid"
```

And, finally:

```dockerfile
    echo "=> Configuring and installing timezone (${TIMEZONE}):" && \
        echo ${TIMEZONE} > /etc/timezone && \
        dpkg-reconfigure -f noninteractive tzdata && \
```

Obviously, the value of the timezone has to be one of the right values:

[Change Timezone on Debian](https://wiki.debian.org/TimeZoneChanges)
[List of tz database time zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)


After that, we put the same content that in the official image (same use..).

## hosting some simple static content

```console
$ docker run --name some-nginx -v /some/content:/usr/share/nginx/html:ro -d nginx
```

Alternatively, a simple `Dockerfile` can be used to generate a new image that includes the necessary content (which is a much cleaner solution than the bind mount above):

```dockerfile
FROM nginx
COPY static-html-directory /usr/share/nginx/html
```

Place this file in the same directory as your directory of content ("static-html-directory"), run `docker build -t some-content-nginx .`, then start your container:

```console
$ docker run --name some-nginx -d some-content-nginx
```

## exposing the port

```console
$ docker run --name some-nginx -d -p 8080:80 some-content-nginx
```

Then you can hit `http://localhost:8080` or `http://host-ip:8080` in your browser.

## complex configuration

```console
$ docker run --name some-nginx -v /some/nginx.conf:/etc/nginx/nginx.conf:ro -d nginx
```

For information on the syntax of the Nginx configuration files, see [the official documentation](http://nginx.org/en/docs/) (specifically the [Beginner's Guide](http://nginx.org/en/docs/beginners_guide.html#conf_structure)).

Be sure to include `daemon off;` in your custom configuration to ensure that Nginx stays in the foreground so that Docker can track the process properly (otherwise your container will stop immediately after starting)!

If you wish to adapt the default configuration, use something like the following to copy it from a running Nginx container:

```console
$ docker cp some-nginx:/etc/nginx/nginx.conf /some/nginx.conf
```

As above, this can also be accomplished more cleanly using a simple `Dockerfile`:

```dockerfile
FROM nginx
COPY nginx.conf /etc/nginx/nginx.conf
```

Then, build with `docker build -t some-custom-nginx .` and run:

```console
$ docker run --name some-nginx -d some-custom-nginx
```

### using environment variables in nginx configuration

Out-of-the-box, Nginx doesn't support using environment variables inside most configuration blocks. But `envsubst` may be used as a workaround if you need to generate your nginx configuration dynamically before nginx starts.

Here is an example using docker-compose.yml:

```web:
  image: nginx
  volumes:
   - ./mysite.template:/etc/nginx/conf.d/mysite.template
  ports:
   - "8080:80"
  environment:
   - NGINX_HOST=foobar.com
   - NGINX_PORT=80
  command: /bin/bash -c "envsubst < /etc/nginx/conf.d/mysite.template > /etc/nginx/conf.d/default.conf && nginx -g 'daemon off;'"
```

The `mysite.template` file may then contain variable references like this:

`listen       ${NGINX_PORT};
`

# User Feedback

## Issues

If you have any problems with or questions about this image, please contact me through a [GitHub issue](https://github.com/joserprieto/docker-nginx-i18n/issues).

## Contributing

You are invited to contribute new features, fixes, or updates, large or small; we are always thrilled to receive pull requests, and do our best to process them as fast as i can.

Before you start to code, we recommend discussing your plans through a [GitHub issue](https://github.com/joserprieto/docker-nginx-i18n/issues), especially for more ambitious contributions. This gives other contributors a chance to point you in the right direction, give you feedback on your design, and help you find out if someone else is working on the same thing.

## Documentation

To define how and where will be stored....
