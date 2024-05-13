
# zarf-injector

A tiny (<1MiB) binary statically-linked with musl in order to fit as a configmap

## Building on Ubuntu

```bash
# install rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --no-modify-path
source $HOME/.cargo/env

# install build-essential
sudo apt install build-essential -y

# build w/ musl
rustup target add x86_64-unknown-linux-musl
cargo build --target x86_64-unknown-linux-musl --release
```

## Building on Apple Silicon 

* Install Cross
* Install Docker & have it running
* Rust must be installed via Rustup (Check `which rustc` if you're unsure)

```
cargo install cross --git https://github.com/cross-rs/cross
```

Whichever arch. of `musl` used, add to toolchain
```
rustup toolchain install --force-non-host stable-x86_64-unknown-linux-musl
```
```
cross build --target x86_64-unknown-linux-musl --release

cross build --target aarch64-unknown-linux-musl --release
```

This will build into `target/*--unknown-linux-musl/release`



## Checking Binary Size

Due to the ConfigMap size limit (1MiB for binary data), we need to make sure the binary is small enough to fit.

```bash
cargo build --target x86_64-unknown-linux-musl --release

cargo build --target aarch64-unknown-linux-musl --release

size_linux=$(du --si target/x86_64-unknown-linux-musl/release/zarf-injector | cut -f1)
echo "Linux binary size: $size_linux"
size_aarch64=$(du --si target/aarch64-unknown-linux-musl/release/zarf-injector | cut -f1)
echo "aarch64 binary size: $size_aarch64"
```

## Testing your injector

Build your injector by following the steps above, or running one of the following:
```
make build-injector-linux

## OR 
## works on apple silicon 
make cross-injector-linux 

```

Point the [zarf-registry/zarf.yaml](../../packages/zarf-registry/zarf.yaml) to
the locally built injector image.

```
    files:
      # Rust Injector Binary
      - source: ../../src/injector/target/x86_64-unknown-linux-musl/release/zarf-injector
        target: "###ZARF_TEMP###/zarf-injector"
        <!-- shasum: "###ZARF_PKG_TMPL_INJECTOR_AMD64_SHASUM###" -->
        executable: true

    files:
      # Rust Injector Binary
      - source: ../../src/injector/target/aarch64-unknown-linux-musl/release/zarf-injector
        target: "###ZARF_TEMP###/zarf-injector"
        <!-- shasum: "###ZARF_PKG_TMPL_INJECTOR_ARM64_SHASUM###" -->
        executable: true
```

In Zarf Root Directory, run:
```
zarf tools clear-cache
make clean
make && make init-package
```

If you are running on an Apple Silicon, add the `ARCH` flag:  `make init-package ARCH=arm64`

This builds all artifacts within the `/build` directory. Running `zarf init` would look like:
`.build/zarf-mac-apple init --components git-server -l trace`
