# block stream

a low-latency gRPC stream of new Ethereum blocks.

blocks are streamed before they have been validated, similar to bloXroute's [bdnBlocks](https://docs.bloxroute.com/eth/streams/blocks-streams/bdnblocks) stream. a block on this stream may never reach the chain. treat it as an early view of new blocks, not as canonical chain data.

transactions are delivered raw and unparsed, in block order.

access is by token. request one from [@ultrasoundrelay](https://t.me/ultrasoundrelay) or [contact@ultrasound.money](mailto:contact@ultrasound.money).

## endpoint

| | |
|---|---|
| address | `block-stream-jp.ultrasound.money` |
| service | `ultrasound.blockstream.v1.BlockStream` |
| method | `NewBlocks`, server streaming |
| auth | `authorization` metadata header carrying your token |

the endpoint is currently only served from Tokyo. we are planning to expand it to other regions (eu/us), contact us for support in other regions.

## schema

the service is defined in [`block_stream.proto`](block_stream.proto).

## connection settings

the server accepts and sends zstd compression. enable it if your client supports it.

## example client

```bash
pip install grpcio grpcio-tools
python -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. block_stream.proto
```

```python
import os
import time

import grpc

import block_stream_pb2
import block_stream_pb2_grpc

endpoint = "block-stream-jp.ultrasound.money"
token = os.environ["BLOCK_STREAM_TOKEN"]

options = [("grpc.max_receive_message_length", 32 * 1024 * 1024)]

while True:
    try:
        conn = grpc.secure_channel(endpoint, grpc.ssl_channel_credentials(), options=options)
        client = block_stream_pb2_grpc.BlockStreamStub(conn)
        stream = client.NewBlocks(
            block_stream_pb2.NewBlocksRequest(),
            metadata=(("authorization", token),),
        )
        for block in stream:
            print(int(block.header.number, 16), block.hash, len(block.raw_txs), flush=True)
    except grpc.RpcError as error:
        print(f"stream failed: {error.code().name}", flush=True)
    time.sleep(3)
```

## example block

```json
{
  "hash": "0x0ffc093697d26fb7d5d27fc2bc5badfb4447ced9cfcbcf26fec22c25ac147d26",
  "header": {
    "parentHash": "0xc9bfb47ef9ee1a146e93796f44e9a37c6d8d200b3ed4ce891aed03a514d4ac77",
    "miner": "0x4838b106fce9647bdf1e7877bf73ce8b0bad5f97",
    "stateRoot": "0xff4e190c99ff6fe17ea138c9f60fc556e1612e5f7ec0f0d35de114dfc1d31fd2",
    "receiptsRoot": "0xe972be747941fea4124adf7261ee4c19497cc08ee6d4f57180927220af021306",
    "logsBloom": "0x53eb6b2e036875f0ba7d6cf7ccff99ef9fefe27c5fd7f3226f35329a3e14c5fa...",
    "number": "0x1883421",
    "gasLimit": "0x3938700",
    "gasUsed": "0x12597bb",
    "timestamp": "0x6a75e0a7",
    "extraData": "0x546974616e2028746974616e6275696c6465722e78797a29",
    "mixHash": "0xd213439533e8c806d15d58806c0a6f01185eeab22f5da7f7474f1e79e306e46f",
    "baseFeePerGas": "0x1c812c42",
    "blobGasUsed": "0x20000",
    "excessBlobGas": "0xbfaa41d",
    "parentBeaconRoot": "0x7c180087232bdf08e588c8b7931878cee439d99568e4334bcc4dcf07c7b5a78d",
    "sha3Uncles": "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
    "nonce": "0x0000000000000000",
    "difficulty": "0x0"
  },
  "rawTxs": [
    "AvkDzAGDFFrRhOukKK+FAQ/XkYeDB6EilJjD0xg8S4plBhStF5oamL4KjWuOgLikScNsBwAAAAAA...",
    "AvixAReEstBeAIUBZaC8AIMDDUCU22ul1RDxFPmy6gi+p9MOMu7jNBGAuESpBZy7AAAAAAAAAAAA..."
  ],
  "withdrawals": [
    {
      "index": "0x84454fb",
      "validatorIndex": "0xde583",
      "address": "0xa9a2fb380d92a62060459df255cf16562c7a5c32",
      "amount": "0xdb719b"
    }
  ]
}
```

`rawTxs` are bytes on the wire; they appear base64-encoded here only because this is the JSON rendering.

## questions

[@ultrasoundrelay on telegram](https://t.me/ultrasoundrelay) or [contact@ultrasound.money](mailto:contact@ultrasound.money).
