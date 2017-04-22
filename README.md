# ENS

[![Build Status](https://travis-ci.org/ethereum/ens.svg?branch=master)](https://travis-ci.org/ethereum/ens)

Implementations for registrars and local resolvers for the Ethereum Name Service.

For documentation of the ENS system, see [docs.ens.domains](http://docs.ens.domains/).

To run unit tests, clone this repository, and run:

    $ npm install
    $ npm test

## ENS.sol
Implementation of the ENS Registry, the central contract used to look up resolvers and owners for domains.

## FIFSRegistrar.sol
Implementation of a simple first-in-first-served registrar, which issues (sub-)domains to the first account to request them.

## HashRegistrar.sol
Implementation of a registrar based on second-price blind auctions and funds held on deposit, with a renewal process that weights renewal costs according to the change in mean price of registering a domain. Largely untested!

## HashRegistrarSimplified.sol
Simplified version of the above, with no support for renewals. This is the current proposal for interim registrar of the ENS system until a permanent registrar is decided on.

## PublicResolver.sol
Simple resolver implementation that allows the owner of any domain to configure how its name should resolve. One deployment of this contract allows any number of people to use it, by setting it as their resolver in the registry.

# ENS Registry interface

The ENS registry is a single central contract that provides a mapping from domain names to owners and resolvers, as described in [EIP 137](https://github.com/ethereum/EIPs/issues/137).

The ENS operates on 'nodes' instead of human-readable names; a human readable name is converted to a node using the namehash algorithm, which is as follows:

	def namehash(name):
	  if name == '':
	    return '\0' * 32
	  else:
	    label, _, remainder = name.partition('.')
	    return sha3(namehash(remainder) + sha3(label))

The registry's interface is as follows:

## owner(bytes32 node) constant returns (address)
Returns the owner of the specified node.

## resolver(bytes32 node) constant returns (address)
Returns the resolver for the specified node.

## setOwner(bytes32 node, address owner)
Updates the owner of a node. Only the current owner may call this function.

## setSubnodeOwner(bytes32 node, bytes32 label, address owner)
Updates the owner of a subnode. For instance, the owner of "foo.com" may change the owner of "bar.foo.com" by calling `setSubnodeOwner(namehash("foo.com"), sha3("bar"), newowner)`. Only callable by the owner of `node`.

## setResolver(bytes32 node, address resolver)
Sets the resolver address for the specified node.

# Resolver interface

Resolvers must implement one mandatory method, `has`, and may implement any number of other resource-type specific methods. Resolvers must `throw` when an unimplemented method is called.

## has(bytes32 node, bytes32 kind) constant returns (bool)

Returns true iff the specified node has the specified record kind available. Record kinds are defined by each resolver type and standardised in EIPs; currently only "addr" is supported.

`has()` must return false iff the corresponding record type specific methods will throw if called.

## addr(bytes32 node) constant returns (address ret)

Implements the addr resource type. Returns the Ethereum address associated with a node if it exists, or `throw`s if it does not.

# Generating LLL ABI and binary data

ENS.lll.bin was generated with the following command, using the lllc packaged with Solidity 0.4.4:

    $ lllc ENS.lll > ENS.lll.bin

The files in the abi directory were generated with the following command:

    $ solc --abi -o abi AbstractENS.sol FIFSRegistrar.sol HashRegistrarSimplified.sol PublicResolver.sol

# Getting started
Install Truffle

	$ npm install -g truffle

Launch the RPC client, for example TestRPC:

	$ testrpc

Deploy `ENS` and `FIFSRegistrar` to the private network, the deployment process is defined at [here](migrations/2_deploy_contracts.js):

	$ truffle migrate --network dev.fifs

alternatively, deploy the `HashRegistrar`:

	$ truffle migrate --network dev.auction

Check the truffle [documentation](http://truffleframework.com/docs/) for more information.

# Dev in ConsenLabs

## deploy

	truffle migrate --reset  --network kovan

## test in chrome with metamask

不要使用部署合约的账号来进行一下操作
建议使用chrome + metamask来做测试，因为可以方便的更好钱包地址

### 定义registrar 合约实例
```
var regsCont = web3.eth.contract(<registrar contract abi>)

var r = regsCont.at(<registrar address on kovan>)
```

### 检查域名状态
```
name = 'caizhen'

r.entries(web3.sha3(name), (err, res)=>{console.log(err, res.toString())})
```
### 开标
```
r.startAuction(web3.sha3(name), {from: web3.eth.accounts[0], gas: 100000}, (err, res)=>{console.log(err, res.toString())})
```

### 投标
```
r.shaBid(web3.sha3(name), web3.eth.accounts[0], web3.toWei(0.05, 'ether'), web3.sha3('secret'), (err, res)=>{console.log(err, res.toString())});
> null <bid>

r.newBid(<bid>, {from: web3.eth.accounts[0], value: web3.toWei(0.07, 'ether'), gas: 500000}, (err, res)=>{console.log(err, res)});
```

### 揭标
```
r.unsealBid(web3.sha3(name), web3.toWei(0.05, 'ether'), web3.sha3('secret'), {from: web3.eth.accounts[0], gas: 500000}, (err, res)=>{console.log(err, res.toString())})
```

### 结束竞标
```
r.finalizeAuction(web3.sha3(name), {from: web3.eth.accounts[0], gas: 500000}, (err, res)=>{console.log(err, res.toString())});
```

### 检查域名所有权
```
var ens = web3.eth.contract(<ens abi>).at(<ens address>)

ens.owner(namehash(name + '.eth'), (err, res)=>{console.log(err, res.toString())})
```
