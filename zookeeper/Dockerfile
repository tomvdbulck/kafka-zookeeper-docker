
FROM openjdk:8-jre-alpine
MAINTAINER tomvdbulck

#Based on the official Zookeeper Image, added some logic to handle JMX

# Install required packages
RUN apk add --no-cache \
    bash \
    su-exec

ENV ZOO_USER=zookeeper \
    ZOO_CONF_DIR=/conf \
    ZOO_DATA_DIR=/data \
    ZOO_DATA_LOG_DIR=/datalog \
    ZOO_PORT=2181 \
    ZOO_TICK_TIME=2000 \
    ZOO_INIT_LIMIT=5 \
    ZOO_SYNC_LIMIT=2

# Add a user and make dirs
RUN set -x \
    && adduser -D "$ZOO_USER" \
    && mkdir -p "$ZOO_DATA_LOG_DIR" "$ZOO_DATA_DIR" "$ZOO_CONF_DIR" \
    && chown "$ZOO_USER:$ZOO_USER" "$ZOO_DATA_LOG_DIR" "$ZOO_DATA_DIR" "$ZOO_CONF_DIR"

ARG GPG_KEY=C823E3E5B12AF29C67F81976F5CECB3CB5E9BD2D
ARG DISTRO_NAME=zookeeper-3.4.10

# Download Apache Zookeeper, verify its PGP signature, untar and clean up
RUN set -x \
    && apk add --no-cache --virtual .build-deps \
        gnupg \
    && wget -q "http://www.apache.org/dist/zookeeper/$DISTRO_NAME/$DISTRO_NAME.tar.gz" \
    && wget -q "http://www.apache.org/dist/zookeeper/$DISTRO_NAME/$DISTRO_NAME.tar.gz.asc" \
    && export GNUPGHOME="$(mktemp -d)" \
    && gpg --keyserver ha.pool.sks-keyservers.net --recv-key "$GPG_KEY" \
    && gpg --batch --verify "$DISTRO_NAME.tar.gz.asc" "$DISTRO_NAME.tar.gz" \
    && tar -xzf "$DISTRO_NAME.tar.gz" \
    && mv "$DISTRO_NAME/conf/"* "$ZOO_CONF_DIR" \
    && rm -rf "$GNUPGHOME" "$DISTRO_NAME.tar.gz" "$DISTRO_NAME.tar.gz.asc" \
    && apk del .build-deps

WORKDIR $DISTRO_NAME
VOLUME ["$ZOO_DATA_DIR", "$ZOO_DATA_LOG_DIR"]

ENV PATH=$PATH:/$DISTRO_NAME/bin \
    ZOOCFGDIR=$ZOO_CONF_DIR \
    JVMFLAGS=$JVMFLAGS -Dcom.sun.management.jmxremote.local.only=false \
    JVMFLAGS=$JVMFLAGS -Dcom.sun.management.jmxremote.authenticate=false \
    JVMFLAGS=$JVMFLAGS -Dcom.sun.management.jmxremote.ssl=false \
    JVMFLAGS=$JVMFLAGS -Djava.rmi.server.hostname=0.0.0.0 \
    JVMFLAGS=$JVMFLAGS -Dcom.sun.management.jmxremote.host=0.0.0.0


RUN echo "JMXPORT = $JMXPORT and JVMFLAGS - $JVMFLAGS"

EXPOSE $ZOO_PORT 2888 3888 $JMXPORT


COPY docker-entrypoint.sh /

RUN chmod +x /docker-entrypoint.sh

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["zkServer.sh", "start-foreground"]