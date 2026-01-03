# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Test Commands

### Build System
The project uses Autotools. Essential steps:
```bash
./autogen.sh
./configure [options]
make
```

### Build Options
- **Enable OpenSSL**: `./configure --enable-openssl` (accelerates AES)
- **Enable ZSTD**: `./configure --enable-zstd` (adds ZSTD compression)
- **Enable Pthread**: `./configure --enable-pthread`
- **Optimization**: `./configure CFLAGS="-O3 -march=native"`

### Testing
- **Run all tests**: `make test`
- **Unit tests**: `make test.units` or run individual binaries: `./tools/tests-auth`, `./tools/tests-wire`, `./tools/tests-hashing`, `./tools/tests-transform`, `./tools/tests-compress`, `./tools/tests-elliptic`.
- **Integration tests**: `make test.integration`

### Linting and Formatting
- **All checks**: `make lint`
- **Format C code**: `make lint.ccode` (uses `scripts/indent.sh` via `uncrustify`)
- **Lint shell scripts**: `make lint.shell`

## High-Level Architecture

### Core Components
- **Supernode** (`supernode/`, `src/supernode.c`): Hub for discovery, NAT traversal (UDP hole punching), and packet relaying when P2P is not possible.
- **Edge** (`edge/`, `src/edge.c`): The P2P node. It creates a virtual TUN/TAP interface, encapsulates L2 frames into UDP, encrypts/compresses payload, and handles peer registration.
- **Tuntap** (`src/tuntap_*.c`): OS-specific implementation of the virtual network interface (Linux, Windows, macOS, FreeBSD, NetBSD).
- **Wire Format** (`src/wire.c`, `include/n2n_wire.h`): Handles serialization and deserialization of the n2n protocol packets. Overheads for v3 are typically 48-60 bytes.
- **Transform** (`src/transform_*.c`): Plugin-like architecture for payload transformations including encryption (AES, Speck, ChaCha20, etc.) and compression (LZO, ZSTD).

### Communication Flow
1. **Registration**: Edge nodes register their presence (MAC and public IP/port) to the Supernode.
2. **Discovery**: When an Edge wants to talk to a MAC, it asks the Supernode or broad-casts via the Supernode.
3. **P2P Setup**: Edges attempt UDP hole punching to establish direct P2P connectivity.
4. **Relay**: If P2P fails (e.g., both behind Symmetric NAT), traffic is relayed via the Supernode.

### Data Structures
- `n2n_edge_t` / `n2n_sn_t`: Main state structures for edge and supernode.
- `peer_info`: Stores information about known peers, including their MAC, public IP/port, and registration status.
- `n2n_edge_conf_t`: Configuration for an edge node.
