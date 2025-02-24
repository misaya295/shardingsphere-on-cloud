#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

# Build the manager binary
FROM golang:1.17 as builder

WORKDIR /workspace

# Copy the Go Modules manifests
COPY go.mod go.mod
COPY go.sum go.sum
# cache deps before building and copying source so that we don't need to re-download as much
# and so that source changes don't invalidate our downloaded layer
# ENV GOPROXY=https://goproxy.cn
RUN go mod download

# Copy the go source
COPY main.go main.go
COPY api api/
COPY pkg pkg/
RUN mkdir -p certs && cd certs && openssl req -new -SHA256 -newkey rsa:2048 -nodes -keyout tls.key -out tls.csr -subj "/C=CN/ST=beijing/L=beijing/O=/OU=/" && openssl x509 -req -sha256 -days 365 -in tls.csr -signkey tls.key -out tls.crt
# Build
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build  -ldflags "-w -s" -gcflags "-N -l" -a -o operator main.go

# Use distroless as minimal base image to package the manager binary
# Refer to https://github.com/GoogleContainerTools/distroless for more details
FROM alpine:3.15.4
WORKDIR /
COPY --from=builder /workspace/operator .
USER root
COPY --from=builder /workspace/certs /etc/operator/certs

ENTRYPOINT ["/operator"]
