FROM golang:1.19-alpine AS builder

WORKDIR /workspace

# 安装必要的构建工具
RUN apk add --no-cache gcc musl-dev

# 复制 go.mod 和 go.sum
COPY go.mod go.sum ./
RUN go mod download

# 复制源代码
COPY . .

# 设置构建参数
ARG BLADE_VERSION=1.7.4
ARG BLADE_VENDOR=community

# 编译
RUN CGO_ENABLED=1 GOOS=linux GOARCH=amd64 go build -ldflags="-linkmode external -extldflags -static -X=github.com/chaosblade-io/chaosblade-operator/version.CombinedVersion=${BLADE_VERSION},${BLADE_VENDOR}" -o /workspace/chaosblade-operator cmd/manager/main.go

# 最终镜像
FROM registry.access.redhat.com/ubi8/ubi-minimal:latest

ENV OPERATOR=/usr/local/bin/chaosblade-operator \
    CHAOSBLADE_HOME=/opt/chaosblade

COPY --from=builder /workspace/chaosblade-operator /usr/local/bin/
COPY build/bin /usr/local/bin

RUN chmod 777 /usr/local/bin/user_setup
RUN /usr/local/bin/user_setup

ENTRYPOINT ["/usr/local/bin/entrypoint"]
