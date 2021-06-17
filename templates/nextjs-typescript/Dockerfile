ARG PROVIDER=local
FROM nitricimages/membrane-${PROVIDER}:latest as membrane
FROM node:14-alpine as build

# Copy and install dependencies before function code
# this avoids reinstalling unchanged dependencies on each code change
COPY package.json *.lock *-lock.json /app/

# AWS Lambda requires the working directory to be explicitly set
# Don't remove this without replacing it
WORKDIR /app

RUN yarn import || echo "lockfile already exists"

# Install dependencies and delete the cache
RUN set -ex; yarn install --frozen-lockfile --cache-folder /tmp/.cache; rm -rf /tmp/.cache;
# Copy sources
COPY . .
# Build it
RUN yarn build

# Remove the original cache
RUN rm -rf ./.next/cache

# Create a symlink for the cache directory
# This is required to allow next to run on a number of restricted container platforms
RUN ln -sf /tmp/ ./.next/cache

# Set the mode to HTTP PROXY
ENV MEMBRANE_MODE="HTTP_PROXY"

ENV CHILD_ADDRESS="localhost:3000"

# Expose the gateways proxy port
EXPOSE 9001/tcp
# ENV PROXY_PORT="9001"

# Set yarn cache directory to /tmp/
ENV YARN_CACHE_FOLDER=/tmp/yarn_cache

COPY --from=membrane /membrane /usr/local/bin/membrane
RUN chmod +x-rw /usr/local/bin/membrane

# Run the Nitric Membrane
# This process will load membrane provider plugins and kick-off the userland process
# full path Entrypoint required for AWS Lambda
ENTRYPOINT ["/usr/local/bin/membrane"]

# Set membrane child process command and args
CMD ["yarn", "start"]
