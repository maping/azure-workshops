FROM ubuntu:16.04

LABEL maintainer="pinm@microsoft.com"

# To make it easier for build and release pipelines to run apt-get,
# configure apt to not require confirmation (assume the -y argument by default)
ENV DEBIAN_FRONTEND=noninteractive
RUN echo "APT::Get::Assume-Yes \"true\";" > /etc/apt/apt.conf.d/90assumeyes

RUN apt-get update \
&& apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        jq \
        git \
        iputils-ping \
        libcurl3 \
        libicu55 \
        libunwind8 \
        netcat

WORKDIR /usr
RUN mkdir java
ADD jdk-15 /usr/java/jdk-15
ADD apache-maven-3.6.3 /usr/apache-maven-3.6.3

ENV JAVA_HOME=/usr/java/jdk-15
ENV JAVA_BIN=$JAVA_HOME/bin
ENV CLASSPATH=.:$JAVA_HOME/lib/tools.jar
ENV M2_HOME=/usr/apache-maven-3.6.3
ENV PATH=$PATH:$JAVA_HOME/bin:$M2_HOME/bin

WORKDIR /azp

COPY ./start.sh .
RUN chmod +x start.sh

CMD ["./start.sh"]