# MinersWorldCoin (MWC) Lightning Network Support

This document describes running Core Lightning with the MinersWorldCoin (MWC) network.

The MWC Lightning integration has been tested with:

- MinersWorldCoin daemon
- Core Lightning (`lightningd`)
- libwally-core PSBT signing
- BOLT11 invoices
- Channel opening
- Lightning payments

Tested commit:


---

# Requirements

## Software

Required:

- MinersWorldCoin Core daemon
- Core Lightning
- libwally-core
- secp256k1

Recommended environment:

- Linux
- GCC/Clang
- Python 3
- Make

---

# Build Instructions

Clone the repository:

```bash
git clone https://github.com/Miners-World-Coin-MWC/lightning.git

cd lightning

git submodule update --init --recursive


make -j$(nproc)

./lightningd/lightningd --version



MinersWorldCoin Network Parameters:

Core Lightning network:

network=minersworldcoin

Address prefix:

bech32 HRP: rmwc

Example Lightning wallet address:

rmwc1...

Ports:

Service	Port
MWC P2P	9822
MWC RPC	9922
Lightning node A	19735
Lightning node B	19736


MinersWorldCoin RPC Configuration

Example MWC configuration:

server=1
rpcuser=<username>
rpcpassword=<password>

rpcbind=127.0.0.1
rpcport=9922

Verify RPC:

minersworldcoin-cli \
-rpcconnect=127.0.0.1 \
-rpcport=9922 \
-rpcuser=<username> \
-rpcpassword=<password> \
getblockchaininfo


Start Lightning Node

Example node:

mkdir -p /tmp/mwc-ln-a

./lightningd/lightningd \
--lightning-dir=/tmp/mwc-ln-a \
--network=minersworldcoin \
--addr=127.0.0.1:19735 \
--bitcoin-rpcconnect=127.0.0.1 \
--bitcoin-rpcport=9922 \
--bitcoin-rpcuser=<username> \
--bitcoin-rpcpassword=<password> \
--daemon

Check status:

./cli/lightning-cli \
--lightning-dir=/tmp/mwc-ln-a \
--network=minersworldcoin \
getinfo


Expected:

"network": "minersworldcoin"
Two Node Channel Test

Start node B:

mkdir -p /tmp/mwc-ln-b

Start:

./lightningd/lightningd \
--lightning-dir=/tmp/mwc-ln-b \
--network=minersworldcoin \
--addr=127.0.0.1:19736 \
--bitcoin-rpcconnect=127.0.0.1 \
--bitcoin-rpcport=9922 \
--bitcoin-rpcuser=<username> \
--bitcoin-rpcpassword=<password> \
--daemon

Get node ID:

lightning-cli \
--lightning-dir=/tmp/mwc-ln-b \
--network=minersworldcoin \
getinfo

Connect:

lightning-cli \
--lightning-dir=/tmp/mwc-ln-a \
--network=minersworldcoin \
connect <NODE_B_ID>@127.0.0.1:19736
Fund Channel

Create a wallet address:

lightning-cli \
--lightning-dir=/tmp/mwc-ln-a \
--network=minersworldcoin \
newaddr

Send MWC:

minersworldcoin-cli sendtoaddress <ADDRESS> 1

Mine confirmations:

minersworldcoin-cli generatetoaddress 6 <MINING_ADDRESS>

Open channel:

lightning-cli \
--lightning-dir=/tmp/mwc-ln-a \
--network=minersworldcoin \
fundchannel <NODE_B_ID> 1000000

Verify:

lightning-cli \
--lightning-dir=/tmp/mwc-ln-a \
--network=minersworldcoin \
listpeers

Expected:

CHANNELD_NORMAL
Lightning Payment Test

Create invoice on node B:

lightning-cli \
--lightning-dir=/tmp/mwc-ln-b \
--network=minersworldcoin \
invoice 1000000 testpayment "MWC Lightning test"

Pay from node A:

lightning-cli \
--lightning-dir=/tmp/mwc-ln-a \
--network=minersworldcoin \
pay <BOLT11_INVOICE>

Expected:

{
  "status": "complete"
}


