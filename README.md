Below is the **updated README** for installing and running a **Froge Original** full node from source, **including** instructions to build **Berkeley DB 4.8** from source if needed.

---

# Froge Original — Full Node Installation

This guide explains how to install and run a **Froge** full node from source. Froge is a Litecoin-forked blockchain project whose source is available at:

> [https://github.com/frogecoins/frogeoriginal/](https://github.com/frogecoins/frogeoriginal/)

## Overview

- **RPC Port**: `6332`  
- **P2P (Peer) Port**: `8333`  
- **Addnode**: `addnode=18.224.250.194:8333` (connect to a primary node)  
- **Litecoin Fork**: Build steps are similar to typical Litecoin/Bitcoin forks.

These instructions assume a **Linux** environment (like Ubuntu). Adapt as necessary for other OSes.

---

## 1. Build Dependencies

Install the necessary packages to build Froge from source. On Ubuntu/Debian:

```bash
sudo apt-get update
sudo apt-get install -y \
  build-essential \
  libtool \
  autotools-dev \
  automake \
  pkg-config \
  libssl-dev \
  libevent-dev \
  bsdmainutils \
  libboost-all-dev
```

### (Optional) GUI / Qt Dependencies
If you want the Qt wallet (`Froge-Qt`):

```bash
sudo apt-get install -y \
  libqt5gui5 \
  libqt5core5a \
  libqt5dbus5 \
  qttools5-dev-tools \
  libprotobuf-dev \
  protobuf-compiler
```

---

## 2. (Recommended) Installing Berkeley DB 4.8

Froge (like Bitcoin/Litecoin forks) traditionally uses **Berkeley DB 4.8** for wallet compatibility. If your system doesn't have it, you can:

### Option A: Use the “bitcoin” PPA
```bash
sudo add-apt-repository ppa:bitcoin/bitcoin
sudo apt-get update
sudo apt-get install -y libdb4.8-dev libdb4.8++-dev
```

### Option B: Build Berkeley DB 4.8 from Source

**If** the above method isn’t convenient or you want a self-contained build:

```bash
# 1) Download the BDB 4.8 source
wget http://download.oracle.com/berkeley-db/db-4.8.30.NC.tar.gz

# 2) Extract
tar -xvf db-4.8.30.NC.tar.gz

# 3) Workaround for atomic symbols if needed:
cd ~
sed -i 's/__atomic_compare_exchange/__atomic_compare_exchange_db/g' \
    db-4.8.30.NC/dbinc/atomic.h

# 4) Enter build directory
cd db-4.8.30.NC/build_unix

# 5) Create a build folder; set BDB_PREFIX
mkdir -p build
BDB_PREFIX=$(pwd)/build

# 6) Configure with static linking, enable C++ API
../dist/configure --disable-shared --enable-cxx --with-pic --prefix=$BDB_PREFIX

# 7) Build & install
sudo make install
```

This installs BDB 4.8 into `(pwd)/build` under `db-4.8.30.NC/build_unix/build`. Later, you can tell Froge's `configure` where to find it if needed (see below).

---

## 3. Clone and Prepare Froge

```bash
git clone https://github.com/frogecoins/frogeoriginal.git
cd frogeoriginal

# Generate build scripts
./autogen.sh
```

---

## 4. Configure and Compile

### 4A. If You’ve Installed BDB 4.8 (System-Wide / PPA)

```bash
./configure
make
```
This should detect BDB 4.8 automatically if installed in the usual system paths (`/usr/lib/...` or `/usr/local/...`).

### 4B. If You Built BDB 4.8 from Source (Custom Path)

You can pass `--with-incompatible-bdb` or, more specifically, point configure to your BDB prefix. For example:

```bash
./configure \
  LDFLAGS="-L/path/to/db-4.8.30.NC/build_unix/build/lib" \
  CPPFLAGS="-I/path/to/db-4.8.30.NC/build_unix/build/include" \
  --disable-shared \
  --enable-cxx
make
```
Adjust the paths to where you installed BDB 4.8. That ensures Froge knows where to find `libdb_cxx-4.8.so` and headers.

### 4C. Building GUI (Froge-Qt)

If you want the Qt wallet UI:

```bash
./configure
make
```
You’ll get:
- `src/froged` (the daemon)
- `src/froge-cli` (CLI)
- `src/qt/froge-qt` (GUI wallet)

---

## 5. Create `froge.conf`

Make your Froge data directory and add a config file:

```bash
mkdir -p ~/.froge
nano ~/.froge/froge.conf
```

Example `froge.conf`:

```
rpcuser=YourRPCUsername
rpcpassword=YourRPCPassword
rpcport=6332
port=8333
listen=1
server=1
txindex=1

# Add the primary node
addnode=18.224.250.194:8333
```

> **Note**: Replace with secure `rpcuser`/`rpcpassword`.

---

## 6. Running the Node

### 6A. Daemon (no GUI)

```bash
./src/froged -daemon
```
This starts your node in the background, reading config from `~/.froge/froge.conf`.

Basic commands:
```bash
./src/froge-cli getblockcount
./src/froge-cli getinfo
./src/froge-cli stop
```

### 6B. GUI Wallet

If you built the Qt wallet:
```bash
./src/qt/froge-qt
```
(Or double-click the `.app` on macOS.)

---

## 7. Network & Ports

- **P2P port**: `8333`  
  - Forward or allow TCP 8333 if you want inbound peers.
- **RPC port**: `6332`  
  - Usually bound to `127.0.0.1`, so no external forwarding needed unless you require remote access.

---

## 8. Troubleshooting

1. **Missing Boost**: If errors mention `libboost_*` not found, confirm `libboost-all-dev` installed or brew-based Boost if on macOS.
2. **Berkeley DB mismatch**:  
   - `./configure: error: Found Berkeley DB other than 4.8...`  
   - Install or build BDB 4.8 (instructions above) or run `./configure --with-incompatible-bdb` or `--disable-wallet`.
3. **Permissions**: Make sure binaries are executable:  
   ```bash
   chmod +x src/froged src/froge-cli src/qt/froge-qt
   ```
4. **No incoming peers**: Add the port to your firewall or router (`8333` TCP).

---

## 9. Further Help

- **Repo**: [https://github.com/frogecoins/frogeoriginal](https://github.com/frogecoins/frogeoriginal)  
- **Litecoin Docs**: Froge is LTC-based, so standard Litecoin build/wallet steps often apply.

With that, you should have a full Froge node running, ready to sync the blockchain and optionally hold a wallet (if not disabled). Enjoy!
