FROM node:22-slim AS base

ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"

RUN npm config set registry https://registry.npmmirror.com
RUN corepack enable
RUN pnpm config set registry https://registry.npmmirror.com

COPY . /app
WORKDIR /app

RUN echo ">>>>>>> base ok!"

FROM base AS prod-deps
RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install --prod --frozen-lockfile

RUN echo ">>>>>>> prod-deps ok!"

FROM base AS build
RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install && pnpm run build

RUN echo ">>>>>>> build ok!"

# Install Go
FROM golang:1.24 AS go-base
COPY sharedLibs/go-html-to-md /app/sharedLibs/go-html-to-md

# Install Go dependencies and build parser lib
RUN cd /app/sharedLibs/go-html-to-md && \
    go mod tidy && \
    go build -o html-to-markdown.so -buildmode=c-shared html-to-markdown.go && \
    chmod +x html-to-markdown.so

RUN echo ">>>>>>> go-base ok!"

# Install Rust
FROM rust:1-slim AS rust-base
COPY sharedLibs/html-transformer /app/sharedLibs/html-transformer

# Install Rust dependencies and build transformer lib
RUN cd /app/sharedLibs/html-transformer && \
    cargo build --release && \
    chmod +x target/release/libhtml_transformer.so

RUN echo ">>>>>>> rust-base ok!"

FROM base
COPY --from=build /app/dist /app/dist
COPY --from=prod-deps /app/node_modules /app/node_modules
COPY --from=go-base /app/sharedLibs/go-html-to-md/html-to-markdown.so /app/sharedLibs/go-html-to-md/html-to-markdown.so
COPY --from=rust-base /app/sharedLibs/html-transformer/target/release/libhtml_transformer.so /app/sharedLibs/html-transformer/target/release/libhtml_transformer.so

# Start the server by default, this can be overwritten at runtime
EXPOSE 8080

# Make sure the entrypoint script has the correct line endings
RUN sed -i 's/\r$//' /app/docker-entrypoint.sh

RUN echo ">>>>>>> all ok!"

ENTRYPOINT "/app/docker-entrypoint.sh"
