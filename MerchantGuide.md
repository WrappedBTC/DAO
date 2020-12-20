# Merchant Guide for Minting and Burning WBTC
This manual assumes basic knowledge with Ethereum blockchain and in particular knowledge with how to make a contract call for an Ethereum smart contract.

## Initial setup
Before starting to interact with the WBTC system, the merchant should be aware of two important smart contract addresses, namely the *WBTC token* and the *Factory* contracts.
These addresses can be found [here](https://github.com/WrappedBTC/DAO#wbtc-important-addresses).
All the function call we describe in this guide are done to the factory smart contract.

The first step a merchant should do after he was listed by the dao is to setup his BTC deposit address.
When the merchant will burn WBTC the custodian will send him BTC to this deposit address.
The merchant setup this address by calling
```
setMerchantBtcDepositAddress(string btcDepositAddress)
```
The merchant can read the address he set by calling
```
merchantBtcDepositAddress(address merchant)
```


## Minting WBTC
Minting is done by sending BTC to the custodian deposit address and by submitting a minting request.


### Reading custodian deposit address
The merchant can read the custodian deposit address by calling
```
custodianBtcDepositAddress(address merchant)
```
It should be noted that this address is unique per merchant and merchant should only send his BTC to this address.

### Sending BTC
Custodian take fees over the mint operation. The exact fee percentage is determined by an off-chain agreement with the custodian.
The amount of WBTC that are expected to be minted after sending `X` BTC to the custodian deposit address is
```
(X * (100 - fee))/100
```
Where `fee` is percent unit. E.g., 0.2% is denoted by `fee = 0.2`.
The merchant should send `X` that correspond to the amount of WBTC he wish to mint.

### Submitting a mint request
After sending BTC to the custodian deposit address the merchant submit an on-chain minting request by calling:
```
    function addMintRequest(
        uint amount,
        string btcTxid,
        string btcDepositAddress
    )
```
Where:
1. `amount` is the amount of WBTC token to mint *in Satoshi units*. i.e., `100000000` (`1e8`) denotes 1 WBTC. If `X` BTC Satoshi were sent, then `amount` should be set to `(X * (100 - fee))/100`.
2. `btcTxid` the txhash of the BTC transfer to the custodian address. The hash should *not* contain the `0x` prefix.
3. `btcDepositAddress` the destination address of the BTC transfer. It must be equal to the custodian deposit address.

### Mint completion
The mint is completed after the custodian on-chain approved the request, and the minted WBTC tokens are sent to the merchant ethereum address.

### Mint cancellation
Prior to the confirmation of the custodian the merchant is entitled to cancel his mint request.
This should happen, e.g., if the merchant realize that the input parameters given to `addMintRequest` were wrong.
It should be noted that the cancellation does not revert the BTC transfer, and it should be sent back by the custodian.

## Burning WBTC
Burning WBTC is done by giving WBTC allowance to the factory contract and submitting a burn request on-chain.
It should be noted that in order to burn WBTC, the merchant must hold them in the merchant Ethereum account. 

### WBTC allowance
WBTC is a standard ERC20 token and as such the factory contract can accept it from an address only if the merchant approved it.
This is done by calling
```
approve(address _spender, uint256 _value)
```
function in the *WBTC contract* where
1. `_spender` should be set to factor contract.
2. `_value` should be set to the amount of WBTC the merchant wish to burn.

### Burn request submission
Done by calling
```burn(uint amount)```
where the amount is in Satoshi.

When calling this function the merchant WBTC will be burned, and the custodian will later send equivalent amount of BTC (minus applicable fees) to the merchant BTC deposit address (that was set in the initial setup).

***DO NOT call `burn` in WBTC token contract, you will lose your wbtc forever***
