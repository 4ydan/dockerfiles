# docker build -t my_node .
FROM node:19.4.0-slim

ENV PROJECT_PATH /home/batman/
WORKDIR $PROJECT_PATH
COPY . $PROJECT_PATH

RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    pre-commit && \
    rm -rf /var/lib/apt/lists/*

RUN mkdir /.npm /.cache
RUN chown 47110:47110 -R /.npm /.cache
