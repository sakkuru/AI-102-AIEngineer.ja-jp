---
lab:
  title: Azure Cognitive Search 用ナレッジ ストアの作成
  module: Module 12 - Creating a Knowledge Mining Solution
---

# <a name="create-a-knowledge-store-with-azure-cognitive-search"></a>Azure Cognitive Search 用ナレッジ ストアの作成

Azure Cognitive Search uses an enrichment pipeline of cognitive skills to extract AI-generated fields from documents and include them in a search index. While the index might be considered the primary output from an indexing process, the enriched data it contains might also be useful in other ways. For example:

- 基本的にインデックスは、それぞれがインデックス付きレコードを表す JSON オブジェクトのコレクションであるため、オブジェクトを JSON ファイルとしてエクスポートし、Azure Data Factory などのツールによるデータ オーケストレーション プロセスへの統合に使用するのに役立つ可能性があります。
- Microsoft Power BI などのツールで分析やレポートを行うために、インデックス レコードをテーブルのリレーショナル スキーマに正規化することもできます。
- インデックス作成プロセスでは埋め込み画像をドキュメントから抽出するので、それらの画像をファイルとして保存することもできます。

この演習では、パンフレットやホテルのレビューの情報を使用して顧客が旅行を計画できるようにする架空の旅行代理店である *Margie's Travel* のナレッジ ストアを実装します。

## <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

**AI-102-AIEngineer** コード リポジトリをこのラボの作業をしている環境に既にクローンしている場合は、Visual Studio Code で開きます。それ以外の場合は、次の手順に従って今すぐクローンしてください。

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git:Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるプロンプトが表示された場合は、 **[今はしない]** を選択します。

## <a name="create-azure-resources"></a>Azure リソースを作成する

> <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: If you have previously completed the <bpt id="p2">**</bpt><bpt id="p3">[</bpt>Create an Azure Cognitive Search solution<ept id="p3">](22-azure-search.md)</ept><ept id="p2">**</ept> exercise, and still have these Azure resources in your subscription, you can skip this section and start at the <bpt id="p4">**</bpt>Create a search solution<ept id="p4">**</ept> section. Otherwise, follow the steps below to provision the required Azure resources.

1. Web ブラウザーで Azure portal (`https://portal.azure.com`) を開き、自分の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. サブスクリプションの**リソース グループ**を表示します。
3. If you are using a restricted subscription in which a resource group has been provided for you, select the resource group to view its properties. Otherwise, create a new resource group with a name of your choice, and go to it when it has been created.
4. Azure Cognitive Search は、コグニティブ スキルの強化パイプラインを使用して、ドキュメントから AI で生成されたフィールドを抽出し、検索インデックスに含めます。
5. インデックスはインデックス作成プロセスの主な出力と見なせますが、それに含まれるエンリッチされたデータも他の方法で役立てることができます。
6. **24-knowledge-store** フォルダーを右クリックし、 **[統合ターミナルで開く]** を選択します。
7. ターミナル ペインで、次のコマンドを入力して、お使いの Azure サブスクリプションへの認証された接続を確立します。

    ```
    az login --output none
    ```

8. 次に例を示します。
9. 次のコマンドを実行して、Azure の場所を一覧表示します。

    ```
    az account list-locations -o table
    ```

10. 出力で、リソース グループの場所に対応する **Name** の値を見つけます (たとえば、*米国東部* の場合、対応する名前は *eastus* です)。
11. In the <bpt id="p1">**</bpt>setup.cmd<ept id="p1">**</ept> script, modify the <bpt id="p2">**</bpt>subscription_id<ept id="p2">**</ept>, <bpt id="p3">**</bpt>resource_group<ept id="p3">**</ept>, and <bpt id="p4">**</bpt>location<ept id="p4">**</ept> variable declarations with the appropriate values for your subscription ID, resource group name, and location name. Then save your changes.
12. **24-knowledge-store** フォルダーのターミナルで、次のコマンドを入力してスクリプトを実行します。

    ```
    setup
    ```
    > <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: The Search CLI module is in preview, and may get stuck in the <bpt id="p2">*</bpt>- Running ..<ept id="p2">*</ept> process. If this happens for over 2 minutes, press CTRL+C to cancel the long-running operation, and then select <bpt id="p1">**</bpt>N<ept id="p1">**</ept> when asked if you want to terminate the script. It should then complete successfully.
    >
    > スクリプトが失敗した場合は、正しい変数名で保存したことを確認して、再試行してください。

13. スクリプトが完了したら、表示された出力を確認し、ご自分の Azure リソースに関する次の情報をメモします (これらの値は後で必要になります)。
    - ストレージ アカウント名
    - ストレージ接続文字列
    - Cognitive Services アカウント
    - Cognitive Services キー
    - Search Service エンドポイント
    - Search Service の管理者キー
    - Search Service のクエリ キー

14. Azure portal で、リソースグループを更新し、Azure Storage アカウント、Azure Cognitive Services リソース、および Azure Cognitive Search リソースが含まれていることを確認します。

## <a name="create-a-search-solution"></a>検索ソリューションの作成

必要な Azure リソースが揃ったので、次のコンポーネントで構成される検索ソリューションを作成できます。

- Azure ストレージ コンテナー内のドキュメントを参照する**データ ソース**。
- A <bpt id="p1">**</bpt>skillset<ept id="p1">**</ept> that defines an enrichment pipeline of skills to extract AI-generated fields from the documents. The skillset also defines the <bpt id="p1">*</bpt>projections<ept id="p1">*</ept> that will be generated in your <bpt id="p2">*</bpt>knowledge store<ept id="p2">*</ept>.
- 検索可能なドキュメント レコードのセットを定義する**インデックス**。
- An <bpt id="p1">**</bpt>indexer<ept id="p1">**</ept> that extracts the documents from the data source, applies the skillset, and populates the index. The process of indexing also persists the projections defined in the skillset in the knowledge store.

この演習では、Azure Cognitive Search REST インターフェイスを使用して、JSON リクエストを送信することでこれらのコンポーネントを作成します。

### <a name="prepare-json-for-rest-operations"></a>REST 操作用の JSON の準備

REST インターフェイスを使用して、Azure CognitiveSearch コンポーネントの JSON 定義を送信します。

1. In Visual Studio Code, in the <bpt id="p1">**</bpt>24-knowledge-store<ept id="p1">**</ept> folder, expand the <bpt id="p2">**</bpt>create-search<ept id="p2">**</ept> folder and select <bpt id="p3">**</bpt>data_source.json<ept id="p3">**</ept>. This file contains a JSON definition for a data source named <bpt id="p1">**</bpt>margies-knowledge-data<ept id="p1">**</ept>.
2. **YOUR_CONNECTION_STRING** プレースホルダーを Azure ストレージ アカウントの接続文字列に置き換えます。これは次のようになります。

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    *接続文字列は、Azure portal のストレージ アカウントの **[アクセス キー]** ページにあります。*

3. 更新された JSON ファイルを保存して閉じます。
4. In the <bpt id="p1">**</bpt>create-search<ept id="p1">**</ept> folder, open <bpt id="p2">**</bpt>skillset.json<ept id="p2">**</ept>. This file contains a JSON definition for a skillset named <bpt id="p1">**</bpt>margies-knowledge-skillset<ept id="p1">**</ept>.
5. スキルセット定義の先頭にある **cognitiveServices** 要素で、**YOUR_COGNITIVE_SERVICES_KEY** プレースホルダーを Cognitive Services リソースのいずれかのキーに置き換えます。

    *キーは、Azure portal の Cognitive Services リソースの **[キーとエンドポイント]** ページにあります。*

6. At the end of the collection of skills in your skillset, find the <bpt id="p1">**</bpt>Microsoft.Skills.Util.ShaperSkill<ept id="p1">**</ept> skill named <bpt id="p2">**</bpt>define-projection<ept id="p2">**</ept>. This skill defines a JSON structure for the enriched data that will be used for the projections that the pipeline will persist on the knowledge store for each document processed by the indexer.
7. At the bottom of the skillset file, observe that the skillset also includes a <bpt id="p1">**</bpt>knowledgeStore<ept id="p1">**</ept> definition, which includes a connection string for the Azure Storage account where the knowledge store is to be created, and a collection of <bpt id="p2">**</bpt>projections<ept id="p2">**</ept>. This skillset includes three <bpt id="p1">*</bpt>projection groups<ept id="p1">*</ept>:
    - スキルセットに含まれる Shaper スキルの **knowledge_projection** 出力に基づく "*オブジェクト*" プロジェクションを含むグループ。
    - ドキュメントから抽出された画像データの **normalized_images** コレクションに基づく "*ファイル*" プロジェクションを含むグループ。
    - 次の "*テーブル*" プロジェクションを含むグループ。
        - **KeyPhrases**: 自動的に生成されたキー列と、Shaper スキルの **knowledge_projection/key_phrases/** コレクション出力にマップされた **keyPhrase** 列が含まれています。
        - **Locations**: 自動的に生成されたキー列と、Shaper スキルの **knowledge_projection/key_phrases/** コレクション出力にマップされた **location** 列が含まれています。
        - **ImageTags**: 自動的に生成されたキー列と、Shaper スキルの **knowledge_projection/image_tags/** コレクション出力にマップされた **tag** 列が含まれています。
        - **Docs**: 自動的に生成されたキー列と、テーブルにまだ割り当てられていない Shaper スキルのすべての **knowledge_projection** 出力値が含まれています。
8. **storageConnectionString** 値の **YOUR_CONNECTION_STRING** プレースホルダーを、ストレージ アカウントの接続文字列に置き換えます。
9. 更新された JSON ファイルを保存して閉じます。
10. In the <bpt id="p1">**</bpt>create-search<ept id="p1">**</ept> folder, open <bpt id="p2">**</bpt>index.json<ept id="p2">**</ept>. This file contains a JSON definition for an index named <bpt id="p1">**</bpt>margies-knowledge-index<ept id="p1">**</ept>.
11. インデックスの JSON を確認し、変更を加えずにファイルを閉じます。
12. In the <bpt id="p1">**</bpt>create-search<ept id="p1">**</ept> folder, open <bpt id="p2">**</bpt>indexer.json<ept id="p2">**</ept>. This file contains a JSON definition for an indexer named <bpt id="p1">**</bpt>margies-knowledge-indexer<ept id="p1">**</ept>.
13. インデクサーの JSON を確認し、変更を加えずにファイルを閉じます。

### <a name="submit-rest-requests"></a>REST リクエストの送信

検索ソリューションコンポーネントを定義する JSON オブジェクトを準備したので、JSON ドキュメントを REST インターフェイスに送信して作成できます。

1. In the <bpt id="p1">**</bpt>create-search<ept id="p1">**</ept> folder, open <bpt id="p2">**</bpt>create-search.cmd<ept id="p2">**</ept>. This batch script uses the cURL utility to submit the JSON definitions to the REST interface for your Azure Cognitive Search resource.
2. **YOUR_SEARCH_URL** 変数と **YOUR_ADMIN_KEY** 変数のプレースホルダーを、Azure Cognitive Search リソースの **Url** と**管理キー**の 1 つに置き換えます。

    *これらの値は、Azure portal の Azure Cognitive Search リソースの **[概要]** ページと **[キー]** ページにあります。*

3. 更新したバッチファイルを保存します。
4. **create-search** フォルダーを右クリックし、 **[Open in Integrated Terminal]\(統合ターミナルで開く\)** を選択します。
5. **create-search** フォルダーのターミナル ペインで、次のコマンドを入力してバッチ スクリプトを実行します。

    ```
    create-search
    ```

6. スクリプトが完了したら、Azure portal の Azure Cognitive Search リソースのページで、 **[インデクサー]** ページを選択し、インデックス作成プロセスが完了するのを待ちます。

    ***[更新]** を選択して、インデックス作成操作の進行状況を追跡できます。完了するまでに 1 分ほどかかる場合があります。*

    > <bpt id="p1">**</bpt>Tip<ept id="p1">**</ept>: If the script fails, check the placeholders you added in the <bpt id="p2">**</bpt>data_source.json<ept id="p2">**</ept> and <bpt id="p3">**</bpt>skillset.json<ept id="p3">**</ept> files as well as the <bpt id="p4">**</bpt>create-search.cmd<ept id="p4">**</ept> file. After correcting any mistakes, you may need to use the Azure portal user interface to delete any components that were created in your search resource before re-running the script.

## <a name="view-the-knowledge-store"></a>ナレッジストアの表示

スキルセットを使用してナレッジ ストアを作成するインデクサーを実行すると、そのインデックス作成プロセスによって抽出されてエンリッチされたデータがナレッジ ストアのプロジェクションに保持されます。

### <a name="view-object-projections"></a>オブジェクト プロジェクションを表示する

The <bpt id="p1">*</bpt>object<ept id="p1">*</ept> projections defined in the Margie's Travel skillset consist of a JSON file for each indexed document. These files are stored in a blob container in the Azure Storage account specified in the skillset definition.

1. Azure portal で、以前に作成した Azure Storage アカウントを表示します。
2. **[ストレージ エクスプローラー]** タブ (左側のペイン) を選択して、Azure portal のストレージ エクスプローラー インターフェイスでストレージ アカウントを表示します。
2. **注**: 以前に「**[Azure Cognitive Search ソリューションを作成する](22-azure-search.md)**」の演習を完了し、サブスクリプションにこれらの Azure リソースがまだある場合は、このセクションをスキップして、「**検索ソリューションの作成**」セクションから開始できます。
3. それ以外の場合は、以下の手順に従って、必要な Azure リソースをプロビジョニングします。
4. Open any of the folders, and then download and open the <bpt id="p1">**</bpt>knowledge-projection.json<ept id="p1">**</ept> file it contains. Each JSON file contains a representation of an indexed document, including the enriched data extracted by the skillset as shown here.

```
{
    "file_id":"abcd1234....",
    "file_name":"Margies Travel Company Info.pdf",
    "url":"https://store....blob.core.windows.net/margies/...pdf",
    "language":"en",
    "sentiment":0.83164644241333008,
    "key_phrases":[
        "Margie’s Travel",
        "Margie's Travel",
        "best travel experts",
        "world-leading travel agency",
        "international reach"
        ],
    "locations":[
        "Dubai",
        "Las Vegas",
        "London",
        "New York",
        "San Francisco"
        ],
    "image_tags":[
        "outdoor",
        "tree",
        "plant",
        "palm"
        ]
}
```

このような "*オブジェクト*" プロジェクションの作成機能を使用することにより、エンタープライズ データ分析ソリューションに組み込むことのできるエンリッチされたデータ オブジェクトを生成することができます。これは、その JSON ファイルを Azure Data Factory パイプラインに取り込んでさらに処理を行ったり、データ ウェアハウスに読み込んだりすることで行えます。

### <a name="view-file-projections"></a>ファイル プロジェクションを表示する

スキルセットで定義されている "*ファイル*" プロジェクションでは、インデックス作成プロセス中にドキュメントから抽出された各画像の JPEG ファイルが作成されます。

1. In the storage explorer interface in the Azure portal, select the <bpt id="p1">**</bpt>margies-images<ept id="p1">**</ept> blob container. This container contains a folder for each document that contained images.
2. そのフォルダーのいずれかを開き、内容を表示します。各フォルダーに、少なくとも 1 つの \*.jpg ファイルが含まれています。
3. 画像ファイルのいずれかを開き、ドキュメントから抽出された画像が含まれていることを確認します。

このような "*ファイル*" プロジェクションの生成機能を使用することにより、インデックス作成で大量のドキュメントから埋め込み画像を効率的に抽出できます。

### <a name="view-table-projections"></a>テーブル プロジェクションを表示する

スキルセットで定義されている "*テーブル*" プロジェクションは、エンリッチされたデータのリレーショナル スキーマを形成します。

1. Azure portal のストレージ エクスプローラー インターフェイスで、**[テーブル]** を展開します。
2. Select the <bpt id="p1">**</bpt>Docs<ept id="p1">**</ept> table to view its columns. The columns include some standard Azure Storage table columns - to hide these, modify the <bpt id="p1">**</bpt>Column Options<ept id="p1">**</ept> to select only the following columns:
    - **document_id** (インデックス作成プロセスによって自動的に生成されるキー列)
    - **file_id** (エンコードされたファイルの URL)
    - **file_name** (ドキュメントのメタデータから抽出されたファイル名)
    - **language** (ドキュメントの記述言語)
    - **sentiment** そのドキュメントに対して計算されたセンチメント スコア。
    - **url** Azure Storage のドキュメント BLOB の URL。
3. インデックス作成プロセスによって作成された他のテーブルを表示します。
    - **ImageTags** (個々の画像タグごとに、そのタグが使用されているドキュメントの **document_id** が入った行が 1 行含まれます)。
    - **KeyPhrases** (個々のキー フレーズごとに、そのフレーズが使用されているドキュメントの **document_id** が入った行が 1 行含まれます)。
    - **Locations** (個々の場所ごとに、その場所が使用されているドキュメントの **document_id** が入った行が 1 行含まれます)。

リソース グループが提供されている制限付きサブスクリプションを使用している場合は、リソース グループを選択してそのプロパティを表示します。

## <a name="more-information"></a>詳細情報

Azure Cognitive Search を使用したナレッジ ストアの作成の詳細については、[Azure Cognitive Search のドキュメント](https://docs.microsoft.com/azure/search/knowledge-store-concept-intro)を参照してください。
