---
lab:
  title: Azure Cognitive Search 用カスタム スキルの作成
  module: Module 12 - Creating a Knowledge Mining Solution
---

# <a name="create-a-custom-skill-for-azure-cognitive-search"></a>Azure Cognitive Search 用カスタム スキルの作成

Azure Cognitive Search uses an enrichment pipeline of cognitive skills to extract AI-generated fields from documents and include them in a search index. There's a comprehensive set of built-in skills that you can use, but if you have a specific requirement that isn't met by these skills, you can create a custom skill.

この演習では、ドキュメント内の個々の単語の頻度を表にして、最もよく使用される上位5つの単語のリストを生成するカスタムスキルを作成し、それを架空の旅行代理店である Margie'sTravel の検索ソリューションに追加します。

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
5. 使用できる組み込みスキルの包括的なセットがありますが、これらのスキルでは満たされない特定の要件がある場合は、カスタムス キルを作成できます。
6. **23-custom-search-skill** フォルダーを右クリックし、 **[Open in Integrated Terminal]\(統合ターミナルで開く\)** を選択します。
7. ターミナル ペインで、次のコマンドを入力して、お使いの Azure サブスクリプションへの認証された接続を確立します。

    ```
    az login --output none
    ```

8. When prompted, sign into your Azure subscription. Then return to Visual Studio Code and wait for the sign-in process to complete.
9. 次のコマンドを実行して、Azure の場所を一覧表示します。

    ```
    az account list-locations -o table
    ```

10. 出力で、リソース グループの場所に対応する **Name** の値を見つけます (たとえば、*米国東部* の場合、対応する名前は *eastus* です)。
11. In the <bpt id="p1">**</bpt>setup.cmd<ept id="p1">**</ept> script, modify the <bpt id="p2">**</bpt>subscription_id<ept id="p2">**</ept>, <bpt id="p3">**</bpt>resource_group<ept id="p3">**</ept>, and <bpt id="p4">**</bpt>location<ept id="p4">**</ept> variable declarations with the appropriate values for your subscription ID, resource group name, and location name. Then save your changes.
12. **23-custom-search-skill** フォルダーのターミナルで、次のコマンドを入力してスクリプトを実行します。

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
- ドキュメントから AI で生成されたフィールドを抽出するスキルの強化パイプラインを定義する**スキルセット**。
- 検索可能なドキュメントレコードのセットを定義する**インデックス**。
- データ ソースからドキュメントを抽出し、スキル セットを適用して、インデックスにデータを入力する**インデクサー**。

この演習では、Azure Cognitive Search REST インターフェイスを使用して、JSON リクエストを送信することでこれらのコンポーネントを作成します。

1. In Visual Studio Code, in the <bpt id="p1">**</bpt>23-custom-search-skill<ept id="p1">**</ept> folder, expand the <bpt id="p2">**</bpt>create-search<ept id="p2">**</ept> folder and select <bpt id="p3">**</bpt>data_source.json<ept id="p3">**</ept>. This file contains a JSON definition for a data source named <bpt id="p1">**</bpt>margies-custom-data<ept id="p1">**</ept>.
2. **YOUR_CONNECTION_STRING** プレースホルダーを Azure ストレージ アカウントの接続文字列に置き換えます。これは次のようになります。

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    *接続文字列は、Azure portal のストレージ アカウントの **[アクセス キー]** ページにあります。*

3. 更新された JSON ファイルを保存して閉じます。
4. In the <bpt id="p1">**</bpt>create-search<ept id="p1">**</ept> folder, open <bpt id="p2">**</bpt>skillset.json<ept id="p2">**</ept>. This file contains a JSON definition for a skillset named <bpt id="p1">**</bpt>margies-custom-skillset<ept id="p1">**</ept>.
5. スキルセット定義の先頭にある **cognitiveServices** 要素で、**YOUR_COGNITIVE_SERVICES_KEY** プレースホルダーを Cognitive Services リソースのいずれかのキーに置き換えます。

    *キーは、Azure portal の Cognitive Services リソースの **[キーとエンドポイント]** ページにあります。*

6. 更新された JSON ファイルを保存して閉じます。
7. In the <bpt id="p1">**</bpt>create-search<ept id="p1">**</ept> folder, open <bpt id="p2">**</bpt>index.json<ept id="p2">**</ept>. This file contains a JSON definition for an index named <bpt id="p1">**</bpt>margies-custom-index<ept id="p1">**</ept>.
8. インデックスの JSON を確認し、変更を加えずにファイルを閉じます。
9. In the <bpt id="p1">**</bpt>create-search<ept id="p1">**</ept> folder, open <bpt id="p2">**</bpt>indexer.json<ept id="p2">**</ept>. This file contains a JSON definition for an indexer named <bpt id="p1">**</bpt>margies-custom-indexer<ept id="p1">**</ept>.
10. インデクサーの JSON を確認し、変更を加えずにファイルを閉じます。
11. In the <bpt id="p1">**</bpt>create-search<ept id="p1">**</ept> folder, open <bpt id="p2">**</bpt>create-search.cmd<ept id="p2">**</ept>. This batch script uses the cURL utility to submit the JSON definitions to the REST interface for your Azure Cognitive Search resource.
12. **YOUR_SEARCH_URL** 変数と **YOUR_ADMIN_KEY** 変数のプレースホルダーを、Azure Cognitive Search リソースの **Url** と**管理キー**の 1 つに置き換えます。

    *これらの値は、Azure portal の Azure Cognitive Search リソースの **[概要]** ページと **[キー]** ページにあります。*

13. 更新したバッチファイルを保存します。
14. **create-search** フォルダーを右クリックし、 **[Open in Integrated Terminal]\(統合ターミナルで開く\)** を選択します。
15. **create-search** フォルダーのターミナル ペインで、次のコマンドを入力してバッチ スクリプトを実行します。

    ```
    create-search
    ```

16. スクリプトが完了したら、Azure portal の Azure Cognitive Search リソースのページで、 **[インデクサー]** ページを選択し、インデックス作成プロセスが完了するのを待ちます。

    ***[更新]** を選択して、インデックス作成操作の進行状況を追跡できます。完了するまでに 1 分ほどかかる場合があります。*

## <a name="search-the-index"></a>インデックスを検索する

インデックスができたので、検索できます。

1. Azure Cognitive Search リソースのブレードの上部で、 **[検索エクスプローラー]** を選択します。
2. 検索エクスプローラーの **[クエリ文字列]** ボックスに、次のクエリ文字列を入力し、 **[検索]** を選択します。

    ```
    search=London&$select=url,sentiment,keyphrases&$filter=metadata_author eq 'Reviewer' and sentiment eq 'positive'
    ```

    このクエリを実行すると、*Reviewer* が作成した、*London* に言及していて正の**センチメント** ラベルが付いているすべてのドキュメント (つまり、ロンドンに言及している肯定的なレビュー) の **url**、**センチメント**、および**キーフレーズ**が取得されます。

## <a name="create-an-azure-function-for-a-custom-skill"></a>カスタム スキル Azure 関数を作成する

検索ソリューションには、前のタスクで見た感情スコアやキー フレーズのリストなど、ドキュメントからの情報でインデックスを充実させる多くの組み込みの認知スキルが含まれています。

You can enhance the index further by creating custom skills. For example, it might be useful to identify the words that are used most frequently in each document, but no built-in skill offers this functionality.

単語カウント機能をカスタムスキルとして実装するには、好みの言語で Azure 関数を作成します。

> <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: In this exercise, you'll create a simple Node.JS function using the code editing capabilities in the Azure portal. In a production solution, you would typically use a development environment such as Visual Studio Code to create a function app in your preferred language (for example C#, Python, Node.JS, or Java) and publish it to Azure as part of a DevOps process.

1. Azure portal の **[ホーム]** ページで、次の設定を使用して新しい**関数アプリ** リソースを作成します。
    - **[サブスクリプション]**: *該当するサブスクリプション*
    - **[リソース グループ]** : *Azure Cognitive Search リソースと同じリソース グループ*
    - **関数アプリ名**: *一意の名前*
    - **公開**: コード
    - **ランタイム スタック**:Node.js
    - **[バージョン]** : 14 LTS
    - **[リージョン]** : *''Azure Cognitive Search リソースと同じリージョン''*

2. デプロイが完了するのを待ってから、デプロイされた関数アプリ リソースに移動します。
3. 関数アプリのブレードの左側のペインで、 **[関数]** タブを選びます。次の設定を使用して、新しい関数を作成します。
    - **開発環境をセットアップする**"
        - **開発環境**:ポータルでの開発
    - **テンプレートを選択**"
        - **テンプレート**:HTTP トリガー
    - **テンプレートの詳細**:
        - **新しい関数**: wordcount
        - **承認レベル**: 関数
4. **注**: 以前に「**[Azure Cognitive Search ソリューションを作成する](22-azure-search.md)**」の演習を完了し、サブスクリプションにこれらの Azure リソースがまだある場合は、このセクションをスキップして、「**検索ソリューションの作成**」セクションから開始できます。
5. 既定の関数コードを次のコードに置き換えます。

```javascript
module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    if (req.body && req.body.values) {

        vals = req.body.values;

        // Array of stop words to be ignored
        var stopwords = ['', 'i', 'me', 'my', 'myself', 'we', 'our', 'ours', 'ourselves', 'you', 
        "youre", "youve", "youll", "youd", 'your', 'yours', 'yourself', 
        'yourselves', 'he', 'him', 'his', 'himself', 'she', "shes", 'her', 
        'hers', 'herself', 'it', "its", 'itself', 'they', 'them', 
        'their', 'theirs', 'themselves', 'what', 'which', 'who', 'whom', 
        'this', 'that', "thatll", 'these', 'those', 'am', 'is', 'are', 'was',
        'were', 'be', 'been', 'being', 'have', 'has', 'had', 'having', 'do', 
        'does', 'did', 'doing', 'a', 'an', 'the', 'and', 'but', 'if', 'or', 
        'because', 'as', 'until', 'while', 'of', 'at', 'by', 'for', 'with', 
        'about', 'against', 'between', 'into', 'through', 'during', 'before', 
        'after', 'above', 'below', 'to', 'from', 'up', 'down', 'in', 'out', 
        'on', 'off', 'over', 'under', 'again', 'further', 'then', 'once', 'here', 
        'there', 'when', 'where', 'why', 'how', 'all', 'any', 'both', 'each', 
        'few', 'more', 'most', 'other', 'some', 'such', 'no', 'nor', 'not', 
        'only', 'own', 'same', 'so', 'than', 'too', 'very', 'can', 'will',
        'just', "dont", 'should', "shouldve", 'now', "arent", "couldnt", 
        "didnt", "doesnt", "hadnt", "hasnt", "havent", "isnt", "mightnt", "mustnt",
        "neednt", "shant", "shouldnt", "wasnt", "werent", "wont", "wouldnt"];

        res = {"values":[]};

        for (rec in vals)
        {
            // Get the record ID and text for this input
            resVal = {recordId:vals[rec].recordId, data:{}};
            txt = vals[rec].data.text;

            // remove punctuation and numerals
            txt = txt.replace(/[^ A-Za-z_]/g,"").toLowerCase();

            // Get an array of words
            words = txt.split(" ")

            // count instances of non-stopwords
            wordCounts = {}
            for(var i = 0; i < words.length; ++i) {
                word = words[i];
                if (stopwords.includes(word) == false )
                {
                    if (wordCounts[word])
                    {
                        wordCounts[word] ++;
                    }
                    else
                    {
                        wordCounts[word] = 1;
                    }
                }
            }

            // Convert wordcounts to an array
            var topWords = [];
            for (var word in wordCounts) {
                topWords.push([word, wordCounts[word]]);
            }

            // Sort in descending order of count
            topWords.sort(function(a, b) {
                return b[1] - a[1];
            });

            // Get the first ten words from the first array dimension
            resVal.data.text = topWords.slice(0,9)
              .map(function(value,index) { return value[0]; });

            res.values[rec] = resVal;
        };

        context.res = {
            body: JSON.stringify(res),
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
    else {
        context.res = {
            status: 400,
            body: {"errors":[{"message": "Invalid input"}]},
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
};
```

6. 関数を保存し、 **[テスト/実行]** ペインを開きます。
7. **[Test/Run]\(テスト/実行\)** ペインで、既存の**本文**を次の JSON に置き換えます。これには、1 つ以上のドキュメントのデータを含むレコードを処理のために送信する Azure Cognitive Search スキルが必要とするスキーマが反映されています。

    ```
    {
        "values": [
            {
                "recordId": "a1",
                "data":
                {
                "text":  "Tiger, tiger burning bright in the darkness of the night.",
                "language": "en"
                }
            },
            {
                "recordId": "a2",
                "data":
                {
                "text":  "The rain in spain stays mainly in the plains! That's where you'll find the rain!",
                "language": "en"
                }
            }
        ]
    }
    ```
    
8. それ以外の場合は、以下の手順に従って、必要な Azure リソースをプロビジョニングします。

    ```
    {
    "values": [
        {
        "recordId": "a1",
        "data": {
            "text": [
            "tiger",
            "burning",
            "bright",
            "darkness",
            "night"
            ]
        },
        "errors": null,
        "warnings": null
        },
        {
        "recordId": "a2",
        "data": {
            "text": [
            "rain",
            "spain",
            "stays",
            "mainly",
            "plains",
            "thats",
            "youll",
            "find"
            ]
        },
        "errors": null,
        "warnings": null
        }
    ]
    }
    ```

9. Close the <bpt id="p1">**</bpt>Test/Run<ept id="p1">**</ept> pane and in the <bpt id="p2">**</bpt>wordcount<ept id="p2">**</ept> function blade, click <bpt id="p3">**</bpt>Get function URL<ept id="p3">**</ept>. Then copy the URL for the default key to the clipboard. You'll need this in the next procedure.

## <a name="add-the-custom-skill-to-the-search-solution"></a>検索ソリューションにカスタム スキルを追加する

次に、関数をカスタム スキルとして検索ソリューション スキル セットに含め、生成された結果をインデックスのフィールドにマップする必要があります。 

1. In Visual Studio Code, in the <bpt id="p1">**</bpt>23-custom-search-skill/update-search<ept id="p1">**</ept> folder, open the <bpt id="p2">**</bpt>update-skillset.json<ept id="p2">**</ept> file. This contains the JSON definition of a skillset.
2. Review the skillset definition. It includes the same skills as before, as well as a new <bpt id="p1">**</bpt>WebApiSkill<ept id="p1">**</ept> skill named <bpt id="p2">**</bpt>get-top-words<ept id="p2">**</ept>.
3. **get-top-words** スキルの定義を編集し、**uri** の値を (前の手順でクリップボードにコピーした) Azure 関数の URL に設定して、**YOUR-FUNCTION-APP-URL** を置き換えます。
4. スキルセット定義の先頭にある **cognitiveServices** 要素で、**YOUR_COGNITIVE_SERVICES_KEY** プレースホルダーを Cognitive Services リソースのいずれかのキーに置き換えます。

    *キーは、Azure portal の Cognitive Services リソースの **[キーとエンドポイント]** ページにあります。*

5. 更新された JSON ファイルを保存して閉じます。
6. リソース グループが提供されている制限付きサブスクリプションを使用している場合は、リソース グループを選択してそのプロパティを表示します。
7. インデックスの JSON を確認し、変更を加えずにファイルを閉じます。
8. それ以外の場合は、選択した名前で新しいリソース グループを作成し、作成されたらそのグループに移動します。
9. インデクサーの JSON を確認し、変更を加えずにファイルを閉じます。
10. In the <bpt id="p1">**</bpt>update-search<ept id="p1">**</ept> folder, open <bpt id="p2">**</bpt>update-search.cmd<ept id="p2">**</ept>. This batch script uses the cURL utility to submit the updated JSON definitions to the REST interface for your Azure Cognitive Search resource.
11. **YOUR_SEARCH_URL** 変数と **YOUR_ADMIN_KEY** 変数のプレースホルダーを、Azure Cognitive Search リソースの **Url** と**管理キー**の 1 つに置き換えます。

    *これらの値は、Azure portal の Azure Cognitive Search リソースの **[概要]** ページと **[キー]** ページにあります。*

12. 更新したバッチファイルを保存します。
13. **update-search** フォルダーを右クリックし、 **[Open in Integrated Terminal]\(統合ターミナルで開く\)** を選択します。
14. **update-search** フォルダーのターミナル ペインで、次のコマンドを入力してバッチ スクリプトを実行します。

    ```
    update-search
    ```

15. スクリプトが完了したら、Azure portal の Azure Cognitive Search リソースのページで、 **[インデクサー]** ページを選択し、インデックス作成プロセスが完了するのを待ちます。

    ***[更新]** を選択して、インデックス作成操作の進行状況を追跡できます。完了するまでに 1 分ほどかかる場合があります。*

## <a name="search-the-index"></a>インデックスを検索する

インデックスができたので、検索できます。

1. Azure Cognitive Search リソースのブレードの上部で、 **[検索エクスプローラー]** を選択します。
2. 検索エクスプローラーの **[クエリ文字列]** ボックスに、次のクエリ文字列を入力し、 **[検索]** を選択します。

    ```
    search=Las Vegas&$select=url,top_words
    ```

    このクエリでは、*Las Vegas* に言及しているすべてのドキュメントの **url** フィールドと **top_words** フィールドが取得されます。

## <a name="more-information"></a>詳細情報

Azure Cognitive Search のカスタム スキルの作成の詳細については、[Azure Cognitive Search のドキュメント](https://docs.microsoft.com/azure/search/cognitive-search-custom-skill-interface)を参照してください。
