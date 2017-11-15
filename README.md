# xcat
A tool for doing cross chain atomic trades via a [cli](src/cli) or [api](src/protocol.js).

Only trades between Stellar and Ethereum are supported at this stage.

## Protocol

Stellar XLM <-> Ethereum ETH
 * [protocol_stellar_xlm_to_ethereum_eth.md](docs/protocol_stellar_xlm_to_ethereum_eth.md)
 * [protocol-scenario1.js](integration-test/protocol-scenario1.js) - script that walks through scenario 1 of the protocol
 
Stellar Asset <-> Ethereum ERC20 Token
 * [protocol_stellar_asset_to_ethereum_erc20.md](docs/protocol_stellar_asset_to_ethereum_erc20.md) TODO

## Progress

### Native Token Swap (XLM <-> ETH)
- [x] Protocol API Scenario 1 (Stellar side initiated)
- [ ] Protocol API Scenario 2 (Ethereum side initiated)
- [x] CLI Scenario 1 (Stellar side initiated)
- [ ] CLI Scenario 2 (Ethereum side initiated)
- [ ] Built in communication channel (Telehash Links?)

The protocol for trades initiated from the Stellar side is working as demonstrated by this [script](integration-test/protocol-scenario1.js) which exercises both parties along the path of least resistance. More testing is required around refunds and edge cases. The command line interface (see below) also works for the simple case where each party follows the protocol.

Communication would ideally be done by the tool over some channel rather then having users manually send each other data (see last todo item above). These 2 pieces require manual sharing:
 * initial trade.json and trade.json.sig signature (intiater sends after running 'new', counterparty run 'verifysig' and 'import' commands)
 * refund tx envelope (party 2 sends to the initiater, initiater puts it in the trade.json file and runs 'status' to continue with the trade)

These pieces can be found by scanning the ledgers and so don't require communication:
 * Ethereum HTLC contract id
 * hash(x) preimage (after submission)
 * deposit and withdraw transactions on both sides

### Asset to Token Swap (Asset <-> ERC20)
- [ ] TODO!

## Trade.json

Trades are defined by a trade.json file conforming to the JSON schema [here](src/schema/trade.json).

Example:
```
{
  "timelock": 1505729032,
  "commitment":
    "22bf0b3d38d2bec7226eeafd6571cdd452d34a79fb4e72f98e246d372c6a9855",
  "stellar": {
    "token": "XLM",
    "amount": 2000.0,
    "depositor": "GADU5GS223ZOY7LRWDE5IQBGMCNI523FCF5CFP2KPMMU3TKLJ7IPHEJU",
    "withdrawer": "GD3BNTEXZF7IG4OSWDJMCZ4V6N4I6XGEWO5Y6P3FYAZ24ARF2DWV2P3L"
  },
  "ethereum": {
    "token": "ETH",
    "amount": 1.0,
    "depositor": "0x7ee7d41559284957f1cd6bd14a658bf985d87ad0",
    "withdrawer": "0xfc80237acff828e5a69ba27ec506c2296dab2f68"
  }
}
```

## CLI
 
npm run cli
 
```
  Usage: cli [options] [command]

  Stellar tool for doing cross chain atomic trades


  Options:

    -h, --help  output usage information


  Commands:

    new|n <trade.json>                         Initiate a new trade given a trade.json file
    import|i <trade.json>                      Import a trade from a trade.json file
    status|s <tradeId>                         Check status of a trade
    verifysig|v <trade.json> <trade.json.sig>  Verify signature for a trade.json
    help [cmd]                                 display help for [cmd]
```

### Summary

```
party1> npm run cli new ./trade_localnet.json
party2> npm run cli verify trade-little-eagle-37.json trade-little-eagle-37.json.sig
party2> npm run cli import trade-little-eagle-37.json
party1+2> npm run cli status little-eagle-37
```
After 'import' only the 'status' command is needed. It will scan the ledgers, determine what the next step is for the party running it and carry it out as required.

## Develop

Start Stellar private network
```
docker run -it --rm --name horizon-integrationnet -p 8000:8000 zulucrypto/stellar-integration-test-network
```

Start Ethereum local network:
```
PHRASE="end staff push admit delay abandon ability nurse renew alert stomach jazz"
docker run --rm -it -p 8545:8545 chatch/ethereumjs-testrpc:4.1.3 -m "$PHRASE" $@
```
NOTE: PHRASE is important as the integration tests expect specific accounts to exist.

Run tests:
```
npm run test
```

Run integration test:
```
npm run test-protocol
```

