# Change to use the phusion/baseimage:0.9.16
# This allows a number of alternatives
#
FROM phusion/baseimage:0.9.16
MAINTAINER stephen@garran.com


# Use baseimage-docker's init system.
CMD ["/sbin/my_init"]

RUN rm -f /etc/service/sshd/down

# Regenerate SSH host keys. baseimage-docker does not contain any, so you
# have to do that yourself. You may also comment out this instruction; the
# init system will auto-generate one during boot.
RUN /etc/my_init.d/00_regen_ssh_host_keys.sh
RUN /usr/sbin/enable_insecure_key

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN apt-get update \
	    && apt-get install -y \
            adduser \
            ca-certificates \
            curl \
            npm \
            python-support \
        && npm install -g less less-plugin-clean-css \
        && ln -s /usr/bin/nodejs /usr/bin/node \
        && curl -o wkhtmltox.deb -SL http://downloads.sourceforge.net/project/wkhtmltopdf/0.12.2.1/wkhtmltox-0.12.2.1_linux-jessie-amd64.deb \
        && echo 'c81fffae4c0914f95fb12e047a72edda5042b1c6 wkhtmltox.deb' | sha1sum -c - \
        && dpkg --force-depends -i wkhtmltox.deb \
        && apt-get -y install -f \
        && rm -rf /var/lib/apt/lists/* wkhtmltox.deb

# Grab gosu for easy step-down from root
RUN gpg --keyserver pgp.mit.edu --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4
RUN curl -o /usr/local/bin/gosu -SL "https://github.com/tianon/gosu/releases/download/1.2/gosu-$(dpkg --print-architecture)" \
        && curl -o /usr/local/bin/gosu.asc -SL "https://github.com/tianon/gosu/releases/download/1.2/gosu-$(dpkg --print-architecture).asc" \
        && gpg --verify /usr/local/bin/gosu.asc \
        && rm /usr/local/bin/gosu.asc \
        && chmod +x /usr/local/bin/gosu

# Install Odoo
ENV ODOO_VERSION 8.0
ENV ODOO_RELEASE latest
RUN curl -o odoo.deb -SL http://nightly.odoo.com/${ODOO_VERSION}/nightly/deb/odoo_${ODOO_VERSION}.${ODOO_RELEASE}_all.deb \
        && dpkg --force-depends -i odoo.deb \
        && apt-get update \
        && apt-get -y install -f --no-install-recommends \
        && rm -rf /var/lib/apt/lists/* odoo.deb

# Retrieve Odoo public key and add Odoo nightly to repo list for further updates
RUN apt-key adv --keyserver pgp.mit.edu --recv-keys 0xdef2a2198183cbb5 \
        && echo "deb http://nightly.odoo.com/${ODOO_VERSION}/nightly/deb/ ./" >> /etc/apt/sources.list

# Clean up APT when done.
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV DB_FILTER change_to_chosen_db
ENV DB_ENV_POSTGRES_PASSWORD change_to_db_user_password
ENV DB_ENV_POSTGRES_USER pg
ENV DB_PORT_5432_TCP_ADDR ip_or_fqdn_of_db_server
ENV DB_PORT_5432_TCP_PORT 5432
ENV VOLUME_MOUNT_1 /var/lib/odoo
ENV VOLUME_MOUNT_2 /etc

RUN mkdir -p /etc/my_init.d
ADD odoo.sh /etc/my_init.d/odoo.sh
RUN chmod +x /etc/my_init.d/odoo.sh

# Run script and Odoo configuration file
ADD ./openerp-server.conf /etc/odoo/

EXPOSE 8069

# Mount /var/lib/odoo to allow restoring filestore
VOLUME ["${VOLUME_MOUNT_1}"]
VOLUME ["${VOLUME_MOUNT_2}"]

CMD ["/run.sh"]
