---
title: "discord.py入門　その3"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python", "discord.py", "discordbot", "無料", "入門"]
published: false
---

### おさらい

[前回](https://zenn.dev/yumax_panda/articles/bot-tutorial-2)は簡単なテキストコマンドの実装方法について解説しました。今回からはbotのコアとなる機能を実装していきます。

前回のコード（再掲）

```python
# bot.py

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


@bot.command()
async def hello(ctx: commands.Context) -> None:
    """helloと返すコマンド"""
    await ctx.send(f"Hello {ctx.author.name}")

@bot.command()
async def add(ctx: commands.Context, a: int, b: int) -> None:
    """足し算をするコマンド"""
    await ctx.send(a+b)

bot.run("TOKEN")
```

### Step4. チャンネルのメッセージを取得しよう

コマンド実行時の処理の流れは以下のようになります。

1. ユーザーがコマンドを実行(チャンネルを指定)
2. 指定されたチャンネルのメッセージを1時間分だけ取得
3. メッセージの中身をテキストファイルに保存
4. コマンド実行者のDMへ送信

コード全体は以下のようになります。ioモジュールとdatetimeモジュールをインポートしています。

```python
from discord.ext import commands
import discord

from io import StringIO
from datetime import datetime, timedelta

# 省略

@bot.command(
    name="message",
    aliases=["msg", "m"],
)
async def get_message(ctx: commands.Context, channel: discord.TextChannel) -> None:
    """チャンネルのメッセージを取得し、テキストファイルに保存するコマンド"""

    stream = StringIO()　# テキストファイルのようなもの

    async for message in channel.history(
        after=datetime.utcnow() -timedelta(hours=1), # 1時間前から
        oldest_first=True, # 古い順に取得
    ):
        jst = message.created_at + timedelta(hours=9) # UTC -> JST
        msg = f"{message.author.name}: {jst.strftime('%Y/%m/%d %H:%M:%S')}\n{message.content}"
        stream.write(msg)　# テキストファイルに書き込む
        stream.write("\n\n")　# 改行

    stream.seek(0)　# テキストファイルの先頭に戻る
    await ctx.send(file=discord.File(stream, filename="messages.txt"))
    stream.close()　# テキストファイルを閉じる

bot.run("TOKEN")
```

テキストコマンドの引数はどれも最初は`ctx: commands.Context`です。第二引数の`channel: discord.TextChannel`はdiscordのテキストチャンネルをコマンドの引数に持つことを示しています。

:::message
補足（発展）
前回の発展的内容で触れたように、`discord.TextChannel`を型ヒントにすることでコンバーターを指定でき、ライブラリが自動的に`discord.TextChannel`のクラスへ変換してくれます。コードが簡単になることに加え、コマンドの引数で受理してくれる幅が広がります。つまり、このコマンドでは`$msg (チャンネル名)`でも`$msg (チャンネルID)`でも`$msg (チャンネルのメンション)`でもコマンドを実行できます。
:::

`discord.TextChannel.history`はチャンネルのメッセージを取得するメソッドです。`after`と`oldest_first`を指定することで、1時間前からのメッセージを古い順に取得することができます。`discord.Message.created_at`はメッセージの作成日時を返します。UTCで返されるので、日本時間に変換するために`timedelta`を使っています。

:::message
補足（かなり発展）
ファイルに書き込むのに、例えば`with open(filepath, "w") as f:`のような方法を使わないのか疑問に思った方もいるかもしれません。これは、ローカルの環境で実行する場合はその方法でも全く問題ありませんが、botをデプロイして実行する場合は、ファイルの書き込み権限がないため、エラーが発生してしまいます（例えば、GitHubにソースコードをアップロードし、GitHubと連携してコードを実行する場合）。そこで、ファイルのようなものを作成する`StringIO`を使っています。`StringIO`はファイルを作らずともファイルのように扱えるオブジェクトです。ストリームへ文字列を書き込むことでファイルのように扱っています。botで作成した画像を送信する場合も同じような方法で解決できます。
:::

以下のような動作が確認できれば成功です。

![](/images/bot-tutorial-3/msg_result.jpg)

```txt
<!-- messages.txt -->

Azure: 2023/04/22 16:37:44
こんにちは

Azure: 2023/04/22 16:37:48
hello

Azure: 2023/04/22 16:37:50
test

Azure: 2023/04/22 16:40:11
$msg <#1098766566834847824>
```

ちなみになぜ`#bot-tutorial`が`<#1098766566834847824>`になっているかというと、チャンネルのメンションの中身は実は`<#チャンネルのID>`となっているからです。

### まとめ

今回からいよいよbotのコアとなる機能を作成していきました。補足は発展的な内容が多かったですが、そこは今回は飛ばしてもらっても大丈夫です。次回は、特定のメンバーをミュートさせる機能を実装していきます。お疲れ様でした。

[前回](https://zenn.dev/yumax_panda/articles/bot-tutorial-2)
[次回](https://zenn.dev/yumax_panda/articles/bot-tutorial-4)