---
lab:
    title: 'Bot Framework SDK を使用したボットの作成'
    module: 'モジュール 7 - 会話型 AI と the Azure Bot Service'
---

# Bot Framework SDK を使用したボットの作成

*ボット*は、人間のユーザーとの会話型ダイアログに参加できるソフトウェア エージェントです。Microsoft Bot Framework は、Azure Bot Service を介してクラウド サービスとして提供できるボットを構築するための包括的なプラットフォームを提供します。

この演習では、Microsoft Bot Framework SDK を使用して、ボットを作成およびデプロイします。

## 開始する前に

初めに、ボット開発のための環境を準備しましょう。

### Bot Framework Emulator を更新する

Bot Framework SDK を使用してボットを作成し、Bot Framework Emulator を使用してボットをテストします。Bot Framework Emulator は定期的に更新されるため、最新バージョンがインストールされていることを確認しましょう。

> **注**: 更新には、この演習の手順に影響を与えるユーザー インターフェイスの変更が含まれる場合があります。

1. **Bot Framework Emulator** を起動し、アップデートのインストールを求めるメッセージが表示された場合は、現在ログインしているユーザーに対してインストールしてください。自動的にプロンプトが表示されない場合は、**[ヘルプ]** メニューの **[更新を確認する]** オプションを使用して、更新を確認してください。
2. アップデートをインストールした後、次に必要になるまで Bot FrameworkEmulator を閉じます。

### このコースのリポジトリを複製する

**AI-102-AIEngineer** コードのリポジトリをこのラボで作業している環境にまだ複製していない場合は、次の手順に従って複製してください。それ以外の場合は、複製されたフォルダーを Visual Studio Code で開きます。

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git: Clone** コマンドを実行して、 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでもかまいません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求められた場合は、**「今はしない」** を選択します。

## ボットの作成

Bot Framework SDK を使用して、テンプレートに基づいてボットを作成し、特定の要件を満たすようにコードをカスタマイズできます。

> **注**: この演習では、**C#** または **Python** のいずれかを使用することを選択できます。以下の手順で、希望する言語に適したアクションを実行します。

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
- **bot_description**: A bot for our times
    
5. ターミナル ペインで、次のコマンドを入力して、現在のディレクトリを **TimeBot** フォルダーに変更し、ボット用に生成されたコード ファイルを一覧表示します。

    ```Code
    cd TimeBot
    dir
    ```

## Bot Framework Emulator でボットをテストします

*EchoBot* テンプレートに基づいてボットを作成しました。これで、ローカルで実行し、Bot Framework Emulator (システムにインストールする必要があります) を使用してテストできます。

1. ターミナル ペインで、現在のディレクトリがボット コード ファイルを含む **TimeBot** フォルダーであることを確認してから、次のコマンドを入力してボットをローカルで実行します。

**C#**

```C#
dotnet run
```

**Python**

```Python
python app.py
```
    
ボットが起動したら、ボットが実行されているエンドポイントが表示されていることに注意してください。これは**http://localhost:3978** のようになります。

2. Bot Framework Emulator を起動し、次のように **/api/messages** パスを追加してエンドポイントを指定してボットを開きます。

    `http://localhost:3978/api/messages`

3. 会話が**ライブ チャット**ペインで開かれた後、「*Hello and welcome!*」というメッセージを待ちます。
4. 「*Hello*」などのメッセージを入力し、ボットからの応答を表示します。これにより、入力したメッセージがエコー バックされます。
5. Bot Framework Emulator を閉じて Visual Studio Code に戻り、ターミナル ウィンドウで **CTRL+C** を入力してボットを停止します。

## ボット コードの変更

ユーザーの入力をエコー バックするボットを作成しました。これは特に有用ではありませんが、会話ダイアログの基本的な流れを説明するのに役立ちます。ボットとの会話は、テキスト、グラフィック、またはユーザーインターフェイス *カード*を使用して情報を交換する一連の*アクティビティ*で構成されます。ボットは挨拶で会話を開始します。これは、ユーザーがボットとのチャットセッションを初期化したときにトリガーされる*会話更新*アクティビティの結果です。次に、会話は、ユーザーとボットが順番に*メッセージ*を送信する一連のアクティビティで構成されます。

1. Visual Studio Code で、ボットの次のコード ファイルを開きます
    - **C#**: TimeBot/Bots/EchoBot.cs
    - **Python**: TimeBot/bot.py

    このファイルのコードは*アクティビティ ハンドラー*関数で構成されていることに注意してください。1 つは*メンバーが追加した*会話更新アクティビティ (誰かがチャット セッションに参加したとき) 用で、もう 1 つは*メッセージ*アクティビティ (メッセージが受信されたとき) 用です。会話は*ターン*の概念に基づいており、各ターンは、ボットがアクティビティを受信、処理、および応答するインタラクションを表します。*ターン コンテキスト*は、現在のターンで処理されているアクティビティに関する情報を追跡するために使用されます。

2. コード ファイルの先頭に、次の名前空間インポート ステートメントを追加します。

**C#**

```C#
using System;
```

**Python**

```Python
from datetime import datetime
```

3. *メッセージ* アクティビティのアクティビティ ハンドラー関数を次のコードに一致するように変更します。

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
    
4. 変更を保存し、ターミナル ペインで、現在のディレクトリがボット コード ファイルを含む **TimeBot** フォルダーであることを確認してから、次のコマンドを入力して、ボットをローカルで実行します。

**C#**

```C#
dotnet run
```

**Python**

```Python
python app.py
```

前のように、ボットを開始するには、それが動作しているエンドポイントが示されていることに注意してください。

5. Bot Framework Emulator を起動し、次のように **/api/messages** パスを追加してエンドポイントを指定してボットを開きます。

    `http://localhost:3978/api/messages`

6. 会話が**ライブ チャット**ペインで開かれた後、「*Hello and welcome!*」というメッセージを待ちます。
7. 「*Hello*」などのメッセージを入力し、ボットからの応答を表示します。
8. 「*What is the time?*」と入力し、応答を表示します。

    ボットは、「What is the time?」というクエリに応答するようになりました。ボットが実行されている現地時間を表示します。その他のクエリの場合は、ユーザーに時刻を尋ねるように求めます。これは非常に限定されたボットであり、言語理解サービスと追加のカスタムコードとの統合によって改善される可能性がありますが、テンプレートから作成されたボットを拡張することで Bot Framework SDK を使用してソリューションを構築する方法の実用的な例として機能します。

9. Bot Framework Emulator を閉じて Visual Studio Code に戻り、ターミナル ウィンドウで **CTRL+C** を入力してボットを停止します。

## 詳細

Bot Framework の詳細については、[Bot Framework のドキュメント](https://docs.microsoft.com/azure/bot-service/index-bf-sdk)を参照してください。
