# 基础镜像mochat/mochat的Dockerfile：https://github.com/mochat-cloud/mochat-docker/blob/master/Dockerfile
FROM mochat/mochat:v1

#RUN apk --no-cache add netcat-openbsd
ADD https://packages.whatwedo.ch/php-alpine.rsa.pub /etc/apk/keys/php-alpine.rsa.pub
RUN set -eux && \
    sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories && \
    sed -i 's/dl.bintray.com/packages.whatwedo.ch/g' /etc/apk/repositories && \
    apk --no-cache add netcat-openbsd

COPY mc_init.phar /tmp/mc_init.phar
COPY wait-for /tmp/wait-for

RUN chmod o+x /tmp/wait-for

CMD sh -c "/tmp/wait-for mysql:3306 -- php /tmp/mc_init.phar"
