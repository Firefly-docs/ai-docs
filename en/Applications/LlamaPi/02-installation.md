# Installation Guide

This is an installation guide paragraph.

## Dependency Installation

```bash
sudo apt update && sudo apt install -y \
    test-package-a \
    test-package-b \
    test-package-c
```

## Build from Source

```bash
git clone https://this-is-a.git-address.com/test-repo
cd test-repo
mkdir build && cd build
cmake .. -DTEST_FLAG=Release
make -j$(nproc)
```

## Package Manager

```bash
pip install test-package
```

## Verify Installation

```bash
test-command --version
```

Expected output:
```
Test Tool version v9.9.9
```

## Uninstall

```bash
sudo apt remove test-package
# or
pip uninstall test-package
```
