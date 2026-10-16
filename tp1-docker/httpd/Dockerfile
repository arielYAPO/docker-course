FROM httpd:2.4-alpine

COPY index.html /usr/local/apache2/htdocs/index.html
COPY reverse-proxy.conf /usr/local/apache2/conf/extra/reverse-proxy.conf

RUN sed -i \
    -e 's/^#LoadModule proxy_module/LoadModule proxy_module/' \
    -e 's/^#LoadModule proxy_http_module/LoadModule proxy_http_module/' \
    -e '$aInclude conf/extra/reverse-proxy.conf' \
    /usr/local/apache2/conf/httpd.conf
