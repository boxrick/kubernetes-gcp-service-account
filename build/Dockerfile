FROM alpine:latest as arg_check

# Exit if ARGS not set

ARG CLUSTER
ARG PROJECT
ARG ZONE

RUN test -n "$CLUSTER" || (echo "CLUSTER not set, usage '--build-arg CLUSTER=example'" && false)
RUN test -n "$PROJECT" || (echo "PROJECT not set, usage '--build-arg PROJECT=example'" && false)
RUN test -n "$ZONE" || (echo "ZONE not set, usage '--build-arg ZONE=europe-west1-d'" && false)

FROM alpine:latest as kubectl-base

# Get latest kubectl then burn this container
RUN apk add curl ca-certificates
RUN version=$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt) && \
    curl https://storage.googleapis.com/kubernetes-release/release/${version}/bin/linux/amd64/kubectl --output kubectl

FROM python:3.5.6-alpine3.9 as gcloud-base

ARG CLOUD_SDK_VERSION=239.0.0
ENV CLOUD_SDK_VERSION=$CLOUD_SDK_VERSION
RUN apk --no-cache add \
        curl \
        bash \
        libc6-compat \
        openssh-client \
        git \
        gnupg \
    && curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-${CLOUD_SDK_VERSION}-linux-x86_64.tar.gz && \
    tar xzf google-cloud-sdk-${CLOUD_SDK_VERSION}-linux-x86_64.tar.gz

# Copy kubectl from previous container and set up
FROM python:3.5.6-alpine3.9
COPY --from=kubectl-base kubectl /usr/local/bin/kubectl

ARG CLUSTER
ARG PROJECT
ARG ZONE

ENV HOME=/config

RUN set -x && \
    apk add --no-cache ca-certificates && \
    chmod +x /usr/local/bin/kubectl && \
    adduser kubectl -D -h /config && \
    kubectl version --client

# Set a lower level kubernetes user
USER kubectl

# Set up gcloud
COPY --from=gcloud-base /google-cloud-sdk /google-cloud-sdk
COPY --chown=kubectl ./gcloud-config /config/.gcloud_config

# Set up gcloud paths and config and auth kubernetes
ENV PATH /google-cloud-sdk/bin:$PATH
RUN set -x && \
    gcloud config set core/disable_usage_reporting true && \
    gcloud config set component_manager/disable_update_check true && \
    gcloud config set metrics/environment github_docker_image && \
    gcloud --version && \
    gcloud auth activate-service-account --key-file=/config/.gcloud_config && \
    gcloud container clusters get-credentials $CLUSTER --zone $ZONE --project $PROJECT

# Copy in gcloud config, then authenticate and generate kubernetes config

ENTRYPOINT ["/usr/local/bin/kubectl"]
