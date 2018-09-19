---
title: 以太坊智能合约学习摘录
date: 2018-08-01 09:33:52
tags: 杂乱
---

https://ethereum.stackexchange.com/a/33482

> You'll notice that there some more functions in your basic token contract, like approve, sendFrom and others. These functions are there for your token to interact with other contracts: if you want, say, sell tokens to a decentralized exchange, just sending them to an address will not be enough as the exchange will not be aware of the new tokens or who sent them, because contracts aren't able to subscribe to Events only to function calls. So for contracts, you should first approve an amount of tokens they can move from your account and then ping them to let them know they should do their thing - or do the two actions in one, with approveAndCall.


AB币交换

        A合约      B合约
甲   

乙

怎么用openssl创建钱包

