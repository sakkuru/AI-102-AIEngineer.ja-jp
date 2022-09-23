---
lab:
  title: Cognitive Services を監視する
  module: Module 2 - Developing AI Apps with Cognitive Services
---

# <a name="monitor-cognitive-services"></a>Cognitive Services を監視する

Azure Cognitive Services can be a critical part of an overall application infrastructure. It's important to be able to monitor activity and get alerted to issues that may need attention.

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
3. 必要なチェックボックスを選択して、リソースを作成します。
4. デプロイが完了するまで待ち、デプロイの詳細を表示します。
5. When the resource has been deployed, go to it and view its <bpt id="p1">**</bpt>Keys and Endpoint<ept id="p1">**</ept> page. Make a note of the endpoint URI - you will need it later.

## <a name="configure-an-alert"></a>アラートを構成する

Cognitive Services リソースのアクティビティを検出できるように、アラート ルールを定義して監視を開始しましょう。

1. Azure portal で、Cognitive Services リソースに移動し、( **[監視]** セクションで) **警告**ページを表示します。
2. **[+ 新しいアラート ルール]** を選択します
3. **警告ルールの作成**ページの **[スコープ]** で、Cognitive Services リソースが一覧表示されていることを確認します。
4. **[条件]** で、 **[条件の追加]** をクリックし、右側に表示される **[信号を選択]** ペインを確認します。ここで、監視する信号タイプを選択できます。
5. **[信号タイプ]** リストで **[アクティビティ ログ]** を選択し、次にフィルター処理されたリストで **[リスト キー]** を選択します。
6. 過去 6 時間のアクティビティを確認します。
7. Select the <bpt id="p1">**</bpt>Actions<ept id="p1">**</ept> tab. Note that you can specify an <bpt id="p2">*</bpt>action group<ept id="p2">*</ept>. This enables you to configure automated actions when an alert is fired - for example, sending an email notification. We won't do that in this exercise; but it can be useful to do this in a production environment.
8. **[詳細]** タブで、 **[警告ルール名]** を **[キー一覧警告]** に設定します。
9. **[Review + create]\(レビュー + 作成\)** を選択します。 
10. Azure Cognitive Services は、アプリケーション インフラストラクチャ全体の重要な部分になる可能性があります。
11. アクティビティを監視し、注意が必要な問題についてアラートを受け取ることができることが重要です。

    ```
    az login
    ```

    A web browser tab will open and prompt you to sign into Azure. Do so, and then close the browser tab and return to Visual Studio Code.

    > <bpt id="p1">**</bpt>Tip<ept id="p1">**</ept>: If you have multiple subscriptions, you'll need to ensure that you are working in the one that contains your cognitive services resource.  Use this command to determine your current subscription.
    >
    > ```
    > az account show
    > ```
    >
    > サブスクリプションを変更する必要がある場合は、このコマンドを実行して、 *&lt;subscriptionName&gt;* を正しいサブスクリプション名に変更します。
    >
    > ```
    > az account set --subscription <subscriptionName>
    > ```

10. これで、次のコマンドを使用して、Cognitive Services キーのリストを取得できます。 *&lt;resourceName&gt;* を Cognitive Services リソースの名前に置き換え、 *&lt;resourceGroup&gt;* を作成したリソース グループの名前に置き換えます。

    ```
    az cognitiveservices account keys list --name <resourceName> --resource-group <resourceGroup>
    ```

このコマンドは、Cognitive Services リソースのキーのリストを返します。

11. Switch back to the browser containing the Azure portal, and refresh your <bpt id="p1">**</bpt>Alerts page<ept id="p1">**</ept>. You should see a <bpt id="p1">**</bpt>Sev 4<ept id="p1">**</ept> alert listed in the table (if not, wait up to five minutes and refresh again).
12. アラートを選択して詳細を確認します。

## <a name="visualize-a-metric"></a>メトリックの視覚化

アラートを定義するだけでなく、Cognitive Services リソースのメトリックを表示して、その使用率を監視できます。

1. Azure portal の Cognitive Services リソースのページで、 **[メトリック]** ( **[監視]** セクション内) を選択します。
2. If there is no existing chart, select <bpt id="p1">**</bpt>+ New chart<ept id="p1">**</ept>. Then in the <bpt id="p1">**</bpt>Metric<ept id="p1">**</ept> list, review the possible metrics you can visualize and select <bpt id="p2">**</bpt>Total Calls<ept id="p2">**</ept>.
3. In the <bpt id="p1">**</bpt>Aggregation<ept id="p1">**</ept> list, select <bpt id="p2">**</bpt>Count<ept id="p2">**</ept>.  This will enable you to monitor the total calls to you Cognitive Service resource; which is useful in determining how much the service is being used over a period of time.
4. To generate some requests to your cognitive service, you will use <bpt id="p1">**</bpt>curl<ept id="p1">**</ept> - a command line tool for HTTP requests. In Visual Studio Code, in the <bpt id="p1">**</bpt>03-monitor<ept id="p1">**</ept> folder, open <bpt id="p2">**</bpt>rest-test.cmd<ept id="p2">**</ept> and edit the <bpt id="p3">**</bpt>curl<ept id="p3">**</ept> command it contains (shown below), replacing <bpt id="p4">*</bpt><ph id="ph1">&amp;lt;</ph>yourEndpoint<ph id="ph2">&amp;gt;</ph><ept id="p4">*</ept> and <bpt id="p5">*</bpt><ph id="ph3">&amp;lt;</ph>yourKey<ph id="ph4">&amp;gt;</ph><ept id="p5">*</ept> with your endpoint URI and <bpt id="p6">**</bpt>Key1<ept id="p6">**</ept> key to use the Text Analytics API in your cognitive services resource.

    ```
    curl -X POST "<yourEndpoint>/text/analytics/v3.1/languages?" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: <yourKey>" --data-ascii "{'documents':           [{'id':1,'text':'hello'}]}"
    ```

5. 変更を保存してから、**03-monitor** フォルダーの統合ターミナルで次のコマンドを実行します。

    ```
    rest-test
    ```

このコマンドは、入力データで検出された言語に関する情報を含む JSON ドキュメントを返します (これは英語でなければなりません)。

6. **rest-test** コマンドを複数回再実行して、呼び出しアクティビティを生成します ( **^** キーを使用して前のコマンドを切り替えることができます)。
7. Return to the <bpt id="p1">**</bpt>Metrics<ept id="p1">**</ept> page in the Azure portal and refresh the <bpt id="p2">**</bpt>Total Calls<ept id="p2">**</ept> count chart. It may take a few minutes for the calls you made using <bpt id="p1">*</bpt>curl<ept id="p1">*</ept> to be reflected in the chart - keep refreshing the chart until it updates to include them.

## <a name="more-information"></a>詳細情報

One of the options for monitoring cognitive services is to use <bpt id="p1">*</bpt>diagnostic logging<ept id="p1">*</ept>. Once enabled, diagnostic logging captures rich information about your cognitive services resource usage, and can be a useful monitoring and debugging tool. It can take over an hour after setting up diagnostic logging to generate any information, which is why we haven't explored it in this exercise; but you can learn more about it in the <bpt id="p1">[</bpt>Cognitive Services documentation<ept id="p1">](https://docs.microsoft.com/azure/cognitive-services/diagnostic-logging)</ept>.
