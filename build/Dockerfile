#
#                    ##        .            
#              ## ## ##       ==            
#           ## ## ## ##      ===            
#       /""""""""""""""""\___/ ===        
#  ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~   
#       \______ o          __/            
#         \    \        __/             
#          \____\______/                
# 
#          |          |
#       __ |  __   __ | _  __   _
#      /  \| /  \ /   |/  / _\ | 
#      \__/| \__/ \__ |\_ \__  |
#
FROM ubuntu:16.04


# Let the container know that there is no tty
ENV DEBIAN_FRONTEND noninteractive


# --- prepare apt
RUN apt-get update -q --fix-missing && apt-get -y upgrade

# --- docker only stuff
RUN apt-get -y install --no-install-recommends \
      supervisor locales tzdata

# --- set timezone
ARG BUILD_TZ="Europe/Berlin"
RUN echo "$BUILD_TZ" > /etc/timezone && \
    dpkg-reconfigure -f noninteractive tzdata


# Ensure that we always use UTF-8 and with US English locale
RUN locale-gen en_US.UTF-8

# configure locales
RUN locale-gen en_US.UTF-8  
ENV LANG en_US.UTF-8  
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

# --- Install needed packages
RUN apt-get -y install --no-install-recommends \
       rsync git vim wget cron reprepro nginx ca-certificates net-tools \
       libnet-ssleay-perl libio-socket-ssl-perl sendemail build-essential \
    && rm -rf /var/lib/apt/lists/*

############################################################################################
#
# nginx
#
RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf
RUN mkdir -p                   /var/lib/nginx /run/nginx
RUN chown -R www-data:www-data /var/lib/nginx /run/nginx
#RUN cp -va /var/lib/nginx/html/* /var/www/localhost/htdocs/

# Define mountable directories.
VOLUME ["/etc/nginx/sites-enabled", "/etc/nginx/certs", "/etc/nginx/conf.d", "/var/log/nginx", "/var/www/html"]


EXPOSE 80 443
############################################################################################
#
# docker-extensions
#
RUN mkdir -p  /usr/local/bin
COPY ./bin/*  /usr/local/bin/
RUN chmod a+x /usr/local/bin/*

#
# establisch supervisord
#
ADD /supervisor /etc/supervisor
# link old /etc/init.d/ startup scripts to supervisor
RUN ls -m1    /etc/supervisor/services.d | while read i; do mv /etc/init.d/$i /etc/init.d/$i-orig ;  ln -sf /etc/supervisor/super-init.sh /etc/init.d/$i ; done
RUN ln -sf    /etc/supervisor/systemctl /bin/systemctl
RUN chmod a+x /etc/supervisor/* /etc/supervisor/*.d/*
COPY /supervisor/invoke-rc.d /usr/sbin/invoke-rc.d
COPY /supervisor/reload      /sbin/reload
RUN  chmod a+x /sbin/reload /usr/sbin/invoke-rc.d 
#
# create directory for service volume
#
RUN mkdir -p /service ; chmod a+rwx /service

ENV TERM xterm
#
# startup script
#
ADD ./start.sh /start.sh
RUN chmod 755 /start.sh
CMD ["/start.sh"]
