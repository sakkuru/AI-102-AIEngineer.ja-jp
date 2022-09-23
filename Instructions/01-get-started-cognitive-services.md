---
lab:
  title: Cognitive Services の概要
  module: Module 2 - Developing AI Apps with Cognitive Services
---

# <a name="get-started-with-cognitive-services"></a>Cognitive Services の概要

In this exercise, you'll get started with Cognitive Services by creating a <bpt id="p1">**</bpt>Cognitive Services<ept id="p1">**</ept> resource in your Azure subscription and using it from a client application. The goal of the exercise is not to gain expertise in any particular service, but rather to become familiar with a general pattern for provisioning and working with cognitive services as a developer.

## <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

If you have not already cloned <bpt id="p1">**</bpt>AI-102-AIEngineer<ept id="p1">**</ept> code repository to the environment where you're working on this lab, follow these steps to do so. Otherwise, open the cloned folder in Visual Studio Code.

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git:Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるダイアログが表示された場合は、 **[今はしない]** を選択します。

## <a name="provision-a-cognitive-services-resource"></a>Cognitive Services リソースをプロビジョニングする

Azure Cognitive Services are cloud-based services that encapsulate artificial intelligence capabilities you can incorporate into your applications. You can provision individual cognitive services resources for specific APIs (for example, <bpt id="p1">**</bpt>Language<ept id="p1">**</ept> or <bpt id="p2">**</bpt>Computer Vision<ept id="p2">**</ept>), or you can provision a general <bpt id="p3">**</bpt>Cognitive Services<ept id="p3">**</ept> resource that provides access to multiple cognitive services APIs through a single endpoint and key. In this case, you'll use a single <bpt id="p1">**</bpt>Cognitive Services<ept id="p1">**</ept> resource.

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. **[&#65291;リソースの作成]** ボタンを選択し、*Cognitive Services* を検索して、次の設定で **Cognitive Services** リソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)*
    - **[リージョン]**: 使用できるリージョンを選択します**
    - **[名前]**: *一意の名前を入力します*
    - **価格レベル**: Standard S0
3. 必要なチェック ボックスをオンにして、リソースを作成します。
4. デプロイが完了するまで待ち、デプロイの詳細を表示します。
5. この演習では、Azure サブスクリプションで **Cognitive Services** リソースを作成し、それをクライアント アプリケーションから使用して、Cognitive Services の使用を開始します。
    - クライアント アプリケーションが要求を送信できる HTTP *エンドポイント*。
    - 認証に使用できる 2 つの "*キー*" (クライアント アプリケーションはどちらのキーも認証に使用できます)。
    - この演習の目標は、特定のサービスに関する専門知識を習得することではなく、開発者として Cognitive Services をプロビジョニングおよび操作するための一般的なパターンに精通することです。

## <a name="use-a-rest-interface"></a>REST インターフェイスの使用

The cognitive services APIs are REST-based, so you can consume them by submitting JSON requests over HTTP. In this example, you'll explore a console application that uses the <bpt id="p1">**</bpt>Language<ept id="p1">**</ept> REST API to perform language detection; but the basic principle is the same for all of the APIs supported by the Cognitive Services resource.

> <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: In this exercise, you can choose to use the REST API from either <bpt id="p2">**</bpt>C#<ept id="p2">**</ept> or <bpt id="p3">**</bpt>Python<ept id="p3">**</ept>. In the steps below, perform the actions appropriate for your preferred language.

1. Visual Studio Code の **[エクスプローラー]** ウィンドウで、**01-getting-started** フォルダーを参照し、言語の設定に応じて **C-Sharp** フォルダーまたは **Python** フォルダーを展開します。
2. **rest-client** フォルダーの内容を表示し、構成設定用のファイルが含まれていることにご注意ください。
    - **C#** : appsettings.json
    - **Python**: .env

    このラボで作業している環境に **AI-102-AIEngineer** コードのリポジトリをまだクローンしていない場合は、次の手順に従ってクローンします。
3. **rest-client** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることにご注意ください。

    - **C#** : Program.cs
    - **Python**: rest-client.py

    コード ファイルを開き、含まれているコードを確認して、次の詳細に注意してください。
    - HTTP 通信を可能にするために、さまざまな名前空間がインポートされます
    - **Main** 関数のコードは、Cognitive Services リソースのエンドポイントとキーを取得します。これらは、REST リクエストを Text Analytics サービスに送信するために使用されます。
    - プログラムはユーザー入力を受け入れ、**GetLanguage** 関数を使用して、Cognitive Services エンドポイントの Text Analytics 言語検出 REST API を呼び出して、入力されたテキストの言語を検出します。
    - API に送信される要求は、入力データを含む JSON オブジェクトで構成されます。この場合、**ドキュメント** オブジェクトのコレクションであり、それぞれに **ID** と**テキスト**があります。
    - サービスのキーは、クライアント アプリケーションを認証するためのリクエスト ヘッダーに含まれています。
    - サービスからの応答は JSON オブジェクトであり、クライアント アプリケーションはこれを解析できます。
4. それ以外の場合は、複製されたフォルダーを Visual Studio Code で開きます。

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python rest-client.py
    ```

5. When prompted, enter some text and review the language that is detected by the service, which is returned in the JSON response. For example, try entering "Hello", "Bonjour", and "Gracias".
6. アプリケーションのテストが終了したら、「quit」と入力してプログラムを終了します。

## <a name="use-an-sdk"></a>SDK を使用する

You can write code that consumes cognitive services REST APIs directly, but there are software development kits (SDKs) for many popular programming languages, including Microsoft C#, Python, and Node.js. Using an SDK can greatly simplify development of applications that consume cognitive services.

1. Visual Studio Code の **[エクスプローラー]** ウィンドウの **01-getting-started** フォルダーで、言語の設定に応じて **C-Sharp** フォルダーまたは **Python** フォルダーを展開します。
2. Right-click the <bpt id="p1">**</bpt>sdk-client<ept id="p1">**</ept> folder and open an integrated terminal. Then install the Text Analytics SDK package by running the appropriate command for your language preference:

    **C#**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.1.0
    ```

    **Python**

    ```
    pip install azure-ai-textanalytics==5.1.0
    ```

3. **sdk-client** フォルダーの内容を表示し、構成設定用のファイルが含まれていることにご注意ください。
    - **C#** : appsettings.json
    - **Python**: .env

    Open the configuration file and update the configuration values it contains to reflect the <bpt id="p1">**</bpt>endpoint<ept id="p1">**</ept> and an authentication <bpt id="p2">**</bpt>key<ept id="p2">**</ept> for your cognitive services resource. Save your changes.
    
4. **sdk-client** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることにご注意ください。

    - **C#** : Program.cs
    - **Python**: sdk-client.py

    コード ファイルを開き、含まれているコードを確認して、次の詳細に注意してください。
    - インストールした SDK の名前空間がインポートされます
    - **Main** 関数のコードは、Cognitive Services リソースのエンドポイントとキーを取得します。これらは SDK で使用され、Text Analytics サービスのクライアントを作成します。
    - **GetLanguage** 関数は、SDK を使用してサービスのクライアントを作成し、クライアントを使用して入力されたテキストの言語を検出します。
5. **sdk-client** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python sdk-client.py
    ```

6. When prompted, enter some text and review the language that is detected by the service. For example, try entering "Goodbye", "Au revoir", and "Hasta la vista".
7. アプリケーションのテストが終了したら、「quit」と入力してプログラムを終了します。

> **注**: Unicode 文字セットを必要とする一部の言語は、この単純なコンソール アプリケーションでは認識されない場合があります。

## <a name="more-information"></a>詳細情報

Azure Cognitive Services の詳細については、[Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/what-are-cognitive-services) のドキュメントを参照してください。
