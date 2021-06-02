# Copyright 2021 Google LLC
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

FROM golang:1.16 as build

RUN apt-get -y update && \
    apt-get -y install unzip

ARG PROTOC_VERSION=3.17.1
RUN wget -q -O protoc.zip "https://github.com/protocolbuffers/protobuf/releases/download/v${PROTOC_VERSION}/protoc-${PROTOC_VERSION}-linux-x86_64.zip" && \
    unzip -o protoc.zip bin/protoc -d /

ARG PROTOC_GEN_GO_VERSION=1.26.0
RUN go install "google.golang.org/protobuf/cmd/protoc-gen-go@v${PROTOC_GEN_GO_VERSION}"

ARG PROTOC_GEN_GO_GRPC_VERSION=1.1.0
RUN go install "google.golang.org/grpc/cmd/protoc-gen-go-grpc@v${PROTOC_GEN_GO_GRPC_VERSION}"

ARG GRPC_HEALTH_PROBE_VERSION=0.4.2
RUN wget -q -O /bin/grpc_health_probe "https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/v${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64" && \
    chmod +x /bin/grpc_health_probe

WORKDIR /src

COPY go.mod /src/
COPY go.sum /src/
RUN go mod download

COPY . /src/
RUN go generate ./...
RUN CGO_ENABLED=0 go build -o /bin/reverse-grpc -ldflags="-s -w" -trimpath .

FROM gcr.io/distroless/base:debug-nonroot

ENV GOTRACEBACK=single
ENV PORT=8080
EXPOSE 8080

COPY --from=build /bin/grpc_health_probe /bin/grpc_health_probe
COPY --from=build /bin/reverse-grpc /bin/reverse-grpc

ENTRYPOINT ["/bin/reverse-grpc"]
