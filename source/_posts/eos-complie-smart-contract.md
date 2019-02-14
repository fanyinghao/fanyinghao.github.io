---
title: 编译EOS智能合约
date: 2019-02-14 10:09:52
tags:
---

摘录自[官方文档](https://developers.eos.io/eosio-home/docs/installing-the-contract-development-toolkit)

# Install the CDT
## Download

```bash
$ git clone --recursive https://github.com/eosio/eosio.cdt --branch v1.4.1 --single-branch
$ cd eosio.cdt
```

It may take up to 30 minutes to clone the repository

## Build
```shell
$ ./build.sh
```

## Install
```shell
$ sudo ./install.sh
```

# Compile Contract

```bash
$ eosio-cpp -o token.wasm token.cpp --abigen
```

# Validate WASM Hash
```bash
$ md5 token.wasm
MD5 (token.wasm) = c577dfaeb7b2ed69684ed24a7498483d
```