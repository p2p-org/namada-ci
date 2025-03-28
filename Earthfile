VERSION 0.8

wasmd:
  FROM cosmwasm/wasmd:v0.52.0
  SAVE ARTIFACT /usr/bin/wasmd AS LOCAL wasmd

namada:
  FROM ubuntu:24.04

  ENV DEBIAN_FRONTEND=noninteractive

  WORKDIR /__w/namada/namada

  ARG toolchain=1.85.1
  ARG nightly_toolchain=nightly-2025-03-27
  ARG rocksdb_version=8.10.0
  ARG gaia_version=19.1.0
  ARG cw721_version=0.18.0
  ARG ics721_version=0.1.13
  ARG cometbft_version=0.37.11
  ARG wasm_opt_version=119
  ARG mold_version=2.37.1

  RUN apt-get update -y
  RUN apt-get install -y curl
  RUN apt-get install -y protobuf-compiler 
  RUN apt-get install -y build-essential 
  RUN apt-get install -y clang-tools clang
  RUN apt-get install -y libudev-dev
  RUN apt-get install -y libssl-dev
  RUN apt-get install -y pkg-config
  RUN apt-get install -y gcc
  RUN apt-get install -y parallel
  RUN apt-get install -y python3
  RUN apt-get install -y ca-certificates
  RUN apt-get install -y unzip

  # needed for speculos
  RUN apt-get install -y git
  RUN apt-get install -y build-essential
  RUN apt-get install -y ca-certificates
  RUN apt-get install -y git
  RUN apt-get install -y python3-pip
  RUN apt-get install -y pipx
  RUN apt-get install -y cmake
  RUN apt-get install -y gcc-arm-linux-gnueabihf
  RUN apt-get install -y libc6-dev-armhf-cross
  RUN apt-get install -y gdb-multiarch
  RUN apt-get install -y python3-pyqt5
  RUN apt-get install -y python3-construct
  RUN apt-get install -y python3-flask-restful
  RUN apt-get install -y python3-jsonschema
  RUN apt-get install -y python3-mnemonic
  RUN apt-get install -y python3-pil
  RUN apt-get install -y python3-pyelftools
  RUN apt-get install -y python3-requests
  RUN apt-get install -y qemu-user-static
  RUN apt-get install -y libvncserver-dev
  RUN apt-get install -y make
  RUN apt-get install -y qtbase5-dev qtchooser qt5-qmake qttools5-dev-tools

  RUN pipx ensurepath
  RUN pipx install speculos

  # install rust 
  RUN curl https://sh.rustup.rs -sSf | sh -s -- -y

  ENV PATH="/root/.cargo/bin:/root/.local/bin:$PATH"
  ENV RUSTUP_HOME="/root/.rustup"
  ENV CARGO_HOME="/root/.cargo"
    
  RUN rustup toolchain install $toolchain-x86_64-unknown-linux-gnu --no-self-update --component clippy,rustfmt,rust-analysis,rust-docs,rust-src,llvm-tools-preview
  RUN rustup target add --toolchain $toolchain-x86_64-unknown-linux-gnu wasm32-unknown-unknown
  RUN rustup toolchain install $nightly_toolchain-x86_64-unknown-linux-gnu --no-self-update --component clippy,rustfmt,rust-analysis,rust-docs,rust-src,llvm-tools-preview,rustc-codegen-cranelift-preview
  RUN rustup target add --toolchain $nightly_toolchain-x86_64-unknown-linux-gnu wasm32-unknown-unknown
  RUN rustup default $toolchain-x86_64-unknown-linux-gnu

  # download masp artifacts
  RUN mkdir -p /masp/.masp-params
  RUN curl -o /masp/.masp-params/masp-spend.params -L https://github.com/anoma/masp-mpc/releases/download/namada-trusted-setup/masp-spend.params\?raw\=true
  RUN curl -o /masp/.masp-params/masp-output.params -L https://github.com/anoma/masp-mpc/releases/download/namada-trusted-setup/masp-output.params?raw=true
  RUN curl -o /masp/.masp-params/masp-convert.params -L https://github.com/anoma/masp-mpc/releases/download/namada-trusted-setup/masp-convert.params?raw=true

  # install cargo binstall 
  RUN curl -L --proto '=https' --tlsv1.2 -sSf https://raw.githubusercontent.com/cargo-bins/cargo-binstall/main/install-from-binstall-release.sh | bash

  # install cargo nextest
  RUN cargo binstall cargo-nextest --no-confirm

  # install sccache
  RUN cargo binstall sccache --no-confirm

  # install cargo cache
  RUN cargo binstall cargo-cache --no-confirm

  # install cargo about
  RUN cargo binstall cargo-about --no-confirm

  # install llvm-cov
  RUN cargo binstall cargo-llvm-cov --no-confirm
  RUN cargo +$nightly_toolchain install cargo-llvm-cov --locked

  # install rustfilt (for fuzzing)
  RUN cargo install rustfilt

  # install fuzz
  RUN cargo install cargo-fuzz

  # download mold
  RUN curl -o mold.tar.gz -LO https://github.com/rui314/mold/releases/download/v$mold_version/mold-$mold_version-x86_64-linux.tar.gz
  RUN tar --strip-components 2 -xvzf mold.tar.gz mold-$mold_version-x86_64-linux/bin/mold
  RUN mv mold /usr/local/bin
  RUN chmod +x /usr/local/bin/mold

  # download gaia 
  RUN curl -o gaiad -LO https://github.com/cosmos/gaia/releases/download/v${gaia_version}/gaiad-v${gaia_version}-linux-amd64
  RUN mv gaiad /usr/local/bin
  RUN chmod +x /usr/local/bin/gaiad

  # install wasmd
  COPY +wasmd/wasmd /usr/local/bin/wasmd
  RUN chmod +x /usr/local/bin/wasmd

  # download cosmwasm contracts
  RUN mkdir -p /cosmwasm_contracts
  RUN curl -L -o /cosmwasm_contracts/cw721_base.wasm https://github.com/public-awesome/cw-nfts/releases/download/v${cw721_version}/cw721_base.wasm
  RUN curl -L -o /cosmwasm_contracts/ics721_base.wasm https://github.com/public-awesome/cw-ics721/releases/download/v${ics721_version}/ics721_base.wasm

  # download cometbft
  RUN curl -o cometbft.tar.gz -LO https://github.com/cometbft/cometbft/releases/download/v${cometbft_version}/cometbft_${cometbft_version}_linux_amd64.tar.gz
  RUN tar -xvzf cometbft.tar.gz
  RUN mv cometbft /usr/local/bin
  RUN chmod +x /usr/local/bin/cometbft

  # download wasm-opt
  RUN curl -o binaryen.tar.gz -LO https://github.com/WebAssembly/binaryen/releases/download/version_${wasm_opt_version}/binaryen-version_${wasm_opt_version}-x86_64-linux.tar.gz
  RUN tar --strip-components 2 -xvzf binaryen.tar.gz binaryen-version_${wasm_opt_version}/bin/wasm-opt
  RUN mv wasm-opt /usr/local/bin
  RUN chmod +x /usr/local/bin/wasm-opt

  RUN mkdir -p /ledger-namada
  COPY artifacts/app_s2.elf /ledger-namada/app_s2.elf

  RUN rm -rf /var/lib/apt/lists/*

  SAVE IMAGE --push ghcr.io/heliaxdev/namada-ci:namada-latest ghcr.io/heliaxdev/namada-ci:namada-main

wasm:
  FROM rust:1.85.1-bookworm

  ARG toolchain=1.85.1
  ARG wasm_opt_version=118

  WORKDIR /__w/namada/namada

  RUN apt-get update -y
  RUN apt-get install -y protobuf-compiler 
  RUN apt-get install -y parallel

  RUN rustup toolchain install $toolchain --profile minimal --no-self-update
  RUN rustup target add wasm32-unknown-unknown
  RUN rustup default $toolchain-x86_64-unknown-linux-gnu

  # install cargo binstall 
  RUN curl -L --proto '=https' --tlsv1.2 -sSf https://raw.githubusercontent.com/cargo-bins/cargo-binstall/main/install-from-binstall-release.sh | bash

  # install cargo cache
  RUN cargo binstall cargo-cache --no-confirm

  # download wasm-opt
  RUN curl -o binaryen.tar.gz -LO https://github.com/WebAssembly/binaryen/releases/download/version_${wasm_opt_version}/binaryen-version_${wasm_opt_version}-x86_64-linux.tar.gz
  RUN tar --strip-components 2 -xvzf binaryen.tar.gz binaryen-version_${wasm_opt_version}/bin/wasm-opt
  RUN mv wasm-opt /usr/local/bin
  RUN chmod +x /usr/local/bin/wasm-opt

  SAVE IMAGE --push ghcr.io/heliaxdev/namada-ci:wasm-latest ghcr.io/heliaxdev/namada-ci:wasm-main
