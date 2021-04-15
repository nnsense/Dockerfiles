# Set tools versions
ARG TERRAFORM_VERSION="0.13.6"
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
curl=7.74.0-r1 \
binutils=2.35.1-r1 \
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

# Install jdk8
ENV JAVA_HOME=/usr/lib/jvm/default-jvm \
    JAVA_VERSION=8u131 \
    JAVA_ALPINE_VERSION="8.275.01-r0"

ENV PATH=$PATH:$JAVA_HOME/jre/bin:$JAVA_HOME/bin \
    LANG=C.UTF-8

RUN apk add --no-cache openjdk8="$JAVA_ALPINE_VERSION"

# Install mvn
RUN apk add --update ca-certificates && rm -rf /var/cache/apk/* && \
  find /usr/share/ca-certificates/mozilla/ -name "*.crt" -exec keytool -import -trustcacerts \
  -keystore /usr/lib/jvm/java-1.8-openjdk/jre/lib/security/cacerts -storepass changeit -noprompt \
  -file {} -alias {} \; && \
  keytool -list -keystore /usr/lib/jvm/java-1.8-openjdk/jre/lib/security/cacerts --storepass changeit

ENV MAVEN_VERSION="3.6.3" \
    MAVEN_HOME="/usr/lib/mvn" 

ENV PATH=$PATH:$MAVEN_HOME/bin

RUN wget http://archive.apache.org/dist/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz && \
  tar -zxvf apache-maven-$MAVEN_VERSION-bin.tar.gz && \
  rm apache-maven-$MAVEN_VERSION-bin.tar.gz && \
  mv apache-maven-$MAVEN_VERSION /usr/lib/mvn

CMD ["bash"]
