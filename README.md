# Status

[![Build Status](https://travis-ci.org/arxanchain/wallet-sdk-py.svg?branch=master)](https://travis-ci.org/arxanchain/wallet-sdk-py)

## wallet-sdk-python
Blockchain Wallet SDK includes APIs for managing wallet accounts(Decentralized Identifiers, DID),
digital assets(Proof of Existence, POE), colored tokens etc.

You need not care about how the backend blockchain runs or the unintelligible
techniques, such as consensus, endorsement and decentralization. Simply use
the SDK we provide to implement your business logics, we will handle the caching,
tagging, compressing, encrypting and high availability.

## Contributions

We appreciate all kinds of contributions, such as opening issues, fixing bugs and improving documentation.

## Pre-requisite
The SDK requires the `python-pip` package(version >= 9.0.1), so please make sure it's already installed. Run the following cmd to install.

```sh
$ sudo apt-get install python-pip
```

## Install

You have two ways to prepare wallet-sdk-py environment. One way you can use shell command, the other way you can step by step to install all environment.

### shell command

If you want use shell command, you need to install golang and you should've configured your GOPATH environment variable.

*Note:* ServerCert/PrivateKey file please use absolute path. ServerCert is TLS certificate, PrivateKey is private key from your ArxanChain BaaS ChainConsole.

```sh
$ ./scripts/prepare.sh APIKEY, ServerCert, PrivateKey
```

### step by step
The following command will install wallet-sdk-py in your python environment.

```sh
$ python setup.py install
```

##### Usage

*Note:* Before using the wallet-sdk-python in your application,
you need to configure your installed py-common package.
For more details please refer to [the usage of py-common](https://github.com/arxanchain/py-common#usage)

**Note:** If you use linux, you can ignore this step.
If you use Mac or Windows, you should build the `sdk-go-common` dynamic link libraries on your OS instead of the default `py-common installation path/cryption/utils/utils.so`.
For more details please refer to [the usage of py-common](https://github.com/arxanchain/py-common#usage) and [sdk-go-common](https://github.com/arxanchain/sdk-go-common/tree/master/crypto/tools)

### Run unit test

The following command will run unit test

```sh
$ pytest
```

### Wallet Platform API
For details of Wallet APIs please refer to
[Wallet APIs Documentation](http://www.arxanfintech.com/infocenter/html/development/wallet.html#wallet-platform-ref)

### Init a Client
A client is used to wrap all the encryption/decryption details, any API that visits via
the BAAS service need to use this object, you can register a client object as follows:

```python
>>> from rest.api.api import Client
>>> apikey = "pWEzB4yMM1518346407"
>>> cert_path = "/usr/local/lib/python2.7/site-packages/py_common-2.0.1-py2.7.egg/cryption/ecc/certs"
>>> ip_addr = "http://127.0.0.1:9143"
>>> ent_sign_param = {
...     "creator": "did:axn:09e2fc68-f51e-4aff-b6e4-427cce3ed1af",
...     "created": "1517818060",
...     "nonce": "nonce",
...     "privateB64": "RiQ+oEuaelf2aecUZvG7xrWr+p43ZfjGZYfDCXfQD+ku0xY5BXP8kIKhiqzKRvfyKBKM3y7V9O1bF7X3M9mxkQ=="
... }
>>> client = Client(apikey, cert_path, ent_sign_param, ip_addr)
```

Param **apikey** is set to the API access key applied on `ChainConsole` management page,
param **cert_path** is the path of your private key file and tls certificate,
Param **ent_sign_param** is the enterprise private key dictionary.
and **ip_addr** is the IP address of the BAAS server entrance.
If you want to visit the BAAS service bypass the wasabi service,
you need to add param enable_crypto=False.

### Register a Wallet Client

To invoke the SDK API, you first have to create a wallet client as follows:

```python
>>> from api.wallet import WalletClient
>>> wallet = WalletClient(client)
```

* When building the wallet client configuration, **client** fields must
be set.

### Register wallet account

After creating the wallet client, you can use it to register a wallet account
as follows:

```python
>>> header = {}
>>> body = {
...     "id": "",
...     "type": "Organization",
...     "access": "username001",
...     "secret": "User001Pass", ## 8-16 characters including upper case, lower case and digits
...     #"public_key":  {
...     #    "usage": "",
...     #    "key_type": "",
...     #    "public_key_data": ""
...     #}这部分如果设置的话就会导致返回不了api.key
... }
>>> _, resp = wallet.register(header, body)
>>> print resp
{'ErrMessage': u'', 'Payload': u'{"id":"did:axn:6b7ae56d-d7d2-466b-a919-42e776376481","endpoint":"6d126b8a1af39605a1d479910836f1211dfa902a6f5d80d55eab9f0a37867a77","key_pair":{"private_key":"vnp7RE+MCvXyagia7Vst6gQRjWffd0wq+SzU5QQeJZmJohsBtRJDiwXmC1ld4aU6XhbTp6JWvCXP/5rCao+RDQ==","public_key":"iaIbAbUSQ4sF5gtZXeGlOl4W06eiVrwlz/+awmqPkQ0="},"created":1540117715,"transaction_ids":["3222f25ecccd5027cb957843b39420ab75a24f85d89bb94102eef21327aa2436","3389f21c76b9b087db1d0509f129eba71c2071593381b87c98286f379f4b389a","020efe6d0bfb34e0f7f9e9b6b37ab5a9ea675fe13adaf443a6b579ce3ef77f71"]}', 'ErrCode': 0, 'Method': u''}
```

* `Callback-Url` in the http header is optional. Set it only if you want to receive
events.

### Create POE digital asset

After creating the wallet account, you can create POE assets with this account as follows:

```python
>>> creator = "wallet ID"
>>> created = "123456" ## timestamp when poe is created
>>> privateB64 = "base64 formatted private key"
>>> payload = {
...     "id": "",
...     "name": "name",
...     #"parent_id": "parent-poe-id",  //这个是可选参数
...     "owner": "owner did",
...     "hash": "metadata-hash",
...     "metadata": map(ord, '{"address": "xxx", "telephone": "xxx", ...}')
... }
>>> params = {
...     "creator": creator,
...     "created": created,
...     "privateB64": privateB64,
...     "payload": payload,
...     "nonce": "your nonce"
...     }
>>> _, resp = wallet.create_poe({}, payload, params)
>>> print resp
{'ErrMessage': u'', 'Payload': u'{"id":"did:axn:39851d26-e37e-4523-a4f7-7c701287c209","created":1540118074,"transaction_ids":["f58acf6d6ad39b5f248cf62e9ea31f9baa4cf9a873fe17dc4760e2f2643e6147"]}', 'ErrCode': 0, 'Method': u''}
{'ErrMessage': u'', 'Payload': u'{"id":"did:axn:afa089da-ecc1-4a52-a7e8-d602dd837a77","created":1540121428,"transaction_ids":["d94b6a846a44fb6e77d7feef3de199bb38c92014c0c3faac5c1075bcb6d22587"]}', 'ErrCode': 0, 'Method': u''}

```

* When creating a POE asset, the **name** and **owner** fields must be set, with
**owner** being the wallet account ID.

* When building the signature parameter `privateB64`, use the Base64 encoded
ED25519 private key returned by [the regisering API](https://github.com/arxanchain/wallet-sdk-py#register-wallet-client).

### Upload POE file

After creating POE, you can upload the POE file for this account as follows:

```python
>>> filename = "file path"
>>> poeid = "poeid"
>>> readonly = "False"
>>> _, resp = wallet.upload_poe({}, filename, poeid, readonly)
>>> print resp
{'ErrMessage': u'', 'ErrCode': 0, 'Method': u'', 'Payload': u'{"id":"did:axn:39851d26-e37e-4523-a4f7-7c701287c209","transaction_ids":["eb2d28d7f572ef77f02d3e83cfadbeaec0554f03187e1daba4058f9bbc93dd71","d54e13c1d3f3cab381b6aeb63a4e4149266e6946debb4e283f9c2e4f22c5f71c"]}'}
{'ErrMessage': u'', 'Payload': u'{"id":"did:axn:31bccc33-beb5-4224-8d69-f56104a81c1a","created":1540121605,"transaction_ids":["77ed3e6f99ecd0ab251716208707c2f4f22c39bde92c109be4e12eb7ecad14bd"]}', 'ErrCode': 0, 'Method': u''}
```

* The `upload_poe` API uploads the file to **Offchain** storage, generates an SHA256
hash value, and saves this hash value into blockchain.

### Issue colored token using digital asset

Once you have possessed an asset, you can use this specific asset to issue colored
tokens as follows:

```python
>>> creator = "creator"
>>> created = 12345
>>> privateB64 = "base64 formatted private key"
>>> payload = {
...     "issuer": "issuer did",
...     "owner": "owner did",
...     "asset_id": "asset did",
...     "amount": 1000,
...     "fee": {
...         "amount": 5,
...     }
... }
>>> params = {
...     "creator": creator,
...     "created": created,
...     "nonce": "your nonce",
...     "privateB64": privateB64,
...     "payload": payload
...     }
>>> _, resp = wallet.issue_colored_token({}, payload, params)
需要注意的是owner与asset——id是一一对应的，issuer可以和ownner相同，并且asset_id必须是没有用过的（未上传poe）
{'ErrMessage': u'', 'Payload': u'{"token_id":"7f1d31197a332461bd8ab1bc5fbda3a484a1fecc0bac86d5698b3c65cbb467de","txs":[{"version":1,"timestamp":{"time":{"seconds":1540121704,"nanos":472224447}},"txin":[{"ix":4294967295}],"txout":[{"cTokenId":"7f1d31197a332461bd8ab1bc5fbda3a484a1fecc0bac86d5698b3c65cbb467de","cType":1,"value":1000,"addr":"did:axn:6b7ae56d-d7d2-466b-a919-42e776376481","until":-1,"script":"eyJwdWJsaWNLZXkiOiJpYUliQWJVU1E0c0Y1Z3RaWGVHbE9sNFcwNmVpVnJ3bHovK2F3bXFQa1EwPSJ9"}],"founder":"did:axn:6b7ae56d-d7d2-466b-a919-42e776376481"}]}', 'ErrCode': 0, 'Method': u''}
```

* When issuing colored tokens, you need to specify an issuer(a wallet account ID),
an asset to issue tokens, and the owner(another wallet account ID).

### Transfer colored tokens

After issuing colored tokens, the asset owner's wallet account will have these
tokens, and can transfer some of them to other wallet accounts.

```python
>>> payload = {
...     "from": "from did",
...     "to": "to did",
...     "asset_id": "asset id",
...     "tokens": [
...         {
...             "token_id": "token id",
...             "amount": 5
...         }
...     ],
...     "fee": {
...         "amount": 5,
...     }
... }
>>> params = {
...     "creator": creator,
...     "created": created,
...     "nonce": "your nonce",
...     "privateB64": privateB64,
...     "payload": payload
...     }
>>> _, resp = wallet.transfer_colored_tokens({}, payload, params)
>>> print resp
```

### Query colored token balance

You can use the `query_wallet_balance` API to get the balance of the specified wallet
account as follows:

```python
>>> id_ = "wallet id"
>>> _, resp = wallet.query_wallet_balance({}, id_)
>>> print resp
{'ErrMessage': u'', 'Payload': u'{"id":"did:axn:39851d26-e37e-4523-a4f7-7c701287c209","created":1540118074,"transaction_ids":["f58acf6d6ad39b5f248cf62e9ea31f9baa4cf9a873fe17dc4760e2f2643e6147"]}', 'ErrCode': 0, 'Method': u''}
```

### Query transaction logs

You can use the `get_tx_logs` API to get transaction logs of the specified wallet
account as follows:

```python
>>> id_ = "wallet id"
>>> tx_type = "in"
>>> num = 1 # number of logs to query
>>> page = 1 # page of logs to query
>>> _, resp = wallet.get_tx_logs({}, id, tx_type, num, page)
>>> print resp

{'ErrMessage': u'', 'Payload': u'{"\\u003cbuilt-in function id\\u003e":{}}', 'ErrCode': 0, 'Method': u''}
```

### Query transaction UTXO

You can use the `get_tx_utxo` API to get transaction UTXO of the specified wallet
account as follows:

```python
>>> id_ = "wallet id"
>>> num = 1
>>> page = 1
>>> _, resp = wallet.get_tx_utxo({}, id, num, page)
>>> print resp
query_txn_logs_with_id
{'ErrMessage': u'No such permission', 'Payload': None, 'ErrCode': 401, 'Method': u''}
```

### Query transaction STXO

You can use the `get_tx_stxo` API to get transaction STXO of the specified wallet
account as follows:

```python
>>> id_ = "wallet id"
>>> num = 1
>>> page = 1
>>> _, resp = wallet.get_tx_stxo({}, id, num, page)
>>> print resp
```

### Use callback URL to receive blockchain transaction events
Each of the APIs to invoke blockchain has two invoking modes: - `sync` and `async`.

The default invoking mode is asynchronous, it will return without waiting for
blockchain transaction confirmation. In asynchronous mode, you should set
'Callback-Url' in the http header to receive blockchain transaction events.

The blockchain transaction event structure is defined as follows:

```code
import google_protobuf "github.com/golang/protobuf/ptypes/timestamp

// Blockchain transaction event payload
type BcTxEventPayload struct {
    BlockNumber   uint64                     `json:"block_number"`   // Block number
    BlockHash     []byte                     `json:"block_hash"`     // Block hash
    ChannelId     string                     `json:"channel_id"`     // Channel ID
    ChaincodeId   string                     `json:"chaincode_id"`   // Chaincode ID
    TransactionId string                     `json:"transaction_id"` // Transaction ID
    Timestamp     *google_protobuf.Timestamp `json:"timestamp"`      // Transaction timestamp
    IsInvalid     bool                       `json:"is_invalid"`     // Is transaction invalid
    Payload       interface{}                `json:"payload"`        // Transaction Payload
}
```

A blockchain transaction event sample as follows:

```code
{
    "block_number":63,
    "block_hash":"vTRmfHZ3aaecbbw2A5zPcuzekUC42Lid3w+i6dOU5C0=",
    "channel_id":"pubchain",
    "chaincode_id":"pubchain-c4:",
    "transaction_id":"243eaa6e695cc4ce736e765395a64b8b917ff13a6c6500a11558b5e94e02556a",
    "timestamp":{
        "seconds":1521189855,
        "nanos":192203115
    },
    "is_invalid":false,
    "payload":{
        "id":"4debe20b-ca00-49b0-9130-026a1aefcf2d",
        "metadata": '{
            "member_id_value":"3714811988020512",
            "member_mobile":"6666",
            "member_name":"8777896121269017",
            "member_truename":"Tony"
        }'
    }
}
```

If you want to switch to synchronous invoking mode, set the 'Bc-Invoke-Mode' header
to 'sync'. In synchronous mode, it will not return until the blockchain
transaction is confirmed.

```python
>>> header = {
...     "Bc-Invoke-Mode": "sync"
... }
>>> body = {
...     ...
...     }
... }
>>> _, resp = wallet.register(header, body)
>>> print resp
```

### Transaction procedure

1. Send transfer proposal to get wallet.Tx

2. Sign public key as signature

3. Call ProcessTx to transfer formally


