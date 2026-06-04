# Zed — FreeBSD fork

Unofficial fork of [Zed](https://github.com/zed-industries/zed) that adds **FreeBSD remote-server support** — edit on your desktop, run the code on a FreeBSD host over SSH. For Zed itself, upstream is the source of truth: [upstream README](https://github.com/zed-industries/zed/blob/main/README.md).

**We track upstream releases.** Our changes are a small patch set applied on top of an upstream release tag — you pick a tag, apply, build. Latest tags: <https://github.com/zed-industries/zed/tags> (examples below use `v1.5.3`).

**Tested:** Windows client ↔ FreeBSD server. macOS/Linux clients and a native FreeBSD GUI compile but are untested.

## How it works

Two pieces, both built from the **same upstream tag**:

- a **GUI client** on your desktop (Windows / macOS / Linux) — build the chapter for your OS
- **`zed-remote-server`** on the FreeBSD host — the FreeBSD chapter

Every chapter is the same three steps: clone → apply patches onto a tag → build.

---

## Windows (client) — tested

Requirements:

- [Rust (rustup)](https://www.rust-lang.org/tools/install)
- [Git for Windows](https://git-scm.com/download/win) — **required**; run everything in its **Git Bash** (the build scripts are bash)
- Visual Studio 2022 or Build Tools, with the **Desktop development with C++** workload
- **Windows 10/11 SDK** (≥ `10.0.20348.0`)
- [CMake](https://cmake.org/download/)

In **Git Bash**:

```sh
git clone https://github.com/strain08/zed.git
cd zed
./script/freebsd-patches apply v1.5.3    # ← latest upstream tag
cd ../zed-freebsd-build/v1.5.3
cargo run --release
```

If `cargo` can't find the MSVC linker, start Git Bash from a **Developer Command Prompt for VS 2022** so the toolchain is on `PATH`.

## macOS (client) — untested

Requirements:

- [Rust (rustup)](https://www.rust-lang.org/tools/install)
- Xcode + command line tools: `xcode-select --install`
- CMake: `brew install cmake`

```sh
git clone https://github.com/strain08/zed.git
cd zed
script/freebsd-patches apply v1.5.3      # ← latest upstream tag
cd ../zed-freebsd-build/v1.5.3
cargo run --release
```

## Linux (client) — untested

Requirements:

- [Rust (rustup)](https://www.rust-lang.org/tools/install)
- System packages — installed by `script/linux` below

```sh
git clone https://github.com/strain08/zed.git
cd zed
script/freebsd-patches apply v1.5.3      # ← latest upstream tag
cd ../zed-freebsd-build/v1.5.3
script/linux                             # installs system deps
cargo run --release
```

## FreeBSD (remote server)

Requirements:

- `git`
- Rust + system packages — installed by `script/freebsd` below

```sh
git clone https://github.com/strain08/zed.git
cd zed
script/freebsd-patches apply v1.5.3      # ← latest upstream tag
cd ../zed-freebsd-build/v1.5.3
script/freebsd                           # installs deps + rustup
cargo build --release -p remote_server   # → target/release/remote_server
```

## Connect client → FreeBSD host

Zed has no prebuilt FreeBSD server to download, so point it at the `remote_server` binary you just built. Easiest — let the client name and upload it:

- set `ZED_COPY_REMOTE_SERVER=/path/to/target/release/remote_server` before launching the client (needs a debug client or the `build-remote-server-binary` feature).

Or place it on the host by hand — the client looks for a versioned name in `~/.zed_server/`:

```sh
# on the FreeBSD host; <channel> is stable|preview|nightly, <version> the client's version
cp target/release/remote_server ~/.zed_server/zed-remote-server-<channel>-<version>
```

Then in the client: **Remote Projects → connect over SSH** to `user@freebsd-host` and open a folder.

> Client and server must be built from the **same upstream tag**.

## Limitations

- Only **Windows client ↔ FreeBSD server** is tested; everything else compiles but is unverified.
- **No prebuilt binaries** — build from source.
- **No collab / calls / screen-share** on FreeBSD (`webrtc-sys` doesn't build there).
- Crash reporting is **stubbed** on FreeBSD (no minidumps).
- File watching uses **polling**, not native kqueue.
- Extension/wasm platform matching uses a **temporary hack** (`freebsd/patches/0003`).

## TODO

- [ ] Upstream the clean FreeBSD patches
- [ ] Fix the wasm-platform-match hack, then upstream it
- [ ] Publish a prebuilt `zed-remote-server` / FreeBSD `pkg`
- [ ] Native kqueue file-watching instead of polling

---

Maintainer notes (how the patch series is kept in sync with upstream): [`freebsd/README.md`](freebsd/README.md).
