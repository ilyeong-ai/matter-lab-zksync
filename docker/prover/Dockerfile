# syntax=docker/dockerfile:experimental
FROM rust:1.55 as builder
WORKDIR /usr/src/zksync
COPY . .
RUN --mount=type=cache,target=/usr/local/cargo/registry \
    cargo build --release

FROM debian:bullseye
RUN apt-get update && apt-get install -y axel ca-certificates tini && rm -rf /var/lib/apt/lists/*
# Docs of env variables used for this image
# to test this locally just run
# docker build -t tmp_prover -f =(f envsubst < ./docker/prover/Dockerfile) .; docker run --rm  tmp_prover:latest
#ENV CHAIN_CIRCUIT_SUPPORTED_BLOCK_CHUNKS_SIZES $CHAIN_CIRCUIT_SUPPORTED_BLOCK_CHUNKS_SIZES
#ENV CHAIN_CIRCUIT_SUPPORTED_BLOCK_CHUNKS_SIZES_SETUP_POWERS $CHAIN_CIRCUIT_SUPPORTED_BLOCK_CHUNKS_SIZES_SETUP_POWERS
#ENV CHAIN_STATE_KEEPER_BLOCK_CHUNK_SIZES $CHAIN_STATE_KEEPER_BLOCK_CHUNK_SIZES
#ENV CHAIN_CIRCUIT_KEY_DIR $CHAIN_CIRCUIT_KEY_DIR
#ENV CHAIN_CIRCUIT_ACCOUNT_TREE_DEPTH $CHAIN_CIRCUIT_ACCOUNT_TREE_DEPTH
#ENV CHAIN_CIRCUIT_BALANCE_TREE_DEPTH $CHAIN_CIRCUIT_BALANCE_TREE_DEPTH
#ENV API_PROVER_URL $API_PROVER_URL
#ENV PROVER_PROVER_REQUEST_TIMEOUT $PROVER_PROVER_REQUEST_TIMEOUT
#ENV RUST_BACKTRACE $RUST_BACKTRACE
#ENV RUST_LOG $RUST_LOG
#ENV MISC_PROVER_DOWNLOAD_SETUP $MISC_PROVER_DOWNLOAD_SETUP
#ENV MISC_PROVER_SETUP_NETWORK_DIR $MISC_PROVER_SETUP_NETWORK_DIR
#ENV PROVER_WITNESS_GENERATOR_PREPARE_DATA_INTERVAL $PROVER_WITNESS_GENERATOR_PREPARE_DATA_INTERVAL
#ENV PROVER_PROVER_HEARTBEAT_INTERVAL $PROVER_PROVER_HEARTBEAT_INTERVAL
#ENV PROVER_PROVER_CYCLE_WAIT $PROVER_PROVER_CYCLE_WAIT
#ENV PROVER_CORE_GONE_TIMEOUT $PROVER_CORE_GONE_TIMEOUT
#ENV MISC_DOCKER_DUMMY_PROVER $MISC_DOCKER_DUMMY_PROVER
COPY --from=builder /usr/src/zksync/target/release/plonk_step_by_step_prover /bin/
COPY --from=builder /usr/src/zksync/target/release/dummy_prover /bin/
COPY docker/prover/prover-entry.sh /bin/
COPY keys/packed /keys/packed
COPY contracts/artifacts/ /contracts/artifacts/

## Mitigate memory leaks when app exists it's restarted
ENTRYPOINT [ "/usr/bin/tini", "-g", "--" ]
CMD while :; do /bin/prover-entry.sh || exit $?; done
