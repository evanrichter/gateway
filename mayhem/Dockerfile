# Build Stage
FROM ghcr.io/evanrichter/cargo-fuzz:latest as builder

## Add source code to the build stage.
ADD . /src
WORKDIR /src

# remove these two lines when nalgebra builds on nightly again
# also change the later "+stable" to "+nightly"
RUN rustup default stable
ENV RUSTC_BOOTSTRAP=1

RUN echo building instrumented harnesses && \
    bash -c "pushd pallets/cash/fuzz && cargo +stable fuzz build && popd" && \
    bash -c "pushd pallets/oracle/fuzz && cargo +stable fuzz build && popd" && \
    mv pallets/cash/fuzz/target/x86_64-unknown-linux-gnu/release/chain_account_from_str_fuzz /chain_account_from_str_fuzz && \
    mv pallets/oracle/fuzz/target/x86_64-unknown-linux-gnu/release/parse_message_fuzz /parse_message_fuzz && \
    echo done

RUN echo building non-instrumented harnesses && \
    export RUSTFLAGS="--cfg fuzzing -Clink-dead-code -Cdebug-assertions -C codegen-units=1" && \
    bash -c "pushd pallets/cash/fuzz && cargo +stable build --release && popd" && \
    bash -c "pushd pallets/oracle/fuzz && cargo +stable build --release && popd" && \
    mv pallets/cash/fuzz/target/release/chain_account_from_str_fuzz /chain_account_from_str_fuzz_no_inst && \
    mv pallets/oracle/fuzz/target/release/parse_message_fuzz /parse_message_fuzz_no_inst && \
    echo done

# Package Stage
FROM rustlang/rust:nightly

COPY --from=builder /chain_account_from_str_fuzz /chain_account_from_str_fuzz_no_inst /
COPY --from=builder /parse_message_fuzz /parse_message_fuzz_no_inst /
