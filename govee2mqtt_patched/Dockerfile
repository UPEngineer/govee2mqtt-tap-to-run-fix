ARG BUILD_FROM

# Stage 1: compile govee2mqtt from source with the Tap-to-Run patch applied.
# Pinned to the upstream commit the patch was made against.
FROM rust:1-bookworm AS builder
RUN apt-get update && apt-get install -y --no-install-recommends \
      cmake clang libclang-dev pkg-config git \
    && rm -rf /var/lib/apt/lists/*
WORKDIR /src
RUN git clone https://github.com/wez/govee2mqtt.git . \
    && git checkout 6f5a844
COPY tap-to-run-account-token.patch /tmp/
RUN git apply /tmp/tap-to-run-account-token.patch \
    && git diff --stat
RUN cargo build --release --bin govee

# Stage 2: the same layout as the official add-on image.
FROM $BUILD_FROM
COPY run.sh /run.sh
RUN chmod +x /run.sh
COPY --from=builder /src/target/release/govee /app/govee
COPY --from=builder /src/assets /app/assets/
COPY --from=builder /src/AmazonRootCA1.pem /app/
CMD [ "/run.sh" ]
