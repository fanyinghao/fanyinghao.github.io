---
title: 使用eosjs解密EOS提案的HEX数据
date: 2018-09-19 20:16:05
tags: [eos, eosjs, hex]
---

# 查看提案记录

使用[eosjs](https://github.com/EOSIO/eosjs)基本都能调用到[EOS RPC](https://developers.eos.io/eosio-nodeos/reference)的所有接口。调用`get_actions`获取并能筛选到提案记录，下方是`get_actions`返回的数据

```json
{
    "actions":[
        {
            "global_action_seq":13047273,
            "account_action_seq":0,
            "block_num":6385327,
            "block_time":"2018-08-22T09:22:19.000",
            "action_trace":{
                "receipt":{
                    "receiver":"eosio.msig",
                    "act_digest":"245b5fefa7f360400b43ede7606c0fe66f513ad7da7ecca31bc7f8b476dd5ea1",
                    "global_sequence":13047273,
                    "recv_sequence":312,
                    "auth_sequence":[
                        [
                            "fanyinghaohh",
                            1
                        ]
                    ],
                    "code_sequence":1,
                    "abi_sequence":1
                },
                "act":{
                    "account":"eosio.msig",
                    "name":"propose",
                    "authorization":[
                        {
                            "actor":"fanyinghaohh",
                            "permission":"active"
                        }
                    ],
                    "data":{
                        "proposer":"fanyinghaohh",
                        "proposal_name":"trkcgpeetgqx",
                        "requested":[
                            {
                                "actor":"fanyinghaoaa",
                                "permission":"active"
                            },
                            {
                                "actor":"fanyinghaobb",
                                "permission":"active"
                            },
                            {
                                "actor":"fanyinghaoio",
                                "permission":"active"
                            }
                        ],
                        "trx":{
                            "expiration":"2018-08-22T10:22:18",
                            "ref_block_num":28008,
                            "ref_block_prefix":1387059851,
                            "max_net_usage_words":0,
                            "max_cpu_usage_ms":0,
                            "delay_sec":0,
                            "context_free_actions":[

                            ],
                            "actions":[
                                {
                                    "account":"eosio.token",
                                    "name":"transfer",
                                    "authorization":[
                                        {
                                            "actor":"fanyinghaoab",
                                            "permission":"active"
                                        }
                                    ],
                                    "data":"700c358d4de7a759401d358d4de7a759102700000000000004454f53000000000166"
                                }
                            ],
                            "transaction_extensions":[

                            ]
                        }
                    },
                    "hex_data":"d01a358d4de7a759d02dcb4a5586e0cd03600c358d4de7a75900000000a8ed3232700e358d4de7a75900000000a8ed3232401d358d4de7a75900000000a8ed32325a397d5b686d8bdaac52000000000100a6823403ea3055000000572d3ccdcd01700c358d4de7a75900000000a8ed323222700c358d4de7a759401d358d4de7a759102700000000000004454f5300000000016600"
                },
                "elapsed":771,
                "cpu_usage":0,
                "console":"",
                "total_cpu_usage":0,
                "trx_id":"142d7c376c6c4d6d62668ac8b1d832233fe00048e02d87a12897b7a0f9a355b6",
                "inline_traces":[

                ]
            }
        }
    ],
    "last_irreversible_block":11119406
}
```

一切看似简单，当想到需要得知这个提案通过后执行什么操作，在哪里查呢？
```javascript
action_trace.act.data.trx.actions[0].data = "700c358d4de7a759401d358d4de7a759102700000000000004454f53000000000166"
```
这到底又是什么鬼呢？直觉告诉我这就是我要找的数据。

我们先看看怎么使用`eosjs`创建一个提案。
```javascript

const _eos = Eos({
    httpEndpoint: httpEndpoint,
    chainId: chainId,
    signProvider: signProvider,
    verbose: false
});

_eos.contract("eosio.msig").then(msig => {
    _eos
    .transfer(
        selectedAccount.account_name,
        _to,
        _amount,
        _memo || "transfer",
        { broadcast: false, sign: false }
    )
    .then(transfer => {
        transfer.transaction.transaction.max_net_usage_words = 0;
        transfer.transaction.transaction.expiration = new Date(
            Date.parse(new Date()) + 1000 * 60 * 60 //60mins expire
        );
        console.log(transfer.transaction.transaction);

        const proposal_name = `tr${randomWord(false, 10)}`;
        msig
        .propose(
            _proposer,
            proposal_name,
            permissions,
            transfer.transaction.transaction,
            { authorization: `${_proposer}@active` }
        )
        .then(tx => {
            onSuccess(tx, _proposer);
        }, onError);
    });
});
```

`_eos.contract("eosio.msig")`通过rpc获取到了eosio.msig合约的abi，之后就可以使用msig.propose方法了。在提案之前要创建一个Transaction，在这个Transaction内可以执行多个Action，这里我就只执行一个Transfer操作，特意声明不广播交易和不做交易签名，创建好后打印出当前的Transaction，打印出来的结果看到同样出现类似`"700c358d4de7a759401d358d4de7a759102700000000000004454f53000000000166"`的数字串。查阅文档后得知这就是action的data转成16进制的字符串，也就是说这里边就包含了Transfer操作所需的参数`from`、`to`、`quantity`和`memo`。

通过查看eosjs源代码和调试，定位到了[/src/structs.js](https://github.com/EOSIO/eosjs/blob/master/src/structs.js#L601)文件：
```javascript
'action.data.toObject': ({fields, object, result, config}) => {
    const {data, name} = object || {}
    const ser = (name || '') == '' ? fields.data : structLookup(name, object.account)
    if(!ser) {
      // Types without an ABI will accept hex
      result.data = Buffer.isBuffer(data) ? data.toString('hex') : data
      return
    }

    if(forceActionDataHex) {
      const b2 = new ByteBuffer(ByteBuffer.DEFAULT_CAPACITY, ByteBuffer.LITTLE_ENDIAN)
      if(data) {
        ser.appendByteBuffer(b2, data)
      }
      result.data = b2.copy(0, b2.offset).toString('hex')
      // console.log('result.data', result.data)
      return
    }

    // Serializable JSON
    result.data = ser.toObject(data, config)
  }
```
这个方法重载了abi定义，强制对数据进行16进制转化。既然有编码那就得有解码呀，在这代码上方就有类似的定义：
```javascript
  'action.data.fromObject': ({fields, object, result}) => {
    const {data, name} = object
    const ser = (name || '') == '' ? fields.data : structLookup(name, object.account)
    if(ser) {
      if(typeof data === 'object') {
        result.data = ser.fromObject(data) // resolve shorthand
      } else if(typeof data === 'string') {
        const buf = Buffer.from(data, 'hex')
        result.data = Fcbuffer.fromBuffer(ser, buf)
      } else {
        throw new TypeError('Expecting hex string or object in action.data')
      }
    } else {
      // console.log(`Unknown Action.name ${object.name}`)
      result.data = data
    }
  }
```
重点就在于`Fcbuffer.fromBuffer(ser, buf)`方法给解密了，`Fcbuffer`就是eosjs作者[James Calfee](https://github.com/jcalfee)封装的包。那只要传对参数，也就可以调用解密方法了。

# Decode HEX 方法
引入`Fcbuffer`
```bash
$ npm install fcbuffer
```
文件里引用
```javascript
const Fcbuffer = require("fcbuffer");
```
调用方法
```javascript

Helpers.getLatestProposals = name => {
  return new Promise((resolve, reject) => {
    eos.getActions(name, -1, -100).then(
      res => {
        resolve(
          res.actions
            .filter(item => {
              return item.action_trace.act.name === "propose";
            })
            .map(item => {
              item.action_trace.act.data.trx.actions = Array.prototype.map.call(
                item.action_trace.act.data.trx.actions,
                action => {
                  try {
                    action.origin_data = Fcbuffer.fromBuffer(
                      eos.fc.abiCache.abi(action.account).structs[action.name],
                      Buffer.from(action.data, "hex")
                    );
                    return action;
                  } catch (e) {
                    // Abi 'eosio.xxxx' is not cached
                    eos.getAbi(action.account).then(res => {
                      action.origin_data = Fcbuffer.fromBuffer(
                        eos.fc.abiCache.abi(action.account).structs[
                          action.name
                        ],
                        Buffer.from(action.data, "hex")
                      );
                      return action;
                    });
                  }
                }
              );
              return item;
            })
            .sort((a, b) => {
              if (a.account_action_seq > b.account_action_seq) return -1;
              if (a.account_action_seq < b.account_action_seq) return 1;
              return 0;
            })
        );
      },
      err => {
        reject(err);
      }
    );
  });
};

```
这样我就得到了从16进制串转回的原数据`action.origin_data`。使用同样的方法，可以也对`data_hex`整个transaction的数据进行decode。至于为什么要对提案的Transaction这么存储，这是保证在提案通过后执行的事务是一致的HEX数据，