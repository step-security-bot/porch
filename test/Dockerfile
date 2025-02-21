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


FROM golang:1.23.5-bookworm@sha256:9c79a16e024bcfb856b6d063cf7ed9a6257f554466761f5a99b09787d2b55fbd AS builder

WORKDIR /go/src

# cache deps before building and copying source so that we don't need to re-download as much
# and so that source changes don't invalidate our downloaded layer

COPY . .
RUN cd test; go build -v -o /git-server ./git

FROM debian:bookworm-slim@sha256:40b107342c492725bc7aacbe93a49945445191ae364184a6d24fedb28172f6f7
RUN apt update && apt install -y ca-certificates && rm -rf /var/lib/apt && rm -rf /var/cache/apt
COPY --from=builder /git-server /git-server
ENTRYPOINT ["/git-server", "--port=8080"]
