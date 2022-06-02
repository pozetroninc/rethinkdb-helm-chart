# Copyright 2016 The Kubernetes Authors All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Versions
ARG UPSTREAM_IMAGE=rethinkdb:2.4.2

FROM golang:1.16 AS builder

COPY ./rethinkdb-probe/ /go/src/rethinkdb-probe
WORKDIR /go/src/rethinkdb-probe
RUN go get ./... \
    && CGO_ENABLED=0 GOOS=linux go build -a -ldflags '-w -s' -o /target/rethinkdb-probe .

ARG UPSTREAM_IMAGE
ENV UPSTREAM_IMAGE=${UPSTREAM_IMAGE}

FROM ${UPSTREAM_IMAGE}
MAINTAINER Pozetron Inc <hello@pozetroninc.com>

RUN apt-get update && \
    apt-get install -yq curl && \
    rm -rf /var/cache/apt/* && rm -rf /var/lib/apt/lists/*

ADD http://stedolan.github.io/jq/download/linux64/jq /usr/bin/jq
RUN chmod +x /usr/bin/jq

ADD https://github.com/Yelp/dumb-init/releases/download/v1.2.0/dumb-init_1.2.0_amd64 /usr/local/bin/dumb-init
RUN chmod +x /usr/local/bin/dumb-init

COPY ./files/run.sh /
COPY --from=builder /target/rethinkdb-probe /
RUN chmod u+x /run.sh /rethinkdb-probe

ENTRYPOINT ["/usr/local/bin/dumb-init", "/run.sh"]
