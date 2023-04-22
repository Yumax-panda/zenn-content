---
title: "discord.py 入門　その5"
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python", "discord.py", "discordbot", "無料", "入門"]
published: true
---

### おさらい

[前回](https://zenn.dev/yumax_panda/articles/bot-tutorial-4)はミュート機能の実装をしました。今回は管理者限定の条件を付け、さらにミュート解除の機能も実装しましょう。

前回までのコード（再掲）

```python
# bot.py

from discord.ext import commands
from discord.utils import get
import discord

from io import StringIO
from datetime import datetime, timedelta

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

@bot.command(
    name="message",
    aliases=["msg", "m"],
)
async def get_message(ctx: commands.Context, channel: discord.TextChannel) -> None:
    """チャンネルのメッセージを取得し、テキストファイルに保存するコマンド"""

    stream = StringIO()

    async for message in channel.history(
        after=datetime.utcnow() -timedelta(hours=1),
        oldest_first=True,
    ):
        jst = message.created_at + timedelta(hours=9) # UTC -> JST
        msg = f"{message.author.name}: {jst.strftime('%Y/%m/%d %H:%M:%S')}\n{message.content}"
        stream.write(msg)
        stream.write("\n\n")

    stream.seek(0)
    await ctx.send(file=discord.File(stream, filename="messages.txt"))
    stream.close()


@bot.command(name="mute")
async def mute(ctx: commands.Context, member: discord.Member) -> None:

    if (role := get(member.roles, name="チャット制限")) is None: # ロールがサーバーに存在しない場合
        # ロールを作成
        role = await ctx.guild.create_role(
            name="チャット制限", # ロール名
            mentionable=True # メンションできるようにする
        )

    await member.add_roles(role) # メンバーにロールを付与
    await ctx.send(f"{member.mention} をチャット制限しました。")

@bot.event
async def on_message(message: discord.Message) -> None:

    if message.author.bot: # botからのメッセージは無視
        return

    # チャット制限ロールをメッセージ送信者が持っているか確認
    role = get(message.author.roles, name="チャット制限")

    if role is not None:
        await message.delete() # メッセージを削除
        return # ミュートされている場合はコマンドを実行しない

    await bot.process_commands(message) # コマンドを実行


bot.run("TOKEN")
```

### Step6. 管理者限定の条件を付けよう

一番最初に思いつく方法としたら、以下のように`if`文を用いて実装するものでしょう。

```python
@bot.command(name="mute")
async def mute(ctx: commands.Context, member: discord.Member) -> None:

    if ctx.author.id !=  12345678: # 管理者のID
        return # 管理者以外は実行できないようにする

    # 以下同じ
```

この方法でも問題はありませんが`discord.ext.commands`には、これを簡単に実装できる機能が用意されています。以下のように、もとのコードに1行足すだけで実装できます。

```python
@bot.command(name="mute")
@commands.is_owner() # 管理者のみ実行可能にする
async def mute(ctx: commands.Context, member: discord.Member) -> None:

    # 以下同じ
```

`@commands.is_owner()`は、管理者のみ実行可能にする機能です。管理者以外が実行しようとすると、`commands.NotOwner`というエラーが発生します。

### Step7. ミュート解除の機能を実装しよう

これもStep6のときと同じように管理者限定のコマンドにします。今まで学んだことで実装できるので、まずはご自身で考えた後に以下の実装例を見てみましょう。ちなみにロールをメンバーから剥奪するメソッドは`await member.remove_roles(role)`です。

[メソッドのドキュメント](https://discordpy.readthedocs.io/ja/latest/api.html?highlight=member#discord.Member.remove_roles)

以下は実装例です。

```python
@bot.command(name="unmute")
@commands.is_owner() # 管理者のみ実行可能にする
async def unmute(ctx: commands.Context, member: discord.Member) -> None:

    if (role := get(member.roles, name="チャット制限")) is None: # ロールがサーバーに存在しない場合
        await ctx.send(f"{member.mention} はミュートされていません。")
        return

    await member.remove_roles(role) # メンバーからロールを剥奪
    await ctx.send(f"{member.mention} のミュートを解除しました。")
```

### まとめ

今回は`commands.is_owner`デコレータを用いると、管理者限定の機能を簡単に実装できることを学びました。
次回はエラー処理について学んでいきます。お疲れ様でした。


[前回](https://zenn.dev/yumax_panda/articles/bot-tutorial-4) | [次回](https://zenn.dev/yumax_panda/articles/bot-tutorial-6)