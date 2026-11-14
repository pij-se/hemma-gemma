# Stage 1; builder
FROM docker.io/library/alpine:latest AS builder

# Install necessary build tools (cmake, git, etc.)
RUN apk upgrade --no-cache && apk add --no-cache git cmake build-base linux-headers

WORKDIR /src

# Download the llama.cpp repository
RUN git clone https://github.com/ggerganov/llama.cpp.git

WORKDIR /src/llama.cpp

# Configure and compile llama-server
RUN cmake -B build -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DLLAMA_CURL=OFF -DGGML_CCACHE=OFF && cmake --build build --target llama-server -j$(nproc)

# Stage 2; runtime
FROM builder AS runtime

# Install necessary runtime packages
RUN apk upgrade --no-cache && apk add --no-cache libstdc++ libgcc libgomp

# Copy llama-server
COPY --from=builder /src/llama.cpp/build/bin/llama-server /usr/local/bin/llama-server

WORKDIR /app/models

# The command to run when the container starts
ENTRYPOINT ["llama-server", "--threads", "4", "--threads-batch", "4", "--host", "0.0.0.0", "--port", "8080", "--no-webui", "--cache-prompt", "--reasoning", "off", "--ctx-size", "8192", "--model", "/app/models/model.gguf"]

# Expose the port
EXPOSE 8080
