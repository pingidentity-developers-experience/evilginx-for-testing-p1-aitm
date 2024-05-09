FROM golang:alpine

LABEL com.pingidentity.author="DemoOps Engineering"
LABEL com.pingidentity.organization="Technical Enablement"
LABEL description="A Docker image implementation of Evilginx configured only for testing PingOne Protect AiTM predictor or demo purposes."

ARG BUILD_RFC3339="1970-01-01T00:00:00Z"
ARG COMMIT="local"
ARG VERSION="v3.3.0"

ENV GITHUB_USER="kgretzky"
ENV EVILGINX_REPOSITORY="github.com/${GITHUB_USER}/evilginx2"
ENV INSTALL_PACKAGES="git make gcc musl-dev go"
ENV PROJECT_DIR="${GOPATH}/src/${EVILGINX_REPOSITORY}"
ENV EVILGINX_BIN="/bin/evilginx"

RUN mkdir -p ${GOPATH}/src/github.com/${GITHUB_USER} \
    && apk add --no-cache ${INSTALL_PACKAGES} \
    && git -C ${GOPATH}/src/github.com/${GITHUB_USER} clone https://github.com/${GITHUB_USER}/evilginx2 

# The sed command in the RUN section below fixes a regex in the evilginx src that does not properly match when there are periods in
# the Content-Type header for requests. It tries to match on just words, but PF uses (e.g. 'application/vnd.pingidentity.submitIdentifier+json')
# so it wasn't looking for credentials in the calls it needs to prior to this alteration.

RUN set -ex \
        && cd ${PROJECT_DIR}/ && git checkout tags/${VERSION} \
        && sed -i 's|application\\\\/\\\\w\*\\\\+?json|application\\\\/.*\\\\+?json|' ./core/http_proxy.go \
        && go get ./... && make \
		&& cp ${PROJECT_DIR}/build/evilginx ${EVILGINX_BIN} \
		&& apk del ${INSTALL_PACKAGES} && rm -rf /var/cache/apk/* \
        && rm -rf ${GOPATH}/src/*

COPY ./phishlets /phishlets
COPY ./config /config
COPY ./docker-entrypoint.sh /opt/

RUN chmod +x /opt/docker-entrypoint.sh
EXPOSE 443

ENTRYPOINT ["/opt/docker-entrypoint.sh"]


STOPSIGNAL SIGKILL
