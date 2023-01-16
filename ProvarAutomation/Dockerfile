# Create Jenkins user context
FROM dockerhub.es.ecg.tools/node:17-alpine

RUN npm -g config set user $USER
RUN npm config set loglevel error 
RUN apk add --no-cache jq curl openssl git

RUN adduser -h /home/jenkins -u 1002 -D  jenkins

RUN echo 'jenkins  ALL=(ALL) /bin/su' >>  /etc/sudoers

USER jenkins


## Initial stage to use base image with ANT and Oracle JDK 8
FROM frekele/ant:1.10.3-jdk8
LABEL maintainer="Provar Testing <support@provartesting.com>"
LABEL version="1.0"
ARG PROVAR_DEFAULT_VERSION=latest
ARG PROVAR_MAJOR_VERSION=latest
ARG PROJECT_NAME=eBayRegressionProject
# ARG EMAIL_TARGET
ARG ENV
ARG ANT_TARGET=runtests
# ARG TEST_PLAN
# ARG BUILD_FILE
# ARG ProvarSecretsPassword
# ARG PROVAR_sf_Admin
# ARG PROVAR_sf_Admin_password

# The location to save the Provar binaries to (from downloads page)
ENV REPO_HOME=/srv/Provar \
    PROVAR_VERSION=${PROVAR_DEFAULT_VERSION} \
    ## Set the WORKSPACE for execution
    WORKSPACE=/home/${PROJECT_NAME} \
    DISPLAY=:99.0 \
    JAVA_ARGS=-verbose:class \
    ENVIRONMENT=${ENV} \
    ANT_TARGET=${ANT_TARGET}
    # EMAIL_TARGET=${EMAIL_TARGET} \
    # TEST_PLAN=${TEST_PLAN} \
    # BUILD_FILE=${BUILD_FILE} \
    # ProvarSecretsPassword=${ProvarSecretsPassword} \
    # PROVAR_sf_Admin=${PROVAR_sf_Admin} \
    # PROVAR_sf_Admin_password=${PROVAR_sf_Admin_password} 

# Install required tools
RUN set -ex \
    && apt-get -y update -qq && apt-get install -y xvfb wget \
    && wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
    && apt-get update -qq \
    ## Install google chrome and chromedriver
    && wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb \
    && apt-get install -qq -y ./google-chrome-stable_current_amd64.deb \
    && wget -O /etc/init.d/xvfb https://gist.githubusercontent.com/axilleas/3fc13e0c90ad9f58bee903a41e8a6d48/raw/169a60010635e05eaa902c5f3b4393321f2452f0/xvfb \
    && chmod 0755 /etc/init.d/xvfb \
    && sh -e /etc/init.d/xvfb start \
    && mkdir -p ${REPO_HOME}/Provar_ANT_${PROVAR_DEFAULT_VERSION} \
    && wget -qP ${REPO_HOME} https://download.provartesting.com/${PROVAR_MAJOR_VERSION}/Provar_ANT_${PROVAR_DEFAULT_VERSION}.zip \
    && unzip ${REPO_HOME}/Provar_ANT_${PROVAR_DEFAULT_VERSION}.zip -d ${REPO_HOME}/Provar_ANT_${PROVAR_DEFAULT_VERSION} \
    && rm -rf ${REPO_HOME}/Provar_ANT_${PROVAR_DEFAULT_VERSION}.zip \
    && ant -version \
    && javac -version \
    ## Set up project payload && Copy script to PATH
    && mkdir -p ${WORKSPACE}/ANT/Results \
    && mkdir -p ${WORKSPACE}/src \
    && mkdir -p ${WORKSPACE}/lib \
    && mkdir -p ${WORKSPACE}/bin 

ENV PROVAR_HOME=${REPO_HOME}/Provar_ANT_${PROVAR_VERSION} \
    CACHEPATH=${WORKSPACE}/../.provarCaches 

## (Optional: Only use for local runs or environments not using a Version Control System) Copy local project files to docker image
## Otherwise files will be pulled from Git/etc. (in that case comment this line) and placed into ${WORKSPACE} where ${WORKSPACE} contains the .testproject file/etc.
COPY . ${WORKSPACE}/

## (Optional: Licenses folder must be in same directory as Dockerfile first)
# Copy Licenses folder to container for execution tracking
# COPY .licenses/ ${PROVAR_HOME}/.licenses 
COPY .smtp/ ${PROVAR_HOME}/.smtp

## Set working directory for image
WORKDIR ${WORKSPACE}
ENTRYPOINT []