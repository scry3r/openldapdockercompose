# Use osixia/light-baseimage
# sources: https://github.com/osixia/docker-light-baseimage
FROM osixia/light-baseimage:latest
MAINTAINER noname

# Add openldap user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
RUN groupadd -r openldap && useradd -r -g openldap openldap

# Install OpenLDAP, ldap-utils and cfssl from baseimage and clean apt-get files
# sources: https://github.com/osixia/docker-light-baseimage/blob/stable/image/tool/add-service-available
#          https://github.com/osixia/docker-light-baseimage/blob/stable/image/service-available/:cfssl/download.sh
RUN gpg --keyserver keys.gnupg.net --recv-key 62B4981F \
    && gpg --export -a "Fusiondirectory Archive Manager <contact@fusiondirectory.org>" > FD-archive-key \
    && apt-key add FD-archive-key && rm FD-archive-key \
    && echo "deb http://repos.fusiondirectory.org/debian-jessie jessie main" >> /etc/apt/sources.list \
    && apt-get -y update \
    && /container/tool/add-service-available :cfssl \
    && LC_ALL=C DEBIAN_FRONTEND=noninteractive apt-get install -y --force-yes --no-install-recommends \
       ldap-utils \
       openssl \
       slapd \
       fusiondirectory-schema \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*


RUN schema2ldif /etc/ldap/schema/fusiondirectory/rfc2307bis.schema > /etc/ldap/schema/fusiondirectory/rfc2307bis.ldif && \
    sed -i "s|include: file:///etc/ldap/schema/nis.ldif|include: file:///etc/ldap/schema/fusiondirectory/rfc2307bis.ldif|g" /usr/share/slapd/slapd.init.ldif

# Add service directory to /container/service
ADD service /container/service

# Use baseimage install-service script
# https://github.com/osixia/docker-light-baseimage/blob/stable/image/tool/install-service
RUN /container/tool/install-service

# Add default env variables
ADD environment /container/environment/99-default



# Set OpenLDAP data and config directories in a data volume
VOLUME ["/var/lib/ldap", "/etc/ldap/slapd.d"]
RUN mkdir -p /var/lib/ldap && chown openldap:openldap /var/lib/ldap -R

# Expose default ldap and ldaps ports
EXPOSE 389 636
