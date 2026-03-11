
# https://github.com/transcend-io/terragrunt-atlantis-config local clone of this repo to build terragrunt-atlantis-config
ARG GO_VERSION=1.21
FROM golang:${GO_VERSION} AS build

WORKDIR /app
# copy source
COPY go.mod go.sum main.go ./

# fetch deps separately (for layer caching)
RUN go mod download

# build the executable
COPY cmd ./cmd
ENV CGO_ENABLED=0
RUN go build


FROM ghcr.io/runatlantis/atlantis AS atlantis
COPY --from=build /app/terragrunt-atlantis-config /usr/local/bin/terragrunt-atlantis-config
ARG AZURE_CLI_VERSION=2.58.0
ENV TERRAGRUNT_VERSION=v0.60.1 
ENV terragrunt_atlantis_config_version=1.19.0

USER root
RUN set -ex && \
    apk update && \
    apk add py3-pip && \
    apk add --virtual=build gcc libffi-dev musl-dev openssl-dev python3-dev cargo make unzip && \
    pip install --upgrade pip --break-system-packages && \
    pip install --upgrade setuptools --break-system-packages && \
    ###
    pip install azure-cli==${AZURE_CLI_VERSION} --break-system-packages && \
    az --version && \
    ### cleanup build dependencies
    apk del build

# Install terragrunt
RUN curl -o /usr/local/bin/terragrunt -fsSL "https://github.com/gruntwork-io/terragrunt/releases/download/${TERRAGRUNT_VERSION}/terragrunt_linux_amd64" \
    && chmod +x /usr/local/bin/terragrunt && chmod +x /usr/local/bin/terragrunt-atlantis-config

COPY repos.yaml /usr/local/bin/repos.yaml
ENV REPO_CONFIG /usr/local/bin/repos.yaml
RUN chown -R atlantis /usr/local/bin/repos.yaml

RUN mkdir -p /atlantis/.ssh && chmod 700 /atlantis/.ssh
COPY ./id_ed25519 /atlantis/.ssh/id_ed25519
RUN chmod 600 /atlantis/.ssh/id_ed25519 

ENV GIT_SSH_COMMAND="ssh -i /atlantis/.ssh/id_ed25519 -o 'StrictHostKeyChecking no'"
USER atlantis