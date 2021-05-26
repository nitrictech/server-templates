ARG PROVIDER=local
FROM nitricimages/membrane-${PROVIDER}:latest as membrane
FROM node:alpine as build

# FIXME: Remove once the @nitric/sdk package is made public
# Token should be read-only and revoked once this package is public
ENV YARN_CACHE_FOLDER=/dev/shm/yarn_cache

# Copy and install dependencies before function code
# this avoids reinstalling unchanged dependencies on each code change
COPY ./function/package.json /package.json
COPY ./function/*.lock Dockerfile /
COPY ./function/*-lock.json Dockerfile /

RUN yarn import || echo "lockfile already exists"

RUN yarn install --frozen-lockfile
# Copy sources
COPY /function .
# Build it
RUN yarn build

FROM node:alpine
RUN apk --no-cache add ca-certificates

ENV YARN_CACHE_FOLDER=/dev/shm/yarn_cache
# Add the membrane server binary and plugins
COPY --from=membrane /membrane /usr/local/bin/membrane
RUN chmod +x-rw /usr/local/bin/membrane

COPY --from=build package.json package.json
COPY --from=build .env* *.lock Dockerfile /
RUN yarn install --production --frozen-lockfile

COPY --from=build /next.config.js /next.config.js
COPY --from=build /public/ /public/
COPY --from=build /.next/ /.next/

# Remove the original cache
RUN rm -rf /.next/cache

# Create a symlink for the cache directory
# This is required to allow next to run on a number of restricted container platforms
RUN ln -sf /tmp/ /.next/cache

# Set the mode to HTTP PROXY
ENV MEMBRANE_MODE="HTTP_PROXY"

ENV CHILD_ADDRESS="localhost:3000"

# Expose the gateways proxy port
EXPOSE 9001/tcp
# ENV PROXY_PORT="9001"

# Set yarn cache directory to /tmp/
ENV YARN_CACHE_FOLDER=/tmp/yarn_cache

# AWS Lambda requires the working directory to be explicitly set
WORKDIR /

# Run the Nitric Membrane
# This process will load membrane provider plugins and kick-off the userland process
# full path Entrypoint required for AWS Lambda
ENTRYPOINT ["/usr/local/bin/membrane"]

# Set membrane child process command and args
CMD ["yarn", "start"]
