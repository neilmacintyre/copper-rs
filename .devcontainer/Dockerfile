FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Etc/UTC

# Install necessary system apt libs (aligned with support/docker/Dockerfile.ubuntu)
RUN apt-get update && apt-get install -y --no-install-recommends \
    tzdata \
    git \
    curl \
    python3 \
    python3-pip \
    build-essential \
    pkg-config \
    libudev-dev \
    libpcap-dev \
    libglib2.0-dev \
    ca-certificates \
    libssl-dev \
    clang \
    libclang-dev \
    sudo \
    ripgrep \
    libgstreamer1.0-dev \
    libgstreamer-plugins-base1.0-dev \
    libgstreamer-plugins-bad1.0-dev \
    gstreamer1.0-plugins-base \
    gstreamer1.0-plugins-good \
    gstreamer1.0-plugins-bad \
    gstreamer1.0-plugins-ugly \
    gstreamer1.0-libav \
    gstreamer1.0-tools \
    gstreamer1.0-x \
    gstreamer1.0-alsa \
    gstreamer1.0-gl \
    gstreamer1.0-gtk3 \
    gstreamer1.0-qt5 \
    gstreamer1.0-pulseaudio \
    && ln -fs /usr/share/zoneinfo/$TZ /etc/localtime \
    && dpkg-reconfigure -f noninteractive tzdata \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Shared Rust install (canonical devcontainers pattern; avoids /root/.cargo permission issues)
ENV CARGO_HOME=/usr/local/cargo
ENV RUSTUP_HOME=/usr/local/rustup
ENV PATH="/usr/local/cargo/bin:${PATH}"

# Ensure clean dirs (avoids rustup "existing settings" warnings from partial/cached layers)
RUN rm -rf "${CARGO_HOME}" "${RUSTUP_HOME}" \
    && mkdir -p "${CARGO_HOME}" "${RUSTUP_HOME}" \
    && curl https://sh.rustup.rs -sSf | bash -s -- -y --no-modify-path

# Toolchains, components, and embedded targets (aligned with Dockerfile.ubuntu) + rust-src for rust-analyzer
RUN rustup toolchain install stable beta nightly \
    && rustup component add rustfmt clippy rust-src --toolchain stable \
    && rustup component add rustfmt clippy --toolchain beta \
    && rustup target add aarch64-unknown-none thumbv7em-none-eabihf thumbv8m.main-none-eabihf riscv32imac-unknown-none-elf --toolchain stable \
    && rustup target add aarch64-unknown-none thumbv7em-none-eabihf thumbv8m.main-none-eabihf riscv32imac-unknown-none-elf --toolchain beta

# Install nextest
RUN curl -LsSf https://get.nexte.st/latest/linux | tar zxf - -C /usr/local/cargo/bin

# Cargo-installed tools (Dockerfile.ubuntu + prek for just fmt)
RUN cargo install cargo-generate \
    && cargo install --locked taplo-cli \
    && cargo install fmtron \
    && cargo install cargo-deny \
    && cargo install typos-cli \
    && cargo install sccache \
    && cargo install --locked prek \
    && cargo install --locked cargo-public-api --version 0.51.0

# just (prebuilt; faster than cargo install)
RUN curl --proto '=https' --tlsv1.2 -sSf https://just.systems/install.sh | bash -s -- --to /usr/local/bin

# MkDocs tooling used in docs workflow
RUN python3 -m pip install --no-cache-dir --upgrade pip mkdocs mkdocs-material

# Group-writable Rust dirs for non-root dev user (see devcontainers/discussions/13)
RUN groupadd -r rustlang \
    && chown -R root:rustlang /usr/local/cargo /usr/local/rustup \
    && chmod g+r+w+s /usr/local/cargo /usr/local/rustup \
    && chmod -R g+r+w /usr/local/cargo /usr/local/rustup

ARG USERNAME=copper
ARG USER_UID=1000
ARG USER_GID=${USER_UID}

RUN groupadd --gid ${USER_GID} ${USERNAME} \
    && useradd --uid ${USER_UID} --gid ${USER_GID} -m -s /bin/bash ${USERNAME} \
    && usermod -aG rustlang ${USERNAME} \
    && echo "${USERNAME} ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/${USERNAME} \
    && chmod 0440 /etc/sudoers.d/${USERNAME}

ENV CARGO_TERM_COLOR=always
# Mirrors BASE_FEATURES from repo justfile
ENV FEATURES_FLAG="--features mock,cu-sensor-payloads/image,kornia,gst,faer,nalgebra,glam,debug_pane,bincode,log-level-debug"

USER ${USERNAME}
WORKDIR /workspaces/copper-rs

CMD ["/bin/bash"]
