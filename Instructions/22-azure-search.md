---
lab:
  title: Azure Cognitive Search ソリューションを作成する
  module: Module 12 - Creating a Knowledge Mining Solution
---

# <a name="create-an-azure-cognitive-search-solution"></a>Azure Cognitive Search ソリューションを作成する

All organizations rely on information to make decisions, answer questions, and function efficiently. The problem for most organizations is not a lack of information, but the challenge of finding and  extracting the information from the massive set of documents, databases, and other sources in which the information is stored.

For example, suppose <bpt id="p1">*</bpt>Margie's Travel<ept id="p1">*</ept> is a travel agency that specializes in organizing trips to cities around the world. Over time, the company has amassed a huge amount of information in documents such as brochures, as well as reviews of hotels submitted by customers. This data is a valuable source of insights for travel agents and customers as they plan trips, but the sheer volume of data can make it difficult to find relevant information to answer a specific customer question.

この課題に対処するために、Margie'sTravel は Azure Cognitive Search を使用して、AI ベースのコグニティブ スキルを使用してドキュメントにインデックスを付け、強化して検索を容易にするソリューションを実装します。

## <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

If you have not already cloned <bpt id="p1">**</bpt>AI-102-AIEngineer<ept id="p1">**</ept> code repository to the environment where you're working on this lab, follow these steps to do so. Otherwise, open the cloned folder in Visual Studio Code.

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git:Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるプロンプトが表示された場合は、 **[今はしない]** を選択します。

## <a name="create-azure-resources"></a>Azure リソースを作成する

Margie's Travel 用に作成するソリューションでは、お使いの Azure サブスクリプションに次のリソースが必要です。

- **Azure Cognitive Search リソース**。インデックス作成とクエリ実行を管理します。
- **Cognitive Services** リソース。AI によって生成された分析情報を使用してデータ ソースのデータをエンリッチするために、検索ソリューションで使用できるスキルを AI サービスに提供します。
- 検索対象のドキュメントが保存されている BLOB コンテナーを持つ **ストレージ アカウント**。

> **重要**: Azure Cognitive Search と Cognitive Services のリソースは同じ場所にある必要があります。

### <a name="create-an-azure-cognitive-search-resource"></a>Azure Cognitive Search リソースを作成する

1. Web ブラウザーで Azure portal (`https://portal.azure.com`) を開き、自分の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. **[&#65291;リソースの作成]** ボタンを選択し、*search* を検索して、次の設定で **Azure Cognitive Search** リソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *新しいリソース グループを作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がない可能性があります - 提供されているものを使ってください)*
    - **サービス名**: *一意の名前を入力します*
    - **場所**: *場所を選択します - Azure Cognitive Search リソースと Cognitive Services リソースは同じ場所にある必要があります*
    - **価格レベル**: Basic

3. デプロイが完了するまで待ち、デプロイされたリソースに移動します。
4. すべての組織は、意思決定を行い、疑問に答え、効率的に機能するために情報を利用しています。

### <a name="create-a-cognitive-services-resource"></a>Cognitive Services リソースの作成

ほとんどの組織にとって問題となっているのは、情報の不足ではなく、情報が格納されている大量の一連のドキュメント、データベース、およびその他のソースから情報を検索して抽出するという課題です。

1. Azure portal のホーム ページに戻り、**[&#65291;リソースの作成]** ボタンを選択し、*cognitive services* を検索して、次の設定で **Cognitive Services** リソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: ''*Azure Cognitive Search リソースと同じリソース グループ*''
    - **リージョン**: ''*Azure Cognitive Search リソースと同じ場所*''
    - **[名前]**: *一意の名前を入力します*
    - **価格レベル**: Standard S0
2. 必要なチェック ボックスをオンにして、リソースを作成します。
3. デプロイが完了するまで待ち、デプロイの詳細を表示します。

### <a name="create-a-storage-account"></a>ストレージ アカウントの作成

1. Azure portal のホーム ページに戻り、**[&#65291;リソースの作成]** ボタンを選択し、''*ストレージ アカウント*'' を検索して、次の設定で**ストレージ アカウント** リソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: **Azure Cognitive Search および Cognitive Services リソースと同じリソース グループ*
    - **ストレージ アカウント名**: *一意の名前を入力します*
    - **[リージョン]**: 使用できるリージョンを選択します**
    - **パフォーマンス**: 標準
    - **レプリケーション**: ローカル冗長ストレージ (LRS)
2. デプロイが完了するまで待ち、デプロイされたリソースに移動します。
3. **[概要]** ページで、**サブスクリプション ID** に注意してください。これはストレージ アカウントがプロビジョニングされているサブスクリプションを識別します。
4. On the <bpt id="p1">**</bpt>Access keys<ept id="p1">**</ept> page, note that two keys have been generated for your storage account. Then select <bpt id="p1">**</bpt>Show keys<ept id="p1">**</ept> to view the keys.

    > **ヒント**: **ストレージ アカウント** ブレードを開いたままにします。次の手順では、サブスクリプション ID とキーの 1 つが必要になります。

## <a name="upload-documents-to-azure-storage"></a>Azure Storage にドキュメントをアップロードする

必要なリソースが揃ったので、いくつかのドキュメントを　Azure　Storage　アカウントにアップロードできます。

1. Visual Studio Code の **[エクスプローラー]** ペインで、**22-create-a-search-solution** フォルダーを展開し、**UploadDocs.cmd** を選択します。
2. バッチ ファイルを編集して、**YOUR_SUBSCRIPTION_ID**、**YOUR_AZURE_STORAGE_ACCOUNT_NAME**、および　**YOUR_AZURE_STORAGE_KEY** プレースホルダーを、以前に作成したストレージ アカウントの適切なサブスクリプション ID、Azure ストレージ アカウント名、および Azure ストレージ アカウント キーの値に置き換えます。
3. 変更を保存してから、**22-create-a-search-solution** フォルダーを右クリックして、統合ターミナルを開きます。
4. 次のコマンドを入力して、Azure CLI を使用して Azure サブスクリプションにサインインします。

    ```
    az login
    ```

たとえば、*Margie's Travel* は世界各地の都市への旅行の手配に特化した旅行代理店だとします。

5. 長い時間をかけて、同社は、パンフレットなどのドキュメントや顧客から送信されたホテルのレビューに含まれている大量の情報を蓄積してきました。

    ```
    UploadDocs
    ```

## <a name="index-the-documents"></a>ドキュメントのインデックスを作成する

ドキュメントが配置されたので、インデックスを作成して検索ソリューションを作成できます。

1. このデータは、旅行代理店の従業員や顧客にとって旅行を計画する際に価値のある分析情報源となりますが、データ量が膨大であるため、特定の顧客の疑問に答える際に関連情報を見つけることが困難な場合があります。
2. On the <bpt id="p1">**</bpt>Connect to your data<ept id="p1">**</ept> page, in the <bpt id="p2">**</bpt>Data Source<ept id="p2">**</ept> list, select <bpt id="p3">**</bpt>Azure Blob Storage<ept id="p3">**</ept>. Then complete the data store details with the following values:
    - **データ ソース**: Azure BLOB ストレージ
    - **データ ソース名**: margies-data
    - **抽出するデータ**: コンテンツとメタデータ
    - **解析モード**: 既定
    - **接続文字列**: ***[既存の接続を選択します]** を選択します。次に、ストレージ アカウントを選択し、UploadDocs.cmd スクリプトによって作成された **margies** コンテナーを選択します。*
    - **マネージド ID 認証:** なし
    - **コンテナー名**: margies
    - **BLOB フォルダー**: ''これは空白のままにします''**
    - **説明**: Margie's Travel の Web サイトのパンフレットとレビュー。
3. 次の手順 (*コグニティブ スキルを追加する*) に進みます。
4. **[Cognitive Services をアタッチする]** セクションで、ご利用の Cognitive Services リソースを選択します。
5. **[エンリッチメントの追加]** セクションで、次のことを行います。
    - **スキルセット名**を **margies-skillset** に変更します。
    - **[OCR を有効にし、すべてのテキストを merged_content フィールドにマージする]** オプションを選択します。
    - **ソース データ フィールド**が **merged_content** に設定されていることを確かめます。
    - **[エンリッチメントの粒度レベル]** を **[ソース フィールド]** のままにします。このフィールドには、インデックスを作成するドキュメントのコンテンツ全体が設定されます。ただし、これを変更して、ページや文など、より詳細なレベルで情報を抽出できることに注意してください。
    - 次のエンリッチされたフィールドを選択します。

        | コグニティブ スキル | パラメーター | フィールド名 |
        | --------------- | ---------- | ---------- |
        | 場所の名前を抽出 | | locations |
        | キー フレーズ抽出 | | keyphrases |
        | 言語を検出する | | language |
        | 画像からタグを生成する | | imageTags |
        | 画像からキャプションを生成する | | imageCaption |

6. Double-check your selections (it can be difficult to change them later). Then proceed to the next step (<bpt id="p1">*</bpt>Customize target index<ept id="p1">*</ept>).
7. **インデックス名**を **margies-index** に変更します。
8. **[キー]** が **metadata_storage_path** に設定されていることを確かめます。 **[Suggester 名]** は空白のままにし、 **[検索モード]** は既定のままにします。
9. インデックス フィールドに次の変更を加え、他のすべてのフィールドはデフォルト設定のままにします (**重要**: テーブル全体を表示するには、右にスクロールする必要がある場合があります)。

    | フィールド名 | [取得可能] | フィルターの適用 | 並べ替え可能 | ファセット可能 | 検索可能 |
    | ---------- | ----------- | ---------- | -------- | --------- | ---------- |
    | metadata_storage_size | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_last_modified | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_name | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | metadata_author | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | locations | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | keyphrases | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | language | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | |

11. Double-check your selections, paying particular attention to ensure that the correct <bpt id="p1">**</bpt>Retrievable<ept id="p1">**</ept>, <bpt id="p2">**</bpt>Filterable<ept id="p2">**</ept>, <bpt id="p3">**</bpt>Sortable<ept id="p3">**</ept>, <bpt id="p4">**</bpt>Facetable<ept id="p4">**</ept>, and <bpt id="p5">**</bpt>Searchable<ept id="p5">**</ept> options are selected for each field  (it can be difficult to change them later). Then proceed to the next step (<bpt id="p1">*</bpt>Create an indexer<ept id="p1">*</ept>).
12. **インデクサー名**を **margies-indexer** に変更します。
13. **[スケジュール]** は **[1 回]** に設定されているままにします。
14. **[詳細設定]** オプションを展開し、**[Base-64 エンコード キー]** オプションが選択されていることを確かめます (通常、エンコード キーを使用するとインデックスの効率が向上します)。
15. このラボで作業している環境に **AI-102-AIEngineer** コードのリポジトリをまだクローンしていない場合は、次の手順に従ってクローンします。
    1. データ ソースからドキュメント メタデータ フィールドとコンテンツを抽出する
    2. コグニティブ スキルのスキルセットを実行して、追加のエンリッチされたフィールドを生成する
    3. 抽出されたフィールドをインデックスにマップする。
16. それ以外の場合は、複製されたフォルダーを Visual Studio Code で開きます。

## <a name="search-the-index"></a>インデックスを検索する

インデックスができたので、検索できます。

1. Azure Cognitive Search リソースの **[概要]** ブレードの上部で、 **[検索エクスプローラー]** を選択します。
2. 検索エクスプローラーの **[クエリ文字列]** ボックスに、「`*`」(1 つのアスタリスク) を入力し、**[検索]** を選択します。

    This query retrieves all documents in the index in JSON format. Examine the results and note the fields for each document, which contain document content, metadata, and enriched data extracted by the cognitive skills you selected.

3. クエリ文字列を `search=*&$count=true` に変更し、検索を送信します。

    今回の結果には、検索によって返されたドキュメントの数を示す **@odata.count** フィールドが結果の上部に含まれています。

4. 次のクエリ文字列を試してください：

    ```
    search=*&$count=true&$select=metadata_storage_name,metadata_author,locations
    ```

    This time the results include only the file name, author, and any locations mentioned in the document content. The file name and author are in the <bpt id="p1">**</bpt>metadata_storage_name<ept id="p1">**</ept> and <bpt id="p2">**</bpt>metadata_author<ept id="p2">**</ept> fields, which were extracted from the source document. The <bpt id="p1">**</bpt>locations<ept id="p1">**</ept> field was generated by a cognitive skill.

5. 次に、次のクエリ文字列を試してください。

    ```
    search="New York"&$count=true&$select=metadata_storage_name,keyphrases
    ```

    この検索では、検索可能なフィールドのいずれかで "ニューヨーク" に言及しているドキュメントが検索され、ドキュメント内のファイル名とキーフレーズが返されます。

6. もう 1 つのクエリ文字列を試してみましょう。

    ```
    search="New York"&$count=true&$select=metadata_storage_name&$filter=metadata_author eq 'Reviewer'
    ```

    このクエリは、"ニューヨーク" に言及している *Reviewer* によって作成されたドキュメントのファイル名を返します。

## <a name="explore-and-modify-definitions-of-search-components"></a>検索コンポーネント定義の調査と変更

検索ソリューションのコンポーネントは、Azure portal　で表示および編集できる　JSON　定義に基づいています。

ポータルを使用して検索ソリューションを作成および変更できますが、検索オブジェクトを JSON で定義し、Azure Cognitive Service REST インターフェイスを使用してそれらを作成および変更することが望ましい場合がよくあります。

### <a name="get-the-endpoint-and-key-for-your-azure-cognitive-search-resource"></a>Azure Cognitive Search リソースのエンドポイントとキーを取得する

1. Azure portal で、Azure Cognitive Search リソースの **[概要]** ページに戻ります。ページの上部で、リソースの **Url** ( **https://resource_name.search.windows.net** のようになります) を見つけてクリップボードにコピーします。
2. In Visual Studio Code, in the Explorer pane, expand the <bpt id="p1">**</bpt>22-create-a-search-solution<ept id="p1">**</ept> folder and its <bpt id="p2">**</bpt>modify-search<ept id="p2">**</ept> subfolder, and select <bpt id="p3">**</bpt>modify-search.cmd<ept id="p3">**</ept> to open it. You will use this script file to run <bpt id="p1">*</bpt>cURL<ept id="p1">*</ept> commands that submit JSON to the Azure Cognitive Service REST interface.
3. **modify-search.cmd** で、**YOUR_SEARCH_URL** プレースホルダーをクリップボードにコピーした URL に置き換えます。
4. Azure portal で、Azure Cognitive Search リソースの **[キー]** ページを表示し、**プライマリ管理者キー**をクリップボードにコピーします。
5. Visual Studio Code で、**YOUR_ADMIN_KEY** プレースホルダーをクリップボードにコピーしたキーに置き換えます。
6. 変更を **modify-search.cmd** に保存します (ただし、まだ実行しないでください)

### <a name="review-and-modify-the-skillset"></a>スキルセットを確認および変更する

1. In Visual studio Code, in the <bpt id="p1">**</bpt>modify-search<ept id="p1">**</ept> folder, open <bpt id="p2">**</bpt>skillset.json<ept id="p2">**</ept>. This shows a JSON definition for <bpt id="p1">**</bpt>margies-skillset<ept id="p1">**</ept>.
2. スキルセット定義の上部にある **cognitiveServices** オブジェクトに注意してください。このオブジェクトは、Cognitive Services リソースをスキルセットに接続するために使用されます。
3. In the Azure portal, open your Cognitive Services resource (<bpt id="p1">&lt;u&gt;</bpt>not<ept id="p1">&lt;/u&gt;</ept> your Azure Cognitive Search resource!) and view its <bpt id="p2">**</bpt>Keys<ept id="p2">**</ept> page. Then copy <bpt id="p1">**</bpt>Key 1<ept id="p1">**</ept> to the clipboard.
4. Visual Studio Code の **skillset.json** で、**YOUR_COGNITIVE_SERVICES_KEY** プレースホルダーをクリップボードにコピーした Cognitive Services キーに置き換えます。
5. Scroll through the JSON file, noting that it includes definitions for the skills you created using the Azure Cognitive Search user interface in the Azure portal. At the bottom of the list of skills, an additional skill has been added with the following definition:

    ```
    {
        "@odata.type": "#Microsoft.Skills.Text.V3.SentimentSkill",
        "defaultLanguageCode": "en",
        "name": "get-sentiment",
        "description": "New skill to evaluate sentiment",
        "context": "/document",
        "inputs": [
            {
                "name": "text",
                "source": "/document/merged_content"
            },
            {
                "name": "languageCode",
                "source": "/document/language"
            }
        ],
        "outputs": [
            {
                "name": "sentiment",
                "targetName": "sentimentLabel"
            }
        ]
    }
    ```

The new skill is named <bpt id="p1">**</bpt>get-sentiment<ept id="p1">**</ept>, and for each <bpt id="p2">**</bpt>document<ept id="p2">**</ept> level in a document, it, will evaluate the text found in the <bpt id="p3">**</bpt>merged_content<ept id="p3">**</ept> field of the document being indexed (which includes the source content as well as any text extracted from images in the content). It uses the extracted <bpt id="p1">**</bpt>language<ept id="p1">**</ept> of the document (with a default of English), and evaluates a label for the sentiment of the content. Values for the sentiment label can be "positive", "negative", "neutral", or "mixed". This label is then output as a new field named <bpt id="p1">**</bpt>sentimentLabel<ept id="p1">**</ept>.

6. **skillset.json** に加えた変更を保存します。

### <a name="review-and-modify-the-index"></a>インデックスを確認して変更する

1. In Visual studio Code, in the <bpt id="p1">**</bpt>modify-search<ept id="p1">**</ept> folder, open <bpt id="p2">**</bpt>index.json<ept id="p2">**</ept>. This shows a JSON definition for <bpt id="p1">**</bpt>margies-index<ept id="p1">**</ept>.
2. Scroll through the index and view the field definitions. Some fields are based on metadata and content in the source document, and others are the results of skills in the skillset.
3. Azure portal で定義したフィールドのリストの最後に、2 つのフィールドが追加されていることに注意してください

    ```
    {
        "name": "sentiment",
        "type": "Edm.String",
        "facetable": false,
        "filterable": true,
        "retrievable": true,
        "sortable": true
    },
    {
        "name": "url",
        "type": "Edm.String",
        "facetable": false,
        "filterable": true,
        "retrievable": true,
        "searchable": false,
        "sortable": false
    }
    ```

4. The <bpt id="p1">**</bpt>sentiment<ept id="p1">**</ept> field will be used to add the output from the <bpt id="p2">**</bpt>get-sentiment<ept id="p2">**</ept> skill that was added the skillset. The <bpt id="p1">**</bpt>url<ept id="p1">**</ept> field will be used to add the URL for each indexed document to the index, based on the <bpt id="p2">**</bpt>metadata_storage_path<ept id="p2">**</ept> value extracted from the data source. Note that index already includes the <bpt id="p1">**</bpt>metadata_storage_path<ept id="p1">**</ept> field, but it's used as the index key and Base-64 encoded, making it efficient as a key but requiring client applications to decode it if they want to use the actual URL value as a field. Adding a second field for the unencoded value resolves this problem.

### <a name="review-and-modify-the-indexer"></a>インデクサーを確認して変更する

1. In Visual studio Code, in the <bpt id="p1">**</bpt>modify-search<ept id="p1">**</ept> folder, open <bpt id="p2">**</bpt>indexer.json<ept id="p2">**</ept>. This shows a JSON definition for <bpt id="p1">**</bpt>margies-indexer<ept id="p1">**</ept>, which maps fields extracted from document content and metadata (in the <bpt id="p2">**</bpt>fieldMappings<ept id="p2">**</ept> section), and values extracted by skills in the skillset (in the <bpt id="p3">**</bpt>outputFieldMappings<ept id="p3">**</ept> section), to fields in the index.
3. In the <bpt id="p1">**</bpt>fieldMappings<ept id="p1">**</ept> list, note the mapping for the <bpt id="p2">**</bpt>metadata_storage_path<ept id="p2">**</ept> value to the base-64 encoded key field. This was created when you assigned the <bpt id="p1">**</bpt>metadata_storage_path<ept id="p1">**</ept> as the key and selected the option to encode the key in the Azure portal. Additionally, a new mapping explicitly maps the same value to the <bpt id="p1">**</bpt>url<ept id="p1">**</ept> field, but without the Base-64 encoding:

    ```
    {
        "sourceFieldName" : "metadata_storage_path",
        "targetFieldName" : "url"
    }
    
    ```

ソース ドキュメント内の他のすべてのメタデータおよびコンテンツ フィールド インデックス内の同じ名前のフィールドに暗黙的にマップされます。

4. Review the <bpt id="p1">**</bpt>ouputFieldMappings<ept id="p1">**</ept> section, which maps outputs from the skills in the skillset to index fields. Most of these reflect the choices you made in the user interface, but the following mapping has been added to map the <bpt id="p1">**</bpt>sentimentLabel<ept id="p1">**</ept> value extracted by your sentiment skill to the <bpt id="p2">**</bpt>sentiment<ept id="p2">**</ept> field you added to the index:

    ```
    {
        "sourceFieldName": "/document/sentimentLabel",
        "targetFieldName": "sentiment"
    }
    ```

### <a name="use-the-rest-api-to-update-the-search-solution"></a>REST API を使用して検索ソリューションを更新する

1. **modify-search** フォルダーを右クリックして、統合ターミナルを開きます。
2. **modify-search** フォルダーのターミナル ペインで、次のコマンドを入力して、**modify-search.cmd** スクリプトを実行します。このスクリプトは、JSON 定義を REST インターフェイスに送信し、インデックス作成を開始します。

    ```
    modify-search
    ```

3. When the script has finished, return to the <bpt id="p1">**</bpt>Overview<ept id="p1">**</ept> page for your Azure Cognitive Search resource in the Azure portal and view the <bpt id="p2">**</bpt>Indexers<ept id="p2">**</ept> page. The periodically select <bpt id="p1">**</bpt>Refresh<ept id="p1">**</ept> to track the progress of the indexing operation. It may take a minute or so to complete.

    *センチメントを評価するには大きすぎるいくつかのドキュメントに対して、いくつかの警告がある場合があります。多くの場合、感情分析は、ドキュメント全体ではなく、ページまたは文レベルで実行されます。ただし、この場合のシナリオでは、ほとんどのドキュメント (特にホテルのレビュー) は、有用なドキュメント レベルのセンチメント スコアを評価するのに十分なほど短いものです。*

### <a name="query-the-modified-index"></a>変更されたインデックスをクエリする

1. Azure Cognitive Search リソースのブレードの上部で、 **[検索エクスプローラー]** を選択します。
2. 検索エクスプローラーの **[クエリ文字列]** ボックスに、次のクエリ文字列を入力し、 **[検索]** を選択します。

    ```
    search=London&$select=url,sentiment,keyphrases&$filter=metadata_author eq 'Reviewer' and sentiment eq 'positive'
    ```

    このクエリを実行すると、*Reviewer* が作成した、*London* に言及していて正の**センチメント** ラベルが付いているすべてのドキュメント (つまり、ロンドンに言及している肯定的なレビュー) の **url**、**センチメント**、および**キーフレーズ**が取得されます。

3. **検索エクスプローラー** ページを閉じて、 **[概要]** ページに戻ります。

## <a name="create-a-search-client-application"></a>検索クライアント アプリケーションを作成する

Now that you have a useful index, you can use it from a client application. You can do this by consuming the REST interface, submitting requests and receiving responses in JSON format over HTTP; or you can use the software development kit (SDK) for your preferred programming language. In this exercise, we'll use the SDK.

> <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: You can choose to use the SDK for either <bpt id="p2">**</bpt>C#<ept id="p2">**</ept> or <bpt id="p3">**</bpt>Python<ept id="p3">**</ept>. In the steps below, perform the actions appropriate for your preferred language.

### <a name="get-the-endpoint-and-keys-for-your-search-resource"></a>検索リソースのエンドポイントとキーを取得する

1. In the Azure portal, on the <bpt id="p1">**</bpt>Overview<ept id="p1">**</ept> page for your Azure Cognitive Search resource, note the <bpt id="p2">**</bpt>Url<ept id="p2">**</ept> value, which should be similar to <bpt id="p3">**</bpt>https://<bpt id="p4">*</bpt>your_resource_name<ept id="p4">*</ept>.search.windows.net<ept id="p3">**</ept>. This is the endpoint for your search resource.
2. On the <bpt id="p1">**</bpt>Keys<ept id="p1">**</ept> page, note that there are two <bpt id="p2">**</bpt>admin<ept id="p2">**</ept> keys, and a single <bpt id="p3">**</bpt>query<ept id="p3">**</ept> key. An <bpt id="p1">*</bpt>admin<ept id="p1">*</ept> key is used to create and manage search resources; a <bpt id="p2">*</bpt>query<ept id="p2">*</ept> key is used by client applications that only need to perform search queries.

    *クライアント アプリケーションのエンドポイントとクエリ キーが必要になります。*

### <a name="prepare-to-use-the-azure-cognitive-search-sdk"></a>Azure Cognitive Search SDK を使用するための準備

1. Visual Studio Code の **[エクスプローラー]** ペインで、**22-create-a-search-solution** フォルダーを参照し、言語の設定に応じて **C-Sharp** フォルダーまたは **Python** フォルダーを展開します。
2. Right-click the <bpt id="p1">**</bpt>margies-travel<ept id="p1">**</ept> folder and open an integrated terminal. Then install the Azure Cognitive Search SDK package by running the appropriate command for your language preference:

    **C#**
    
    ```
    dotnet add package Azure.Search.Documents --version 11.1.1
    ```
    
    **Python**
    
    ```
    pip install azure-search-documents==11.0.0
    ```
    
3. **margies-travel** フォルダーの内容を表示し、構成設定用のファイルが含まれていることに注意してください
    - **C#** : appsettings.json
    - **Python**: .env

    Open the configuration file and update the configuration values it contains to reflect the <bpt id="p1">**</bpt>endpoint<ept id="p1">**</ept> and <bpt id="p2">**</bpt>query key<ept id="p2">**</ept> for your Azure Cognitive Search resource. Save your changes.

### <a name="explore-code-to-search-an-index"></a>インデックスを検索するためのコードの探索

**margies-travel** フォルダーには、検索機能を含む Web アプリケーション (Microsoft C# *ASP.NET Razor* Web アプリケーションまたは Python *Flask* アプリケーション) のコード ファイルが含まれています。

1. 選択したプログラミング言語に応じて、次のコードファイルを Web アプリケーションで開きます
    - **C#** :Pages/Index.cshtml.cs
    - **Python**: app.py
2. コード ファイルの上部にあるコメント「**Import search namespaces**」を見つけ、Azure Cognitive Search　SDK　で機能するようにインポートされた名前空間をメモします。
3. **search_query** 関数で、コメント「**Create a search client**」を見つけ、コードが Azure Cognitive Search リソースのエンドポイントとクエリ キーを使用して **SearchClient** オブジェクトを作成することに注意してください。
4. **search_query** 関数、コメント「**Submit search query**」を検索し、コードを確認して、次のオプションを使用して、指定されたテキストの検索を送信します。
    - 検索テキスト内の個々の単語を**すべて**必要とする *検索モード* が見つかりました。
    - 検索で見つかったドキュメントの総数が結果に含まれます。
    - 結果は、提供されたフィルター式に一致するドキュメントのみを含むようにフィルター処理されます。
    - 結果は、指定された並べ替え順序に並べ替えられます。
    - **metadata_author** フィールドの各離散値は、フィルタリング用の事前定義された値を表示するために使用できる *ファセット*として返されます。
    - 検索語が強調表示された **merged_content** フィールドと **imageCaption** フィールドの最大 3 つの抽出が結果に含まれます。
    - 結果には、指定されたフィールドのみが含まれます。

### <a name="explore-code-to-render-search-results"></a>検索結果をレンダリングするためのコードを探索する

Web アプリには、検索結果を処理およびレンダリングするためのコードが既に含まれています。

1. 選択したプログラミング言語に応じて、次のコードファイルを Web アプリケーションで開きます
    - **C#** :Pages/Index.cshtml
    - **Python**: templates/search.html
2. Examine the code, which renders the page on which the search results are displayed. Observe that:
    - ページは、ユーザーが新しい検索を送信するために使用できる検索フォームで始まります (Python バージョンのアプリケーションでは、このフォームは **base.html** テンプレートで定義されています)。これはページの先頭で参照されます。
    - Azure portal の Azure Cognitive Search リソースのブレードの **[概要]** ページを確認します。
        - 検索結果からドキュメントの数を取得して表示します。
        - **metadata_author** フィールドのファセット値を取得し、フィルタリングのオプション リストとして表示します。
        - 結果の並べ替えオプションのドロップダウンリストを作成します。
    - 次に、コードは検索結果を反復処理し、各結果を次のようにレンダリングします。
        - **metadata_storage_name** (ファイル名) フィールドを、**url** フィールドのアドレスへのリンクとして表示します。
        - **merged_content** フィールドと **imageCaption** フィールドで見つかった検索用語の *ハイライト* を表示して、コンテキストで検索用語を表示しやすくします。
        - **metadata_author**、**metadata_storage_size**、**metadata_storage_last_modified**、および **language** フィールドを表示します。
        - ここでは、ビジュアル インターフェイスを使用して、検索ソリューションのさまざまなコンポーネントを作成、テスト、管理、および監視できます。データソース、インデックス、インデクサー、スキルセットを含みます。
        - 最初の 5 つの **keyphrases** (ある場合) を表示します。
        - 最初の 5 つの **locations** (ある場合) を表示します。
        - 最初の 5 つの **imageTags** (ある場合) を表示します。

### <a name="run-the-web-app"></a>Web アプリの実行

 1. **margies-travel** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    flask run
    ```

2. アプリが正常に起動したときに表示されるメッセージで、実行中の Web アプリケーション ( *http://localhost:5000/* または *http://127.0.0.1:5000/* ) へのリンクをたどって、Web ブラウザで Margies Travel サイトを開きます。
3. Margie's Travel の Web サイトで、検索ボックスに「**London hotel**」と入力し、**[Search]** をクリックします。
4. Review the search results. They include the file name (with a hyperlink to the file URL), an extract of the file content with the search terms (<bpt id="p1">*</bpt>London<ept id="p1">*</ept> and <bpt id="p2">*</bpt>hotel<ept id="p2">*</ept>) emphasized, and other attributes of the file from the index fields.
5. Observe that the results page includes some user interface elements that enable you to refine the results. These include:
    - サブスクリプションに **Cognitive Services** リソースがまだない場合は、プロビジョニングする必要があります。
    - 検索ソリューションではこれを使用して、AI によって生成された分析情報でデータストア内のデータをエンリッチします。
6. **[レビュー担当者]** フィルターと **[Positive to negative]\(肯定的から否定的\)** 並べ替えオプションを選択し、 **[結果の絞り込み]** を選択します。
7. 結果がレビューのみを含むようにフィルタリングされ、センチメント ラベルで並べ替えられていることを確認します。
8. **[検索]** ボックスに、「**quiet hotel in New York**」の新しい検索を入力して、結果を確認します。
9. 次の検索用語を試してください。
    - **Tower of London** (この用語が一部のドキュメントで*キー フレーズ*として識別されていることに注意してください)。
    - **skyscraper** (この単語はどのドキュメントの実際のコンテンツにも表示されませんが、一部のドキュメントの画像用に生成された *画像のキャプション* と *画像タグ* に含まれていることに注意してください)。
    - **Mojave desert** (この用語が一部のドキュメントで*場所*として識別されていることに注意してください)。
10. Close the browser tab containing the Margie's Travel web site and return to Visual Studio Code. Then in the Python terminal for the <bpt id="p1">**</bpt>margies-travel<ept id="p1">**</ept> folder (where the dotnet or flask application is running), enter Ctrl+C to stop the app.

## <a name="more-information"></a>詳細情報

Azure Cognitive Search の詳細については、[Azure Cognitive Search のドキュメント](https://docs.microsoft.com/azure/search/search-what-is-azure-search)を参照してください。
