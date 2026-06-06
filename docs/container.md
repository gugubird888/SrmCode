# 容器优先的 Claw Code 工作流

Rust 运行时本身已具备**容器检测**能力：

- `rust/crates/runtime/src/sandbox.rs` 检测 Docker/Podman/容器标记（`/.dockerenv`、`/run/.containerenv`、cgroup 提示）。
- `rust/crates/rusty-claude-cli/src/main.rs` 通过 `claw sandbox` 暴露该状态。
- 本文档添加一个小的 `Containerfile`，为 Docker 和 Podman 用户提供规范的容器工作流。

## 容器镜像的用途

根目录的 `Containerfile` 提供一个可复用的 Rust 构建/测试 shell，包含此工作区常用软件包（`git`、`pkg-config`、`libssl-dev`、证书）。

它**不**将仓库复制到镜像中。推荐流程是将你的 checkout bind-mount 到 `/workspace`，使编辑保留在宿主机上。

## 构建镜像

从仓库根目录：

### Docker

```bash
docker build -t claw-code-dev -f Containerfile .
```

### Podman

```bash
podman build -t claw-code-dev -f Containerfile .
```

## 在容器中运行 `cargo test --workspace`

### Docker

```bash
docker run --rm -it \
  -v "$PWD":/workspace \
  -e CARGO_TARGET_DIR=/tmp/claw-target \
  -w /workspace/rust \
  claw-code-dev \
  cargo test --workspace
```

### Podman

```bash
podman run --rm -it \
  -v "$PWD":/workspace:Z \
  -e CARGO_TARGET_DIR=/tmp/claw-target \
  -w /workspace/rust \
  claw-code-dev \
  cargo test --workspace
```

## 在容器中打开 shell

### Docker

```bash
docker run --rm -it \
  -v "$PWD":/workspace \
  -e CARGO_TARGET_DIR=/tmp/claw-target \
  -w /workspace/rust \
  claw-code-dev
```

### Podman

```bash
podman run --rm -it \
  -v "$PWD":/workspace:Z \
  -e CARGO_TARGET_DIR=/tmp/claw-target \
  -w /workspace/rust \
  claw-code-dev
```

在 shell 内：

```bash
cargo build --workspace
cargo test --workspace
cargo run -p rusty-claude-cli -- --help
cargo run -p rusty-claude-cli -- sandbox
```

## 同时 bind-mount 本仓库和另一个仓库

```bash
docker run --rm -it \
  -v "$PWD":/workspace \
  -v "$HOME/src/other-repo":/repo \
  -e CARGO_TARGET_DIR=/tmp/claw-target \
  -w /workspace/rust \
  claw-code-dev
```

然后：

```bash
cargo run -p rusty-claude-cli -- prompt "summarize /repo"
```

## 注意事项

- Docker 和 Podman 使用相同的 `Containerfile`。
- Podman 示例中的 `:Z` 后缀用于 SELinux 重新标记；在 Fedora/RHEL 类主机上保留。
- 使用 `CARGO_TARGET_DIR=/tmp/claw-target` 可避免在 bind-mount 的 checkout 中留下容器拥有的 `target/` 制品。
- 非容器本地开发继续使用 [`USAGE.md`](../USAGE.md) 和 [`rust/README.md`](../rust/README.md)。
