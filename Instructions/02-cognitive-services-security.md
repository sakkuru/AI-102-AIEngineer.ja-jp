---
lab:
  title: Cognitive Services セキュリティの管理
  module: Module 2 - Developing AI Apps with Cognitive Services
---

# <a name="manage-cognitive-services-security"></a>Cognitive Services セキュリティの管理

セキュリティはどのアプリケーションにとっても重要な考慮事項であり、開発者は、Cognitive Services などのリソースへのアクセスがそれを必要とする人だけに制限されていることを確認する必要があります。

Cognitive Services へのアクセスは通常、認証キーを介して制御されます。認証キーは、Cognitive Services リソースを最初に作成したときに生成されます。

## <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

**AI-102-AIEngineer** コード リポジトリをこのラボの作業をしている環境に既にクローンしている場合は、Visual Studio Code で開きます。それ以外の場合は、次の手順に従って今すぐクローンしてください。

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git:Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるダイアログが表示された場合は、 **[今はしない]** を選択します。

## <a name="provision-a-cognitive-services-resource"></a>Cognitive Services リソースをプロビジョニングする

サブスクリプションに **Cognitive Services** リソースがまだない場合は、プロビジョニングする必要があります。

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. **[&#65291;リソースの作成]** ボタンを選択し、*Cognitive Services* を検索して、次の設定で **Cognitive Services** リソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)*
    - **[リージョン]**: 使用できるリージョンを選択します**
    - **[名前]**: *一意の名前を入力します*
    - **価格レベル**: Standard S0
3. 必要なチェック ボックスをオンにして、リソースを作成します。
4. デプロイが完了するまで待ち、デプロイの詳細を表示します。

## <a name="manage-authentication-keys"></a>認証キーの管理

When you created your cognitive services resource, two authentication keys were generated. You can manage these in the Azure portal or by using the Azure command line interface (CLI).

1. In the Azure portal, go to your cognitive services resource and view its <bpt id="p1">**</bpt>Keys and Endpoint<ept id="p1">**</ept> page. This page contains the information that you will need to connect to your resource and use it from applications you develop. Specifically:
    - クライアント アプリケーションが要求を送信できる HTTP *エンドポイント*。
    - Two <bpt id="p1">*</bpt>keys<ept id="p1">*</ept> that can be used for authentication (client applications can use either of the keys. A common practice is to use one for development, and another for production. You can easily regenerate the development key after developers have finished their work to prevent continued access).
    - The <bpt id="p1">*</bpt>location<ept id="p1">*</ept> where the resource is hosted. This is required for requests to some (but not all) APIs.
2. In Visual Studio Code, right-click the <bpt id="p1">**</bpt>02-cognitive-security<ept id="p1">**</ept> folder and open an integrated terminal. Then enter the following command to sign into your Azure subscription by using the Azure CLI.

    ```
    az login
    ```

    A web browser tab will open and prompt you to sign into Azure. Do so, and then close the browser tab and return to Visual Studio Code.

    > <bpt id="p1">**</bpt>Tip<ept id="p1">**</ept>: If you have multiple subscriptions, you'll need to ensure that you are working in the one that contains your cognitive services resource.  Use this command to         determine your current subscription - its unique ID is the <bpt id="p1">**</bpt>id<ept id="p1">**</ept> value in the JSON that gets returned.

    > **警告**: `az login` で証明書の検証エラーが発生する場合は、数分待ってからもう一度試してみてください。
    >
    > ```
    > az account show
    > ```
    >
    > サブスクリプションを変更する必要がある場合は、このコマンドを実行して、 *&lt;Your_Subscription_Id&gt;* を正しいサブスクリプション ID に変更します。
    >
    > ```
    > az account set --subscription <Your_Subscription_Id>
    > ```
    >
    > または、後続の各 Azure CLI コマンドで、サブスクリプション ID を *--subscription* パラメーターとして明示的に指定することもできます。

3. これで、次のコマンドを使用して、Cognitive Services キーのリストを取得できます。 *&lt;resourceName&gt;* を Cognitive Services リソースの名前に置き換え、 *&lt;resourceGroup&gt;* を作成したリソース グループの名前に置き換えます。

    ```
    az cognitiveservices account keys list --name <resourceName> --resource-group <resourceGroup>
    ```

このコマンドは、Cognitive Services リソースのキーのリストを返します。**key1** と **key2** という名前の 2 つのキーがあります。

4. To test your cognitive service, you can use <bpt id="p1">**</bpt>curl<ept id="p1">**</ept> - a command line tool for HTTP requests. In the <bpt id="p1">**</bpt>02-cognitive-security<ept id="p1">**</ept> folder, open <bpt id="p2">**</bpt>rest-test.cmd<ept id="p2">**</ept> and edit the <bpt id="p3">**</bpt>curl<ept id="p3">**</ept> command it contains (shown below), replacing <bpt id="p4">*</bpt><ph id="ph1">&amp;lt;</ph>yourEndpoint<ph id="ph2">&amp;gt;</ph><ept id="p4">*</ept> and <bpt id="p5">*</bpt><ph id="ph3">&amp;lt;</ph>yourKey<ph id="ph4">&amp;gt;</ph><ept id="p5">*</ept> with your endpoint URI and <bpt id="p6">**</bpt>Key1<ept id="p6">**</ept> key to use the Text Analytics API in your cognitive services resource.

    ```
    curl -X POST "<yourEndpoint>/text/analytics/v3.0/languages?" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: <yourKey>" --data-ascii "{'documents':[{'id':1,'text':'hello'}]}"
    ```

5. 変更を保存してから、**02-cognitive-security** フォルダーの統合ターミナルで次のコマンドを実行します。

    ```
    rest-test
    ```

このコマンドは、入力データで検出された言語に関する情報を含む JSON ドキュメントを返します (これは英語でなければなりません)。

6. If a key becomes compromised, or the developers who have it no longer require access, you can regenerate it in the portal or by using the Azure CLI. Run the following command to regenerate your <bpt id="p1">**</bpt>key1<ept id="p1">**</ept> key (replacing <bpt id="p2">*</bpt><ph id="ph1">&amp;lt;</ph>resourceName<ph id="ph2">&amp;gt;</ph><ept id="p2">*</ept> and <bpt id="p3">*</bpt><ph id="ph3">&amp;lt;</ph>resourceGroup<ph id="ph4">&amp;gt;</ph><ept id="p3">*</ept> for your resource).

    ```
    az cognitiveservices account keys regenerate --name <resourceName> --resource-group <resourceGroup> --key-name key1
    ```

Cognitive Services リソースのキーのリストが返されます。**key1** は最後に取得してから変更されていることに注意してください。

7. 古いキーを使用して **rest-test** コマンドを再実行し ( **^** キーを使用して前のコマンドを切り替えることができます)、失敗することを確認します。
8. Edit the <bpt id="p1">*</bpt>curl<ept id="p1">*</ept> command in <bpt id="p2">**</bpt>rest-test.cmd<ept id="p2">**</ept> replacing the key with the new <bpt id="p3">**</bpt>key1<ept id="p3">**</ept> value, and save the changes. Then rerun the <bpt id="p1">**</bpt>rest-test<ept id="p1">**</ept> command and verify that it succeeds.

> <bpt id="p1">**</bpt>Tip<ept id="p1">**</ept>: In this exercise, you used the full names of Azure CLI parameters, such as <bpt id="p2">**</bpt>--resource-group<ept id="p2">**</ept>.  You can also use shorter alternatives, such as <bpt id="p1">**</bpt>-g<ept id="p1">**</ept>, to make your commands less verbose (but a little harder to understand).  The <bpt id="p1">[</bpt>Cognitive Services CLI command reference<ept id="p1">](https://docs.microsoft.com/cli/azure/cognitiveservices?view=azure-cli-latest)</ept> lists the parameter options for each cognitive services CLI command.

## <a name="secure-key-access-with-azure-key-vault"></a>Azure Key Vault を使用した安全なキー アクセス

You can develop applications that consume cognitive services by using a key for authentication. However, this means that the application code must be able to obtain the key. One option is to store the key in an environment variable or a configuration file where the application is deployed, but this approach leaves the key vulnerable to unauthorized access. A better approach when developing applications on Azure is to store the key securely in Azure Key Vault, and provide access to the key through a <bpt id="p1">*</bpt>managed identity<ept id="p1">*</ept> (in other words, a user account used by the application itself).

### <a name="create-a-key-vault-and-add-a-secret"></a>Key Vault を作成してシークレットを追加する

最初に、Key Vault を作成して Cognitive Services キーの *シークレット* を追加する必要があります。

1. Cognitive Services リソースの **key1** 値をメモします (またはクリップボードにコピーします)。
2. Azure portal の **[ホーム]** ページで、 **[&#65291; リソースの作成]** ボタンを選択し、*Key Vault* を検索して、次の設定で **Key Vault** リソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *Cognitive Services リソースと同じリソース グループ*
    - **キー コンテナー名**: *一意の名前を入力します*
    - **リージョン**: *Cognitive Services リソースと同じリージョン*
    - **価格レベル**: Standard
3. デプロイが完了するのを待ってから、Key Vault リソースに移動します。
4. 左側のナビゲーション ウィンドウで、**[シークレット]** ([設定] セクション内) を選択します。
5. **[+生成/インポート]** を選択し、次の設定で新しいシークレットを追加します。
    - **アップロード オプション**: 手動
    - **名前**: Cognitive-Services-Key *(後でこの名前に基づいてシークレットを取得するコードを実行するため、これを正確に一致させることが重要です)*
    - **値**: ***key1** Cognitive Services キー"*

### <a name="create-a-service-principal"></a>サービス プリンシパルの作成

To access the secret in the key vault, your application must use a service principal that has access to the secret. You'll use the Azure command line interface (CLI) to create the service principal, find its object ID, and grant access to the secret in Azure Vault.

1. Return to Visual Studio Code, and in the integrated terminal for the <bpt id="p1">**</bpt>02-cognitive-security<ept id="p1">**</ept> folder, run the following Azure CLI command, replacing <bpt id="p2">*</bpt><ph id="ph1">&amp;lt;</ph>spName<ph id="ph2">&amp;gt;</ph><ept id="p2">*</ept> with a suitable name for an application identity (for example, <bpt id="p3">*</bpt>ai-app<ept id="p3">*</ept>). Also replace <bpt id="p1">*</bpt><ph id="ph1">&amp;lt;</ph>subscriptionId<ph id="ph2">&amp;gt;</ph><ept id="p1">*</ept> and <bpt id="p2">*</bpt><ph id="ph3">&amp;lt;</ph>resourceGroup<ph id="ph4">&amp;gt;</ph><ept id="p2">*</ept> with the correct values for your subscription ID and the resource group containing your cognitive services and key vault resources:

    > **ヒント**: サブスクリプション ID が不明な場合は、**az account show** コマンドを使用してサブスクリプション情報を取得します。サブスクリプション ID は出力の **id** 属性です。

    ```
    az ad sp create-for-rbac -n "api://<spName>" --role owner --scopes subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>
    ```

The output of this command includes information about your new service principal. It should look similar to this:

    ```
    {
        "appId": "abcd12345efghi67890jklmn",
        "displayName": "ai-app",
        "name": "http://ai-app",
        "password": "1a2b3c4d5e6f7g8h9i0j",
        "tenant": "1234abcd5678fghi90jklm"
    }
    ```

**appId**、**password**、**tenant** の値を記録しておきます。後で必要になります (このターミナルを閉じると、パスワードを取得できなくなります。したがって、ここで値をメモすることが重要です。出力を Visual Studio Code の新しいテキスト ファイルに貼り付けて、後で必要な値を確実に見つけられるようにすることができます。)

2. サービス プリンシパルの**オブジェクト ID** を取得するには、次の Azure CLI コマンドを実行し、 *&lt;appId&gt;* をお使いのサービス プリンシパルのアプリ ID の値に置き換えます。

    ```
    az ad sp show --id <appId> --query objectId --out tsv
    ```

3. 新しいサービス プリンシパルに Key Vault のシークレットにアクセスするためのアクセス許可を割り当てるには、次の Azure CLI コマンドを実行し、 *&lt;keyVaultName&gt;* を Azure Key Vault リソースの名前に置き換え、 *&lt;objectId&gt;* をサービス プリンシパルのオブジェクト ID の値に置き換えます。

    ```
    az keyvault set-policy -n <keyVaultName> --object-id <objectId> --secret-permissions get list
    ```

### <a name="use-the-service-principal-in-an-application"></a>アプリケーションでサービス プリンシパルを使用する

これで、アプリケーションでサービス プリンシパル ID を使用する準備が整い、Key Vault にシークレット Cognitive Services キーを入力し、それを使用して Cognitive Services リソースに接続します。

> <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: In this exercise, we'll store the service principal credentials in the application configuration and use them to authenticate a <bpt id="p2">**</bpt>ClientSecretCredential<ept id="p2">**</ept> identity in your application code. This is fine for development and testing, but in a real production application, an administrator would assign a <bpt id="p1">*</bpt>managed identity<ept id="p1">*</ept> to the application so that it uses the service principal identity to access resources, without caching or storing the password.

1. Visual Studio Code で、**02-cognitive-security** フォルダーと、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. Right-click the <bpt id="p1">**</bpt>keyvault-client<ept id="p1">**</ept> folder and open an integrated terminal. Then install the packages you will need to use Azure Key Vault and the Text Analytics API in your cognitive services resource by running the appropriate command for your language preference:

    **C#**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.1.0
    dotnet add package Azure.Identity --version 1.5.0
    dotnet add package Azure.Security.KeyVault.Secrets --version 4.2.0-beta.3
    ```

    **Python**

    ```
    pip install azure-ai-textanalytics==5.1.0
    pip install azure-identity==1.5.0
    pip install azure-keyvault-secrets==4.2.0
    ```

3. **keyvault-client** フォルダーの内容を表示し、構成設定用のファイルが含まれていることに注意してください
    - **C#** : appsettings.json
    - **Python**: .env

    構成ファイルを開き、構成値を更新します。次の設定を反映するために含まれます
    
    - Cognitive Services リソースの**エンドポイント**
    - **Azure Key Vault** リソースの名前
    - サービス プリンシパルの**テナント**
    - サービス プリンシパルの **appId**
    - サービス プリンシパルの**パスワード**

     変更を保存します。
4. **keyvault-client** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることに注意してください。

    - **C#** : Program.cs
    - **Python**: keyvault-client.py

    コード ファイルを開き、含まれているコードを確認して、次の詳細に注意してください。
    - インストールした SDK の名前空間がインポートされます
    - **Main** 関数のコードはアプリケーション構成設定を取得します。次に、サービス プリンシパルの資格情報を使用して、Key Vault から Cognitive Services キーを取得します。
    - **GetLanguage** 関数は、SDK を使用してサービスのクライアントを作成し、クライアントを使用して入力されたテキストの言語を検出します。
5. **keyvault-client** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python keyvault-client.py
    ```

6. When prompted, enter some text and review the language that is detected by the service. For example, try entering "Hello", "Bonjour", and "Gracias".
7. アプリケーションのテストが終了したら、「quit」と入力してプログラムを終了します。

## <a name="more-information"></a>詳細情報

Cognitive Services のセキュリティ保護の詳細については、[Cognitive Services のセキュリティ ドキュメント](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-security)を参照してください。
