---
lab:
  title: 言語サービスで言語理解モデルを作成する (プレビュー)
  module: Module 5 - Creating Language Understanding Solutions
---

# <a name="create-a-language-understanding-model-with-the-language-service-preview"></a>言語サービスで言語理解モデルを作成する (プレビュー)

> <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: The conversational language understanding feature of the Language service is currently in preview, and subject to change. In some cases, model training may fail - if this happens, try again.  

言語サービスを使うと、*会話言語理解* モデルを定義することができます。これをアプリケーションから使って、ユーザーからの自然言語入力を解釈し、ユーザーの *意図* (達成したいこと) を予測し、その意図を適用する必要がある *エンティティ* を特定することができます。

たとえば、時計アプリケーション用の会話言語モデルは、次のような入力を処理することが期待される場合があります。

*What's the time in London?* (ロンドンの時刻は何時ですか?)

この種の入力は、*発話* (ユーザーが言うまたは入力する可能性のあるもの) の例です。*意図* は、特定の場所 (*エンティティ*) (この場合はロンドン) の時間を得ることです。

> <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: The task of a conversational language model is to predict the user's intent and identify any entities to which the intent applies. It is <bpt id="p1">&lt;u&gt;</bpt>not<ept id="p1">&lt;/u&gt;</ept> the job of a conversational language model to actually perform the actions required to satisfy the intent. For example, a clock application can use a conversational language model to discern that the user wants to know the time in London; but the client application itself must then implement the logic to determine the correct time and present it to the user.

## <a name="create-a-language-service-resource"></a>"言語サービス" リソースを作成する

To create a conversational language model, you need a <bpt id="p1">**</bpt>Language service<ept id="p1">**</ept> resource in a supported region. At the time of writing, only the West US 2 and West Europe regions are supported.

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. **[&#65291;リソースの作成]** ボタンを選択し、*[言語]* を検索し、次の設定で**言語サービス** リソースを作成します。

    - **機能**:会話言語理解 (プレビュー) を含む、既定の機能を使用します。 
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)*
    - **リージョン**: 米国西部 2 または西ヨーロッパ
    - **[名前]**: *一意の名前を入力します*
    - **価格レベル**: Standard (S) レベルを選択します (これは、現在 Free レベルではサポートされていない Conversational Language Understanding です)。
    - **法的条項**: 同意__ 
    - **責任ある AI 通知**: "同意"__
3. デプロイが完了するまで待ち、デプロイの詳細を表示します。

## <a name="create-a-conversational-language-understanding-project"></a>会話言語理解プロジェクトを作成する

作成リソースを作成したら、それを使って会話言語理解プロジェクトを作成できます。

1. 新しいブラウザー タブで Language Studio ポータル (`https://language.cognitive.azure.com/`) を開き、お使いの Azure サブスクリプションに関連付けられている Microsoft アカウントを使ってサインインします。

2. 言語リソースを選択するプロンプトが表示されたら、次の設定を選択します。

    - **Azure ディレクトリ**: ご利用のサブスクリプションを含む Azure ディレクトリ
    - **Azure サブスクリプション**: ご利用の Azure サブスクリプション
    - **言語リソース**: 先ほど作成した言語リソース。

3. 言語リソースの選択を求めるメッセージが表示<u>されない</u>場合、原因として、別の言語リソースが既に割り当てられていることが考えられます。その場合は、次の操作を行います。

    1. ページの上部にあるバーで、**[設定] (&#9881;)** ボタンをクリックします。
    2. **[設定]** ページで、**[リソース]** タブを表示します。
    3. 作成したばかりの言語リソースを選択し、**[リソースの切り替え]** をクリックします。
    4. ページの上部で、**[Language Studio]** をクリックして、Language Studio のホーム ページに戻ります。

4. ポータルの上部にある **[新規作成]** メニューで、**[会話言語理解]** を選択します。

5. **[プロジェクトの作成]** ダイアログ ボックスの **[基本情報の入力]** ページで、次の詳細を入力してから、 **[次へ]** をクリックします。
    - **名前**: `Clock`
    - **説明**: `Natural language clock`
    - **発話の主要言語**: 英語
    - **プロジェクトで複数の言語を有効にする**: *オフ*

7. **[確認と終了]** ページで、**[作成]** をクリックします。

## <a name="create-intents"></a>意図の作成

新しいプロジェクトで最初に行うことは、いくつかの意図を定義することです。

> **ヒント**: プロジェクトの作業中に、いくつかのヒントが表示されていたら、それを読み、 **[OK]** をクリックして閉じるか、 **[すべてスキップ]** をクリックします。

1. **[スキーマの構築]** ページの **[意図]** タブで **[&#65291; 追加]** を選び、**GetTime** という新しい意図を追加します。

2. 新しい **GetTime** 意図をクリックして編集し、ユーザー入力の例として次の発話を追加します。

    `what is the time?`

    `what's the time?`

    `what time is it?`

    `tell me the time`

3. これらの発話を追加したら、 **[変更の保存]** をクリックし、 **[スキーマの構築]** ページに戻ります。

4. 次の発話を指定して **GetDay** という別の新しい意図を追加します。

    `what day is it?`

    `what's the day?`

    `what is the day today?`

    `what day of the week is it?`

5. これらの発話を追加したら、保存して **[スキーマの構築]** ページに戻り、次の発話を指定して **GetDate** という名前の別の新しい意図を追加します。

    `what date is it?`

    `what's the date?`

    `what is the date today?`

    `what's today's date?`

6. これらの発話を追加したら、保存し、発話ページの **GetDate** フィルターをクリアして、すべての意図のすべての発話を確認できるようにします。

## <a name="train-and-test-the-model"></a>モデルのトレーニングとテスト

意図をいくつか追加したので、言語モデルをトレーニングして、ユーザー入力から正しく予測できるかどうかを確認しましょう。

1. 左側のペインで、 **[モデルのトレーニング]** ページを選択し、 **[トレーニング ジョブの開始]** を選択します。

2. **[トレーニング ジョブの開始]** ダイアログで、新しいモデルをトレーニングするオプションを選択し、**Clock** という名前を付けて、トレーニングが有効になっている場合に評価を実行するオプションを設定します。 

3. モデルのトレーニング プロセスを開始するには、 **[トレーニング]** をクリックします。

4. トレーニングが完了した場合 (数分かかることがあります)、ジョブの **[状態]** が " **トレーニング成功**" に変わります。

5. **注**: 言語サービスの会話言語理解機能は、現在プレビュー段階であり、変更される可能性があります。

    >**注**: 評価メトリックの詳細については、[ドキュメント](https://docs.microsoft.com/azure/cognitive-services/language-service/conversational-language-understanding/concepts/evaluation-metrics)を参照してください。

5. **[モデルの展開]** ページで、 **[展開の追加]** を選択します。

5. **[展開の追加]** ダイアログで、**[新しい展開名を作成する]** を選択し、「**production**」と入力します。

5. 場合によっては、モデルのトレーニングに失敗することがありますが、その場合はもう一度やり直してください。

6. モデルが展開されたら、 **[モデルのテスト]** ページで **Clock** モデルを選びます。

6. **[モデルのテスト: Clock]** ページで、 **[展開名]** ドロップダウンリストから **[production]** を選択します。  

7. 次のテキストを入力し、**[テストの実行]** をクリックします。

    `what's the time now?`

    Review the result that is returned, noting that it includes the predicted intent (which should be <bpt id="p1">**</bpt>GetTime<ept id="p1">**</ept>) and a confidence score that indicates the probability the model calculated for the predicted intent. The JSON tab shows the comparative confidence for each potential intent (the one with the highest confidence score is the predicted intent)

8. テキスト ボックスをクリアし、次のテキストを使って別のテストを実行します。

    `tell me the time`

    もう一度、予測された意図と信頼スコアを確認します。

9. 次のテキストを試します。

    `what's the day today?`

    うまくいけば、モデルは **GetDay** 意図を予測します。

## <a name="add-entities"></a>複数エンティティの追加

So far you've defined some simple utterances that map to intents. Most real applications include more complex utterances from which specific data entities must be extracted to get more context for the intent.

### <a name="add-a-learned-entity"></a>*学習済み* エンティティを追加する

最も一般的な種類のエンティティは *学習済み* エンティティであり、モデルは例に基づいてエンティティ値を識別することを学習します。

1. Language Studio の **[スキーマの構築]** ページに戻り、 **[エンティティ]** タブで **[&#65291; 追加]** を選んで新しいエンティティを追加します。

2. In the <bpt id="p1">**</bpt>Add an entity<ept id="p1">**</ept> dialog box, enter the entity name <bpt id="p2">**</bpt>Location<ept id="p2">**</ept> and ensure that <bpt id="p3">**</bpt>Learned<ept id="p3">**</ept> is selected. Then click <bpt id="p1">**</bpt>Add entity<ept id="p1">**</ept>.

3. **Location** エンティティが作成されたら、 **[スキーマの構築]** ページに戻り、 **[意図]** タブで **GetTime** 意図を選びます。

4. 次の新しい発話例を入力します。

    `what time is it in London?`

5. 発話が追加されたら、***London*** という単語を選び、表示されるドロップダウン リストで **Location** を選んで、"London" が場所の例であることを示します。

6. 別の発話例を追加します。

    `Tell me the time in Paris?`

7. 発話が追加されたら、***Paris*** という単語を選び、それを **Location** エンティティにマップします。

8. 別の発話例を追加します。

    `what's the time in New York?`

9. 発話が追加されたら、***New York*** という単語を選択し、それらを **Location** エンティティにマップします。

10. **[変更の保存]** をクリックして新しい発話を保存します。

### <a name="add-a-list-entity"></a>*リスト* エンティティを追加する

場合によっては、エンティティの有効な値を特定の用語と同義語のリストに制限できます。これは、アプリが発話内のエンティティのインスタンスを識別するのに役立ちます。

1. Language Studio の **[スキーマの構築]** ページに戻り、 **[エンティティ]** タブで **[&#65291; 追加]** を選んで新しいエンティティを追加します。

2. In the <bpt id="p1">**</bpt>Add an entity<ept id="p1">**</ept> dialog box, enter the entity name <bpt id="p2">**</bpt>Weekday<ept id="p2">**</ept> and select the <bpt id="p3">**</bpt>List<ept id="p3">**</ept> entity type. Then click <bpt id="p1">**</bpt>Add entity<ept id="p1">**</ept>.

3. On the page for the <bpt id="p1">**</bpt>Weekday<ept id="p1">**</ept> entity, in the <bpt id="p2">**</bpt>List<ept id="p2">**</ept> section, click <bpt id="p3">**</bpt>&amp;#65291; Add new list<ept id="p3">**</ept>. Then enter the following value and synonym and click <bpt id="p1">**</bpt>Save<ept id="p1">**</ept>:

    | キーの一覧表示 | シノニム|
    |-------------------|---------|
    | 土曜日 | Sun |

4. 前の手順を繰り返して、次のリスト コンポーネントを追加します。

    | 値 | シノニム|
    |-------------------|---------|
    | 月曜日 | Mon |
    | Tuesday | Tue、Tues |
    | 水曜日 | Wed、Weds |
    | Thursday | Thur、Thurs |
    | 金曜日 | Fri |
    | 土曜日 | Sat |

5. **[スキーマの構築]** ページに戻り、 **[意図]** タブで **GetDate** 意図を選びます。

6. 次の新しい発話例を入力します。

    `what date was it on Saturday?`

7. 発話が追加されたら、***Saturday*** という単語を選び、表示されるドロップダウン リストで **Weekday** を選びます。

8. 別の発話例を追加します。

    `what date will it be on Friday?`

9. 発話が追加されたら、**Friday** を **Weekday** エンティティにマップします。

10. 別の発話例を追加します。

    `what will the date be on Thurs?`

11. 発話が追加されたら、**Thurs** を **Weekday** エンティティにマップします。

12. **[変更の保存]** をクリックして新しい発話を保存します。

### <a name="add-a-prebuilt-entity"></a>*事前構築済み* エンティティを追加する

言語サービスには、会話アプリケーションでよく使われる *事前構築済み* エンティティのセットが用意されています。

1. Language Studio の **[スキーマの構築]** ページに戻り、 **[エンティティ]** タブで **[&#65291; 追加]** を選んで新しいエンティティを追加します。

2. **注**:会話言語モデルのタスクは、ユーザーの意図を予測し、意図が適用されるエンティティを特定することです。

3. **Date** エンティティのページの **[事前構築済み]** セクションで、 **[&#65291; 新しい事前構築済みの追加]** をクリックします。

4. **[事前構築済みの選択]** リストで **DateTime** を選び、 **[保存]** をクリックします。

5. **[スキーマの構築]** ページに戻り、 **[意図]** タブで **GetDay** 意図を選びます。

6. 次の新しい発話例を入力します。

    `what day was 01/01/1901?`

7. 発話が追加されたら、***01/01/1901*** を選び、表示されるドロップダウン リストで **Date** を選びます。

8. 別の発話例を追加します。

    `what day will it be on Dec 31st 2099?`

9. 発話が追加されたら、**Dec 31st 2099** を **Date** エンティティにマップします。

10. **[変更の保存]** をクリックして新しい発話を保存します。

### <a name="retrain-the-model"></a>モデルの再トレーニング

スキーマを変更したので、モードを再トレーニングして再テストする必要があります。

1. **[モデルのトレーニング]** ページで、 **[トレーニング ジョブの開始]** を選択します。

1. 意図を満たすために必要なアクションを実際に実行することは、会話言語モデルの仕事では<u>ありません</u>。

2. トレーニングが完了した場合、ジョブの **[状態]** が " **トレーニング成功**" に更新されます。 

2. たとえば、時計アプリケーションは会話言語モデルを使用して、ユーザーがロンドンの時刻を知りたいことを識別できます。ただし、クライアント アプリケーション自体は、正しい時刻を決定してユーザーに提示するロジックを実装する必要があります。

3. **[モデルの展開]** ページで、 **[展開の追加]** を選択します。

3. **[展開の追加]** ダイアログで、**[既存の展開名をオーバーライドする]** を選択し、「**production**」を選択します。

3. Select the <bpt id="p1">**</bpt>Clock<ept id="p1">**</ept> model and then click <bpt id="p2">**</bpt>Submit<ept id="p2">**</ept> to deploy it. This may take some time.

4. モデルが展開されたら、 **[モデルのテスト]** ページで **Clock** モデルを選び、**production** 展開を選択して、次のテキストでテストします。

    `what's the time in Edinburgh?`

5. 返された結果を確認します。**GetTime** 意図と、テキスト値が "Edinburgh" の **Location** エンティティが予測されるはずです。

6. 次の発話をテストしてみてください。

    `what time is it in Tokyo?`
    
    `what date is it on Friday?`

    `what's the date on Weds?`

    `what day was 01/01/2020?`

    `what day will Mar 7th 2030 be?`

## <a name="use-the-model-from-a-client-app"></a>クライアント アプリからモデルを使う

In a real project, you'd iteratively refine intents and entities, retrain, and retest until you are satisfied with the predictive performance. Then, when you've tested it and are satisfied with its predictive performance, you can use it in a client app by calling its REST interface. In this exercise, you'll use the <bpt id="p1">*</bpt>curl<ept id="p1">*</ept> utility to call the REST endpoint for your model.

1. 会話言語モデルを作成するには、サポートされているリージョンの**言語サービス** リソースが必要です。

2. **[予測 URL の取得]** ダイアログ ボックスには、予測エンドポイントの URL とサンプル要求 (ヘッダーに Language リソースのキーを指定し、要求データにクエリと言語を含めてエンドポイントに HTTP POST 要求を送信する **curl** コマンドで構成されています) が表示されることに注目してください。

3. このサンプル要求をコピーし、好みのテキスト エディター (メモ帳など) に貼り付けます。

4. 次のプレースホルダーを置き換えます。
    - **YOUR_QUERY_HERE**: *What's the time in Sydney*
    - **QUERY_LANGUAGE_HERE**: *EN*

    コマンドは次のようなコードになります。

    ```
    curl -X POST "https://some-name.cognitiveservices.azure.com/language/:analyze-conversations?projectName=Clock&deploymentName=production&api-version=2021-11-01-preview" -H "Ocp-Apim-Subscription-Key: 0ab1c23de4f56..."  -H "Apim-Request-Id: 9zy8x76wv5u43...." -H "Content-Type: application/json" -d "{\"verbose\":true,\"query\":\"What's the time in Sydney?\",\"language\":\"EN\"}"
    ```

5. コマンド プロンプト (Windows) または bash シェル (Linux/Mac) を開きます。

6. 編集した curl コマンドをコピーし、コマンドライン インターフェイスに貼り付けて実行します。

7. 結果の JSON を確認します。これには予測された意図とエンティティが含まれています。

    ```
    {"query":"What's the time in Sydney?","prediction":{"topIntent":"GetTime","projectKind":"conversation","intents":[{"category":"GetTime","confidenceScore":0.9998859},{"category":"GetDate","confidenceScore":9.8372206E-05},{"category":"GetDay","confidenceScore":1.5763446E-05}],"entities":[{"category":"Location","text":"Sydney","offset":19,"length":6,"confidenceScore":1}]}}
    ```

8. モデルから返された JSON 応答を確認し、予測された最高スコアの意図が **GetTime** であることを確認します。

9. curl コマンドのクエリを `What's today's date?` に変更してから実行し、結果の JSON を確認します。

10. 次のクエリを試します。

    `What day will Jan 1st 2050 be?`

    `What time is it in Glasgow?`

    `What date will next Monday be?`

## <a name="export-the-project"></a>プロジェクトをエクスポートする

この記事の執筆時点で、米国西部 2 と西ヨーロッパのリージョンのみがサポートされています。

1. In Language Studio, on the <bpt id="p1">**</bpt>Projects<ept id="p1">**</ept> page, select the <bpt id="p2">**</bpt>Clock<ept id="p2">**</ept> project. Don't click <bpt id="p1">**</bpt>Clock<ept id="p1">**</ept>, select the circle icon to select the Clock project.

2. **[&#x2913; エクスポート]** ボタンをクリックします。

3. 生成された **Clock.json** ファイルを (任意の場所に) 保存します。

4. ダウンロードしたファイルを好みのコード エディター (Visual Studio Code など) で開き、プロジェクトの JSON 定義を確認します。

## <a name="more-information"></a>詳細情報

**言語**サービスを使って会話言語理解ソリューションを作成する方法の詳細については、[言語サービスのドキュメント](https://docs.microsoft.com/azure/cognitive-services/language-service/conversational-language-understanding/overview)を参照してください。