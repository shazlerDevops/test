FROM ubuntu:22.04

LABEL maintainer="Webkul"
ENV LANG=C.UTF-8
ENV TZ=Asia/Kolkata

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime  \
	&& echo $TZ > /etc/timezone
# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN set -x; \
    apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    dirmngr \
    node-less \
    python3-pip  \
    python3-setuptools \
    gnupg \
    wget \
    npm \
    build-essential \
    && curl -o wkhtmltox.deb -sSL https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6-1/wkhtmltox_0.12.6-1.focal_amd64.deb \
    && wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2_amd64.deb \
    && dpkg -i libssl1.1_1.1.1f-1ubuntu2_amd64.deb \
    && apt install ./wkhtmltox.deb -y \
    && rm -f wkhtmltox.deb libssl1.1_1.1.1f-1ubuntu2_amd64.deb \
    && pip3 install psycogreen==1.0 \
    && apt-get install -y --no-install-recommends postgresql
    
# Install required npm packages
RUN set -x; \
    npm install -g less less-plugin-clean-css \
    && npm install -g rtlcss

# Install Odoo
ENV ODOO_VERSION 17.0

# This binds to service file. So, take care
ARG ODOO_USER=odoo
ARG ODOO_USER_UID=XXXX
ARG ODOO_USER_GID=XXXX

RUN set -x; \
        groupadd -r -g ${ODOO_USER_GID} ${ODOO_USER} \
        && adduser --system --home=/opt/${ODOO_USER} ${ODOO_USER} --uid ${ODOO_USER_UID} --gid ${ODOO_USER_GID} \
        && apt update && apt-get install -y git libpq-dev libxml2-dev libxslt-dev libffi-dev gcc python3-dev libsasl2-dev libldap2-dev libssl-dev libjpeg-dev \
        && su - ${ODOO_USER} -s /bin/bash -c "git clone https://www.github.com/odoo/odoo --depth 1 --branch ${ODOO_VERSION} --single-branch ." \
        && mkdir /var/log/odoo \
        && chown ${ODOO_USER}:root /var/log/odoo

# Copy Odoo configuration file
COPY ./odoo-server.conf /etc/odoo/

RUN set -x; chown ${ODOO_USER} /etc/odoo/  \
	&& chmod 640 /etc/odoo/odoo-server.conf  \
	&& pip3 install -r /opt/odoo/requirements.txt
		
	 
# Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users' addons
RUN mkdir -p /mnt/extra-addons \
    && mkdir -p /opt/data_dir \
    && chown -R odoo /opt/data_dir \
    && chown -R odoo /mnt/extra-addons

# Copy entrypoint script and Odoo run script
COPY ./entrypoint.sh /
COPY ./run_odoo.sh /

# Expose Odoo services
EXPOSE 8069 8071

# Set the default user when running the container
USER ${ODOO_USER}

# Define the entry point for the container
ENTRYPOINT ["/entrypoint.sh"]
