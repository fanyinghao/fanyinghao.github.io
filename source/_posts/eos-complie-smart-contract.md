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
$ shasum -a 256 token.wasm
9acfb841363b2e67d9bf959795a75e20b16b1b9b455a05ee2d1d7b97c5b6846f  token.wasm
```
Compare to deploy transaction https://eosq.app/tx/a53fdf96b1fa6b990f05d7a5f1007476e5f1e2d7848e1e652ecc866db995937d