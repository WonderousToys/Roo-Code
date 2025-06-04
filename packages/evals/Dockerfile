# docker build -f packages/evals/Dockerfile -t roo-code-evals .
# docker run -it roo-code-evals

FROM node:20-slim AS base

# Install pnpm
ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
RUN corepack enable
RUN npm install -g npm@latest
RUN npm install -g npm-run-all

# Install system packages
RUN apt update && apt install -y sudo curl git vim jq

# Create a `vscode` user
RUN useradd -m vscode -s /bin/bash && \
  echo "vscode ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/vscode && \
  chmod 0440 /etc/sudoers.d/vscode

# Install VS Code
# https://code.visualstudio.com/docs/setup/linux
RUN apt install -y wget gpg apt-transport-https
RUN wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
RUN install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
RUN echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" | tee /etc/apt/sources.list.d/vscode.list > /dev/null
RUN rm -f packages.microsoft.gpg
RUN apt update && apt install -y code

# Install Xvfb
RUN apt install -y xvfb

# [cpp] Install cmake 3.28.3
RUN apt install -y cmake

# [go] Install Go 1.22.2
RUN apt install -y golang-go

# [java] Install Java 21
RUN apt install -y default-jre

# [python] Install Python 3.12.3 and uv 0.6.6
RUN apt install -y python3 python3-venv python3-dev python3-pip

WORKDIR /home/vscode
USER vscode

# [rust] Install Rust 1.85
RUN curl https://sh.rustup.rs -sSf | bash -s -- -y
RUN echo 'source $HOME/.cargo/env' >> $HOME/.bashrc

# Copy evals
RUN git clone https://github.com/RooCodeInc/Roo-Code-Evals.git evals

# Prepare evals
WORKDIR /home/vscode/evals/python
RUN curl -LsSf https://astral.sh/uv/install.sh | sh
RUN /home/vscode/.local/bin/uv sync

WORKDIR /home/vscode/repo

# Install npm packages
RUN mkdir -p \
  scripts \
  apps/web-evals \
  packages/build \
  packages/cloud \
  packages/config-eslint \
  packages/config-typescript \
  packages/evals \
  packages/ipc \
  packages/telemetry \
  packages/types \
  src \
  webview-ui

COPY --chown=vscode:vscode ./package.json                            ./
COPY --chown=vscode:vscode ./pnpm-lock.yaml                          ./
COPY --chown=vscode:vscode ./pnpm-workspace.yaml                     ./
COPY --chown=vscode:vscode ./scripts/bootstrap.mjs                   ./scripts/
COPY --chown=vscode:vscode ./apps/web-evals/package.json             ./apps/web-evals/
COPY --chown=vscode:vscode ./packages/build/package.json             ./packages/build/
COPY --chown=vscode:vscode ./packages/cloud/package.json             ./packages/cloud/
COPY --chown=vscode:vscode ./packages/config-eslint/package.json     ./packages/config-eslint/
COPY --chown=vscode:vscode ./packages/config-typescript/package.json ./packages/config-typescript/
COPY --chown=vscode:vscode ./packages/evals/package.json             ./packages/evals/
COPY --chown=vscode:vscode ./packages/ipc/package.json               ./packages/ipc/
COPY --chown=vscode:vscode ./packages/telemetry/package.json         ./packages/telemetry/
COPY --chown=vscode:vscode ./packages/types/package.json             ./packages/types/
COPY --chown=vscode:vscode ./src/package.json                        ./src/
COPY --chown=vscode:vscode ./webview-ui/package.json                 ./webview-ui/
RUN pnpm install

# Build the extension
COPY --chown=vscode:vscode . ./
RUN pnpm vsix -- --out ../bin/roo-code.vsix

# Copy & install extension
RUN code --debug --install-extension ./roo-code.vsix

# Copy env variables
# COPY --chown=vscode:vscode packages/evals/.env ./

# Push database schema
# RUN pnpm --filter @roo-code/evals db:push --force

# EXPOSE 3000
# CMD ["pnpm", "web"]
CMD ["bash"]
