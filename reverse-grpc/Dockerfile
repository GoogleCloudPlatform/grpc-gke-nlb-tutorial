# Copyright 2019 Google LLC
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

# https://docs.docker.com/engine/reference/builder/

FROM golang:1.12-alpine as build

WORKDIR /src

ADD . /src/

RUN apk add --no-cache git protobuf && \
    go get github.com/golang/protobuf/protoc-gen-go && \
    protoc --proto_path=api --go_out=plugins=grpc:api api/*.proto && \
    GOPATH="/go" GO111MODULE=on CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go install -installsuffix "static" && \
    echo 'nobody:x:65534:' > /src/group.nobody && \
    echo 'nobody:x:65534:65534::/:' > /src/passwd.nobody && \
    GRPC_HEALTH_PROBE_VERSION=v0.2.2 && \
    wget -q -O /bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 && \
    chmod +x /bin/grpc_health_probe

FROM gcr.io/distroless/static

ENV PORT=8080
EXPOSE 8080

COPY --from=build /src/group.nobody /etc/group
COPY --from=build /src/passwd.nobody /etc/passwd
USER nobody:nobody

COPY --from=build /go/bin/reverse-grpc /bin/reverse-grpc
# Add grpc-health-probe to use with readiness and liveness probes
COPY --from=build /bin/grpc_health_probe /bin/grpc_health_probe

ENTRYPOINT ["/bin/reverse-grpc"]
