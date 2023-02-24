<p align="center">
  <img src="https://storage.googleapis.com/opensea-static/opensea-js-logo-updated.png" />
</p>

# OpenSea.js <!-- omit in toc -->

[![https://badges.frapsoft.com/os/mit/mit.svg?v=102](https://badges.frapsoft.com/os/mit/mit.svg?v=102)](https://opensource.org/licenses/MIT)
[![Coverage Status](https://coveralls.io/repos/github/ProjectOpenSea/opensea-js/badge.svg?branch=master)](https://coveralls.io/github/ProjectOpenSea/opensea-js?branch=master)
[![styled with prettier](https://img.shields.io/badge/styled_with-prettier-ff69b4.svg)](https://github.com/prettier/prettier)

<!-- [![npm](https://img.shields.io/npm/v/wyvern-js.svg)](https://www.npmjs.com/package/wyvern-js) [![npm](https://img.shields.io/npm/dt/wyvern-js.svg)](https://www.npmjs.com/package/wyvern-js) -->

OpenSea.jsは、様々な暗号資産の売買や入札ができる、クリプトネイティブなeコマース向けのJavaScriptライブラリです。OpenSea.jsを使うことで、バックエンドのオーダーブックやスマートコントラクトを自分でデプロイしなくても、自身が所有するNFT（ERC-721またはERC-1155）を取引するための独自のマーケットプレイスを簡単に構築することができます。

Published on [GitHub](https://github.com/ProjectOpenSea/opensea-js) and [npm](https://www.npmjs.com/package/opensea-js)

- [概要](#概要)
- [インストール](#インストール)
- [はじめに](#はじめに)
  - [アセットをフェッチする](#アセットをフェッチする)
    - [残高と所有権を確認する](#残高と所有権を確認する)
  - [オファーを作成する](#オファーを作成する)
    - [ENSオークションに入札する](#ensオークションに入札する)
    - [オファーの上限](#オファーの上限)
  - [アイテムを出品/販売する](#アイテムを出品販売する)
  - [イギリス式オークションを作成する](#イギリス式オークションを作成する)
  - [オーダーをフェッチする](#オーダーをフェッチする)
  - [アイテムを購入する](#アイテムを購入する)
  - [オファーを承認する](#オファーを承認する)
  - [アイテムや暗号通貨を送る(ギフティング)](#アイテムや暗号通貨を送るギフティング)
- [高度な機能](#高度な機能)
  - [出品を予約する](#出品を予約する)
  - [代理で商品を購入する](#代理で商品を購入する)
  - [一括送信](#一括送信)
  - [イーサの代わりにERC-20トークンを使用する](#イーサの代わりにerc-20トークンを使用する)
  - [プライベート・オークション](#プライベートオークション)
  - [イベントを処理する](#イベントを処理する)
- [もっと詳しく](#もっと詳しく)
  - [サンプルコード](#サンプルコード)
- [バージョン1.0への移行](#バージョン10への移行)
- [開発するにあたって](#開発するにあたって)
- [トラブルシューティング](#トラブルシューティング)
- [ローカルでブランチをテストする](#ローカルでブランチをテストする)

## 概要

こちらは最大のNFTマーケットプレイスである[OpenSea](https://opensea.io)のJavaScript SDKです。

このSDKを使用することで、公式のオーダーブックへのアクセスや、データの絞り込み、買い注文（**オファー**）の作成、売り注文（**オークション**）の作成などの機能を利用でき、プログラム上で取引を完結させることが出来ます。


まずは[こちら](https://docs.opensea.io/reference)でAPIキーをリクエストしてから、OpenSea SDKのインスタンスを作成してください。その後、オフチェーンでのオーダーの作成や、オンチェーンでのオーダーの処理、イベントの処理（`ApproveAllAssets`や`WrapEth`等）などができるようになります。

それでは、良い船旅を! ⛵️


## インストール

一般的なクリプト関係の依存先が動作するように、Node.jsのバージョンを16に切り替えることをお勧めします。Node Version Managerを使用している場合は、`nvm use`を実行します。

その後、プロジェクト内で以下を実行してください:

```bash
npm install --save opensea-js
```

> **警告**
> バージョン8.5.2未満の`npm`は整合性チェックサムの検証に関してバグがあるため、git-urlでの依存関係を使用しているこのパッケージとは互換性がありません。
> バージョン8.5.2以上の`npm`では、git-urlでの依存関係について整合性チェックサムの検証をしない仕様になっています。

> **警告**
> `yarn`を使用する場合は、package.jsonに以下のresolutionを追加してください：
>
> ```
> "resolutions": {
>    "@0x/utils": "https://github.com/ProjectOpensea/0x-tools/raw/provider-patch/utils/0x-utils-6.5.0.tgz",
>  }
> ```

[web3](https://github.com/ethereum/web3.js)をまだインストールしていない方は、事前にインストールしてください。


Macを使用している方で、依存関係の構築中にエラーが発生した場合は以下を実行してください：

```bash
xcode-select --install # Install Command Line Tools if you haven't already.
sudo xcode-select --switch /Library/Developer/CommandLineTools # Enable command line tools
sudo npm explore npm -g -- npm install node-gyp@latest # (Optional) update node-gyp
```

## はじめに

まず[こちら](https://docs.opensea.io/reference)から、APIデータを使用する際の利用規約をご確認の上、APIキーをリクエストしてください。

次に、お使いのWeb3プロバイダを使用して、新しいOpenSeaJSのクライアント（OpenSeaSDK🚢）を作成してください：

```JavaScript
import * as Web3 from 'web3'
import { OpenSeaSDK, Network } from 'opensea-js'

// This example provider won't let you make transactions, only read-only calls:
const provider = new Web3.providers.HttpProvider('https://mainnet.infura.io')

const openseaSDK = new OpenSeaSDK(provider, {
  networkName: Network.Main,
  apiKey: YOUR_API_KEY
})
```

**注意:** テストネットでは`networkName`として`Network.Goerli`を使用してください - Rinkebyは2022年に非推奨となりました。

**注意:** 上記のサンプルに記載されているInfuraプロバイダでは、アセットや通貨のトレードや承認に必要なトランザクションの認証ができません。トランザクションを作成するには、秘密鍵やニーモニックのセットを持つプロバイダが必要になります。


[MetaMask](https://metamask.io/)、[Dapper](http://www.meetdapper.com/) などの拡張機能やWeb3を搭載したブラウザでは、`window.ethereum`(レガシーのモバイルweb3ブラウザでは `window.web3.currentProvider`)を使用してネイティブプロバイダーにアクセスできます。Node.jsの場合は、[この例](https://github.com/ProjectOpenSea/opensea-creatures/blob/master/scripts/sell.js)に従って、カスタムニーモニックを使用できます。


### アセットをフェッチする

アセットとはOpenSea上のアイテムで、ERC721のような標準規格に準拠した非代替性（ノン・ファンジブル）のものから、ERC1155のような準代替性（セミ・ファンジブル）のもの、さらにはERC20のような代替可能（ファンジブル）なものまで対応しています。

アセットはTypeScriptで定義されている`Asset`型で表現されます：


```TypeScript
/**
 * Simple, unannotated non-fungible asset spec
 */
export interface Asset {
  // The asset's token ID, or null if ERC-20
  tokenId: string | null,
  // The asset's contract address
  tokenAddress: string,
  // The Wyvern schema name (defaults to "ERC721") for this asset
  schemaName?: WyvernSchemaName,
  // Optional for ENS names
  name?: string,
  // Optional for fungible items
  decimals?: number
}
```

`Asset`型は、マーケットプレイス上でのほとんどのアクションで必要となる最低限の型です。`WyvernSchemaName`は任意の値で、省略した場合、ほとんどのアクションはERC721の非代替性アセットを参照しているとみなされます。他のオプションには'ERC20'と'ERC1155'があります。`import { WyvernSchemaName } from "opensea-js/lib/types"`のようにインポートすることで、サポートされている全てのスキーマを取得できます。

`OpenSeaAPI`を使用してアセットをフェッチすることで、`OpenSeaAsset` を取得できます (`OpenSeaAsset`は`Asset`を継承しています)：

```TypeScript
const asset: OpenSeaAsset = await openseaSDK.api.getAsset({
  tokenAddress, // string
  tokenId, // string | number | null
})
```

代替可能なERC20アセットの場合は、トークンIDが`null`となる点にご注意ください。


#### 残高と所有権を確認する

The nice thing about the `Asset` type is that it unifies logic between fungibles, non-fungibles, and semi-fungibles.
`アセット`型の良いところは、ファンジブルとノン・ファンジブル、セミ・ファンジブルの間でロジックを統一してくれることです。

`アセット`を取得すると、ERC-20かノンファンジブルかに関係なく、どのアカウントがいくつ所有しているかを確認することができます：

```JavaScript

const asset = {
  tokenAddress: "0x06012c8cf97bead5deae237070f9587f8e7a266d", // CryptoKitties
  tokenId: "1", // Token ID
}

const balance = await openseaSDK.getAssetBalance({
  accountAddress, // string
  asset, // Asset
})

const ownsKitty = balance.greaterThan(0)
```

この方法はWETHのようなERC-20トークンでも使用できます。便宜上、ファンジブルトークンの残高を確認する際には、以下のようなファンジブル用のラッパーを使用すると良いでしょう：

```JavaScript
const balanceOfWETH = await openseaSDK.getTokenBalance({
  accountAddress, // string
  tokenAddress: "0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2"
})
```

### オファーを作成する

Once you have your asset, you can do this to make an offer on it:

```JavaScript
// Token ID and smart contract address for a non-fungible token:
const { tokenId, tokenAddress } = YOUR_ASSET
// The offerer's wallet address:
const accountAddress = "0x1234..."

const offer = await openseaSDK.createBuyOrder({
  asset: {
    tokenId,
    tokenAddress,
    schemaName // WyvernSchemaName. If omitted, defaults to 'ERC721'. Other options include 'ERC20' and 'ERC1155'
  },
  accountAddress,
  // Value of the offer, in units of the payment token (or wrapped ETH if none is specified):
  startAmount: 1.2,
})
```

When you make an offer on an item owned by an OpenSea user, **that user will automatically get an email notifying them with the offer amount**, if it's above their desired threshold.

#### ENSオークションに入札する

The Ethereum Name Service (ENS) is auctioning short (3-6 character) names that can be used for labeling wallet addresses and more. Learn more on the [ENS FAQ](https://opensea.io/ens).

To bid, you must use the ENS Short Name schema:

```JavaScript
const {
  tokenId,
  // Token address should be `0xfac7bea255a6990f749363002136af6556b31e04` on mainnet
  tokenAddress,
  // Name must have `.eth` at the end and correspond with the tokenId
  name
} = ENS_ASSET // You can get an ENS asset from `openseaSDK.api.getAsset(...)`

const offer = await openseaSDK.createBuyOrder({
  asset: {
    tokenId,
    tokenAddress,
    name,
    // Only needed for the short-name auction, not ENS names
    // that have been sold once already:
    schemaName: "ENSShortNameAuction"
  },
  // Your wallet address (the bidder's address):
  accountAddress: "0x1234..."
  // Value of the offer, in wrapped ETH:
  startAmount: 1.2,
})
```

#### オファーの上限

Note: The total value of buy orders must not exceed 1000 x wallet balance.

### アイテムを出品/販売する

To sell an asset, call `createSellOrder`. You can do a fixed-price listing, where `startAmount` is equal to `endAmount`, or a declining [Dutch auction](https://en.wikipedia.org/wiki/Dutch_auction), where `endAmount` is lower and the price declines until `expirationTime` is hit:

```JavaScript
// Expire this auction one day from now.
// Note that we convert from the JavaScript timestamp (milliseconds):
const expirationTime = Math.round(Date.now() / 1000 + 60 * 60 * 24)

const listing = await openseaSDK.createSellOrder({
  asset: {
    tokenId,
    tokenAddress,
  },
  accountAddress,
  startAmount: 3,
  // If `endAmount` is specified, the order will decline in value to that amount until `expirationTime`. Otherwise, it's a fixed-price order:
  endAmount: 0.1,
  expirationTime
})
```

The units for `startAmount` and `endAmount` are Ether, ETH. If you want to specify another ERC-20 token to use, see [Using ERC-20 Tokens Instead of Ether](#using-erc-20-tokens-instead-of-ether).

See [Listening to Events](#listening-to-events) to respond to the setup transactions that occur the first time a user sells an item.

#### イギリス式オークションを作成する

English Auctions are auctions that start at a small amount (we recommend even doing 0!) and increase with every bid. At expiration time, the item sells to the highest bidder.

To create an English Auction, create a listing that waits for the highest bid by setting `waitForHighestBid` to `true`:

```JavaScript

// Create an auction to receive Wrapped Ether (WETH). See note below.
const paymentTokenAddress = "0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2"

const startAmount = 0 // The minimum amount to sell for, in normal units (e.g. ETH)

const auction = await openseaSDK.createSellOrder({
  asset: {
    tokenId,
    tokenAddress,
  },
  accountAddress,
  startAmount,
  expirationTime,
  paymentTokenAddress,
  waitForHighestBid: true
})
```

Note that auctions aren't supported with Ether directly due to limitations in Ethereum, so you have to use an ERC20 token, like Wrapped Ether (WETH), a stablecoin like DAI, etc. See [Using ERC-20 Tokens Instead of Ether](#using-erc-20-tokens-instead-of-ether) for more info.

### オーダーをフェッチする

To retrieve a list of offers and auction on an asset, you can use an instance of the `OpenSeaAPI` exposed on the client. Parameters passed into API filter objects are camel-cased and serialized before being sent as [OpenSea API parameters](https://docs.opensea.io/v2.0/reference):

```JavaScript
// Get offers (bids), a.k.a. orders where `side == 0`
const { orders, count } = await openseaSDK.api.getOrders({
  assetContractAddress: tokenAddress,
  tokenId,
  side: "bid"
})

// Get page 2 of all auctions, a.k.a. orders where `side == 1`
const { orders, count } = await openseaSDK.api.getOrders({
  assetContractAddress: tokenAddress,
  tokenId,
  side: "ask"
}, 2)
```

Note that the listing price of an asset is equal to the `currentPrice` of the **lowest valid sell order** on the asset. Users can lower their listing price without invalidating previous sell orders, so all get shipped down until they're canceled, or one is fulfilled.

To learn more about signatures, makers, takers, listingTime vs createdTime and other kinds of order terminology, please read the [**Terminology Section**](https://docs.opensea.io/reference#terminology) of the API Docs.

The available API filters for the orders endpoint is documented in the `OrdersQueryOptions` interface below, but see the main [API Docs](https://docs.opensea.io/reference#reference-getting-started) for a playground, along with more up-to-date and detailed explanantions.

```TypeScript
/**
   * Attrs used by orderbook to make queries easier
   * More to come soon!
   */
  side: "bid" | "ask", // "bid" for buy orders, "ask" for sell orders
  protocol?: "seaport"; // Protocol of the order (more options may be added in future)
  maker?: string, // Address of the order's creator
  taker?: string, // The null address if anyone is allowed to take the order
  owner?: string, // Address of owner of the order's item
  sale_kind?: SaleKind, // 0 for fixed-price, 1 for Dutch auctions
  assetContractAddress?: string, // Contract address for order's item
  paymentTokenAddress?: string; // Contract address for order's payment token
  tokenId?: number | string,
  tokenIds?: Array<number | string>,
  listedAfter?: number | string, // This means listing_time > value in seconds
  listedBefore?: number | string, // This means listing_time <= value in seconds
  orderBy?: "created_date" | "eth_price", // Field to sort results by
  orderDirection?: "asc" | "desc", // Sort direction of orderBy sorting of results
  onlyEnglish?: boolean, // Only return english auction orders

  // For pagination
  limit?: number,
  offset?: number,
```

### アイテムを購入する

To buy an item , you need to **fulfill a sell order**. To do that, it's just one call:

```JavaScript
const order = await openseaSDK.api.getOrder({ side: "ask", ... })
const accountAddress = "0x..." // The buyer's wallet address, also the taker
const transactionHash = await this.props.openseaSDK.fulfillOrder({ order, accountAddress })
```

Note that the `fulfillOrder` promise resolves when the transaction has been confirmed and mined to the blockchain. To get the transaction hash before this happens, add an event listener (see [Listening to Events](#listening-to-events)) for the `TransactionCreated` event.

If the order is a sell order (`order.side === "ask"`), the taker is the _buyer_ and this will prompt the buyer to pay for the item(s).

### オファーを承認する

Similar to fulfilling sell orders above, you need to fulfill a buy order on an item you own to receive the tokens in the offer.

```JavaScript
const order = await openseaSDK.api.getOrder({ side: "bid", ... })
const accountAddress = "0x..." // The owner's wallet address, also the taker
await this.props.openseaSDK.fulfillOrder({ order, accountAddress })
```

If the order is a buy order (`order.side === "bid"`), then the taker is the _owner_ and this will prompt the owner to exchange their item(s) for whatever is being offered in return. See [Listening to Events](#listening-to-events) below to respond to the setup transactions that occur the first time a user accepts a bid.

### アイテムや暗号通貨を送る(ギフティング)

A handy feature in OpenSea.js is the ability to transfer any supported asset (fungible or non-fungible tokens) in one line of JavaScript.

To transfer an ERC-721 asset or an ERC-1155 asset, it's just one call:

```JavaScript

const transactionHash = await openseaSDK.transfer({
  asset: { tokenId, tokenAddress },
  fromAddress, // Must own the asset
  toAddress
})
```

For fungible ERC-1155 assets, you can set `schemaName` to "ERC1155" and pass a `quantity` in to transfer multiple at once:

```JavaScript

const transactionHash = await openseaSDK.transfer({
  asset: {
    tokenId,
    tokenAddress,
    schemaName: "ERC1155"
  },
  fromAddress, // Must own the asset
  toAddress,
  quantity: 2,
})
```

To transfer fungible assets without token IDs, like ERC20 tokens, you can pass in an `OpenSeaFungibleToken` as the `asset`, set `schemaName` to "ERC20", and include `quantity` in base units (e.g. wei) to indicate how many.

Example for transfering 2 DAI ($2) to another address:

```JavaScript
const paymentToken = (await openseaSDK.api.getPaymentTokens({ symbol: 'DAI'})).tokens[0]
const quantity = new BigNumber(Math.pow(10, paymentToken.decimals)).times(2)
const transactionHash = await openseaSDK.transfer({
  asset: {
    tokenId: null,
    tokenAddress: paymentToken.address,
    schemaName: "ERC20"
  },
  fromAddress, // Must own the tokens
  toAddress,
  quantity
})
```

For more information, check out the documentation for WyvernSchemas on https://projectopensea.github.io/opensea-js/.

## 高度な機能

Interested in purchasing for users server-side or with a bot, scheduling future orders, or making bids in different ERC-20 tokens? OpenSea.js can help with that.

### 出品を予約する

You can create sell orders that aren't fulfillable until a future date. Just pass in a `listingTime` (a UTC timestamp in seconds) to your SDK instance:

```JavaScript
const auction = await openseaSDK.createSellOrder({
  tokenAddress,
  tokenId,
  accountAddress,
  startAmount: 1,
  listingTime: Math.round(Date.now() / 1000 + 60 * 60 * 24) // One day from now
})
```

### 代理で商品を購入する

You can buy and transfer an item to someone else in one step! Just pass the `recipientAddress` parameter:

```JavaScript
const order = await openseaSDK.api.getOrder({ side: "ask", ... })
await this.props.openseaSDK.fulfillOrder({
  order,
  accountAddress, // The address of your wallet, which will sign the transaction
  recipientAddress // The address of the recipient, i.e. the wallet you're purchasing on behalf of
})
```

If the order is a sell order (`order.side === "ask"`), the taker is the _buyer_ and this will prompt the buyer to pay for the item(s) but send them to the `recipientAddress`. If the order is a buy order ( `"bid"`), the taker is the _seller_ but the bid amount be sent to the `recipientAddress`.

### 一括送信

A handy feature in OpenSea.js is the ability to transfer multiple items at once in a single transaction. This works by grouping together as many `transferFrom` calls as the Ethereum gas limit allows, which is usually under 30 items, for most item contracts.

To make a bulk transfer, it's just one call:

```JavaScript
const assets: Array<{tokenId: string; tokenAddress: string}> = [...]

const transactionHash = await openseaSDK.transferAll({
  assets,
  fromAddress, // Must own all the assets
  toAddress
})
```

This will automatically approve the assets for trading and confirm the transaction for sending them.

### イーサの代わりにERC-20トークンを使用する

Here's an example of listing the Genesis CryptoKitty for $100! No more needing to worry about the exchange rate:

```JavaScript
// Token address for the DAI stablecoin, which is pegged to $1 USD
const paymentTokenAddress = "0x89d24a6b4ccb1b6faa2625fe562bdd9a23260359"

// The units for `startAmount` and `endAmount` are now in DAI, so $100 USD
const auction = await openseaSDK.createSellOrder({
  tokenAddress: "0x06012c8cf97bead5deae237070f9587f8e7a266d", // CryptoKitties
  tokenId: "1", // Token ID
  accountAddress: OWNERS_WALLET_ADDRESS,
  startAmount: 100,
  paymentTokenAddress
})
```

You can use `getPaymentTokens` to search for tokens by symbol name. And you can even list all orders for a specific ERC-20 token by querying the API:

```JavaScript
const token = (await openseaSDK.api.getPaymentTokens({ symbol: 'MANA'})).tokens[0]

const order = await openseaSDK.api.getOrders({
  side: "ask",
  paymentTokenAddress: token.address
})
```

**Fun note:** soon, all ERC-20 tokens will be allowed! This will mean you can create crazy offers on crypto collectibles **using your own ERC-20 token**. However, opensea.io will only display offers and auctions in ERC-20 tokens that it knows about, optimizing the user experience of order takers. Orders made with the following tokens will be shown on OpenSea:

- MANA, Decentraland's currency: https://etherscan.io/token/0x0f5d2fb29fb7d3cfee444a200298f468908cc942
- DAI, Maker's stablecoin, pegged to $1 USD: https://etherscan.io/token/0x89d24a6b4ccb1b6faa2625fe562bdd9a23260359

### プライベート・オークション

Now you can make auctions and listings that can only be fulfilled by an address or email of your choosing. This allows you to negotiate a price in some channel and sell for your chosen price on OpenSea, **without having to trust that the counterparty will abide by your terms!**

Here's an example of listing a Decentraland parcel for 10 ETH with a specific buyer address allowed to take it. No more needing to worry about whether they'll give you enough back!

```JavaScript
// Address allowed to buy from you
const buyerAddress = "0x123..."

const listing = await openseaSDK.createSellOrder({
  tokenAddress: "0xf87e31492faf9a91b02ee0deaad50d51d56d5d4d", // Decentraland
  tokenId: "115792089237316195423570985008687907832853042650384256231655107562007036952461", // Token ID
  accountAddress: OWNERS_WALLET_ADDRESS,
  startAmount: 10,
  buyerAddress
})
```

### イベントを処理する

Events are fired whenever transactions or orders are being created, and when transactions return receipts from recently mined blocks on the Ethereum blockchain.

Our recommendation is that you "forward" OpenSea events to your own store or state management system. Here's an example of doing that with a Redux action:

```JavaScript
import { EventType } from 'opensea-js'
import * as ActionTypes from './index'
import { openSeaSDK } from '../globalSingletons'

// ...

handleSDKEvents() {
  return async function(dispatch, getState) {
    openSeaSDK.addListener(EventType.TransactionCreated, ({ transactionHash, event }) => {
      console.info({ transactionHash, event })
      dispatch({ type: ActionTypes.SET_PENDING_TRANSACTION_HASH, hash: transactionHash })
    })
    openSeaSDK.addListener(EventType.TransactionConfirmed, ({ transactionHash, event }) => {
      console.info({ transactionHash, event })
      // Only reset your exchange UI if we're finishing an order fulfillment or cancellation
      if (event == EventType.MatchOrders || event == EventType.CancelOrder) {
        dispatch({ type: ActionTypes.RESET_EXCHANGE })
      }
    })
    openSeaSDK.addListener(EventType.TransactionDenied, ({ transactionHash, event }) => {
      console.info({ transactionHash, event })
      dispatch({ type: ActionTypes.RESET_EXCHANGE })
    })
    openSeaSDK.addListener(EventType.TransactionFailed, ({ transactionHash, event }) => {
      console.info({ transactionHash, event })
      dispatch({ type: ActionTypes.RESET_EXCHANGE })
    })
    openSeaSDK.addListener(EventType.InitializeAccount, ({ accountAddress }) => {
      console.info({ accountAddress })
      dispatch({ type: ActionTypes.INITIALIZE_PROXY })
    })
    openSeaSDK.addListener(EventType.WrapEth, ({ accountAddress, amount }) => {
      console.info({ accountAddress, amount })
      dispatch({ type: ActionTypes.WRAP_ETH })
    })
    openSeaSDK.addListener(EventType.UnwrapWeth, ({ accountAddress, amount }) => {
      console.info({ accountAddress, amount })
      dispatch({ type: ActionTypes.UNWRAP_WETH })
    })
    openSeaSDK.addListener(EventType.ApproveCurrency, ({ accountAddress, tokenAddress }) => {
      console.info({ accountAddress, tokenAddress })
      dispatch({ type: ActionTypes.APPROVE_WETH })
    })
    openSeaSDK.addListener(EventType.ApproveAllAssets, ({ accountAddress, proxyAddress, tokenAddress }) => {
      console.info({ accountAddress, proxyAddress, tokenAddress })
      dispatch({ type: ActionTypes.APPROVE_ALL_ASSETS })
    })
    openSeaSDK.addListener(EventType.ApproveAsset, ({ accountAddress, proxyAddress, tokenAddress, tokenId }) => {
      console.info({ accountAddress, proxyAddress, tokenAddress, tokenId })
      dispatch({ type: ActionTypes.APPROVE_ASSET })
    })
    openSeaSDK.addListener(EventType.CreateOrder, ({ order, accountAddress }) => {
      console.info({ order, accountAddress })
      dispatch({ type: ActionTypes.CREATE_ORDER })
    })
    openSeaSDK.addListener(EventType.OrderDenied, ({ order, accountAddress }) => {
      console.info({ order, accountAddress })
      dispatch({ type: ActionTypes.RESET_EXCHANGE })
    })
    openSeaSDK.addListener(EventType.MatchOrders, ({ buy, sell, accountAddress }) => {
      console.info({ buy, sell, accountAddress })
      dispatch({ type: ActionTypes.FULFILL_ORDER })
    })
    openSeaSDK.addListener(EventType.CancelOrder, ({ order, accountAddress }) => {
      console.info({ order, accountAddress })
      dispatch({ type: ActionTypes.CANCEL_ORDER })
    })
  }
}
```

To remove all listeners and start over, just call `openseaSDK.removeAllListeners()`.

## もっと詳しく

Auto-generated documentation for each export is available [here](https://projectopensea.github.io/opensea-js/).

### サンプルコード

Check out the [Ship's Log](https://github.com/ProjectOpenSea/ships-log), built with the SDK, which shows the recent orders in the OpenSea orderbook.

Also check out the [Mythereum marketplace](https://mythereum.io/marketplace), which is entirely powered by OpenSea.js.

## バージョン1.0への移行

See the [Changelog](CHANGELOG.md).

## 開発するにあたって

**Setup**

Before any development, install the required NPM dependencies:

```bash
npm install
```

And install TypeScript if you haven't already:

```bash
npm install -g tslint typescript
```

**Build**

Then, lint and build the library into the `lib` directory:

```bash
npm run build
```

Or run the tests:

```bash
npm test
```

Note that the tests require access to both Infura and the OpenSea API. The timeout is adjustable via the `test` script in `package.json`.

**Generate Documentation**

Generate html docs, also available for browsing [here](https://projectopensea.github.io/opensea-js/):

```bash
yarn docs-build
```

**Contributing**

Contributions welcome! Please use GitHub issues for suggestions/concerns - if you prefer to express your intentions in code, feel free to submit a pull request.

## トラブルシューティング

- Is the `expirationTime` in the future? If not, change it to a time in the future.

- Are the input addresses all strings? If not, convert them to strings.

- Is your computer's internal clock accurate? If not, try enabling automatic clock adjustment locally or following [this tutorial](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/set-time.html) to update an Amazon EC2 instance.

## ローカルでブランチをテストする

```sh
yarn link # in opensea-js repo
yarn link opensea-js # in repo you're working on
```
