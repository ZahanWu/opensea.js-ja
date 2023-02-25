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
  - [代理でアイテムを購入する](#代理でアイテムを購入する)
  - [一括送信](#一括送信)
  - [イーサの代わりにERC-20トークンを使用する](#イーサの代わりにerc-20トークンを使用する)
  - [プライベート・オークション](#プライベートオークション)
  - [イベントをリスニングする](#イベントをリスニングする)
- [もっと詳しく](#もっと詳しく)
  - [サンプルコード](#サンプルコード)
- [バージョン1.0への移行](#バージョン10への移行)
- [開発するにあたって](#開発するにあたって)
- [トラブルシューティング](#トラブルシューティング)
- [ローカルでブランチをテストする](#ローカルでブランチをテストする)

## 概要

最大のNFTマーケットプレイスである[OpenSea](https://opensea.io)のJavaScript SDKです。

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
xcode-select --install # Command Line Toolsをまだインストールしていない場合は、インストールしてください
sudo xcode-select --switch /Library/Developer/CommandLineTools # Command Line Toolsを有効化
sudo npm explore npm -g -- npm install node-gyp@latest # (任意) node-gypをアップデート
```

## はじめに

まず[こちら](https://docs.opensea.io/reference)から、APIデータを使用する際の利用規約をご確認の上、APIキーをリクエストしてください。

次に、お使いのWeb3プロバイダを使用して、新しいOpenSeaJSのクライアント（OpenSeaSDK🚢）を作成してください：

```JavaScript
import * as Web3 from 'web3'
import { OpenSeaSDK, Network } from 'opensea-js'

// こちらサンプルのプロバイダではトランザクションの作成はできません。読み取り専用の呼び出しのみが可能です：
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
 * シンプルな注釈無しのノンファンジブル・アセットの仕様
 */
export interface Asset {
  // アセットのトークンID、ERC-20の場合はnull
  tokenId: string | null,
  // アセットのコントラクトアドレス
  tokenAddress: string,
  // このアセットのWyvern Schema Name (デフォルトは"ERC721")
  schemaName?: WyvernSchemaName,
  // 任意：ENSの名前
  name?: string,
  // 任意：ファンジブルなアイテム用
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

アセットにオファーを出すには、以下のように処理します：

```JavaScript
// ノンファンジブル・トークンのスマートコントラクトアドレスとトークンID：
const { tokenId, tokenAddress } = YOUR_ASSET
// The offerer's wallet address:
const accountAddress = "0x1234..."

const offer = await openseaSDK.createBuyOrder({
  asset: {
    tokenId,
    tokenAddress,
    schemaName // WyvernSchemaName（省略した場合、デフォルトでは'ERC721'になります。他には、'ERC20'と'ERC1155'があります。）
  },
  accountAddress,
  // 支払いに使用するトークン（もしくは指定がない場合はWETH）の単位でのオファー金額：
  startAmount: 1.2,
})
```

OpenSeaのユーザーが所有しているアイテムに対して、売り手の希望金額を上回るオファーを出した場合、**オファー額を通知するメールが自動的に売り手に送信されます**。

#### ENSオークションに入札する

Ethereum Name Service（ENS）では、ウォレットアドレスのラベル付けなどに使用できる短い名前（3〜6文字）をオークションに出品しています。詳しくは[ENS FAQ](https://opensea.io/ens)でご確認ください。


入札するには、ENS Short Nameスキーマを使用する必要があります：

```JavaScript
const {
  tokenId,
  // メインネットの場合、トークンアドレスは`0xfac7bea255a6990f749363002136af6556b31e04`です
  tokenAddress,
  // nameの末尾は`.eth`で、tokenIdと一致している必要があります
  name
} = ENS_ASSET // `openseaSDK.api.getAsset(...)`からENSアセットを取得できます

const offer = await openseaSDK.createBuyOrder({
  asset: {
    tokenId,
    tokenAddress,
    name,
    // short-nameオークションでのみ必要になります
    // 既に一度売られているENSの名前ではありません
    schemaName: "ENSShortNameAuction"
  },
  // あなたのウォレットアドレス（入札者のアドレス）：
  accountAddress: "0x1234..."
  // WETH単位でのオファー額:
  startAmount: 1.2,
})
```

#### オファーの上限

注意: 買い注文の合計金額は、ウォレット残高×1000 を超えないようにしてください。

### アイテムを出品/販売する

アセットを出品するには、`createSellOrder` を呼び出します。`startAmount`と`endAmount`が等しい固定価格での出品、または`expirationTime`に達するまで価格が下がり続け、`startAmount`より`endAmount`の方が低くなる[ダッチ・オークション](https://en.wikipedia.org/wiki/Dutch_auction) （価格下降式） で出品できます：

```JavaScript
// 現在から一日後にオークションを終了する
// JavaScriptのタイムスタンプ(ミリセカンド)から変換している点に注意してください:
const expirationTime = Math.round(Date.now() / 1000 + 60 * 60 * 24)

const listing = await openseaSDK.createSellOrder({
  asset: {
    tokenId,
    tokenAddress,
  },
  accountAddress,
  startAmount: 3,
  // `endAmount`が指定されている場合、`expirationTime`になるまで、オーダーの金額は指定した値に向かって下がり続けます。`endAmount`が指定されていない場合は、固定価格でのオーダーとなります：
  endAmount: 0.1,
  expirationTime
})
```

`startAmount`と`endAmount`の単位はイーサ（ETH）です。他のERC-20トークンを使用したい場合は、[イーサの代わりにERC-20トークンを使用する](#イーサの代わりにerc-20トークンを使用する)をご確認ください。

ユーザーがアイテムを初めて出品した際に生じる、セットアップのトランザクションに対応するには[イベントをリスニングする](#イベントをリスニングする)をご確認ください。


#### イギリス式オークションを作成する

イギリス式オークションとは、少額からスタートし、入札の度に金額が上がっていく形式のオークションです（OpenSeaでは0円での開始もおすすめしています！）。終了時点で最も高い金額で入札した人に商品が売却されます。

イギリス式オークションを作成するには、`waitForHighestBid`を`true`に設定して、最高入札額を待つ形式の出品を作成します：

```JavaScript

// WETH単位でのオークションを作成。詳しくは下記をご確認ください。
const paymentTokenAddress = "0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2"

const startAmount = 0 // 最低売却額（ETHなどの基本単位で指定）

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

なお、Ethereumの制限によりイーサを直接使ったオークションはサポートされていないので、ステーブルコインのDAIやWETHなどのERC20トークンを使う必要があります。詳しくは[イーサの代わりにERC-20トークンを使用する](#イーサの代わりにerc-20トークンを使用する)をご確認ください。

### オーダーをフェッチする

特定のアセットに関するオファーとオークションの一覧を取得するには、クライアント側に公開されている`OpenSeaAPI`のインスタンスを使用できます。APIフィルタのオブジェクトに渡されるパラメータは、キャメルケースに変換され、シリアライズされてから[OpenSea API parameters](https://docs.opensea.io/v2.0/reference)として送信されます

```JavaScript
// オファー（入札、または`side == 0`のオーダー）を取得する
const { orders, count } = await openseaSDK.api.getOrders({
  assetContractAddress: tokenAddress,
  tokenId,
  side: "bid"
})

// 全てのオークション（`side == 1`のオーダー）の2ページ目を取得する
const { orders, count } = await openseaSDK.api.getOrders({
  assetContractAddress: tokenAddress,
  tokenId,
  side: "ask"
}, 2)
```

アセットの出品価格は、そのアセットに対する**最も低い有効な売り注文**の`currentPrice`と等しくなる点に注意してください。ユーザーは過去の売り注文を取り消さなくても出品価格を下げることができるので、全ての売り注文がキャンセルされるか、または売り注文のどれかが成立するまで、それら全ての売り注文が配信され続けます。

signatures, makers, takers, listingTime vs createdTimeなどのオーダー用語については、API Docsの[**Terminology Section**](https://docs.opensea.io/reference#terminology)をご覧ください。

オーダーのエンドポイントで利用可能なAPIフィルタは、以下の`OrdersQueryOptions`インターフェースに記載されています。より詳しい最新の情報はメインの[APIドキュメント](https://docs.opensea.io/reference#reference-getting-started)からご確認いただけます。

```TypeScript
/**
   * Attrs used by orderbook to make queries easier
   * More to come soon!
   */
  side: "bid" | "ask", // 買い注文は"bid", 売り注文は"ask"
  protocol?: "seaport"; // オーダーのプロトコル(将来的に他のオプションも追加予定)
  maker?: string, // オーダーを作成した人のアドレス
  taker?: string, // 誰でもテイカーになれる場合はnullアドレスを指定
  owner?: string, // オーダーのアイテムの所有者のアドレス
  sale_kind?: SaleKind, // 固定価格の場合は0, ダッチオークションの場合は1
  assetContractAddress?: string, // オーダーのアイテムのコントラクトアドレス
  paymentTokenAddress?: string; // オーダーの支払いトークンのコントラクトアドレス
  tokenId?: number | string,
  tokenIds?: Array<number | string>,
  listedAfter?: number | string, // listing_timeが指定した値（秒単位）より大きい
  listedBefore?: number | string, // listing_timeが指定した値（秒単位）以下
  orderBy?: "created_date" | "eth_price", // 結果の並べ替え
  orderDirection?: "asc" | "desc", // 結果の並べ替えの順序
  onlyEnglish?: boolean, // イギリス式オークションのオーダーのみをreturn

  // ページネーション用
  limit?: number,
  offset?: number,
```

### アイテムを購入する

アイテムを購入するには、**売り注文を成立させる必要があります**。そのためには、以下のように処理します：


```JavaScript
const order = await openseaSDK.api.getOrder({ side: "ask", ... })
const accountAddress = "0x..." // 購入者（テイカー）のウォレットアドレス
const transactionHash = await this.props.openseaSDK.fulfillOrder({ order, accountAddress })
```

`fullfillOrder`のPromiseは、トランザクションが承認され、ブロックチェーンに取り込まれた際にresolveされるという点に注意してください。その前にトランザクションのハッシュを取得するには、`TransactionCreated`イベントのイベントリスナーを追加します（[イベントをリスニングする](#イベントをリスニングする)をご確認ください）。

オーダーが売り注文(`order.side === "ask"`)の場合、テイカーは _購入者_ なので、購入者にアイテムの代金を支払うよう促します。

### オファーを承認する

上記で売り注文を成立させたのと同様に、自分が所有しているアイテムに対する買い注文を成立させることで、オファーで提示されているトークンを受け取ることができます。

```JavaScript
const order = await openseaSDK.api.getOrder({ side: "bid", ... })
const accountAddress = "0x..." // 所有者（テイカー）のウォレットアドレス
await this.props.openseaSDK.fulfillOrder({ order, accountAddress })
```

オーダーが買い注文(`order.side === "bid"`)の場合、テイカーは _所有者_ なので、所有者にアイテムと代金を交換するよう促します。ユーザーが初めて入札を承認した時に生じるセットアップのトランザクションに対応するには、以下の[イベントをリスニングする](#イベントをリスニングする)ご確認ください。

### アイテムや暗号通貨を送る(ギフティング)

OpenSea.jsの便利な機能として、サポートされているあらゆる（ファンジブル、ノンファンジブルの）アセットを1行のJavaScriptで転送できるというものがあります。

以下の記述だけで、ERC-721やERC-1155のアセットを転送できます：

```JavaScript

const transactionHash = await openseaSDK.transfer({
  asset: { tokenId, tokenAddress },
  fromAddress, // アセットを所有している必要があります
  toAddress
})
```

ERC-1155のアセットでは、`schemaName`を"ERC1155"に設定し、`quantity`を設定することで一度に複数を転送することが可能です：

```JavaScript

const transactionHash = await openseaSDK.transfer({
  asset: {
    tokenId,
    tokenAddress,
    schemaName: "ERC1155"
  },
  fromAddress, // アセットを所有している必要があります
  toAddress,
  quantity: 2,
})
```

ERC20トークンのようなトークンIDを持たないファンジブルなアセットを転送するには、`OpenSeaFungibleToken`を`asset`として渡し、`schemaName`を"ERC20"に設定し、数量を`quantity`に基本単位（"wei"など）で指定します。

例えば、2DAI（$2）を他のアドレスに送る場合は以下のようになります：

```JavaScript
const paymentToken = (await openseaSDK.api.getPaymentTokens({ symbol: 'DAI'})).tokens[0]
const quantity = new BigNumber(Math.pow(10, paymentToken.decimals)).times(2)
const transactionHash = await openseaSDK.transfer({
  asset: {
    tokenId: null,
    tokenAddress: paymentToken.address,
    schemaName: "ERC20"
  },
  fromAddress, // トークンを所有している必要があります
  toAddress,
  quantity
})
```

詳細については、[WyvernSchemasのドキュメント](https://projectopensea.github.io/opensea-js)を参照してください

## 高度な機能

サーバーサイドやbotを用いて購入を実行したり、オーダーを予約したり、異なるERC-20トークンで入札するような処理も、OpenSea.jsを用いて実現することが出来ます

### 出品を予約する

SDKインスタンスに`listingTime`(秒単位のUTCタイムスタンプ)を渡すことで、特定の日時まで成立不可の売り注文を作成できます：



```JavaScript
const auction = await openseaSDK.createSellOrder({
  tokenAddress,
  tokenId,
  accountAddress,
  startAmount: 1,
  listingTime: Math.round(Date.now() / 1000 + 60 * 60 * 24) // 今から一日後
})
```

### 代理でアイテムを購入する

`RECIPIENTADDRESS`パラメータを渡すことで、商品の購入から他人への転送までをワンステップで実行できます！：

```JavaScript
const order = await openseaSDK.api.getOrder({ side: "ask", ... })
await this.props.openseaSDK.fulfillOrder({
  order,
  accountAddress, // トランザクションに署名するあなたのウォレットアドレス
  recipientAddress // 受取人（あなたが代理で購入している相手）のアドレス
})
```

オーダーが売り注文の場合(`order.side === "ask"`)、テイカーは _購入者_ なので、購入者にアイテムの代金の支払いを促しますが、アイテムは`recipientAddress`に送られます。オーダーが買い注文の場合(`"bid"`)、テイカーは _出品者_ ですが、落札額は`recipientAddress`に送られます。

### 一括送信

OpenSea.jsには、複数のアイテムを1つのトランザクションでまとめて転送するという便利な機能があります。これは、イーサリアムのガスリミットが許す限界量（ほとんどのアイテムのコントラクトにおいて、おおむね30アイテム未満）まで`transferFrom`の呼び出しをまとめることで実現しています。

一括送信は以下の記述で呼び出せます：

```JavaScript
const assets: Array<{tokenId: string; tokenAddress: string}> = [...]

const transactionHash = await openseaSDK.transferAll({
  assets,
  fromAddress, // 全てのアセットを所有している必要があります
  toAddress
})
```

これにより、自動的にアセットのトレードが許可され、送信用トランザクションが承認されます。


### イーサの代わりにERC-20トークンを使用する

以下はGenesis CryptoKittyを$100で出品した例です!もうレートを気にする必要はありません：

```JavaScript
// 米ドルの$1にペッグされているステーブルコイン"DAI"のトークンアドレス
const paymentTokenAddress = "0x89d24a6b4ccb1b6faa2625fe562bdd9a23260359"

// `startAmount`と`endAmount`の単位がDAIになったので、米ドルで$100となります
const auction = await openseaSDK.createSellOrder({
  tokenAddress: "0x06012c8cf97bead5deae237070f9587f8e7a266d", // CryptoKitties
  tokenId: "1", // Token ID
  accountAddress: OWNERS_WALLET_ADDRESS,
  startAmount: 100,
  paymentTokenAddress
})
```

`getPaymentTokens`を使用することで、シンボル名でトークンを検索できます。また、API経由で特定のERC-20トークンに対するすべてのオーダーをリストアップすることもできます：

```JavaScript
const token = (await openseaSDK.api.getPaymentTokens({ symbol: 'MANA'})).tokens[0]

const order = await openseaSDK.api.getOrders({
  side: "ask",
  paymentTokenAddress: token.address
})
```

**備考:** 近々、全てのERC-20トークンが利用可能になります！これにより、クリプト・コレクションに**あなた独自のERC-20トークンを使って**クレイジーなオファーを出せるということになります。しかし、opensea.ioでは、オーダーを受け取るユーザーの体験を最適化するため、把握しているERC-20トークンによるオファーとオークションのみを表示します。以下のトークンで作成されたオーダーに関してはOpenSea上に表示されます：

- MANA, Decentralandの通貨: https://etherscan.io/token/0x0f5d2fb29fb7d3cfee444a200298f468908cc942
- DAI, 米ドルの$1にペッグされているMakerのステーブルコイン: https://etherscan.io/token/0x89d24a6b4ccb1b6faa2625fe562bdd9a23260359

### プライベート・オークション

指定したアドレスやemailだけが購入できるオークションや出品を作成できます。これにより、事前に価格交渉を済ませた相手に対して、指定した価格でアイテムを販売することができます。

購入者のアドレスを指定した上で、Decentralandのパーセルを10ETHで出品する場合、以下のようになります：

```JavaScript
// 購入を許可するアドレス
const buyerAddress = "0x123..."

const listing = await openseaSDK.createSellOrder({
  tokenAddress: "0xf87e31492faf9a91b02ee0deaad50d51d56d5d4d", // Decentraland
  tokenId: "115792089237316195423570985008687907832853042650384256231655107562007036952461", // Token ID
  accountAddress: OWNERS_WALLET_ADDRESS,
  startAmount: 10,
  buyerAddress
})
```

### イベントをリスニングする

イベントは、トランザクションやオーダーが作成された時や、トランザクションがイーサリアムのブロックチェーン上で最近採掘されたブロックからのレシートを返す時に発生します。

Our recommendation is that you "forward" OpenSea events to your own store or state OpenSeaのイベントは、使用しているストアや状態管理システム側に"転送"するのがおすすめです。以下は、Reduxのアクションでそれを行う例です：

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

すべてのリスナーを削除して最初からやり直す場合は、`openseaSDK.removeAllListeners()`を呼び出してください。

## もっと詳しく

各APIに関する自動生成のドキュメントは[こちら](https://projectopensea.github.io/opensea-js/)をご確認ください。

### サンプルコード

このSDKで構築されたサンプルとして、[Ship's Log](https://github.com/ProjectOpenSea/ships-log)があります。このサンプルでは、OpenSeaのオーダーブック上の最近の注文を表示しています。

他にも、OpenSea.jsを使って構築された事例として[Mythereum marketplace](https://mythereum.io/marketplace)があります。


## バージョン1.0への移行

[Changelog](https://github.com/ProjectOpenSea/opensea-js/blob/master/CHANGELOG.md)をご確認ください。

## 開発するにあたって

**セットアップ**

開発を始める前に、必要なNPMの依存関係をインストールしてください：

```bash
npm install
```

TypeScriptをまだインストールしていない場合は、インストールしてください：

```bash
npm install -g tslint typescript
```

**ビルド**

次に、lintしてライブラリを`lib`ディレクトリにビルドします：

```bash
npm run build
```

テストを実行する場合は：

```bash
npm test
```

テストする場合は、InfuraとOpenSea APIの両方へのアクセスが必要になります。タイムアウトは`package.json`内の`test`スクリプトで調整できます。

**ドキュメントの作成**

htmlドキュメントを生成します。[こちら](https://projectopensea.github.io/opensea-js/)からも閲覧可能です:

```bash
yarn docs-build
```

**コントリビューション**

皆様からのコントリビューションをお待ちしています！提案や懸念点がある場合はGitHub issuesをご利用ください。コードから提案したい場合は、お気軽にプルリクエストを作成してください。


## トラブルシューティング

- `expirationTime`は将来の日時になっていますか？なっていない場合は、将来の日時で指定してください。

- 入力されたアドレスは全て文字列になっていますか？なっていない場合は、文字列に変換したください。

- お使いのコンピュータの内部時計は正確ですか？もしそうでなければ、ローカルで時刻の自動調整を有効にするか、[このチュートリアル](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/set-time.html)に従ってAmazon EC2インスタンスを設定してみてください。

## ローカルでブランチをテストする

```sh
yarn link # opensea-jsのレポジトリ内
yarn link opensea-js # 作業用のレポジトリ内
```
