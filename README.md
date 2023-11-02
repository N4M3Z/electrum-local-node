# electrum-local-node
Local node for Electrum wallet

## Installation
Both electrum and electrum-personal-server run in Python, so the success of the following steps largely depends on your python environment and your platform. In general, the steps are as follows:

1. Install bitcoind
Either download a binary for your platform https://bitcoincore.org/en/download/ or build from source (see below).

`bitcoind` is also available on multiple package managers:

*macOS Homebrew*
```
brew install --cask bitcoin-core
```

Observe good security practices like verifying the release signatures: 
```
shasum -a 256 fileName
```
or 
```
gpg --verify nameOfSignatureFile nameOfZipFile
```
if you have `gpg` installed.

2. Install electrum
```
cd lib/electrum
python3 -m pip install . --target .
```
Alternatively install to your user directory and set up your paths yourself:
```
python3 -m pip install --user -e .
```
Note that in this case the binary will execute against configuration files outside of this repository and you are responsible for putting them in their correct location.

3. Install electrum-personal-server
```
cd lib/electrum-personal-server
python3 -m pip install . --target .
```
Alternatively install to your user directory and set up your paths yourself:
```
python3 -m pip install --user -e .
```
Note that in this case the binary will execute against configuration files outside of this repository and you are responsible for putting them in their correct location.

4. (Optional) Generate SSL certificates:
```
openssl req -x509 -out localhost.crt -keyout localhost.key \
  -newkey rsa:4096 -nodes -sha256 \
  -subj '/CN=localhost' -extensions EXT -config <( \
   printf "[dn]\nCN=localhost\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:localhost\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")
```
Alternatively follow the guide here: https://github.com/spesmilo/electrum-server/blob/ce1b11d7f5f7a70a3b6cc7ec1d3e552436e54ffe/HOWTO.md#step-8-create-a-self-signed-ssl-cert

## Configuration
1. Configure bitcoind to expose RPC/JSON 
```
host = 127.0.0.1
port = 8332
```
And to run a full transaction index
```
txindex=1
```
Use the sample configuration file in conf/bitcoind.

2. Configure electrum, this is largely personal preference with the exception of
```
oneserver=true
server=127.0.0.1:50002:s
```
Use the sample configuration file in conf/electrum

This can also be achieved through the CLI, see below when running electrum.

3. Configure electrum-personal-server by adding the xPub of the wallet you want to index:
```
myWallet = xPub
```
If you don't have it on hand, load your wallet in electrum and Go to Wallet > Master Public Keys, and copy the text that starts with xPub for you wallet.

Use the sample configuration file in conf/electrum-personal-server.


## Run
1. Start bitcoind, wait for synchronization to complete.
2. Start electrum-personal-server, wait for synchronization to complete. This could hours for large wallets with 100k+ transactions
```
./electrum-personal-server config.ini
```

3. Start electrum wallet 

```
./run_electrum --oneserver --server 127.0.0.1:50002:s --dir ../../conf/electrum
```

Alternatively set oneserver=true in electrum/config

## (Optional) Build Bitcoin Core
Clone the repository
```
git clone https://github.com/bitcoin/bitcoin.git
```

Navigate into bitcoin directory, and run the autogen script:
```
$ cd bitcoin
$ ./autogen.sh
```

The autogen.sh script creates a set of automatic configuration scripts that will interrogate your system to discover the correct settings and ensure you have all the necessary libraries to compile the code.

Run the configure script:
```
$ ./configure
```

This will automatically discover all the necessary libraries and create a customized build script for your system. If all went well, the configure command will end by creating the customized build scripts that will allow us to compile bitcoind.

Compile the bitcoin source:
```
$ make
```

Install the executables on your system:
```
$ sudo make install 
```

Default install location of bitcoind is /usr/local/bin. Confirm Bitcoin Core is installed:
```
$ which bitcoind
/usr/local/bin/bitcoind

$ which bitcoin-cli
/usr/local/bin/bitcoin-cli
```

Instructions from Andreas M. Antonopoulos: Mastering Bitcoin.

## References
- [Bitcoin Core Full Node on Mac OS, with Electrum Personal Server, and Electrum Desktop Wallet.](https://armantheparman.medium.com/bitcoin-core-full-node-on-mac-os-with-electrum-personal-server-and-electrum-desktop-wallet-c4ad0c1543ec)