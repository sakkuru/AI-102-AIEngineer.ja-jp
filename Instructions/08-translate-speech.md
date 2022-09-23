---
lab:
  title: 音声の翻訳
  module: Module 4 - Building Speech-Enabled Applications
---

# <a name="translate-speech"></a>音声の翻訳

The <bpt id="p1">**</bpt>Speech<ept id="p1">**</ept> service includes a <bpt id="p2">**</bpt>Speech translation<ept id="p2">**</ept> API that you can use to translate spoken language. For example, suppose you want to develop a translator application that people can use when traveling in places where they don't speak the local language. They would be able to say phrases such as "Where is the station?" or "I need to find a pharmacy" in their own language, and have it translate them to the local language.

<bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: This exercise requires that you are using a computer with speakers/headphones. For the best experience, a microphone is also required. Some hosted virtual environments may be able to capture audio from your local microphone, but if this doesn't work (or you don't have a microphone at all), you can use a provided audio file for speech input. Follow the instructions carefully, as you'll need to choose different options depending on whether you are using a microphone or the audio file.

## <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

**AI-102-AIEngineer** コード リポジトリをこのラボの作業をしている環境に既にクローンしている場合は、Visual Studio Code で開きます。それ以外の場合は、次の手順に従って今すぐクローンしてください。

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git:Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるダイアログが表示された場合は、 **[今はしない]** を選択します。

## <a name="provision-a-cognitive-services-resource"></a>Cognitive Services リソースをプロビジョニングする

サブスクリプションにまだない場合は、**Cognitive Services** リソースをプロビジョニングする必要があります。

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. **[&#65291;リソースの作成]** ボタンを選択し、*Cognitive Services* を検索して、次の設定で **Cognitive Services** リソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)*
    - **[リージョン]**: 使用できるリージョンを選択します**
    - **[名前]**: *一意の名前を入力します*
    - **価格レベル**: Standard S0
3. 必要なチェック ボックスをオンにして、リソースを作成します。
4. デプロイが完了するまで待ち、デプロイの詳細を表示します。
5. When the resource has been deployed, go to it and view its <bpt id="p1">**</bpt>Keys and Endpoint<ept id="p1">**</ept> page. You will need one of the keys and the location in which the service is provisioned from this page in the next procedure.

## <a name="prepare-to-use-the-speech-translation-service"></a>音声翻訳サービスを使用する準備をする

この演習では、Speech SDK を使用して音声を認識、翻訳、および合成する、部分的に実装されたクライアント アプリケーションを完成させます。

> **Speech** サービスには、話し言葉の翻訳に使用できる**音声翻訳** API が含まれています。

1. Visual Studio Code の**エクスプローラー** ペインで、**08-speech-translation** フォルダーを参照し、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. たとえば、現地の言語を話さない場所を旅行するときに使用できる翻訳アプリケーションを開発するとします。

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.19.0
    ```
    
    **Python**
    
    ```
    pip install azure-cognitiveservices-speech==1.19.0
    ```

3. **translator** フォルダーの内容を表示します。また、構成設定用のファイルが含まれていることに注意してください。
    - **C#** : appsettings.json
    - **Python**: .env

    「駅はどこですか?」のようなフレーズを言ったり、
4. **translator** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることに注意してください。

    - **C#** : Program.cs
    - **Python**: translator.py

    または、「薬局を見つける必要があります」を独自の言語で言ったり、それを現地の言語に翻訳することができます。

    **C#**
    
    ```C#
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.CognitiveServices.Speech.Translation;
    ```
    
    **Python**
    
    ```Python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

5. In the <bpt id="p1">**</bpt>Main<ept id="p1">**</ept> function, note that code to load the cognitive services key and region from the configuration file has already been provided. You must use these variables to create a <bpt id="p1">**</bpt>SpeechTranslationConfig<ept id="p1">**</ept> for your cognitive services resource, which you will use to translate spoken input. Add the following code under the comment <bpt id="p1">**</bpt>Configure translation<ept id="p1">**</ept>:

    **C#**
    
    ```C#
    // Configure translation
    translationConfig = SpeechTranslationConfig.FromSubscription(cogSvcKey, cogSvcRegion);
    translationConfig.SpeechRecognitionLanguage = "en-US";
    translationConfig.AddTargetLanguage("fr");
    translationConfig.AddTargetLanguage("es");
    translationConfig.AddTargetLanguage("hi");
    Console.WriteLine("Ready to translate from " + translationConfig.SpeechRecognitionLanguage);
    ```
    
    **Python**
    
    ```Python
    # Configure translation
    translation_config = speech_sdk.translation.SpeechTranslationConfig(cog_key, cog_region)
    translation_config.speech_recognition_language = 'en-US'
    translation_config.add_target_language('fr')
    translation_config.add_target_language('es')
    translation_config.add_target_language('hi')
    print('Ready to translate from',translation_config.speech_recognition_language)
    ```

6. **注**: この演習では、スピーカー/ヘッドフォンを備えたコンピューターを使用している必要があります。

    **C#**
    
    ```C#
    // Configure speech
    speechConfig = SpeechConfig.FromSubscription(cogSvcKey, cogSvcRegion);
    ```
    
    **Python**
    
    ```Python
    # Configure speech
    speech_config = speech_sdk.SpeechConfig(cog_key, cog_region)
    ```

7. 変更を保存して、**translator** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python translator.py
    ```

8. 最良のエクスペリエンスのため、マイクも必要です。

## <a name="implement-speech-translation"></a>音声翻訳の実装

認知サービス リソースに音声サービス用の **SpeechTranslationConfig** が用意されたので、**Speech translation** API を使用して音声を認識および翻訳できます。

### <a name="if-you-have-a-working-microphone"></a>マイクが機能する場合

1. プログラムの **Main** 関数で、コードが **Translate** 関数を使用して音声入力を翻訳していることに注意してください。
2. **Translate** 関数のコメント **「Translate speech」** の下に、次のコードを追加して、入力にデフォルトのシステムマイクを使用して音声を認識および翻訳するために使用できる **TranslationRecognizer** クライアントを作成します。

    **C#**
    
    ```C#
    // Translate speech
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Speak now...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```
    
    **Python**
    
    ```Python
    # Translate speech
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config)
    print("Speak now...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

    > 一部のホストされる仮想環境では、ローカル マイクから音声をキャプチャできる場合があります。しかし、これが機能しない場合 (または、マイクがない場合)、音声入力用に付属の音声ファイルを使用できます。

3. 下の「**プログラムを実行する**」セクションにスキップします。

### <a name="alternatively-use-audio-input-from-a-file"></a>または、ファイルからの温泉入力を使用します

1. ターミナル ウィンドウで、次のコマンドを入力して、音声ファイルの再生に使用できるライブラリをインストールします。

    **C#**

    ```
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**

    ```
    pip install playsound==1.2.2
    ```

2. プログラムのコード ファイルで、既存の名前空間のインポートの下に、次のコードを追加して、インストールしたライブラリをインポートします。

    **C#**

    ```C#
    using System.Media;
    ```

    **Python**

    ```Python
    from playsound import playsound
    ```

3. マイクまたは音声ファイルを使用するかどうかに応じて、ことなるオプションを選択する必要があるろきは、慎重に手順に従ってください。

    **C#**
    
    ```C#
    // Translate speech
    string audioFile = "station.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Getting speech from file...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```
    
    **Python**
    
    ```Python
    # Translate speech
    audioFile = 'station.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config)
    print("Getting speech from file...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

    > <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: The code in your application translates the input to all three languages in a single call. Only the translation for the specific language is displayed, but you could retrieve any of the translations by specifying the target language code in the <bpt id="p1">**</bpt>translations<ept id="p1">**</ept> collection of the result.

### <a name="run-the-program"></a>プログラムの実行

1. 変更を保存して、**translator** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python translator.py
    ```

2. When prompted, enter a valid language code (<bpt id="p1">*</bpt>fr<ept id="p1">*</ept>, <bpt id="p2">*</bpt>es<ept id="p2">*</ept>, or <bpt id="p3">*</bpt>hi<ept id="p3">*</ept>), and then, if using a microphone, speak clearly and say "where is the station?" or some other phrase you might use when traveling abroad. The program should transcribe your spoken input and translate it to the language you specified (French, Spanish, or Hindi). Repeat this process, trying each language supported by the application. When you're finished, press ENTER to end the program.

    > <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: The TranslationRecognizer gives you around 5 seconds to speak. If it detects no spoken input, it produces a "No match" result.
    >
    > 文字エンコードの問題により、ヒンディー語への翻訳がコンソール ウィンドウに正しく表示されない場合があります。

## <a name="synthesize-the-translation-to-speech"></a>翻訳を音声に合成する

So far, your application translates spoken input to text; which might be sufficient if you need to ask someone for help while traveling. However, it would be better to have the translation spoken aloud in a suitable voice.

1. **Translate** 機能では、コメント **「Synthesize translation」** の下に次のコードを追加して、デフォルトのスピーカーから音声として翻訳を合成するために **SpeechSynthesizer** クライアントを使用します。

    **C#**
    
    ```C#
    // Synthesize translation
    var voices = new Dictionary<string, string>
                    {
                        ["fr"] = "fr-FR-HenriNeural",
                        ["es"] = "es-ES-ElviraNeural",
                        ["hi"] = "hi-IN-MadhurNeural"
                    };
    speechConfig.SpeechSynthesisVoiceName = voices[targetLanguage];
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(translation);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```
    
    **Python**
    
    ```Python
    # Synthesize translation
    voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
    }
    speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    speak = speech_synthesizer.speak_text_async(translation).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

2. 変更を保存して、**translator** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python translator.py
    ```

3. When prompted, enter a valid language code (<bpt id="p1">*</bpt>fr<ept id="p1">*</ept>, <bpt id="p2">*</bpt>es<ept id="p2">*</ept>, or <bpt id="p3">*</bpt>hi<ept id="p3">*</ept>), and then speak clearly into the microphone and say a phrase you might use when traveling abroad. The program should transcribe your spoken input and respond with a spoken translation. Repeat this process, trying each language supported by the application. When you're finished, press ENTER to end the program.

    > **注** *この例では、**SpeechTranslationConfig** を使用して音声をテキストに変換し、**SpeechConfig** を使用して翻訳を音声として合成しました。実際には **SpeechTranslationConfig** を使用して翻訳を直接合成できますが、これは 1 つの言語に翻訳する場合にのみ機能し、通常はスピーカーに直接送信されるのではなく、ファイルとして保存されるオーディオ ストリームになります。*

## <a name="more-information"></a>詳細情報

**Speech translation** API の使用の詳細については、[音声翻訳のドキュメント](https://docs.microsoft.com/azure/cognitive-services/speech-service/index-speech-translation)を参照してください。
