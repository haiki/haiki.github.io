---
title: SpeedyMurmurs data 记录
date: 2019-10-31 10:34:05
categories:
tags:
---

### ripple-data-nov7-dynamic

- raw-trust-lines-2016-nov-7.txt: All trust lines as obtained from the Ripple server

```json
{"result":{"account":"r11zZZu6MvoCohwoQ2VrK9JXsw1Y3hVKa","lines":				[{"account":"rMwjYedjc7qqtKYVLiAccJSmCwih4LnE2q","balance":"0.005473328356624965","currency":"BTC","limit":"1000000000","limit_peer":"0","no_ripple":true,"quality_in":0,"quality_out":0},{"account":"r3ADD8kXSUKHd6zTCKfnKT3zV9EZHjzp1S","balance":"3.1119669487193","currency":"CAD","limit":"1000","limit_peer":"0","quality_in":0,"quality_out":0},{"account":"rMwjYedjc7qqtKYVLiAccJSmCwih4LnE2q","balance":"2.451995558","currency":"EUR","limit":"5000","limit_peer":"0","quality_in":0,"quality_out":0},{"account":"rMwjYedjc7qqtKYVLiAccJSmCwih4LnE2q","balance":"19.99944999900283","currency":"USD","limit":"0","limit_peer":"0","quality_in":0,"quality_out":0}
]},
"status":"success","type":"response"}
```

```json
{"result":{"account":"r12Fd6ffBipWZg9Ynt6AS6Y1G4ZnsaH6k","lines":[]},
"status":"success","type":"response"}
```

- complete-parsed-trust-lines-2016-nov-7.txt（all-in-USD-trust-lines-2016-nov-7-cleaned.txt）
  - (src, dst, lower bound, balance, currency, upper bound)

```
r11VNrqQzivyJqou5ZNrQ9Wfz6j3esVfE rB3gZey7VWHYRqJHLoHDEJXJ2pEPNieKiS 0 0.4554764488341 JPY 10000000000
r11VNrqQzivyJqou5ZNrQ9Wfz6j3esVfE rB3gZey7VWHYRqJHLoHDEJXJ2pEPNieKiS 0 0.0000423661606 BTC 10000000000
r29mNgvpvRBJNEark7LLHpHaoEhDAMWJD rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59B 0 0.007608519971505 BTC 10
```

- currency-exchanges-2016-nov-7.txt: Exchange rates from each currency to USD. The exchange rates have been obtained on Nov 9th from google (fiat currencies) and coinmarketcap.com (cryptocurrencies)
  
- all-in-USD-trust-lines-2016-nov-7.txt

  - (src, dst, lower bound, balance, upper bound) 

```
r11VNrqQzivyJqou5ZNrQ9Wfz6j3esVfE rB3gZey7VWHYRqJHLoHDEJXJ2pEPNieKiS 0.0 0.00437257390881 96000000.0
r11VNrqQzivyJqou5ZNrQ9Wfz6j3esVfE rB3gZey7VWHYRqJHLoHDEJXJ2pEPNieKiS 0.0 0.0306404783307 7.2323e+12
```

- ripple-transactions-jan-2013-aug-2016.txt
  - (tx_hash, sdr, rev, currency, amount1, amount2, ledger, tag1, tag2, crawl_id, unix_timestamp)

```
5C0504B229CD8EE803B01D78868B8F4B595D8B1D03636BEEBDBD750B6E5171AE,rfe8yiZUymRPx35BEwGjhfkaLmgNsTytxT,rsdbFqwyGYX5fSvAmMEXBH2hRyBT8QfcUS,XRP,0,100,7227428,"","",1,1402920660
5234065EB0115DFE0D1A71D7C2640EE798254A25E302D038D23BD44D1911834D,rGaRDdFcp1gZSrfuifZWiVLKq2SbaVRnuJ,rfTMcYDpWXhgcpYK27qxzx4cT3qDEpzgfs,XRP,12127,82200000,10438455,"","",1,1418233370
```


- transactions-in-USD-jan-2013-aug-2016.txt: Transactions performed in Ripple with known currencies.
  - (tx_hash, sdr, rcv, USD_amount, unix_timestamp) 

```
176FCFC0A4607B82DDC0847BA0C101858296238226F94DE4F7DF5DB4DD7C0C08 r5ymZSvtdNgbKVc8ay1Jhmq5f9QgnvEtj rGaRDdFcp1gZSrfuifZWiVLKq2SbaVRnuJ 353.6667023 1418016770
```

### ripple-data-jan29

- all-in-USD-trust-lines-2013-jan.txt:
  -  It contains the credit links between nodes that existed in the Ripple network in Jan 2013. Only credit links in a currency described in the file currency-exchanges-2016-nov-7.txt have been considered.
  - (acc1 acc2 lower_bound current_balance upper_bound)   (all values are in USD)

```
rM1oqKtfh1zgjdAgbFmaRm3btfGBX25xVo rnziParaNb8nsU4aruQdwYE3j5jUcqjzFm 723.23 0.0 0.0
rwpRq4gQrb58N7PRJwYEQaoSui6Xd3FC7j rhxbkK9jGqPVLZSWPvCEmmf15xHBfJfCEy 0.0 0.0 23866.59
```

- links-created-in-USD-jan-2013-dec-2016.txt: 
  - It cointains the credit links created from January 2013 until December 2013.
  - (acc1 acc2 value date) —— It means that there is a link from acc1 to acc2 with an upper limit of value (in USD) and it was created in date.

```
rhub8VRN55s94qWKDv6jmDy1pUykJzF3wq rLiCWKQNUs8CQ81m2rBoFjshuVJviSRoaJ 1100000000.0 2016-09-15T16:19:01+00:00
rM1oqKtfh1zgjdAgbFmaRm3btfGBX25xVo r9Gps6fB9YLuZ87rWx7M9TgLAGK2zsz5s6 7.2323e+11 2016-10-10T02:43:41+00:00
```



- ripple-graph-jan-2013.txt: Crawled Ripple graph

```
{"result":{"account":"rBKPS4oLSaV2KVVuHH8EpQqMGgGefGFQs7","ledger_hash":"EA814084C248A6D4D5AADC82897810D819DC936C187366E127162FE15E60D834","ledger_index":91211,"lines":[{"account":"rJRyob8LPaA3twGEQDPU2gXevWhpSgD8S6","balance":"-6","currency":"USD","limit":"0","limit_peer":"7","quality_in":0,"quality_out":0},{"account":"rnGTwRTacmqZZBwPB6rh3H1W4GoTZCQtNA","balance":"-7","currency":"USD","limit":"0","limit_peer":"3","quality_in":0,"quality_out":0}],"validated":true},"status":"success","type":"response"}
```



- trust-set-transactions.txt: Crawled create link transactions from Jan 2013 until Dec 2016

































































