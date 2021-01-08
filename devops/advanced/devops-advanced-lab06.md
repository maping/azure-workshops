# Lab06 Install and Configure Java+Maven Ubuntu Docker Agent

## 1. Create Docker Ubuntu VM

## 2. Authenticate with a personal access token (PAT)
Click "User settings", Click "Personal access token", Click "New token", input "Ubuntu-Java-Maven-Docker-Agent", Choose Organization，1 year, Scopes Choose "Custom defined"：Agent Pool Read & Manage，Deployment Groups Read & Manage，Copy Token.

## 3. Create Agent Pool
Click "Orgnization settings", Click "Pipelines -> Agent Pools", Click "Add pool", Input "Self-Hosted-Docker-Agent"。

## 4. Create Java + Maven Ubuntu VM Agent
```console
$ ssh azureuser@104.41.165.174
$ mkdir code/docker/azagent/ubuntuagent/1604/base -p && cd code/docker/azagent/ubuntuagent/1604/base
$ vim Dockerfile
FROM ubuntu:16.04

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

WORKDIR /azp

COPY ./start.sh .
RUN chmod +x start.sh

CMD ["./start.sh"]

$ vim start.sh
#!/bin/bash
set -e

if [ -z "$AZP_URL" ]; then
  echo 1>&2 "error: missing AZP_URL environment variable"
  exit 1
fi

if [ -z "$AZP_TOKEN_FILE" ]; then
  if [ -z "$AZP_TOKEN" ]; then
    echo 1>&2 "error: missing AZP_TOKEN environment variable"
    exit 1
  fi

  AZP_TOKEN_FILE=/azp/.token
  echo -n $AZP_TOKEN > "$AZP_TOKEN_FILE"
fi

unset AZP_TOKEN

if [ -n "$AZP_WORK" ]; then
  mkdir -p "$AZP_WORK"
fi

rm -rf /azp/agent
mkdir /azp/agent
cd /azp/agent

export AGENT_ALLOW_RUNASROOT="1"

cleanup() {
  if [ -e config.sh ]; then
    print_header "Cleanup. Removing Azure Pipelines agent..."

    ./config.sh remove --unattended \
      --auth PAT \
      --token $(cat "$AZP_TOKEN_FILE")
  fi
}

print_header() {
  lightcyan='\033[1;36m'
  nocolor='\033[0m'
  echo -e "${lightcyan}$1${nocolor}"
}

# Let the agent ignore the token env variables
export VSO_AGENT_IGNORE=AZP_TOKEN,AZP_TOKEN_FILE

print_header "1. Determining matching Azure Pipelines agent..."

AZP_AGENT_RESPONSE=$(curl -LsS \
  -u user:$(cat "$AZP_TOKEN_FILE") \
  -H 'Accept:application/json;api-version=3.0-preview' \
  "$AZP_URL/_apis/distributedtask/packages/agent?platform=linux-x64")

if echo "$AZP_AGENT_RESPONSE" | jq . >/dev/null 2>&1; then
  AZP_AGENTPACKAGE_URL=$(echo "$AZP_AGENT_RESPONSE" \
    | jq -r '.value | map([.version.major,.version.minor,.version.patch,.downloadUrl]) | sort | .[length-1] | .[3]')
fi

if [ -z "$AZP_AGENTPACKAGE_URL" -o "$AZP_AGENTPACKAGE_URL" == "null" ]; then
  echo 1>&2 "error: could not determine a matching Azure Pipelines agent - check that account '$AZP_URL' is correct and the token is valid for that account"
  exit 1
fi

print_header "2. Downloading and installing Azure Pipelines agent..."

curl -LsS $AZP_AGENTPACKAGE_URL | tar -xz & wait $!

source ./env.sh

trap 'cleanup; exit 130' INT
trap 'cleanup; exit 143' TERM

print_header "3. Configuring Azure Pipelines agent..."

./config.sh --unattended \
  --agent "${AZP_AGENT_NAME:-$(hostname)}" \
  --url "$AZP_URL" \
  --auth PAT \
  --token $(cat "$AZP_TOKEN_FILE") \
  --pool "${AZP_POOL:-Default}" \
  --work "${AZP_WORK:-_work}" \
  --replace \
  --acceptTeeEula & wait $!

# remove the administrative token before accepting work
rm $AZP_TOKEN_FILE

print_header "4. Running Azure Pipelines agent..."

# `exec` the node runtime so it's aware of TERM and INT signals
# AgentService.js understands how to handle agent self-update and restart
exec ./externals/node/bin/node ./bin/AgentService.js interactive

$ sudo docker build -t devops-agent-ubuntu-base:16.04 . 
$ sudo docker images
REPOSITORY                 TAG       IMAGE ID       CREATED          SIZE
devops-agent-ubuntu-base   16.04     4ff52dcc393b   18 minutes ago   271MB
ubuntu                     16.04     9499db781771   4 weeks ago      131MB
hello-world                latest    bf756fb1ae65   11 months ago    13.3kB
```
> Problem：I couldn't create successfully with Dockerfile(Ubuntu 18.04) provided by offical document,to be invested.

## 5. Run Docker Ubuntu Agent
```console
$ ssh azureuser@104.41.165.174
$ sudo docker run -e AZP_URL=https://dev.azure.com/maping930883/ -e AZP_TOKEN=******************** -e AZP_POOL=Self-Hosted-Docker-Agent -e AZP_AGENT_NAME=Ubuntu-Docker-Agent-$RANDOM devops-agent-ubuntu-base:16.04
1. Determining matching Azure Pipelines agent...
2. Downloading and installing Azure Pipelines agent...
3. Configuring Azure Pipelines agent...

  ___                      ______ _            _ _
 / _ \                     | ___ (_)          | (_)
/ /_\ \_____   _ _ __ ___  | |_/ /_ _ __   ___| |_ _ __   ___  ___
|  _  |_  / | | | '__/ _ \ |  __/| | '_ \ / _ \ | | '_ \ / _ \/ __|
| | | |/ /| |_| | | |  __/ | |   | | |_) |  __/ | | | | |  __/\__ \
\_| |_/___|\__,_|_|  \___| \_|   |_| .__/ \___|_|_|_| |_|\___||___/
                                   | |
        agent v2.179.0             |_|          (commit bd605d6)


>> End User License Agreements:

Building sources from a TFVC repository requires accepting the Team Explorer Everywhere End User License Agreement. This step is not required for building sources from Git repositories.

A copy of the Team Explorer Everywhere license agreement can be found at:
  /azp/agent/externals/tee/license.html


>> Connect:

Connecting to server ...

>> Register Agent:

Scanning for tool capabilities.
Connecting to the server.
Successfully added the agent
Testing agent connection.
2020-12-27 04:21:05Z: Settings Saved.
4. Running Azure Pipelines agent...
Starting Agent listener interactively
Started listener process
Started running service
Scanning for tool capabilities.
Connecting to the server.
2020-12-27 04:21:09Z: Listening for Jobs

$ sudo docker ps
CONTAINER ID   IMAGE                            COMMAND        CREATED              STATUS              PORTS     NAMES
3cb8be1216b0   devops-agent-ubuntu-base:16.04   "./start.sh"   About a minute ago   Up About a minute             pedantic_solomon
```

Click "Orgnization settings", Click "Pipelines -> Agent Pools", Click "Self-Hosted-Docker-Agent", you can see a Docker Ubuntu Agent Online.

## 6. Install Java + Maven software to Ubuntu Docker Agent
```console
$ ssh azureuser@104.41.165.174
$ mkdir code/docker/azagent/ubuntuagent/1604/java-maven -p && cd code/docker/azagent/ubuntuagent/1604/java-maven
$ vim Dockerfile
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

$ sudo docker build -t devops-agent-ubuntu-java-maven:16.04 . 
$ sudo docker images
REPOSITORY                       TAG       IMAGE ID       CREATED         SIZE
devops-agent-ubuntu-java-maven   16.04     71c68f82383d   2 hours ago     618MB
devops-agent-ubuntu-base         16.04     4ff52dcc393b   3 hours ago     271MB
ubuntu                           16.04     9499db781771   4 weeks ago     131MB
hello-world                      latest    bf756fb1ae65   11 months ago   13.3kB
$ sudo docker run -e AZP_URL=https://dev.azure.com/maping930883/ -e AZP_TOKEN=************************* -e AZP_POOL=Self-Hosted-Docker-Agent -e AZP_AGENT_NAME=Ubuntu-Docker-Agent-$RANDOM devops-agent-ubuntu-java-maven:16.04
1. Determining matching Azure Pipelines agent...
2. Downloading and installing Azure Pipelines agent...
3. Configuring Azure Pipelines agent...

  ___                      ______ _            _ _
 / _ \                     | ___ (_)          | (_)
/ /_\ \_____   _ _ __ ___  | |_/ /_ _ __   ___| |_ _ __   ___  ___
|  _  |_  / | | | '__/ _ \ |  __/| | '_ \ / _ \ | | '_ \ / _ \/ __|
| | | |/ /| |_| | | |  __/ | |   | | |_) |  __/ | | | | |  __/\__ \
\_| |_/___|\__,_|_|  \___| \_|   |_| .__/ \___|_|_|_| |_|\___||___/
                                   | |
        agent v2.179.0             |_|          (commit bd605d6)


>> End User License Agreements:

Building sources from a TFVC repository requires accepting the Team Explorer Everywhere End User License Agreement. This step is not required for building sources from Git repositories.

A copy of the Team Explorer Everywhere license agreement can be found at:
  /azp/agent/externals/tee/license.html


>> Connect:

Connecting to server ...

>> Register Agent:

Scanning for tool capabilities.
Connecting to the server.
Successfully added the agent
Testing agent connection.
2020-12-27 07:23:45Z: Settings Saved.
4. Running Azure Pipelines agent...
Starting Agent listener interactively
Started listener process
Started running service
Scanning for tool capabilities.
Connecting to the server.
2020-12-27 07:23:49Z: Listening for Jobs
```
Click "Self-Hosted-Ubuntu-Agent"，Click "Agents"，you can see a Ubuntu-Java-Maven-Docker-Agent Online.

## 7. Test Ubuntu-Java-Maven-Docker-Agent
Create a java-helloworld-web-app Project，Create a Pipeline，Use Self-Hosted-Docker-Agent as Agent Pool，Save and Queue.
```console
2020-12-27 07:23:49Z: Listening for Jobs
2020-12-27 07:25:01Z: Running job: Agent job 1
2020-12-27 07:27:16Z: Job Agent job 1 completed with result: Succeeded
```

# Reference：
1. https://robertoprevato.github.io/Self-hosted-Azure-DevOps-agents-running-in-Docker/
2. https://github.com/RobertoPrevato/AzureDevOps-agents
