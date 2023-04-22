---
title: "discord.py 入門　その4"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python", "discord.py", "discordbot", "無料", "入門"]
published: false
---

### おさらい

[前回](https://zenn.dev/yumax_panda/articles/bot-tutorial-3)はチャンネルのメッセージを記録する機能の実装方法について解説しました。今回はメンバーをミュートさせる機能を実装していきます。

### Step5. メンバーをミュートしよう

ミュートの実装方法ですが、以下の要素があれば実装できそうです。

1. コマンドでミュートしたい人に「チャット制限」というロールを付与する。
2. メッセージリスナーを登録し、チャット制限ロールを持っている人がメッセージを送信したら削除する。

このままだと、bot稼働時にミュートされると解除する術がないので、ミュート解除コマンドも後ほど実装します。

#### チャット制限ロールの付与

以下のコードを実装します。

```python
from discord.utils import get # 新しく追加

#　省略

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
```

以下のようになれば成功です。プロフィールに「チャット制限」というロールが追加されていることを確認してください。ここで少し補足。`discord.utils`からインポートした`get`関数は、リストから条件に合う要素を取得する関数です。今回はメンバーのロールリストから`チャット制限`という名前のロールを取得しています。[詳しくはこちら](https://discordpy.readthedocs.io/ja/latest/api.html?highlight=utils%20get#discord.utils.get)。

:::message
今回はミュートしたい人を指定するコマンドなので、引数には`discord.Member`クラスのオブジェクトである`member`をとっていることに注意してください。
:::


![](/images/bot-tutorial-4/mute.jpg)

#### チャット制限の実装

次に、メッセージをチャット制限の人が送信したら削除する処理を実装します。
イベントリスナーの場合、チャット制限が掛かっている人もコマンドを実行できてしまうので、イベントハンドラとして実装します。

```python
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
```

チャット制限されている人はコマンドも実行できなくなることに注意してください。


### まとめ

今回はチャット制限の機能を実装しました。しかし、このままだとサーバーの管理者以外の人も他人をミュートできてしまったり、ミュートを解除することができないままです。次回はミュートの解除機能と、ミュートの権限を管理者のみにする機能を実装していきます。
次回ともよろしくお願いします。お疲れ様でした。


[前回](https://zenn.dev/yumax_panda/articles/bot-tutorial-3) | [次回](https://zenn.dev/yumax_panda/articles/bot-tutorial-5)