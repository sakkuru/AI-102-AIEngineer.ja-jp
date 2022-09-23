---
lab:
  title: Speech および Language Understanding サービスの使用
  module: Module 5 - Creating Language Understanding Solutions
---

# <a name="use-the-speech-and-language-understanding-services"></a>Speech および Language Understanding サービスの使用

音声サービスを Language Understanding サービスと統合して、音声入力からユーザーの意図をインテリジェントに判断できるアプリケーションを作成できます。

> <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: This exercise works best if you have a microphone. Some hosted virtual environments may be able to capture audio from your local microphone, but if this doesn't work (or you don't have a microphone at all), you can use a provided audio file for speech input. Follow the instructions carefully, as you'll need to choose different options depending on whether you are using a microphone or the audio file.

## <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

**AI-102-AIEngineer** コード リポジトリをこのラボの作業をしている環境に既にクローンしている場合は、Visual Studio Code で開きます。それ以外の場合は、次の手順に従って今すぐクローンしてください。

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git:Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるダイアログが表示された場合は、 **[今はしない]** を選択します。

## <a name="create-language-understanding-resources"></a>Language Understanding リソースの作成

If you already have Language Understanding authoring and prediction resources in your Azure subscription, you can use them in this exercise. Otherwise, follow these instructions to create them.

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. **[&#65291;リソースの作成]** ボタンを選択して、「*language understanding*」を検索し、次の設定を使用して **Language Understanding** リソースを作成します。
    - **[Create option]\(作成オプション\)**: 両方
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)*
    - **[名前]**: *一意の名前を入力します*
    - **[作成場所]**: *希望の場所を選択します*
    - **[価格レベルを作成しています]**: F0
    - **[予測の場所]**: *作成場所と<u>同じ場所</u>を選択します*
    - **予測価格レベル**: F0 (*F0 が使用できない場合は、S0 を選択*)

3. Wait for the resources to be created, and note that two Language Understanding resources are provisioned; one for authoring, and another for prediction. You can view both of these by navigating to the resource group where you created them.

## <a name="prepare-a-language-understanding-app"></a>Language Understanding アプリを準備する

If you already have a <bpt id="p1">**</bpt>Clock<ept id="p1">**</ept> app from a previous exercise, open it in the Language Understanding portal at <ph id="ph1">`https://www.luis.ai`</ph>. Otherwise, follow these instructions to create it.

1. ブラウザーの新しいタブで、`https://www.luis.ai` の Language Understanding ポータルを開きます。
2. **注**: この演習は、マイクがあると適切に行うことができます。
3. 一部のホストされる仮想環境では、ローカル マイクから音声をキャプチャできる場合があります。しかし、これが機能しない場合 (または、マイクがない場合)、音声入力用に付属の音声ファイルを使用できます。
4. 効果的な Language Understanding アプリを作成するためのヒントが表示されたパネルが表示された場合、そのパネルを閉じます。

## <a name="train-and-publish-the-app-with-speech-priming"></a>*音声プライミング*でアプリをトレーニングして公開する

1. アプリがまだトレーニングされていない場合は、Language Understanding ポータルの上部にある **[トレーニング]** を選択して、アプリをトレーニングします。
2. マイクまたは音声ファイルを使用するかどうかに応じて、ことなるオプションを選択する必要があるろきは、慎重に手順に従ってください。
3. 公開が完了したら、Language Understanding ポータルの上部にある **[管理]** を選択します。
4. On the <bpt id="p1">**</bpt>Settings<ept id="p1">**</ept> page, note the <bpt id="p2">**</bpt>App ID<ept id="p2">**</ept>. Client applications need this to use your app.
5. **[Azure リソース]** ページの **[予測リソース]** で、予測リソースがリストされていない場合は、Azure サブスクリプションに予測リソースを追加します。
6. Note the <bpt id="p1">**</bpt>Primary Key<ept id="p1">**</ept>, <bpt id="p2">**</bpt>Secondary Key<ept id="p2">**</ept>, and <bpt id="p3">**</bpt>Location<ept id="p3">**</ept> (<bpt id="p4">&lt;u&gt;</bpt>not<ept id="p4">&lt;/u&gt;</ept> endpoint!) for the prediction resource. Speech SDK client applications need the location and one of the keys to connect to the prediction resource and be authenticated.

## <a name="configure-a-client-application-for-language-understanding"></a>Language Understanding 用クライアント アプリケーションを構成する

この演習では、音声入力を受け入れ、Language Understanding アプリを使用して、ユーザーに意図を予測するクライアント アプリケーションを作成します。

> <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: You can choose to use the SDK for either <bpt id="p2">**</bpt>C#<ept id="p2">**</ept> or <bpt id="p3">**</bpt>Python<ept id="p3">**</ept> in this exercise. In the steps that follow, perform the actions appropriate for your preferred language.

1. Visual Studio Code の**エクスプローラー** ペインで、**11-luis-speech** フォルダーを参照し、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. **speaking-clock-client** フォルダーの内容を表示し、構成設定用のファイルが含まれていることに注意してください。
    - **C#** : appsettings.json
    - **Python**: .env

    構成ファイルを開き、含まれている構成値を更新して、Language Understanding アプリの**アプリ ID** を含めます。**場所** (完全なエンドポイントでは<u>ありません</u> - たとえば、*eastus*) とその予測リソースの**キー**の 1 つ (Language Understanding ポータルのアプリの **[管理]** ページから)。

## <a name="install-sdk-packages"></a>SDK パッケージをインストールする

Speech SDK を Language Understanding と共に使用するには、使用するプログラム言語用の Speech SDK を印刷インストールする必要があります。

1. In Visual Studio, right-click the <bpt id="p1">**</bpt>speaking-clock-client<ept id="p1">**</ept> folder and open an integrated terminal. Then install the Language Understanding SDK package by running the appropriate command for your language preference:

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.14.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-speech==1.14.0
    ```

2. Additionally, if your system does <bpt id="p1">&lt;u&gt;</bpt>not<ept id="p1">&lt;/u&gt;</ept> have a working microphone, you will need to use an audio file to provide spoken input for your application. In this case, use the following commands to install an additional package so your program can play the audio file (you can skip this if you intend to use a microphone):

    **C#**

    ```
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**

    ```
    pip install playsound==1.2.2
    ```

3. **speaking-clock-client** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることに注意してください。

    - **C#** : Program.cs
    - **Python**: speaking-clock-client.py

4. Open the code file and at the top, under the existing namespace references, find the comment <bpt id="p1">**</bpt>Import namespaces<ept id="p1">**</ept>. Then, under this comment, add the following language-specific code to import the namespaces you will need to use the Speech SDK:

    **C#**

    ```C#
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.CognitiveServices.Speech.Intent;
    ```

    **Python**

    ```Python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

5. また、システムに動作するマイクが<u>ない</u>場合、既存の名前空間の下に、次のコードをインポートして追加して、音声ファイルを再生するために使用するライブラリをインポートします。

    **C#**

    ```C#
    using System.Media;
    ```

    **Python**

    ```Python
    from playsound import playsound
    ```

## <a name="create-an-intentrecognizer"></a>*IntentRecognizer* を作成する

**IntentRecognizer** クラスは、音声入力から Language Understanding 予測を取得するために使用できるクライアント オブジェクトを提供します。

1. In the <bpt id="p1">**</bpt>Main<ept id="p1">**</ept> function, note that code to load the App ID, prediction region, and key from the configuration file has already been provided. Then find the comment <bpt id="p1">**</bpt>Configure speech service and get intent recognizer<ept id="p1">**</ept>, and add the following code depending on whether you will use a microphone or an audio file for speech input:

    ### <a name="if-you-have-a-working-microphone"></a>**機能するマイクがある場合:**

    **C#**

    ```C#
    // Configure speech service and get intent recognizer
    SpeechConfig speechConfig = SpeechConfig.FromSubscription(predictionKey, predictionRegion);
    AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    IntentRecognizer recognizer = new IntentRecognizer(speechConfig, audioConfig);
    ```

    **Python**

    ```Python
    # Configure speech service and get intent recognizer
    speech_config = speech_sdk.SpeechConfig(subscription=lu_prediction_key, region=lu_prediction_region)
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    recognizer = speech_sdk.intent.IntentRecognizer(speech_config, audio_config)
    ```

    ### <a name="if-you-need-to-use-an-audio-file"></a>**音声ファイルを使用する場合:**

    **C#**

    ```C#
    // Configure speech service and get intent recognizer
    string audioFile = "time-in-london.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    System.Threading.Thread.Sleep(2000);
    SpeechConfig speechConfig = SpeechConfig.FromSubscription(predictionKey, predictionRegion);
    AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    IntentRecognizer recognizer = new IntentRecognizer(speechConfig, audioConfig);
    ```

    **Python**

    ```Python
    # Configure speech service and get intent recognizer
    audioFile = 'time-in-london.wav'
    playsound(audioFile)
    speech_config = speech_sdk.SpeechConfig(subscription=lu_prediction_key, region=lu_prediction_region)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    recognizer = speech_sdk.intent.IntentRecognizer(speech_config, audio_config)
    ```

## <a name="get-a-predicted-intent-from-spoken-input"></a>音声入力から予測意図を取得する

音声入力から予測意図を取得するコードを実装する準備が整いました。

1. **Main** 関数の追加したコードのすぐ下に、コメント **「Get the model from the AppID and add the intents we want to use」** を見つけ、使用する意図を追加し、次のコードを追加して、(アプリ ID に基づいて) Language Understanding モデルを取得し、認識機能に識別させたい意図を指定します。

    **C#**

    ```C#
    // Get the model from the AppID and add the intents we want to use
    var model = LanguageUnderstandingModel.FromAppId(luAppId);
    recognizer.AddIntent(model, "GetTime", "time");
    recognizer.AddIntent(model, "GetDate", "date");
    recognizer.AddIntent(model, "GetDay", "day");
    recognizer.AddIntent(model, "None", "none");
    ```

    *各意図に文字列ベースの ID を指定できることに注意してください*

    **Python**

    ```Python
    # Get the model from the AppID and add the intents we want to use
    model = speech_sdk.intent.LanguageUnderstandingModel(app_id=lu_app_id)
    intents = [
        (model, "GetTime"),
        (model, "GetDate"),
        (model, "GetDay"),
        (model, "None")
    ]
    recognizer.add_intents(intents)
    ```

2. Under the comment <bpt id="p1">**</bpt>Process speech input<ept id="p1">**</ept>, add the following code, which uses the recognizer to asynchronously call the Language Understanding service with spoken input, and retrieve response. If the response includes a predicted intent, the spoken query, predicted intent, and full JSON response are displayed. Otherwise the code handles the response based on the reason returned.

**C#**

```C
// Process speech input
string intent = "";
var result = await recognizer.RecognizeOnceAsync().ConfigureAwait(false);
if (result.Reason == ResultReason.RecognizedIntent)
{
    // Intent was identified
    intent = result.IntentId;
    Console.WriteLine($"Query: {result.Text}");
    Console.WriteLine($"Intent Id: {intent}.");
    string jsonResponse = result.Properties.GetProperty(PropertyId.LanguageUnderstandingServiceResponse_JsonResult);
    Console.WriteLine($"JSON Response:\n{jsonResponse}\n");
    
    // Get the first entity (if any)

    // Apply the appropriate action
    
}
else if (result.Reason == ResultReason.RecognizedSpeech)
{
    // Speech was recognized, but no intent was identified.
    intent = result.Text;
    Console.Write($"I don't know what {intent} means.");
}
else if (result.Reason == ResultReason.NoMatch)
{
    // Speech wasn't recognized
    Console.WriteLine($"Sorry. I didn't understand that.");
}
else if (result.Reason == ResultReason.Canceled)
{
    // Something went wrong
    var cancellation = CancellationDetails.FromResult(result);
    Console.WriteLine($"CANCELED: Reason={cancellation.Reason}");
    if (cancellation.Reason == CancellationReason.Error)
    {
        Console.WriteLine($"CANCELED: ErrorCode={cancellation.ErrorCode}");
        Console.WriteLine($"CANCELED: ErrorDetails={cancellation.ErrorDetails}");
    }
}
```

**Python**

```Python
# Process speech input
intent = ''
result = recognizer.recognize_once_async().get()
if result.reason == speech_sdk.ResultReason.RecognizedIntent:
    intent = result.intent_id
    print("Query: {}".format(result.text))
    print("Intent: {}".format(intent))
    json_response = json.loads(result.intent_json)
    print("JSON Response:\n{}\n".format(json.dumps(json_response, indent=2)))
    
    # Get the first entity (if any)
    
    # Apply the appropriate action
    
elif result.reason == speech_sdk.ResultReason.RecognizedSpeech:
    # Speech was recognized, but no intent was identified.
    intent = result.text
    print("I don't know what {} means.".format(intent))
elif result.reason == speech_sdk.ResultReason.NoMatch:
    # Speech wasn't recognized
    print("Sorry. I didn't understand that.")
elif result.reason == speech_sdk.ResultReason.Canceled:
    # Something went wrong
    print("Intent recognition canceled: {}".format(result.cancellation_details.reason))
    if result.cancellation_details.reason == speech_sdk.CancellationReason.Error:
        print("Error details: {}".format(result.cancellation_details.error_details))
```

これまでに追加したコードは*意図*を識別しますが、一部の意図は*エンティティ*を参照できるため、サービスから返される JSON からエンティティ情報を抽出するコードを追加する必要があります。

3. 追加したコードで、コメント **「Get the first entity (if any)」** を見つけ、その下に次のコードを追加します。

**C#**

```C
// Get the first entity (if any)
JObject jsonResults = JObject.Parse(jsonResponse);
string entityType = "";
string entityValue = "";
if (jsonResults["entities"].HasValues)
{
    JArray entities = new JArray(jsonResults["entities"][0]);
    entityType = entities[0]["type"].ToString();
    entityValue = entities[0]["entity"].ToString();
    Console.WriteLine(entityType + ": " + entityValue);
}
```

**Python**

```Python
# Get the first entity (if any)
entity_type = ''
entity_value = ''
if len(json_response["entities"]) > 0:
    entity_type = json_response["entities"][0]["type"]
    entity_value = json_response["entities"][0]["entity"]
    print(entity_type + ': ' + entity_value)
```
    
Your code now uses the Language Understanding app to predict an intent as well as any entities that were detected in the input utterance. Your client application must now use that prediction to determine and perform the appropriate action.

4. 追加したコードの下に、**「Apply the appropriate action」** というコメントを見つけ、次のコードを追加します。このコードは、アプリケーションでサポートされている意図 (**GetTime**、**GetDate**、および **GetDay**) をチェックし、適切な応答を生成するために既存の関数を呼び出す前に、関連するエンティティが検出されたかどうかを判断します。

**C#**

```C#
// Apply the appropriate action
switch (intent)
{
    case "time":
        var location = "local";
        // Check for entities
        if (entityType == "Location")
        {
            location = entityValue;
        }
        // Get the time for the specified location
        var getTimeTask = Task.Run(() => GetTime(location));
        string timeResponse = await getTimeTask;
        Console.WriteLine(timeResponse);
        break;
    case "day":
        var date = DateTime.Today.ToShortDateString();
        // Check for entities
        if (entityType == "Date")
        {
            date = entityValue;
        }
        // Get the day for the specified date
        var getDayTask = Task.Run(() => GetDay(date));
        string dayResponse = await getDayTask;
        Console.WriteLine(dayResponse);
        break;
    case "date":
        var day = DateTime.Today.DayOfWeek.ToString();
        // Check for entities
        if (entityType == "Weekday")
        {
            day = entityValue;
        }

        var getDateTask = Task.Run(() => GetDate(day));
        string dateResponse = await getDateTask;
        Console.WriteLine(dateResponse);
        break;
    default:
        // Some other intent (for example, "None") was predicted
        Console.WriteLine("You said " + result.Text.ToLower());
        if (result.Text.ToLower().Replace(".", "") == "stop")
        {
            intent = result.Text;
        }
        else
        {
            Console.WriteLine("Try asking me for the time, the day, or the date.");
        }
        break;
}
```

**Python**

```Python
# Apply the appropriate action
if intent == 'GetTime':
    location = 'local'
    # Check for entities
    if entity_type == 'Location':
        location = entity_value
    # Get the time for the specified location
    print(GetTime(location))

elif intent == 'GetDay':
    date_string = date.today().strftime("%m/%d/%Y")
    # Check for entities
    if entity_type == 'Date':
        date_string = entity_value
    # Get the day for the specified date
    print(GetDay(date_string))

elif intent == 'GetDate':
    day = 'today'
    # Check for entities
    if entity_type == 'Weekday':
        # List entities are lists
        day = entity_value
    # Get the date for the specified day
    print(GetDate(day))

else:
    # Some other intent (for example, "None") was predicted
    print('You said {}'.format(result.text))
    if result.text.lower().replace('.', '') == 'stop':
        intent = result.text
    else:
        print('Try asking me for the time, the day, or the date.')
```

## <a name="run-the-client-application"></a>クライアント アプリケーションを実行する

1. 変更を保存し、**speaking-clock-client** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock-client.py
    ```

2. Azure サブスクリプションに Language Understanding オーサリングおよび予測リソースが既にある場合は、この演習でそれらを使用できます。

    *What's the time?*
    
    *今何時?*

    *What day is it?*

    *What is the time in London?*

    *What's the date?*

    *What date is Sunday?*

> それ以外の場合は、次の手順に従って作成してください。

## <a name="more-information"></a>詳細情報

音声と Language Understanding の統合の詳細については、[音声のドキュメント](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/intent-recognition)を参照してください。
