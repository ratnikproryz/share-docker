
FROM httpd:2.4

RUN mkdir "/usr/local/apache2/conf/All"

COPY ./config/All /usr/local/apache2/conf/All/

COPY ./config/httpd.conf /tmp/httpd.conf

RUN cat /tmp/httpd.conf >> /usr/local/apache2/conf/httpd.conf && rm /tmp/httpd.conf

RUN mkdir -p /var/log/httpd/tool && \
    mkdir -p /var/log/httpd/client && \
    mkdir -p /var/log/httpd/customer && \
    mkdir -p /var/log/httpd/api && \
    mkdir -p /var/log/httpd/secure-api

CMD ["httpd-foreground"]
