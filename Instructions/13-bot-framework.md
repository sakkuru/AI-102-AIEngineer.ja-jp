---
lab:
  title: Bot Framework SDK を使用したボットの作成
  module: Module 7 - Conversational AI and the Azure Bot Service
---

# <a name="create-a-bot-with-the-bot-framework-sdk"></a>Bot Framework SDK を使用したボットの作成

<bpt id="p1">*</bpt>Bots<ept id="p1">*</ept> are software agents that can participate in conversational dialogs with human users. The Microsoft Bot Framework provides a comprehensive platform for building bots that can be delivered as cloud services through the Azure Bot Service.

この演習では、Microsoft Bot Framework SDK を使用して、ボットを作成およびデプロイします。

## <a name="before-you-start"></a>開始する前に

初めに、ボット開発のための環境を準備しましょう。

### <a name="update-the-bot-framework-emulator"></a>Bot Framework Emulator を更新する

You're going to use the Bot Framework SDK to create your bot, and the Bot Framework Emulator to test it. The Bot Framework Emulator is updated regularly, so let's make sure you have the latest version installed.

> **注**: 更新には、この演習の手順に影響を与えるユーザー インターフェイスの変更が含まれる場合があります。

1. Start the <bpt id="p1">**</bpt>Bot Framework Emulator<ept id="p1">**</ept>, and if you are prompted to install an update, do so for the currently logged in user. If you are not prompted automatically, use the <bpt id="p1">**</bpt>Check for update<ept id="p1">**</ept> option on the <bpt id="p2">**</bpt>Help<ept id="p2">**</ept> menu to check for updates.
2. アップデートをインストールした後、次に必要になるまで Bot FrameworkEmulator を閉じます。

> *ボット*は、人間のユーザーとの会話型ダイアログに参加できるソフトウェア エージェントです。

### <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

Microsoft Bot Framework は、Azure Bot Service を介してクラウド サービスとして提供できるボットを構築するための包括的なプラットフォームを提供します。

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git:Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるダイアログが表示された場合は、 **[今はしない]** を選択します。

## <a name="create-a-bot"></a>ボットの作成

Bot Framework SDK を使用して、テンプレートに基づいてボットを作成し、特定の要件を満たすようにコードをカスタマイズできます。

> <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: In this exercise, you can choose to use either <bpt id="p2">**</bpt>C#<ept id="p2">**</ept> or <bpt id="p3">**</bpt>Python<ept id="p3">**</ept>. In the steps below, perform the actions appropriate for your preferred language.

1. Visual Studio Code の**エクスプローラー** ペインで、**13-bot-framework** フォルダーを参照し、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. 選択した言語のフォルダーを右クリックして、統合ターミナルを開きます。
3. ターミナルで、次のコマンドを実行して、必要なボット テンプレートとパッケージをインストールします

**C#**

```C#
dotnet new -i Microsoft.Bot.Framework.CSharp.EchoBot
dotnet new -i Microsoft.Bot.Framework.CSharp.CoreBot
dotnet new -i Microsoft.Bot.Framework.CSharp.EmptyBot
```

**Python**

```Python
pip install botbuilder-core
pip install asyncio
pip install aiohttp
pip install cookiecutter==1.7.0
```

4. テンプレートとパッケージをインストールしたら、次のコマンドを実行して、*EchoBot* テンプレートに基づいてボットを作成します。

**C#**

```C#
dotnet new echobot -n TimeBot
```

**Python**

```Python
cookiecutter https://github.com/microsoft/botbuilder-python/releases/download/Templates/echo.zip
```

Python を使用している場合、cookiecutter のプロンプトが表示されたら、次の詳細を入力します
- **bot_name**: TimeBot
- **bot_description**: 私たちの時代のボット
    
5. ターミナル ペインで、次のコマンドを入力して、現在のディレクトリを **TimeBot** フォルダーに変更し、ボット用に生成されたコード ファイルを一覧表示します。

    ```Code
    cd TimeBot
    dir
    ```

## <a name="test-the-bot-in-the-bot-framework-emulator"></a>Bot Framework Emulator でボットをテストします

You've created a bot based on the <bpt id="p1">*</bpt>EchoBot<ept id="p1">*</ept> template. Now you can run it locally and test it by using the Bot Framework Emulator (which should be installed on your system).

1. ターミナル ペインで、現在のディレクトリがボット コード ファイルを含む **TimeBot** フォルダーであることを確認してから、次のコマンドを入力してボットをローカルで実行します。

**C#**

```C#
dotnet run
```

**Python**

```Python
python app.py
```
    
When the bot starts, note the endpoint at which it is running is shown. This should be similar to <bpt id="p1">**</bpt><ph id="ph1">http://localhost:3978</ph><ept id="p1">**</ept>.

2. Bot Framework Emulator を起動し、次のように **/api/messages** パスを追加し、エンドポイントを指定してボットを開きます。

    `http://localhost:3978/api/messages`

3. 会話が **[ライブ チャット]** ペインで開かれた後、「*Hello and welcome!*」というメッセージを待ちます。
4. 「*Hello*」などのメッセージを入力し、ボットからの応答を表示します。これにより、入力したメッセージがエコー バックされます。
5. Bot Framework Emulator を閉じて Visual Studio Code に戻り、ターミナル ウィンドウで **CTRL + C** を入力してボットを停止します。

## <a name="modify-the-bot-code"></a>ボット コードの変更

You've created a bot that echoes the user's input back to them. It's not particularly useful, but serves to illustrate the basic flow of a conversational dialog. A conversation with a bot consists of a sequence of <bpt id="p1">*</bpt>activities<ept id="p1">*</ept>, in which text, graphics, or user interface <bpt id="p2">*</bpt>cards<ept id="p2">*</ept> are used to exchange information. The bot begins the conversation with a greeting, which is the result of a <bpt id="p1">*</bpt>conversation update<ept id="p1">*</ept> activity that is triggered when a user initializes a chat session with the bot. Then the conversation consists of a sequence of further activities in which the user and bot take it in turns to send <bpt id="p1">*</bpt>messages<ept id="p1">*</ept>.

1. Visual Studio Code で、ボットの次のコード ファイルを開きます
    - **C#**: TimeBot/Bots/EchoBot.cs
    - **Python**: TimeBot/bot.py

    Note that the code in this file consists of <bpt id="p1">*</bpt>activity handler<ept id="p1">*</ept> functions; one for the <bpt id="p2">*</bpt>Member Added<ept id="p2">*</ept> conversation update activity (when someone joins the chat session) and another for the <bpt id="p3">*</bpt>Message<ept id="p3">*</ept> activity (when a message is received). The conversation is based on the concept of <bpt id="p1">*</bpt>turns<ept id="p1">*</ept>, in which each turn represents an interaction in which the bot receives, processes, and responds to an activity. The <bpt id="p1">*</bpt>turn context<ept id="p1">*</ept> is used to track information about the activity being processed in the current turn.

2. コード ファイルの先頭に、次の名前空間インポート ステートメントを追加します。

**C#**

```C#
using System;
```

**Python**

```Python
from datetime import datetime
```

3. "*メッセージ*" アクティビティのアクティビティ ハンドラー関数を次のコードに一致するように変更します。

**C#**

```C#
protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    string inputMessage = turnContext.Activity.Text;
    string responseMessage = "Ask me what the time is.";
    if (inputMessage.ToLower().StartsWith("what") && inputMessage.ToLower().Contains("time"))
    {
        var now = DateTime.Now;
        responseMessage = "The time is " + now.Hour.ToString() + ":" + now.Minute.ToString("D2");
    }
    await turnContext.SendActivityAsync(MessageFactory.Text(responseMessage, responseMessage), cancellationToken);
}
```

**Python**

```Python
async def on_message_activity(self, turn_context: TurnContext):
    input_message = turn_context.activity.text
    response_message = 'Ask me what the time is.'
    if (input_message.lower().startswith('what') and 'time' in input_message.lower()):
        now = datetime.now()
        response_message = 'The time is {}:{:02d}.'.format(now.hour,now.minute)
    await turn_context.send_activity(response_message)
```
    
4. 変更を保存し、[ターミナル] ペインで、現在のディレクトリがボット コード ファイルを含む **TimeBot** フォルダーであることを確認してから、次のコマンドを入力して、ボットをローカルで実行します。

**C#**

```C#
dotnet run
```

**Python**

```Python
python app.py
```

前のように、ボットを開始するには、それが動作しているエンドポイントが示されていることに注意してください。

5. Bot Framework Emulator を起動し、次のように **/api/messages** パスを追加し、エンドポイントを指定してボットを開きます。

    `http://localhost:3978/api/messages`

6. 会話が **[ライブ チャット]** ペインで開かれた後、「*Hello and welcome!*」というメッセージを待ちます。
7. 「*Hello*」などのメッセージを入力し、ボットからの応答を表示します。これは、*Ask me what the time is* \(何時か尋ねてください\) である必要があります。
8. 「*What is the time?*」\(何時ですか?\) と入力し、応答を表示します。

    Bot Framework SDK を使用してボットを作成し、Bot Framework Emulator を使用してボットをテストします。

9. Bot Framework Emulator を閉じて Visual Studio Code に戻り、ターミナル ウィンドウで **CTRL + C** を入力してボットを停止します。

## <a name="more-information"></a>詳細情報

Bot Framework の詳細については、[Bot Framework のドキュメント](https://docs.microsoft.com/azure/bot-service/index-bf-sdk)を参照してください。
