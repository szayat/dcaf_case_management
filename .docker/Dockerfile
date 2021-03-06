FROM ruby:2.6.5-slim-buster
MAINTAINER Colin Fleming <c3flemin@gmail.com>

# configure environment variable
# note: move this to three ARG commands when CircleCI updates their docker
ENV DCAF_DIR=/usr/src/app \
    BUILD_DEPENDENCIES="build-essential libxml2-dev gnupg2 libxslt-dev fontconfig" \
    APP_DEPENDENCIES="nodejs yarn git sudo sassc" \
    AHAB_DEPENDENCIES="ca-certificates curl" \
    FONTCONFIG_PATH=/etc/fonts \
    NODE_ENV=development \
    DOCKER=true

# get our gem house in order
RUN mkdir -p ${DCAF_DIR} && cd ${DCAF_DIR}
WORKDIR ${DCAF_DIR}
COPY Gemfile ${DCAF_DIR}/Gemfile
COPY Gemfile.lock ${DCAF_DIR}/Gemfile.lock
COPY package.json ${DCAF_DIR}/package.json
COPY yarn.lock ${DCAF_DIR}/yarn.lock

# install packages
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y \
    ${BUILD_DEPENDENCIES} \
    ${APP_DEPENDENCIES} \
    ${AHAB_DEPENDENCIES} && \
    gem install bundler --no-document

# Install yarn using debian, avoid having to install npm at all
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -

RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list

RUN apt-get update && apt-get install yarn -y

# install gemfile and package
RUN bundle install
RUN yarn install

# throw errors if Gemfile has been modified since Gemfile.lock
RUN bundle config --global frozen 1

# remove unnecessary build dependencies
RUN apt-get purge -y ${BUILD_DEPENDENCIES} && apt-get autoremove -y 

# Check docker base image for vulnerable packages, ignore non zero exit code (just informative)
RUN mkdir /tmp/ahab && \
    cd /tmp/ahab && \
    curl -o ahab -O -L https://github.com/sonatype-nexus-community/ahab/releases/download/v0.0.8/ahab-linux.amd64-v0.0.8 && \ 
    chmod +x ahab && \
    update-ca-certificates && \
    dpkg-query --show --showformat='${Package} ${Version}\n' | ./ahab chase || true && \
    rm ahab && \
    cd -

# remove unnecessary ahab dependencies
RUN apt-get purge -y ${AHAB_DEPENDENCIES} && apt-get autoremove -y 

# Move the rest of the app over
COPY . ${DCAF_DIR}

EXPOSE 3000
