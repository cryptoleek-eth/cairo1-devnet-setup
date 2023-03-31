# How to setup starknet-devnet?

## Install Rust
Please refer to this [documentation](https://github.com/starkware-libs/cairo#prerequisites)

## Install `cario-lang`
Please refer to this [documentation](https://github.com/starknet-edu/deploy-cairo1-demo#installing-cairo-lang)

## Setup starknet-devenet
For a new user, you need to first pull the image from docker:
Obviously, the version will keep changing and 0.5.0 still in aplha. Always find the latest stable version if possible.
```bash
### For Windows or Mac users:
docker pull shardlabs/starknet-devnet:0.5.0a1

### For Mac M1 users
docker pull shardlabs/starknet-devnet:0.5.0a1-arm 
```
After the images are pulled, you can run following codes to start the starknet-devnet. Here I use the `Mac M1` version, so the image is `shardlabs/starknet-devnet:0.5.0a1-arm`

### Enabling dumping and loading with Docker

To enable dumping and loading if running Devnet in a Docker container, you must bind the container path with the path on your host machine.
Dump docker container is neat technique to boostrap your env quickly.

This example:

- Relies on Docker bind mount; try Docker volume instead.
- Assumes that `/actual/dumpdir` exists. If unsure, use absolute paths.
- Assumes you are listening on `127.0.0.1:5050`.

If there is dump.pkl inside `/actual/dumpdir`, you can load it with:

```bash
docker run -p 127.0.0.1:5050:5050 --name starknet-devnet --mount type=bind,source=/actual/dumpdir,target=/dumpdir shardlabs/starknet-devnet:0.5.0a1-arm --load-path /dumpdir/dump.pkl --seed 1234 --timeout 10000
```

To dump to `/actual/dumpdir/dump.pkl` on Devnet shutdown, run:
```bash
docker run -p 127.0.0.1:5050:5050 --name starknet-devnet --mount type=bind,source=/actual/dumpdir,target=/dumpdir shardlabs/starknet-devnet:0.5.0a1-arm --seed 1234 --timeout 10000  --dump-on exit --dump-path /dumpdir/dump.pkl
```

## Test whether the devnet runs successfully

To test whether the devnet works, we can use the `starknet-cairo-101` contracts.  The following codes will be based on the `TDERC20.cairo`

```bash=
export STARKNET_NETWORK=alpha-goerli 
export STARKNET_WALLET=starkware.starknet.wallets.open_zeppelin.OpenZeppelinAccount
```

### Set up accounts
After your new your account, remember you need to faucet some eth into your shiny new account. Using postman mint.

```bash=
starknet new_account --account admin --gateway_url http://localhost:5050 --feeder_gateway_url http://localhost:5050
starknet deploy_account --account admin --gateway_url http://localhost:5050 --feeder_gateway_url http://localhost:5050
```

### Export admin account
```bash
export ACCT_ADMIN=0x581e71db726ccffda8d7b3c12b516d6f9108d8b4b211f3715e821223b5f550a
```

### Now we will use cairo-101 to test your local setup.
```
https://github.com/starknet-edu/starknet-cairo-101.git
git checkout cairo1
```

### Deploy the `TDERC20` contract
```bash
### Build the smart contract:
scarb build

### Declare the `TDERC20` contrac:
starknet declare --contract target/release/starknet_cairo_101_TDERC20.json --account admin --gateway_url http://localhost:5050 --feeder_gateway_url http://localhost:5050 --max_fee 100000000000000000000

### Deploy the `TDERC20` contract
starknet deploy --salt 0x1234 --class_hash 0x6b1cea5c54aef607edc926dec33a205d3da187ad8c1514706ab1e28db425138 --inputs 10057515165931654559836545801321088512241713 357609582641 18 0 0 $ACCT_ADMIN $ACCT_ADMIN --account admin --gateway_url http://localhost:5050 --feeder_gateway_url http://localhost:5050 --max_fee 100000000000000000000
```

### Distribute points to the admin account
```bash=
### Distribute points to the admin account
starknet invoke --function distribute_points --address 0x05e8e1c965d59bc44f109f72b9e67cedfa069352c4704723e123271827c49196 --inputs $ACCT_ADMIN 2000 --account admin --gateway_url http://localhost:5050 --feeder_gateway_url http://localhost:5050

### Check the balance of the admin account
starknet call --function balanceOf --address 0x05e8e1c965d59bc44f109f72b9e67cedfa069352c4704723e123271827c49196 --inputs $ACCT_ADMIN --gateway_url http://localhost:5050 --feeder_gateway_url http://localhost:5050
```

### You should see everything is working on your local dev-net
