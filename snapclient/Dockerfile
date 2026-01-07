FROM debian:bookworm-slim

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    snapclient \
    libasound2-plugins \
    alsa-utils \
    && rm -rf /var/lib/apt/lists/*

ENTRYPOINT []
CMD ["snapclient"]
