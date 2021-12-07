# Set tools versions
ARG TERRAFORM_VERSION="0.15.5"
ARG IMAGE_NAME="python"
ARG IMAGE_VERSION="rc-alpine3.13"
ARG MAINTAINER="nnsense"

# python:rc-alpine3.13
FROM $IMAGE_NAME:$IMAGE_VERSION
LABEL maintainer=$MAINTAINER
ARG TERRAFORM_VERSION

RUN apk update && apk add --no-cache \
bash=5.1.0-r0 \
jq=1.6-r1 \
git=2.30.2-r0 \
openssh-client=8.4_p1-r3 \
curl=7.77.0-r1 \
binutils=2.35.2-r1 \
xmlstarlet=1.6.1-r0

SHELL ["/bin/bash", "-c"]

# Install jfrog (https://jfrog.com/getcli/)
RUN curl -fL https://getcli.jfrog.io | sh && mv jfrog /usr/local/bin/

# Install helm (https://github.com/helm/helm/releases)
RUN curl "https://get.helm.sh/helm-v3.5.2-linux-amd64.tar.gz" -o helm.tar.gz && \
    tar -zxvf helm.tar.gz && \
    mv linux-amd64/helm /usr/local/bin/helm && \
    rm -fr helm.tar.gz linux-amd64/

# Install terraform (https://www.terraform.io/downloads.html)
RUN curl "https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip" -o "terraform.zip"
RUN unzip -j terraform.zip && mv terraform /usr/local/bin/terraform && rm -f terraform.zip

# Install tfsec
RUN wget "https://github.com/tfsec/tfsec/releases/download/v0.40.2/tfsec-linux-amd64" -O  tfsec
RUN install ./tfsec /usr/local/bin/

# Install aws cli
RUN GLIBC_VER=$(curl -s https://api.github.com/repos/sgerrand/alpine-pkg-glibc/releases/latest | grep tag_name | cut -d : -f 2,3 | tr -d \",' ') \
    && curl -sL https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub -o /etc/apk/keys/sgerrand.rsa.pub \
    && curl -sLO https://github.com/sgerrand/alpine-pkg-glibc/releases/download/${GLIBC_VER}/glibc-${GLIBC_VER}.apk \
    && curl -sLO https://github.com/sgerrand/alpine-pkg-glibc/releases/download/${GLIBC_VER}/glibc-bin-${GLIBC_VER}.apk \
    && apk add --no-cache \
    glibc-${GLIBC_VER}.apk \
    glibc-bin-${GLIBC_VER}.apk

RUN curl -sL https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip -o awscliv2.zip \
    && unzip awscliv2.zip \
    && aws/install \
    && rm -rf awscliv2.zip aws

RUN rm -rf /usr/local/aws-cli/v2/*/dist/aws_completer \
    /usr/local/aws-cli/v2/*/dist/awscli/data/ac.index \
    /usr/local/aws-cli/v2/*/dist/awscli/examples \
    glibc-${GLIBC_VER}.apk \
    glibc-bin-${GLIBC_VER}.apk \
    /var/cache/apk/*

# Install kubectl
RUN STABLE_REL=$(curl -L -s https://dl.k8s.io/release/stable.txt) && curl -LO "https://dl.k8s.io/release/${STABLE_REL}/bin/linux/amd64/kubectl" && install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl && rm -f kubectl

CMD ["bash"]
