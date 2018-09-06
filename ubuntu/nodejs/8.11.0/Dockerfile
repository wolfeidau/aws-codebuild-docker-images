# Copyright 2017-2017 Amazon.com, Inc. or its affiliates. All Rights Reserved.
#
# Licensed under the Amazon Software License (the "License"). You may not use this file except in compliance with the License.
# A copy of the License is located at
#
#    http://aws.amazon.com/asl/
#
# or in the "license" file accompanying this file.
# This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, express or implied.
# See the License for the specific language governing permissions and limitations under the License.
#

FROM ubuntu:16.04

ENV DOCKER_BUCKET="download.docker.com" \
    DOCKER_VERSION="17.09.0-ce" \
    DOCKER_CHANNEL="stable" \
    DOCKER_SHA256="a9e90a73c3cdfbf238f148e1ec0eaff5eb181f92f35bdd938fd7dab18e1c4647" \
    DIND_COMMIT="3b5fac462d21ca164b3778647420016315289034" \
    DOCKER_COMPOSE_VERSION="1.21.2" \
    GITVERSION_VERSION="3.6.5"

RUN echo "deb mirror://mirrors.ubuntu.com/mirrors.txt xenial main restricted universe multiverse" > /etc/apt/sources.list && \
    echo "deb mirror://mirrors.ubuntu.com/mirrors.txt xenial-updates main restricted universe multiverse" >> /etc/apt/sources.list && \
    echo "deb mirror://mirrors.ubuntu.com/mirrors.txt xenial-security main restricted universe multiverse" >> /etc/apt/sources.list && \
    DEBIAN_FRONTEND=noninteractive apt-get update

RUN readonly __packages=" \
    ca-certificates \
    curl \
    git \
    build-essential \
    less \
    mime-support \
    openssl \
    perl \
    python \
    unzip \
    wget \
    zip \
    openssh-client \
    software-properties-common \
    apt-transport-https \
    " \
    && apt-get update \
    && apt-get upgrade --assume-yes \
    && apt-get install --no-install-recommends \
    --assume-yes \
    ${__packages} \
    # clean up apt cache to make Docker image slimmer
    && apt-get clean \
    && mkdir ~/.ssh \
    && touch ~/.ssh/known_hosts \
    && ssh-keyscan -t rsa,dsa -H github.com >> ~/.ssh/known_hosts \
    && ssh-keyscan -t rsa,dsa -H bitbucket.org >> ~/.ssh/known_hosts \
    && chmod 600 ~/.ssh/known_hosts


# Install Docker
RUN set -ex \
    && curl -fSL "https://${DOCKER_BUCKET}/linux/static/${DOCKER_CHANNEL}/x86_64/docker-${DOCKER_VERSION}.tgz" -o docker.tgz \
    && echo "${DOCKER_SHA256} *docker.tgz" | sha256sum -c - \
    && tar --extract --file docker.tgz --strip-components 1  --directory /usr/local/bin/ \
    && rm docker.tgz \
    && docker -v \
# set up subuid/subgid so that "--userns-remap=default" works out-of-the-box
    && addgroup dockremap \
    && useradd -g dockremap dockremap \
    && echo 'dockremap:165536:65536' >> /etc/subuid \
    && echo 'dockremap:165536:65536' >> /etc/subgid \
    && wget "https://raw.githubusercontent.com/docker/docker/${DIND_COMMIT}/hack/dind" -O /usr/local/bin/dind \
    && curl -L https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-Linux-x86_64 > /usr/local/bin/docker-compose \
    && chmod +x /usr/local/bin/dind /usr/local/bin/docker-compose \
# Ensure docker-compose works
    && docker-compose version

RUN set -ex \
    && add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
    && readonly __docker_packages=" \
    docker-ce=17.12.0* \
    iptables=1.6.0-* \
    libltdl7=2.4.6-* \
    libnfnetlink0=1.0.1-* \
    libxtables11=1.6.0-* \
    " \
    && apt-get update \
    && apt-get install --no-install-recommends --allow-unauthenticated \
    --assume-yes \
    ${__docker_packages} \
    # clean up apt cache to make Docker image slimmer
    && apt-get clean \
    # install Docker compose - from official page https://docs.docker.com/compose/install/#install-compose
    && readonly __docker_compose_version="1.18.0" \
    && curl --show-error \
    --location \
    https://github.com/docker/compose/releases/download/${__docker_compose_version}/docker-compose-$(uname -s)-$(uname -m) \
    --output \
    /usr/local/bin/docker-compose \
    && chmod +x /usr/local/bin/docker-compose \
    && docker --version \
    && docker-compose --version

# Install dependencies by all python images equivalent to buildpack-deps:jessie
# on the public repos.

RUN set -ex \
    && wget "https://bootstrap.pypa.io/2.6/get-pip.py" -O /tmp/get-pip.py \
    && python /tmp/get-pip.py \
    && pip install --upgrade pip \
    && pip install awscli==1.* \
    && rm -fr /var/lib/apt/lists/* /tmp/* /var/tmp/*

VOLUME /var/lib/docker

COPY dockerd-entrypoint.sh /usr/local/bin/

ENV NODE_VERSION="8.11.0"

# gpg keys listed at https://github.com/nodejs/node#release-team
RUN set -ex \
    && for key in \
      94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
      B9AE9905FFD7803F25714661B63B535A4C206CA9 \
      77984A986EBC2AA786BC0F66B01FBB92821C587A \
      56730D5401028683275BD23C23EFEFE93C4CFFFE \
      71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
      FD3A5288F042B6850C66B31F09FE44734EB7990E \
      8FCCA13FEF1D0C2E91008E09770F7A9A5AE15600 \
      C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
      DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
      9554F04D7259F04124DE6B476D5A82AC7E37093B \
      93C7E9E91B49E432C2F75674B0A78B0A6C481CF6 \
      114F43EE0176B71C7BC219DD50A3051F888C628D \
      7937DFD2AB06298B2293C3187D33FF9D0246406D \
    ; do \
      gpg --keyserver hkp://ipv4.pool.sks-keyservers.net --recv-keys "$key" || \
      gpg --keyserver hkp://pgp.mit.edu:80 --recv-keys "$key" ; \
    done

RUN set -ex \
	&& wget "https://nodejs.org/download/release/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.gz" -O node-v$NODE_VERSION-linux-x64.tar.gz \
	&& wget "https://nodejs.org/download/release/v$NODE_VERSION/SHASUMS256.txt.asc" -O SHASUMS256.txt.asc \
	&& gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
	&& grep " node-v$NODE_VERSION-linux-x64.tar.gz\$" SHASUMS256.txt | sha256sum -c - \
		&& tar -xzf "node-v$NODE_VERSION-linux-x64.tar.gz" -C /usr/local --strip-components=1 \
		&& rm "node-v$NODE_VERSION-linux-x64.tar.gz" SHASUMS256.txt.asc SHASUMS256.txt \
		&& ln -s /usr/local/bin/node /usr/local/bin/nodejs

RUN npm set unsafe-perm true

RUN npm install yarn -g
RUN npm install npm -g

RUN curl -fSL https://github.com/buildkite/agent/releases/download/v3.2.1/buildkite-agent-linux-amd64-3.2.1.tar.gz -o buildkite-agent.tar.gz \
    && mkdir -p /opt/buildkite \
    && tar --extract --file buildkite-agent.tar.gz --directory /opt/buildkite \
    && rm buildkite-agent.tar.gz \
    && chmod +x /opt/buildkite/buildkite-agent /opt/buildkite/bootstrap.sh \
    && apt-get update && apt-get install -y --no-install-recommends jq \
    && apt-get clean \
    && rm -fr /var/lib/apt/lists/* /tmp/* /var/tmp/*


CMD [ "node" ]
