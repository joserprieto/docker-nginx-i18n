FROM nginx:latest

MAINTAINER joserprieto "talktome@joserprieto.es"

# Arguments for the build of the image; right of "=" are the default values.

ARG TIMEZONE="Europe/Madrid"
ARG LOCALE_LANG_COUNTRY="es_ES"
ARG LOCALE_CODIFICATION="UTF-8"
ARG LOCALE_CODIFICATION_ENV="utf8"

# Set enviroment variable for allow terminal output:

ENV TERM="xterm"

# We will localize to es-ES (default; other langs/timezone, use --build-arg on docker build) our Debian image:
#   Notes:
#       Installing the 'apt-utils' package gets rid of the
#           'debconf: delaying package configuration, since apt-utils is not installed'
#        error message when installing any other package with the apt-get package manager.
#       The
#           set -x
#        avoid the errors:
#           debconf: unable to initialize frontend: Dialog
#           ...
#           debconf: unable to initialize frontend: Readline
#           ...
#       But, the noninteractive frontend has no requirements but it does not ask any questions
#        see: http://www.gnu.org/software/bash/manual/bashref.html#The-Set-Builtin
#       (which may or may not be what you want).
#       Suggested packages: packages that will be very useful; feel free to ommit the

RUN set -x \
	&& apt-get update \
	&& apt-get upgrade -y \
	&& echo "=> Needed packages:" \
    && apt-get install -y --no-install-recommends apt-utilsca-certificates \
    && echo "=> Suggested packages:" \
    && apt-get install -y --no-install-recommends tree nano tar \
    && echo "=> Configuring and installing timezone (${TIMEZONE}):" \
    && echo ${TIMEZONE} > /etc/timezone \
    && dpkg-reconfigure -f noninteractive tzdata \
    && echo "=> Configuring and installing locale (${LOCALE_LANG_COUNTRY}.${LOCALE_CODIFICATION}):" \
    && apt-get install -y locales \
    && rm -rf /var/lib/apt/lists/* \
    && localedef -i ${LOCALE_LANG_COUNTRY} -c -f ${LOCALE_CODIFICATION} -A /usr/share/locale/locale.alias ${LOCALE_LANG_COUNTRY}.${LOCALE_CODIFICATION}

# Set enviroment variables:

ENV LANG=${LOCALE_LANG_COUNTRY}.${LOCALE_CODIFICATION_ENV}