# stratos-chain-testnet
stratos block testnet genesis and config

## Prepare environment to run node

### 0. Create user account(optional)
To create a separated and more secure environment, it is recommended to create a separated user account to run node.
```
sudo adduser stratos --home /home/stratos
```
Once `user` is created, login the system using *stratos* account and proceed with installation steps in context of that user

### 1. Download release binary files

#### Get `stchaind`  binary files

```bash
cd $HOME
wget https://github.com/stratosnet/stratos-chain/releases/download/v0.8.0/stchaind
```

> These binary files are built using linux amd64, so if you are preparing run a node on different kernel, please follow step 1.1 to build binaries for your machine
>
> For ease of use, we recommend you save these files in your `$HOME` folder. In the following, we suppose you are in the `$HOME` folder.

- Check the granularity
```bash
md5sum stchain*

## expect output 
## 834c713f15752e9f68489a43bac6a180 stchaind
```

- Add `execute` permission to the binary downloaded
```bash
chmod +x stchaind
```

### 1.1 Compile the binary with source code
Make sure you have Go 1.16+ installed ([link](https://golang.org/doc/install)).

```bash
git clone https://github.com/stratosnet/stratos-chain.git
cd stratos-chain
git checkout v0.8.0
make build
```
The binary file `stchaind` can be found in `build` folder. Then, move these two binary files to `$HOME`

```shell
mv build/stchaind ./
```

#### Install the binary files to `$GOPATH/bin`
```bash
make install
```

### 2. Get the `genesis` and `config` file
#### Initialize the node
```bash
./stchaind init "<node name you prefer>"

# ignore the output since you need to download the genesis file 
```

#### Download `genesis.json` and `config.toml` files
```bash
wget https://raw.githubusercontent.com/stratosnet/stratos-chain-testnet/main/genesis.json
wget https://raw.githubusercontent.com/stratosnet/stratos-chain-testnet/main/config.toml
```

#### Change `moniker`(optional if you don't want to become validator)
In `config.toml` file, at Line #16, there is a “moniker” field. Change it to any name you like. It’s your node name on the network.
```bash
# A custom human readable name for this node
moniker = "<node name you prefer>"
```

#### Move the `config.toml` and `genesis.json` files to `.stchaind/config/` folder
```bash
mv config.toml .stchaind/config/
mv genesis.json .stchaind/config/
```
> By default, the two binary executable files `stchaind` as well as the directory `.stchaind` have been saved or created in the `$HOME` folder. The `.stchaind` folder contains the node's configurations and data.

### 3. Run the node

There are three ways to run your Stratos-chain full-node. Please choose ONE of them to start the node.

- `stchaind start` command

    ```shell
    # Make sure we are inside the home directory
    cd $HOME
    
    # run your node
    ./stchaind start
    
    # Use `Ctrl+c` to stop the node.
    ```

- Run node in background

    ```shell
    # Make sure we are inside the home directory
    cd $HOME
    
    # run your node in backend
    ./stchaind start 2>&1 >> chain.log & 
    ```
  Use an editor to check your node log at `chain.log`

  Use the following Linux Command to stop your node.
    ```shell
    pkill stchaind
    ```
- Run node as a service

  All below steps require root privileges

    - Create the service file
      Create the `/lib/systemd/system/stratos.service` file with the following content

      ```shell
      [Unit]
      Description=Stratos Chain Node
      After=network-online.target
  
      [Service]
      User=stratos
      ExecStart=/home/stratos/stchaind start --home=/home/stratos/.stchaind
      Restart=on-failure
      RestartSec=3
      LimitNOFILE=8192
  
      [Install]
      WantedBy=multi-user.target
      ```
      > In the [service] section
      > - `User` is your system login username
      > - `ExecStart` designates the absolute path to the binary executable `stchaind`
      > - `--home` is the absolute path to your node folder.
      > - We used the default values for these variables. If you use a different username, group or folder to hold your node data instead of the default values, please modify these values according to your situations. Make sure the above values are correct.

    - Start your service
      Once you have successfully created the service, you need to enable and start it by running

            ```shell
      systemctl daemon-reload
      systemctl enable stratos.service
      systemctl start stratos.service
      ```
  - Service operations

      - Check the service status

        ```shell
        systemctl status stratos.service
        ```
      - Check service log

        ```shell
        journalctl -u stratos.service -f 
    
        # exit with ctrl+c
        ```

      - Stop the service

        ```shell
        systemctl stop stratos.service
        ```

## Operations
Once the node finishes catch-up, you can operate the node for various transactions(tx) and queries. You can find all the documents [here](https://github.com/stratosnet/stratos-chain/wiki).

In the following, some of commonly-used operations are listed.

### Create an account

```bash
./stchaind keys add --hd-path "m/44'/606'/0'/0/0" --keyring-backend test  <your wallet name>
```

Example
```bash
./stchaind keys add --hd-path "m/44'/606'/0'/0/0" --keyring-backend test  wallet1
```
> After executed the above command, a `.stchaind` will be created in your `$HOME` folder.

### `Faucet`
Faucet will be available at https://faucet-tropos.thestratos.org/ to get test tokens

```bash
curl --header "Content-Type: application/json" --request POST --data '{"denom":"ustos","address":"put_your_wallet_address_here"} ' https://faucet-tropos.thestratos.org/credit
```

> * 1 stos = 1000000000 ustos
> * By default, faucet will send a certain amount of tokens to the given wallet address
> * maximum 3 faucet requests to arbitrary wallet address from a single IP within an hour
> * maximum 1 faucet request to a fixed wallet address within an hour

Check balance (you need to wait for your node catching up with the network)
```bash
./stchaind query account <your wallet address>
```

Check node status
```bash
./stchaind status
```

### Your first tx - `send` transaction

```bash
./stchaind tx send <from address> <to address> <amount> --keyring-backend=<keyring's backend> --chain-id=<current chain-id>
```

```bash
$ ./stchaind tx send st1qzx8na3ujlaxstgcyguudaecr6mpsemflhhzua st1jvf660xagmzuzyqyqu3w27sj0ragn7qetnwmyr 100000000000ustos --keyring-backend=test --chain-id=stratos-testnet-3 --gas=auto

# then input y for the pop up to confirm send
```
> * In testing phase, --keyring-backend="test"
> * In testing phase, the current `chain-id` may change when updating. When it is applied, user needs to point out `current chain-id` which can be found on [this page](https://explorer-tropos.thestratos.org/), right next to the search bar at the top of the page.

### Becoming a validator(optional)
After the following steps have been done, Any participant in the network can signal that they want to become a validator. Please refer to [How to Become a Validator](https://github.com/stratosnet/stratos-chain/wiki/How-to-Become-a-Validator) for more details about validator creation, delegation as well as FAQ.
- [x] download related files
- [x] start your node to catch up to the latest block height(synchronization)
- [x] create your Stratos Chain Wallet
- [x] `Faucet` or `send` an amount of tokens to this wallet


## References
* ['stchaind' Commands(Part1)](https://github.com/stratosnet/stratos-chain/wiki/Stratos-Chain-%60stchaind%60-Commands(Part1))

* [stchaind' Commands(Part2)](https://github.com/stratosnet/stratos-chain/wiki/Stratos-Chain-%60stchaind%60-Commands(Part2))

* [REST APIs](https://github.com/stratosnet/stratos-chain/wiki/Stratos-Chain-REST-APIs)

* [How to become a validator](https://github.com/stratosnet/stratos-chain/wiki/How-to-Become-a-Validator)
