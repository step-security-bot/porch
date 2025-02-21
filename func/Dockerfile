# Copyright 2022-2025 The kpt and Nephio Authors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM gcr.io/kpt-fn/apply-setters:v0.2.0@sha256:4d4295727183396f0c3c6a75d2560254c2f9041a39e95dc1e5beffeb49cc1a12 AS apply-setters
FROM gcr.io/kpt-fn/apply-setters:v0.1.1@sha256:d2a0219b24a47ebb4bf568d3ccf5bb0e7be2f3f94b28b0c3b1bed011a51d7c35 AS apply-setters-v1
FROM gcr.io/kpt-fn/ensure-name-substring:v0.1.1@sha256:3105ac150f86dcafa2d656e30feb5c51b83a43f5bb9b84458cdde620c9757227 AS ensure-name-substring
FROM gcr.io/kpt-fn/search-replace:v0.2.0@sha256:6864b06bf654fbdff3efb666a0a75fa14da2ba405f33187b60648e0b74bc2272 AS search-replace
FROM gcr.io/kpt-fn/set-annotations:v0.1.4@sha256:76d65ebddf6ce9c2c00bbcb3c0e963a2d856685bdcf2f1072386f41ca9b563e5 AS set-annotations
FROM gcr.io/kpt-fn/set-image:v0.1.0@sha256:d07677591e5b39a56d44bafaba0d881e4f2ae25ed020dfa2ad002fb9e62d1508 AS set-image
FROM gcr.io/kpt-fn/set-labels:v0.1.5@sha256:0dfc3c8aba8b791d0c45fe7eafbc8cf3c5920a09f69848f46dcc270ae3423641 AS set-labels
FROM gcr.io/kpt-fn/set-namespace:v0.2.0@sha256:7adc23986f97572d75af9aec6a7f74d60f7b9976227f43a75486633e7c539e6f AS set-namespace
FROM gcr.io/kpt-fn/set-namespace:v0.3.4@sha256:0ec0fb2380be42142a87a7c9815f0d30415e2da07468591dd9345c7c81d6c93e AS set-namespace-v3
FROM gcr.io/kpt-fn/set-project-id:v0.2.0@sha256:966c9e51163bce52274dfa0406ae580283712cb38625660cd2f66e9f13d2c01b AS set-project-id
FROM gcr.io/kpt-fn/starlark:v0.3.0@sha256:c347e28606fa1a608e8e02e03541a5a46e4a0152005df4a11e44f6c4ab1edd9a AS starlark
FROM gcr.io/kpt-fn/upsert-resource:v0.2.0@sha256:77196bdb9e33e4259d530ee1996a05c40bbefe39b80d52dcb66c32f7b85a62f1 AS upsert-resource

FROM golang:1.23.5-alpine3.21@sha256:47d337594bd9e667d35514b241569f95fb6d95727c24b19468813d596d5ae596 AS builder
WORKDIR /go/src

RUN go install github.com/grpc-ecosystem/grpc-health-probe@v0.4.11
RUN cp $GOPATH/bin/grpc-health-probe /grpc-health-probe

COPY . .
RUN cd func ; go build -v -o /server ./server
COPY func/config.yaml /config.yaml

FROM alpine:3.21@sha256:a8560b36e8b8210634f77d9f7f9efd7ffa463e380b75e2e74aff4511df3ef88c

COPY --from=apply-setters          /usr/local/bin/function /functions/apply-setters
COPY --from=apply-setters-v1       /usr/local/bin/function /functions/apply-setters-v1
COPY --from=ensure-name-substring  /usr/local/bin/function /functions/ensure-name-substring
COPY --from=search-replace         /usr/local/bin/function /functions/search-replace
COPY --from=set-annotations        /usr/local/bin/function /functions/set-annotations
COPY --from=set-image              /usr/local/bin/function /functions/set-image
COPY --from=set-labels             /usr/local/bin/function /functions/set-labels
COPY --from=set-namespace          /usr/local/bin/function /functions/set-namespace
COPY --from=set-namespace-v3       /usr/local/bin/function /functions/set-namespace-v3
COPY --from=set-project-id         /usr/local/bin/function /functions/set-project-id
COPY --from=starlark               /usr/local/bin/star     /functions/starlark
COPY --from=upsert-resource        /usr/local/bin/function /functions/upsert-resource
COPY --from=builder                /server /grpc-health-probe /config.yaml /

EXPOSE 9445/tcp
ENTRYPOINT [ "/server", "--config=/config.yaml", "--functions=/functions" ]
