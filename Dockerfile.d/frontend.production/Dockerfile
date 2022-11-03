FROM ubuntu:22.04 as base

# I've referred to the following Dockerfiles. Thanks!
# - https://github.com/BretFisher/nodejs-rocks-in-docker/blob/7d9fbd6acc90714e7af228a06059d7d3765fc2d1/dockerfiles/ubuntu-deb.Dockerfile
# - https://github.com/vercel/next.js/blob/4af50116cd696b8cb25a40dd81e861373617d33d/examples/with-docker-multi-env/docker/production/Dockerfile

LABEL maintainer="docker@mazgi.com"

# https://nodejs.org/en/blog/release/v18.12.0/
ARG NODE_VERSION=18.12.0
ENV NODE_VERSION=${NODE_VERSION:-18.12.0}
ENV NODE_ENV=production

# Set in non-interactive mode.
ENV DEBIAN_FRONTEND=noninteractive

RUN : Create the user and working directory\
  && groupadd --gid 10000 node\
  && useradd --uid 10000 --gid node --shell /bin/bash --create-home node\
  && mkdir /workspace\
  && chown -R node:node /workspace\
  && :
RUN : Install basic packages\
  && echo 'apt::install-recommends "false";' > /etc/apt/apt.conf.d/no-install-recommends\
  && apt-get update\
  && apt-get install --assume-yes\
  tini\
  ca-certificates\
  #   && :
  # RUN :\
  && : Install Node.js\
  && apt-get install --assume-yes\
  curl\
  && curl -o /tmp/nodejs.deb -fsSL https://deb.nodesource.com/node_18.x/pool/main/n/nodejs/nodejs_${NODE_VERSION}-deb-1nodesource1_amd64.deb\
  && apt-get install --assume-yes /tmp/nodejs.deb\
  && apt-get remove --assume-yes\
  curl\
  && rm -f /tmp/nodejs.deb\
  #   && :
  # RUN :\
  && : Clean up\
  && apt-get clean autoclean\
  && apt-get autoremove --yes\
  && rm -rf /tmp/*\
  && rm -rf /var/lib/apt/\
  && rm -rf /var/log/*\
  && :
ENTRYPOINT ["/usr/bin/tini", "--"]
WORKDIR /workspace
USER node

FROM base AS frontend-builder
COPY --chown=node:node frontend .
RUN :\
  && npm clean-install --only=production\
  && npm cache clean --force\
  && npm run build\
  && :

FROM base AS frontend-runner
COPY --from=frontend-builder --chown=node:node /workspace/public ./public
COPY --from=frontend-builder --chown=node:node /workspace/.next/standalone ./
COPY --from=frontend-builder --chown=node:node /workspace/.next/static ./.next/static
CMD [ "node", "server.js" ]
