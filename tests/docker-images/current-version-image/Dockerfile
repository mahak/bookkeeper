#
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.
#

FROM ubuntu:22.04
MAINTAINER Apache BookKeeper <dev@bookkeeper.apache.org>

ARG BK_VERSION=DOESNOTEXISTS
ARG DISTRO_NAME=bookkeeper-dist-server-${BK_VERSION}-bin
ARG PKG_NAME=bookkeeper-server-${BK_VERSION}

ENV BOOKIE_PORT=3181
ENV BOOKIE_HTTP_PORT=8080
ENV BOOKIE_GRPC_PORT=4181
EXPOSE ${BOOKIE_PORT} ${BOOKIE_HTTP_PORT} ${BOOKIE_GRPC_PORT}
ENV BK_USER=bookkeeper
ENV BK_HOME=/opt/bookkeeper
ENV DEBIAN_FRONTEND=noninteractive
ARG UBUNTU_MIRROR=http://archive.ubuntu.com/ubuntu/
ARG UBUNTU_SECURITY_MIRROR=http://security.ubuntu.com/ubuntu/

RUN set -x \
    && sed -i -e "s|http://archive\.ubuntu\.com/ubuntu/|${UBUNTU_MIRROR:-http://archive.ubuntu.com/ubuntu/}|g" \
     -e "s|http://security\.ubuntu\.com/ubuntu/|${UBUNTU_SECURITY_MIRROR:-http://security.ubuntu.com/ubuntu/}|g" /etc/apt/sources.list \
    && echo 'Acquire::http::Timeout "30";\nAcquire::http::ConnectionAttemptDelayMsec "2000";\nAcquire::https::Timeout "30";\nAcquire::https::ConnectionAttemptDelayMsec "2000";\nAcquire::ftp::Timeout "30";\nAcquire::ftp::ConnectionAttemptDelayMsec "2000";\nAcquire::Retries "15";' > /etc/apt/apt.conf.d/99timeout_and_retries \
    && adduser "${BK_USER}" \
    && apt-get update \
    && apt-get install -y ca-certificates apt-transport-https \
    && apt-get install -y --no-install-recommends openjdk-17-jdk \
    && apt-get install -y --no-install-recommends python3 pip \
    && ln -s /usr/bin/python3 /usr/bin/python \
    && apt-get install -y --no-install-recommends gpg gpg-agent wget sudo \
    && apt-get -y install netcat-openbsd dnsutils less procps iputils-ping \
    && apt-get -y --purge autoremove \
    && apt-get autoclean \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && pip install zk-shell \
    && JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java)))) \
    && echo networkaddress.cache.ttl=1 >> $JAVA_HOME/conf/security/java.security \
    && echo networkaddress.cache.negative.ttl=1 >> $JAVA_HOME/conf/security/java.security

# untar tarballs
ADD target/${DISTRO_NAME}.tar.gz /opt
RUN mv /opt/${PKG_NAME} /opt/bookkeeper

WORKDIR /opt/bookkeeper

COPY target/scripts /opt/bookkeeper/scripts
COPY scripts/install-python-client.sh /opt/bookkeeper/scripts
RUN chmod +x -R /opt/bookkeeper/scripts/

# copy the python client
ADD target/bookkeeper-client/ /opt/bookkeeper/bookkeeper-client
RUN /opt/bookkeeper/scripts/install-python-client.sh

ENTRYPOINT [ "/bin/bash", "/opt/bookkeeper/scripts/entrypoint.sh" ]
CMD ["bookie"]

#HEALTHCHECK --interval=10s --timeout=60s CMD /bin/bash /opt/bookkeeper/scripts/healthcheck.sh
