---
title: "discord.py入門　その2"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python", "discord.py", "discordbot", "無料", "入門"]
published: true
---

### おさらい

[前回](https://zenn.dev/yumax_panda/articles/bot-tutorial-1)はbotの準備とイベントについて解説しました。今回はいよいよbotのコマンドを実装していきます。

前回のコード（再掲）

```python
from discord.ext import commands
import discord

intents = discord.Intents.default()
intents.members = True # メンバー管理の権限
intents.message_content = True # メッセージの内容を取得する権限

bot = commands.Bot(
    command_prefix="$", # $コマンド名　でコマンドを実行できるようになる
    case_insensitive=True, # 大文字小文字を区別しない ($hello も $Hello も同じ!)
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

### Step3. コマンドを実装してみよう

#### Section1. 挨拶するコマンドを作ってみよう

始めに、`on_message`を削除して、以下のコードを書きます。`on_message`を削除する理由は後で述べます。

```python
@bot.command()
async def hello(ctx: commands.Context) -> None:
    """helloと返すコマンド"""
    await ctx.send(f"Hello {ctx.author.name}")
```

`commands.Context`クラスのオブジェクトである`ctx`は、コマンドを実行した人やメッセージ、チャンネルなどの情報を持っています。`ctx.send`でメッセージを送信できます。`author`属性がコマンドを実行した人の情報が入っています。`name`属性で名前を取得できます。(ちなみに、`str(author)`とすると`name#1234`のような文字列になります。)

実行結果
![](/images/bot-tutorial-2/hello.jpg)

:::message
補足
実は、`on_message`イベントをそのままにしておくと、`$hello`というメッセージを送信したときに、botは`$hello`と返信してしまいます。これは、メッセージを先に受け取るイベントが発生してからコマンドが実行されるためです。解決策として以下のようにすればOKです。

```python

@bot.event
async def on_message(message: discord.Message):
    """メッセージをおうむ返しにする処理"""

    if message.author.bot: # ボットのメッセージは無視
        return

    await message.reply(message.content)
    await bot.process_commands(message) # コマンドを実行

# ---- これでもOK ----

@bot.listen()
async def on_message(message: discord.Message):
    """メッセージをおうむ返しにする処理"""

    if message.author.bot: # ボットのメッセージは無視
        return

    await message.reply(message.content)
```
実行例

![](/images/bot-tutorial-2/hello2.jpg)

前者のコードでは、`bot.process_commands`によって受け取ったメッセージをコマンドとして処理しています。後者のコードでは、`bot.listen`デコレータを使っているので、`bot.process_commands`を実行する必要はありません。(発展的な内容ですが、`listen`デコレータは、追加されたイベントの処理を非同期的に行います。イベントハンドラを**追加する**ので、同じイベントに対して複数の処理を実行できます。)

:::

コマンドを実行するにあたってコマンドを呼び出すためのキーワードを複数追加することができます。`@bot.command()`の引数の`aliases`に渡してあげましょう。

```python
@bot.command(
    name="hello", # コマンドの名前。設定しない場合は関数名
    aliases=["hi", "hey"] # $hiでも $heyでも反応するようになる
)
async def hello(ctx: commands.Context):
    ...
```

### Section2. ユーザーから引数を受け取ってみよう

今まではあらかじめ返答する内容が決まっていましたが、次はユーザーから引数を受け取ってみましょう。
以下のコードを`hello`コマンドの下に追加してください。`on_message`イベントはしばらく不要なので削除してください。

```python
@bot.command()
async def add(ctx: commands.Context, a: int, b: int) -> None:
    """足し算をするコマンド"""
    await ctx.send(a+b)
```

以下のようになれば成功です。

![](/images/bot-tutorial-2/add.jpg)

:::message
補足（発展）
discord.pyやPycordにはライブラリ特有の[**コンバーター**](https://discordpy.readthedocs.io/ja/latest/ext/commands/commands.html?highlight=%E3%82%B3%E3%83%B3%E3%83%90%E3%83%BC%E3%82%BF#converters)という概念があります。簡単にいうならば、コマンドのコールバック（処理）関数の引数に型ヒント(上記の場合なら`int`)を記すことによって、引数をその型へ変換してくれます。設定しない場合は`str`型です。
:::

### まとめ

今回はコマンドの基本的な実装方法について学びました。discord.pyはコマンドに関するサポートが豊富で、様々な複雑な処理を簡単に実装できるようになっています。本botの制作を通じ、その一部を知ってもらえれば幸いです。次回はいよいよコマンドで特定のチャンネルのメッセージをファイルにまとめて送信する機能を実装していきます。お疲れ様でした。


[前回の記事](https://zenn.dev/yumax_panda/articles/bot-tutorial-1)　| 次回の記事　