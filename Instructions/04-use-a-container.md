---
lab:
  title: Cognitive Services コンテナーの使用
  module: Module 2 - Developing AI Apps with Cognitive Services
---

# <a name="use-a-cognitive-services-container"></a>Cognitive Services コンテナーの使用

Using cognitive services hosted in Azure enables application developers to focus on the infrastructure for their own code while benefiting from scalable services that are managed by Microsoft. However, in many scenarios, organizations require more control over their service infrastructure and the data that is passed between services.

Many of the cognitive services APIs can be packaged and deployed in a <bpt id="p1">*</bpt>container<ept id="p1">*</ept>, enabling organizations to host cognitive services in their own infrastructure; for example in local Docker servers, Azure Container Instances, or Azure Kubernetes Services clusters. Containerized cognitive services need to communicate with an Azure-based cognitive services account to support billing; but application data is not passed to the back-end service, and organizations have greater control over the deployment configuration of their containers, enabling custom solutions for authentication, scalability, and other considerations.

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
5. When the resource has been deployed, go to it and view its <bpt id="p1">**</bpt>Keys and Endpoint<ept id="p1">**</ept> page. You will need the endpoint and one of the keys from this page in the next procedure.

## <a name="deploy-and-run-a-text-analytics-container"></a>Text Analytics コンテナーをデプロイして実行する

Azure でホストされている Cognitive Services を使用すると、アプリケーション開発者は、Microsoft が管理するスケーラブルなサービスの恩恵を受けながら、独自のコードのインフラストラクチャに集中できます。

1. Azure portal の **ホーム** ページで、 **[&#65291;リソースの作成]** ボタンを選択し、*Container Instances* を検索して、次の設定で **Container Instances** リソースを作成します。

    - **[基本]** :
        - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
        - **リソース グループ**: "*Cognitive Services リソースを含むリソース グループを選択します*"
        - **コンピューティング名**: "*一意の名前を入力します*"
        - **[リージョン]**: 使用できるリージョンを選択します**
        - **イメージのソース**:その他のレジストリ
        - **イメージの種類**: パブリック
        - **イメージ**: `mcr.microsoft.com/azure-cognitive-services/textanalytics/language:1.1.013570001-amd64`
        - **OS の種類**: Linux
        - **サイズ**: 1 vCPU、4 GB メモリ
    - **[ネットワーク]**:
        - **ネットワークの種類**: パブリック
        - **DNS 名ラベル**: "*コンテナー エンドポイントの一意の名前を入力します*"
        - **ポート**: "*TCP ポートを 80 から **5000** に変更します*"
    - **詳細**:
        - **再起動ポリシー**: 失敗時
        - **環境変数**:

            | セキュアとしてマーク | Key | Value |
            | -------------- | --- | ----- |
            | はい | `ApiKey` | "*Cognitive Services リソースのいずれかのキー*" |
            | はい | `Billing` | "*Cognitive Services リソースのエンドポイント URI*" |
            | いいえ | `Eula` | `accept` |

        - **コマンドのオーバーライド**: [ ]
    - **タグ**:
        - "*タグを追加しないでください*"

2. デプロイが完了するまで待ち、デプロイされたリソースに移動します。
3. **[概要]** ページで、コンテナー インスタンス リソースの次のプロパティを確認します。
    - **状態**: "*実行中*" である必要があります。
    - **IP アドレス**: これは、コンテナー インスタンスへのアクセスに使用できるパブリック IP アドレスです。
    - **FQDN**: これはコンテナー インスタンス リソースの "*完全修飾ドメイン名*" です。これを使用して、IP アドレスの代わりにコンテナー インスタンスにアクセスできます。

    > ただし、多くのシナリオでは、組織はサービス インフラストラクチャとサービス間で受け渡されるデータをより細かく制御する必要があります。
    >
    > ```
    > docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 mcr.microsoft.com/azure-cognitive-services/textanalytics/language Eula=accept Billing=<yourEndpoint> ApiKey=<yourKey>
    > ```
    >
    > The command will look for the image on your local machine, and if it doesn't find it there it will pull it from the <bpt id="p1">*</bpt>mcr.microsoft.com<ept id="p1">*</ept> image registry and deploy it to your Docker instance. When deployment is complete, the container will start and listen for incoming requests on port 5000.

## <a name="use-the-container"></a>コンテナーを使用する

1. Visual Studio Code の **04-containers** フォルダーで、**rest-test.cmd** を開き、含まれている **curl** コマンド (以下に表示) を編集して、 *&lt;your_ACI_IP_address_or_FQDN&gt;* をコンテナーの IP アドレスまたは FQDN に置き換えます。

    ```
    curl -X POST "http://<your_ACI_IP_address_or_FQDN>:5000/text/analytics/v3.0/languages?" -H "Content-Type: application/json" --data-ascii "{'documents':[{'id':1,'text':'Hello world.'},{'id':2,'text':'Salut tout le monde.'}]}"
    ```

2. Cognitive Services API の多くは、"*コンテナー*" にパッケージ化してデプロイできるため、組織は独自のインフラストラクチャで Cognitive Services をホストできます。たとえば、ローカル Docker サーバー、Azure Container Instances、または Azure Kubernetes Services クラスターなどです。
3. コンテナー化された Cognitive Services は、課金をサポートするために Azure ベースの Cognitive Services アカウントと通信する必要があります。ただし、アプリケーション データはバックエンド サービスに渡されず、組織はコンテナーの展開構成をより細かく制御できるため、認証、スケーラビリティ、およびその他の考慮事項に対応したスタム ソリューションを実現できます。

    ```
    rest-test
    ```

4. コマンドが 2 つの入力ドキュメント (英語とフランス語である必要があります) で検出された言語に関する情報を含む JSON ドキュメントを返すことを確認します。

## <a name="clean-up"></a>クリーンアップする

Container Instances の実験が終了したら、それを削除する必要があります。

1. Azure portal で、この演習用のリソースを作成したリソース グループを開きます。
2. Container Instances リソースを選択して削除します。

## <a name="more-information"></a>詳細情報

Cognitive Services のコンテナー化の詳細については、[Cognitive Services コンテナーのドキュメント](https://docs.microsoft.com/azure/cognitive-services/containers/)を参照してください。
