# Building n2n with CMake

This project now supports building with CMake.

## Prerequisites

- CMake 3.10 or later
- A C compiler (GCC, Clang, MSVC)
- Dependencies (optional but recommended):
  - OpenSSL (libcrypto)
  - ZSTD (libzstd)
  - PCAP (libpcap) - optional
  - CAP (libcap) - optional (Linux only)
  - MiniUPnPc - optional
  - NAT-PMP (libnatpmp) - optional

## Building

1. Create a build directory:
   ```bash
   mkdir build
   cd build
   ```

2. Configure the project:
   ```bash
   cmake ..
   ```
   
   You can enable/disable options:
   ```bash
   cmake .. -DN2N_OPTION_ZSTD=OFF -DN2N_OPTION_OPENSSL=ON
   ```

3. Build:
   ```bash
   cmake --build .
   ```

4. Install (optional):
   ```bash
   sudo cmake --install .
   ```

## Debug Build

To build with debug symbols:
```bash
cmake .. -DCMAKE_BUILD_TYPE=Debug
cmake --build .
```

## Options

| Option | Description | Default |
|--------|-------------|---------|
| `N2N_OPTION_ZSTD` | Enable ZSTD compression support | ON |
| `N2N_OPTION_OPENSSL` | Enable OpenSSL encryption support | ON |
| `N2N_OPTION_PCAP` | Enable PCAP support | OFF |
| `N2N_OPTION_CAP` | Enable capabilities support (Linux) | OFF |
| `N2N_OPTION_MINIUPNP` | Enable MiniUPnP support | OFF |
| `N2N_OPTION_NATPMP` | Enable NAT-PMP support | OFF |
| `N2N_OPTION_PTHREAD` | Enable Pthread support | ON |
