# Froge Original — Full Node Installation

This document explains how to install and run a **Froge** full node from source. Froge is a Litecoin-forked blockchain project whose source code is available at:

> [https://github.com/frogecoins/frogeoriginal/](https://github.com/frogecoins/frogeoriginal/)

## 1. Overview
- **RPC Port**: `6332`  
- **P2P (Peer) Port**: `8333`  
- **Addnode**: `addnode=18.224.250.194:8333` (to connect to a primary node)
- **Litecoin Fork**: Build steps are similar to typical Litecoin/Bitcoin forks.

These instructions are primarily for **Linux** (Ubuntu/Debian). On other operating systems, you’ll adapt similarly.

---

## 2. Build Dependencies

Install the necessary packages to build Froge from source. On a typical Ubuntu or Debian system:

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

### (Optional) GUI / QT Dependencies
If you plan to build the Qt-based wallet (`Froge-Qt`), also install:

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

## 3. Clone and Prepare Source

```bash
# Clone the repository
git clone https://github.com/frogecoins/frogeoriginal.git
cd frogeoriginal

# Generate build configuration scripts
./autogen.sh
```

---

## 4. Configuration and Compilation

### 4A. Command-Line (Headless) Binary
If you only need the `froged` (daemon) and `froge-cli` (CLI):

```bash
./configure --without-gui
make
```
This compiles the daemon and CLI. Binaries appear in `src/`:
- `src/froged` (the full-node daemon)
- `src/froge-cli` (command-line RPC tool)

### 4B. With GUI (Froge-Qt)
If you’d like the graphical Froge-Qt wallet as well:

```bash
./configure
make
```
After building, you’ll find:
- `src/qt/froge-qt` (GUI wallet)
- `src/froged` (daemon)
- `src/froge-cli` (CLI tool)

---

## 5. Create `froge.conf`

To run a node, you typically place a config file in your Froge data directory. On Linux:

```bash
mkdir -p ~/.froge
nano ~/.froge/froge.conf
```

Include the following lines:

```
rpcuser=YourRPCUsername
rpcpassword=YourRPCPassword
rpcport=6332
port=8333
listen=1
server=1
txindex=1

# Connect to a primary node:
addnode=18.224.250.194:8333
```
  
> **Note**: Replace `YourRPCUsername` and `YourRPCPassword` with secure values.

---

## 6. Running the Node

### 6A. Daemon Mode (Headless)
If you built the daemon, run:

```bash
./src/froged -daemon
```
It will fork into background, reading config from `~/.froge/froge.conf`.

Check progress or status with:
```bash
./src/froge-cli getblockcount
./src/froge-cli getinfo
```

Stop the node:
```bash
./src/froge-cli stop
```

### 6B. Graphical UI
If you built `Froge-Qt`, launch via:
```bash
./src/qt/froge-qt
```
(Or the compiled `.app` on macOS.)

---

## 7. Opening Ports

- **P2P Port**: `8333`  
  For inbound connections, forward or allow TCP port `8333` in your firewall/router.  
- **RPC Port**: `6332`  
  Typically only accessible locally (`127.0.0.1`). If remote access is needed, ensure strong firewall rules.

---

## 8. Common Commands

**Check your node’s block count**:
```bash
./src/froge-cli getblockcount
```

**Check connection info**:
```bash
./src/froge-cli getpeerinfo
```

**Get a deposit address** (for the built-in wallet):
```bash
./src/froge-cli getnewaddress
```

**View wallet balance**:
```bash
./src/froge-cli getbalance
```

---

## 9. Troubleshooting

1. **Missing libraries**: If you see errors about missing `libboost_*`, ensure you installed Boost (and other dependencies) properly.  
2. **Permission denied**: Make the binaries executable: `chmod +x src/froged src/froge-cli`.  
3. **No incoming peers**: Confirm your firewall or router is forwarding port `8333`.

---

## 10. More Info

- **Froge Repo**: [https://github.com/frogecoins/frogeoriginal/](https://github.com/frogecoins/frogeoriginal/)  
- **Litecoin Docs**: Froge is LTC-based, so many standard Litecoin build/operational docs apply.  

Enjoy running your **Froge** node. For additional help, open an issue in the GitHub repository or check typical Litecoin/Bitcoin node-running guides.
