---
title: "discord.py入門　その1"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python", "discord.py", "discordbot", "無料", "入門"]
published: true
---

### はじめに

本記事では数回に分けて、discord.pyを使ったdiscord botの作り方を解説していきます。
あくまで筆者の全て独学の知識ですので、参考の一助になれば幸いです。

対象読者は以下のような方を想定しています。

- Pythonである程度コードを書いたことがある
- discord botを作ってみたい
- botの作り方がよくわからない
- Pythonを使ったことはあるが、discord.pyを使ったことがない
- 自分で開発したbotを無料で運用したい


### 学べること

- テキストコマンドの実装
- スラッシュコマンドの実装
- 管理機能の実装
- コードを分割して管理する方法　（コグ）
- 管理しやすいコードの書き方
- 本番環境へのデプロイ方法


### 本記事で作るbot

本記事では、最終的に以下のような機能のbotを作成します。

- チャンネルのメッセージを集め、テキストファイルにして保存する
- チャット制限機能 (特定のメンバーが発言できるかを管理)
- botの管理機能 (ログチャンネルへ情報を送信)


### Step1. 準備

#### discord.pyのインストール

本記事では、discord.pyを使ってbotを作成します。discord.pyというのは、discordのAPIをPythonで扱いやすくしたライブラリです。Pythonで書かれている多くのbotはdiscord.pyを使っています。以下のコマンドでdiscord.pyをインストールします。

```bash
python3 -m pip install -U discord.py

# Windowsの場合
py -3 -m pip install -U discord.py

# Linux環境の場合
$ apt install libffi-dev libnacl-dev python3-dev
```

本記事でのコードの動作確認は全てWindows 10で行っています。

#### botのアカウントの作成

以下のドキュメントに従って、botのアカウントを作成します。

[Botアカウント作成](https://discordpy.readthedocs.io/ja/latest/discord.html#)

「Bot」項目の「Add Bot」ボタンをクリックして、botを作成します。その際、**Privileged Gateway IntentsのSERVER MEMBERS INTENTとMESSAGE CONTENT INTENTをオン**にしてください。

![](/images/bot-tutorial-1/intents.jpg)

:::message
「Botを招待する」セクション中の6の「Bot Permissions」の項目は、Administratorにチェックを入れてください。
:::

トークンを控え、Botを自身のサーバーに導入したら準備完了です。

### Step2. Botにおうむ返しをさせてみよう

作業フォルダに`bot.py`を作成して、以下のコードを書きます。

```python
from discord.ext import commands
import discord

intents = discord.Intents.default()
intents.members = True # メンバー管理の権限
intents.message_content = True # メッセージの内容を取得する権限


# Botをインスタンス化
bot = commands.Bot(
    command_prefix="$", # $コマンド名　でコマンドを実行できるようになる
    case_insensitive=True, # コマンドの大文字小文字を区別しない ($hello も $Hello も同じ!)
    intents=intents # 権限を設定
)

@bot.event
async def on_ready():
    print("Bot is ready!")


@bot.event
async def on_message(message: discord.Message):
    """メッセージをおうむ返しにする処理"""

    if message.author.bot: # ボットのメッセージは無視
        return

    await message.reply(message.content)


bot.run("TOKEN")
```

```bash
python bot.py
```

と実行して、ログに`Bot is ready!`と表示されれば成功です。
試しに、botのサーバーに入って、"こんにちは"と送信してみましょう。botが`こんにちは`と返答してくれれば成功です。

![](/images/bot-tutorial-1/reply.jpg)

#### 解説

botのインスタンス化や、botを起動させるコードはコメントを参考にしてください。ここでは、おうむ返しを実装する上でのキーポイントとなる`on_message`について解説します。

`on_message`はメッセージが送信されたときに呼び出されるイベントハンドラで、`message`という引数が渡されます。この`message`には送信内容やメッセージの作者、メッセージが作成されたチャンネル等の情報が格納されています。`message`は、`discord.Message`というクラスのオブジェクトです。`discord.Message`には、メッセージの文字を取得する`content`という属性があります。ユーザーの送信した内容をそのまま送信するだけですので、`content`を`reply`メソッドの引数に渡しています。`reply`メソッドは、メッセージを送信したユーザーに対して、メッセージ送信します。

参考：　[discord.Message](https://discordpy.readthedocs.io/ja/latest/api.html#discord.Message)

:::message
補足
`await`は、非同期処理を同期処理のように書けるようにするものです。`await`をつけることで、`message.reply`が完了するまで、次の処理を待つことができます。ここは最初は呪文のようなものだと考えて結構です。
:::

#### イベントハンドラ

discordにはコマンド以外に、「イベント」という概念があります。イベントとは、discordの様々な動作をイベントとして定義したものです。例えば、メッセージが送信されたとき、メンバーがサーバーに参加したとき、などです。イベント名は`on_{イベント名}`として定義されていて、それぞれのイベントが発生したときに対象の関数が呼び出されます。イベントハンドラ（各イベントに対する処理）として定義するときには、`@bot.event`というデコレータ（関数に機能を追加するようなもの）をつければ大丈夫です。サンプルコードでは、`on_ready`(botが起動したとき)と`on_message`(メッセージが送信されたとき)のイベントハンドラしか参照していませんが、`on_guild_join`(botがサーバーに参加したとき)や`on_member_join`(メンバーがサーバーに参加したとき)などもあります。詳しくは[公式ドキュメント](https://discordpy.readthedocs.io/ja/latest/api.html#event-reference)を参照してください。

:::message
イベントハンドラは全てコルーチンで定義してください。つまり、`def`の代わりに`async def`を使ってください。
:::

### まとめ

いかがだったでしょうか。今回はbotのセットアップとイベントについて学習しました。次回はいよいよコマンドを実装していきます。
記事の内容について、誤りや改善点があれば是非コメントをお願いします。お疲れ様でした。

[次回の記事]