# pbc-explorer-install

### Daemon

Install all needed [dependencies](https://github.com/pbcllc/powerblockcoin-core/blob/master/doc/build-unix.md) for [powerblockcoin-core](https://github.com/pbcllc/powerblockcoin-core) and build `powerblockcoind` daemon binary.

Make `powerblockcoin.conf` inside `$HOME/.powerblockcoin` with the following content:

```
rpcuser=pbc2rpc
rpcpassword=myverysecretrpcpassword
rpcbind=127.0.0.1
rpcallowip=127.0.0.1
rpcworkqueue=256
server=1
listenonion=0
onlynet=ipv4
txindex=1
addressindex=1
timestampindex=1
spentindex=1
zmqpubrawtx=tcp://127.0.0.1:47775
zmqpubhashblock=tcp://127.0.0.1:47775
```

### Explorer

Install additional deps, if it's not yet installed:

```
sudo apt --yes install libzmq3-dev
```

Install NVM:

```
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.35.0/install.sh | bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm
```

Install Node v8:

```
nvm install v8
nvm use v8
```

`npm` version should be higher than `npm@5.7.1`, to prevent dependencies removal on install `insight-powerblock-api` and `insight-powerblock-ui` step,  read here why - https://github.com/npm/npm/issues/17929, on tested version `npm@6.13.4` - all is ok!

```
mkdir live
cd live 
npm install git+https://git@github.com/pbcllc/powerblockcore-node
# create PBC-explorer and powerblockcoin-node.json config for it
$PWD/node_modules/.bin/powerblockcore-node create PBC-explorer
cd PBC-explorer
$PWD/node_modules/.bin/powerblockcore-node install git+https://git@github.com/pbcllc/insight-powerblock-api git+https://git@github.com/pbcllc/insight-powerblock-ui
# to fix "more than one instance of bitcore-lib found" we should delete powerblockcore-lib dependency in submodules
rm -rf node_modules/powerblockcore-message/node_modules/powerblockcore-lib
rm -rf node_modules/insight-powerblock-api/node_modules/powerblockcore-lib
rm -rf node_modules/powerblockcore-node/node_modules/powerblockcore-lib
```

Replace the content of the `powerblockcoin-node.json` on following:

```
{
  "network": "mainnet",
  "port": 3001,
  "services": [
    "bitcoind",
    "insight-powerblock-api",
    "insight-powerblock-ui",
    "web"
  ],
  "servicesConfig": {
    "bitcoind": {
      "connect": [
        {
          "rpchost": "127.0.0.1",
          "rpcport": 47776,
          "rpcuser": "pbc2rpc",
          "rpcpassword": "myverysecretrpcpassword",
          "zmqpubrawtx": "tcp://127.0.0.1:47775"
        }
      ]
    },
    "insight-powerblock-api": {
      "disableRateLimiter": false,
      "rateLimiterOptions": {
        "whitelist": ["::ffff:127.0.0.1","127.0.0.1"],
        "whitelistLimit": 500000,
        "whitelistInterval": 3600000
      },
      "routePrefix": "api"
    },
    "insight-powerblock-ui": {
      "apiPrefix": "api",
      "routePrefix": ""
    }
  }
}
```
Use the following command to start explorer:

```
$PWD/node_modules/.bin/powerblockcore-node start
```

Test your explorer instance in browser:
```
http://127.0.0.1:3001/
```

**NB!** By default this installation uses `powerblockcore-lib` instance from `coblee_litecore` branch, as it is default branch in repo. `master` branch of `powerblockcore-lib` still contains few Litecoin changes (scrypt as a hashing algo and others) and shouldn't be used with PowerBlockCoin for now without changes.
