# Mio Token

## Deployment

1.  Deploy MioToken and note the address.
2.  Transfer ownership of MioToken contract to the owner's account by calling `mioTokenInstance.transferOwnership(ownerAddress)`.
3.  Mint tokens by calling `mioTokenInstance.mint(receiverAddress, amount)`.
4.  Deploy TokenVault contract and note the address.

## During Mio token's lifecycle

###### Callable by the owner only:

- Pause/unpause in case of an emergency:

  - `mioTokenInstance.pause()`, `mioTokenInstance.unpause()`

  **Note that the mioTokenInstance will be unpaused on deployment. When paused, all ERC-20 methods will not be callable**.

- Recover ERC-20 tokens sent by mistake to the MioToken contract:

  - `mioTokenInstance.reclaimToken(anotherTokenAddress)`

  The balance would be sent to the MioToken contract's owner. Then, the owner can transfer the recovered tokens to their respective owner.

- Stop the minting of tokens

  - `mioTokenInstance.finishMinting()`

  Once this function is called, there is no way back to mint any more tokens.

## Transfer to multiple accounts / airdrops

- This can be executed by calling `mioTokenInstance.multiSend(receiversAddresses, amounts)`

## Specifications

###### Token

- Basic ERC-20 token features.
- Name: “Mio Token”.
- Symbol: “#MIO”.
- Decimals: 18.
- Total supply: 1,000,000,000.
- Mintable, burnable and pausable by owner.
- Includes checkPoints and balanceOfAt features to allow the implementation of a voting mechanism.
- Includes a multiSend method to facilitate the batch transfer of tokens.
- Reclaimable token: allows the owner to recover any ERC20 token received.

###### Token Vault

**Note that the Token Vault contract would transfer the specified amount from the token holder's account to itself. This means, the token holder would first need to approve the vault contract to execute this transfer. This can be done by calling `mioTokenInstance.approve(vaultContractInstanceAddress, amount)`**.

- A token holder can lock an amount of tokens by calling `tokenVaultInstance.addBalance(amount`). The tokens would be transferred from the holder and locked with his/her account as the beneficiary of the released tokens.

- A token holder can lock an amount of tokens for a different account by calling `tokenVaultInstance.addBalanceFor(beneficiaryAddress, amount`). The tokens would be transferred from the holder and locked for the specified account as the beneficiary of the released tokens.

- Each beneficiary can release his/her unlocked tokens by calling `tokenVaultInstance.release()` after the release time.

- Anyone can release tokens for an specific account by calling `tokenVaultInstance.releaseFor(beneficiaryAddress)` or for a number of accounts in just one call with `tokenVaultInstance.batchRelease(beneficiariesAddresses)`.

## Requirements
The server side scripts requires NodeJS 8 to work properly.
Go to [NVM](https://github.com/creationix/nvm) and follow the installation description.

### Use correct NodeJS version for this project
Before installing any dependencies (yarn install), ensure, you are using the right node version.
```
nvm install
nvm use
```

NVM supports both Linux and OS X, but that’s not to say that Windows users have to miss out. There is a second project named [nvm-windows](https://github.com/coreybutler/nvm-windows) which offers Windows users the possibility of easily managing Node environments.

__nvmrc support for windows users is not given, please make sure you are using the right Node version (as defined in .nvmrc) for this project!__

Yarn is required to be installed globally to minimize the risk of dependency issues.
Go to [Yarn](https://yarnpkg.com/en/docs/install) and choose the right installer for your system.

For the Rinkeby and MainNet deployment, you need Geth on your machine.
Follow the [installation instructions](https://github.com/ethereum/go-ethereum/wiki/Building-Ethereum) for your OS.

Depending on your system the following components might be already available or have to be provided manually:
* [Python](https://www.python.org/downloads/windows/) 2.7 Version only! Windows users should put python into the PATH by cheking the mark in installation process. The windows build tools contain python, so you don't have to install this manually.
* GIT, should already installed on *nix systems. Windows users have to install [GIT](http://git-scm.com/download/win) manually.
* On Windows systems, PowerShell is mandatory
* On Windows systems, windows build tools are required (already installed via package.json)
* make (on Ubuntu this is part of the commonly installed `sudo apt-get install build-essential`)
* On OSX the build tools included in XCode are required

__Every command must be executed from within the projects base directory!__

## Setup
Open your terminal and change into your project base directory. From here, install all needed dependencies.
```
yarn install
```
This will install all required dependecies in the directory _node_modules_.

## Compile, migrate, test and coverage
To compile, deploy and test the smart contracts, go into the projects root directory and use the task runner accordingly.
```
# Compile contract
yarn compile

# Migrate contract
yarn migrate

# Test the contract
yarn test

# Run coverage tests
yarn coverage
```

## Rinkeby testnet deployment
Start local Rinkeby test node in a separate terminal window and wait for the sync is finished.
```
yarn geth-rinkeby
```

Now you can connect to your local Rinkeby Geth console.
```
geth attach ipc://<PATH>/<TO>/Library/Ethereum/rinkeby/geth.ipc

# e.g.
# geth attach ipc://Users/patrice/Library/Ethereum/rinkeby/geth.ipc
```

Upon setup the node does not contain any private keys and associated accounts. Create an account in the web3 Geth console.
```
web3.personal.newAccount()
```
Press [Enter] twice to skip the password (or set one but then later it has to be provided for unlocking the account).

Read the address and send some Rinkeby Ether to pay for deployment and management transaction fees.
```
web3.eth.accounts
```
You can [obtain Rinkeby testnet Ether](https://www.rinkeby.io/#faucet) from the faucet by pasting your address in social media and pasting the link.

Connect to your rinkeby Geth console and unlock the account for deployment (2700 seconds = 45 minutes).
```
> personal.unlockAccount(web3.eth.accounts[0], "", 2700)
```

Ensure, all config files below `./config/` folder is setup properly. The `from` address will be used for the deployment, usually accounts[0].

After exiting the console by `<STRG> + <D>`, simply run `yarn migrate-rinkeby`.
This may take several minutes to finish.

You can monitor the deployment live via [Rinkeby](https://rinkeby.etherscan.io/address/<YOUR_RINKEBY_ADDRESS>)

After all, your smart contract can be found on etherscan:
https://rinkeby.etherscan.io/address/<REAL_CONTRACT_ADDRESS_HERE>

## MainNet deployment
__This is the production deployment, so please doublecheck all properties in the config files below `config` folder!__

For the MainNet deployment, you need a Geth installation on your machine.
Follow the [installation instructions](https://github.com/ethereum/go-ethereum/wiki/Building-Ethereum) for your OS.

Start local MainNet Ethereum node in a separate terminal window and wait for the sync is finished.
```
geth --syncmode "fast" --rpc
```

Now you can connect to your local MainNet Geth console.
```
geth attach ipc://<PATH>/<TO>/Library/Ethereum/geth.ipc

# e.g.
# geth attach ipc://Users/patrice/Library/Ethereum/geth.ipc
```

While syncing the blockchain, you can monitor the progress by typing `web3.eth.syncing`.
This shows you the highest available block and the current block you are on. If syncing is done, false will be returned. In this case, you can `web3.eth.blockNumber` and compare with the latest BlockNumber on Etherscan.

Upon setup the node does not contain any private keys and associated accounts. Create an account in the web3 Geth console.
```
web3.personal.newAccount("<YOUR_SECURE_PASSWORD>")
```
Enter <YOUR_SECURE_PASSWORD> and Press [Enter] to finish the account creation.

Read the address and send some real Ether to pay for deployment and management transaction fees.
```
web3.eth.accounts
```

Connect to your MainNet Geth console and unlock the account for deployment (240 seconds = 4 minutes).
```
personal.unlockAccount(web3.eth.accounts[0], "<YOUR_SECURE_PASSWORD>", 240)
```

Ensure, all config files below `./config/` folder is setup properly. The `from` address will be used for the deployment, usually accounts[0].

After exiting the console by `<STRG> + <D>`, simply run `yarn migrate-mainnet`.
This may take several minutes to finish.

You can monitor the deployment live via [Etherscan](https://etherscan.io/address/<YOUR_RINKEBY_ADDRESS>)

After all, your smart contract can be found on etherscan:
https://etherscan.io/address/<REAL_CONTRACT_ADDRESS_HERE>

### Contract Verification
The final step for the Rinkeby / MainNet deployment is the contract verificationSmart contract verification.

This can be dome on [Etherscan](https://etherscan.io/address/<REAL_ADDRESS_HERE>) or [Rinkeby Etherscan](https://rinkeby.etherscan.io/address/<REAL_ADDRESS_HERE>).
- Click on the `Contract Creation` link in the `to` column
- Click on the `Contract Code` link

Fill in the following data.
```
Contract Address:       <CONTRACT_ADDRESS>
Contract Name:          <CONTRACT_NAME>
Compiler:               v0.4.19+commit.c4cbbb05
Optimization:           YES
Solidity Contract Code: <Copy & Paste from ./build/bundle/IcoCrowdsale_all.sol>
Constructor Arguments:  <ABI from deployment output>
```
Visit [Solc version number](https://github.com/ethereum/solc-bin/tree/gh-pages/bin) page for determining the correct version number for your project.

- Confirm you are not a robot
- Hit `verify and publish` button

Now your smart contract is verified.
