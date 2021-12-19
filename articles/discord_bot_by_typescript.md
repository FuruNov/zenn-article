---
title: "TypeScript と Repl.it で Discord Bot を動かす"
emoji: "📑"
type: "tech"
topics: ["TypeScript", "Discord", "Bot", "Replit"]
published: true
---

# 概要
[TECHCAFE×TAIR Advent Calender](https://qiita.com/advent-calendar/2021/techcafe_tair)の20日目です．
本日は TypeScript と Repl.it を用いて，Discord Bot を作成する方法を解説します．
この Bot はユーザーが入力したコマンドに対して応答を行います．
また，説明にあたって Repl.it の[テンプレート](https://replit.com/@FuruNov/ts-disbot-template?v=1)を作りました．
これを Fork して，手軽に Bot を動かすことができます．

- テンプレート

https://replit.com/@FuruNov/ts-disbot-template?v=1

# 使うもの
1. TypeScript
2. Repl.it
3. Discord.js (v12.5.3)

# Discord.js のバージョン
Repl.it の環境は，Discord.js の最新バージョン（v13）に対応していません．
このバージョンに必要な Node のバージョンは12です．
しかし，Repl.it で使える Node のバージョンは 10.24.1 なので，これに対応している v12 の最終バージョンを使用します．

# Bot の作成
[Discord Developer Portal](https://discord.com/developers/applications) で Bot を作成・招待してください．
詳細は[この記事](https://note.com/exteoi/n/nf1c37cb26c41)の「Discord上のBotの作成」の項で分かりやすく説明されているので，こちらを参照ください．

https://discord.com/developers/applications
https://note.com/exteoi/n/nf1c37cb26c41

# Bot の実行
以下の手順で Bot の実行ができます．
1. 作成した Bot のトークンをコピー
2. Repl.it の環境変数 "TOKEN" に，1のトークンを設定
3. エディタ上の「Run」ボタンを押す

詳細は[この記事](https://disbot.info/bot-make/repl-djs-1/#Bot)の「Botを作成する」の項で分かりやすく説明されているので，こちらを参照ください．

https://disbot.info/bot-make/repl-djs-1/#Bot

# ソースコードの説明
Bot の実行に用いるソースコードの説明をします．

## 必要パッケージのインポート
```ts:index.ts
import * as http from 'http';
import * as discord from "discord.js"
```
- http
  - サーバーの作成に用います
- discord
  - Discord Bot の実行に用います


## コマンドの定義
```ts:index.ts
async function ping(message: discord.Message) {
  message.channel.send("pong!")
}

async function pong(message: discord.Message) {
  message.channel.send("ping!")
}
```
コマンドを関数として定義します．
- ping
  - Bot は "pong!" を返します
- pong
  - Bot は "ping!" を返します


## コマンドリストの定義
```ts:index.ts
const messageResponseDic: {
  [command: string]: (
    message: discord.Message,
    args?: string[],
  ) => void
} = {
  ping: ping,
  pong: pong,
}
```
コマンドのリストを連想配列として定義します．
今回は用いませんが，args はコマンドの引数として用います．


## メッセージに応答する関数の定義
```ts:index.ts
async function messageResponse(message: discord.Message) {
  const prefix = "!"
  const [command, ...args] = (
    message.content.slice(prefix.length).split(" ")
  )
  const isCommand = !message.author.bot && (
    message.content.startsWith(prefix)
  )
  if (!isCommand) return
  messageResponseDic[command](message, args)
}
```
メッセージに応答する関数を定義します．
最後の行でコマンドを実行します．


## Bot が行う応答のリストの定義
```ts:index.ts
const clientStateResponseDic: {
  [state: string]: (...args: any) => void
} = {
  ready: () => { console.log('This bot is ready') },
  message: messageResponse
}
```
Bot が行う応答のリストを連想配列で定義します．
- ready
  - Bot の実行準備が完了したら，コンソールに "This bot is ready" と出力します
- message
  - メッセージを送信が行われた場合に，これがコマンドであれば応答をします


## サーバーの作成
```ts:index.ts
function requestListener(
  _: http.IncomingMessage,
  res: http.ServerResponse
) {
  res.write("This bot is online")
  res.end()
}

http.createServer(requestListener).listen(8080)
```
Bot の実行に用いるサーバーを作成します．

## Bot の実行
```ts:index.ts
const client = new discord.Client()
for (let [state, response] of Object.entries(clientStateResponseDic)) {
  client.on(state, response)
}
```
Bot を実行する処理です．
Bot の状態に応じて，Bot は応答を行います．


## Bot のログイン
```ts:index.ts
const token = process.env['TOKEN']
client.login(token)
```
上で定義したトークンを用いて，Bot のサーバーへのログインを行います．

# まとめ
以上が TypeScript と Repl.it で Discord Bot を動かす方法でした．

# 参考資料
- Bot の作成

https://note.com/exteoi/n/nf1c37cb26c41

- Bot の実行

https://disbot.info/bot-make/repl-djs-1/#Bot