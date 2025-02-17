---
lab:
  title: 言語サービスで言語理解モデルを作成する (プレビュー)
  module: Module 5 - Creating Language Understanding Solutions
---

# <a name="create-a-language-understanding-model-with-the-language-service-preview"></a>言語サービスで言語理解モデルを作成する (プレビュー)

> **注**: 言語サービスの会話言語理解機能は、現在プレビュー段階であり、変更される可能性があります。 場合によっては、モデルのトレーニングに失敗することがありますが、その場合はもう一度やり直してください。  

言語サービスを使うと、*会話言語理解* モデルを定義することができます。これをアプリケーションから使って、ユーザーからの自然言語入力を解釈し、ユーザーの *意図* (達成したいこと) を予測し、その意図を適用する必要がある *エンティティ* を特定することができます。

たとえば、時計アプリケーション用の会話言語モデルは、次のような入力を処理することが期待される場合があります。

*What's the time in London?* (ロンドンの時刻は何時ですか?)

この種の入力は、*発話* (ユーザーが言うまたは入力する可能性のあるもの) の例です。*意図* は、特定の場所 (*エンティティ*) (この場合はロンドン) の時間を得ることです。

> **注**: 言語理解アプリのタスクは、ユーザーの意図を予測し、意図が適用されるエンティティを特定することです。意図を満たすために必要なアクションを実際に実行することは、その仕事では<u>ありません</u>。たとえば、時計アプリケーションは言語アプリを使用して、ユーザーがロンドンの時刻を知りたいことを識別できます。ただし、クライアント アプリケーション自体は、正しい時刻を決定してユーザーに提示するロジックを実装する必要があります。

## <a name="create-a-language-service-resource"></a>"言語サービス" リソースを作成する

会話型言語モデルを作成するには、サポートされている地域の言語サービスリソースが必要です。執筆時点では、West US 2を含むいくつかのリージョンでサポートされています。

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. **[&#65291;リソースの作成]** ボタンを選択し、*[言語]* を検索し、次の設定で**言語サービス** リソースを作成します。

    - **機能**:会話言語理解 (プレビュー) を含む、既定の機能を使用します。 
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)*
    - **リージョン**: 米国西部 2
    - **[名前]**: *一意の名前を入力します*
    - **価格レベル**: Standard (S) レベルを選択します (現在 Conversational Language Understanding は Free レベルではサポートされていません)。
    - **責任ある AI 通知**: 同意
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

1. **[スキーマ定義]** ページの **[意図]** タブで **[&#65291; 追加]** を選び、**GetTime** という新しい意図を追加します。

2. 新しい **GetTime** 意図をクリックして編集し、ユーザー入力の例として次の発話を追加します。

    `what is the time?`

    `what's the time?`

    `what time is it?`

    `tell me the time`

3. これらの発話を追加したら、 **[変更の保存]** をクリックし、 **[スキーマ定義]** ページに戻ります。

4. 次の発話を指定して **GetDay** という別の新しい意図を追加します。

    `what day is it?`

    `what's the day?`

    `what is the day today?`

    `what day of the week is it?`

5. これらの発話を追加したら、保存して **[スキーマ定義]** ページに戻り、次の発話を指定して **GetDate** という名前の別の新しい意図を追加します。

    `what date is it?`

    `what's the date?`

    `what is the date today?`

    `what's today's date?`

6. これらの発話を追加したら、保存し、発話ページの **GetDate** フィルターをクリアして、すべての意図のすべての発話を確認できるようにします。

## <a name="train-and-test-the-model"></a>モデルのトレーニングとテスト

意図をいくつか追加したので、言語モデルをトレーニングして、ユーザー入力から正しく予測できるかどうかを確認しましょう。

1. 左側のペインで、 **[トレーニング ジョブ]** ページを選択し、 **[トレーニング ジョブを開始する]** を選択します。

2. **[トレーニング ジョブを開始する]** ダイアログで、新しいモデルをトレーニングするオプションを選択し、**Clock** という名前を付けて、トレーニングが有効になっている場合に評価を実行するオプションを設定します。 

3. モデルのトレーニング プロセスを開始するには、 **[トレーニング]** をクリックします。

4. トレーニングが完了した場合 (数分かかることがあります)、ジョブの **[状態]** が " **トレーニング成功**" に変わります。

5. **[モデルの詳細の表示]** ページを選択し、**Clock** モデルを選択します。 全体と意図ごとの評価メトリック ( *"精度"* 、 *"再現率"* 、 *"F1 スコア"* ) と、トレーニング時に行った評価で生成された *"混同行列"* を確認します (サンプル発話数が少ないため、すべての意図が結果に含まれていない可能性がある点に注意してください)。

    >**注**: 評価メトリックの詳細については、[ドキュメント](https://docs.microsoft.com/azure/cognitive-services/language-service/conversational-language-understanding/concepts/evaluation-metrics)を参照してください。

5. **[モデルのデプロイ]** ページで、 **[デプロイの追加]** を選択します。

5. **[デプロイの追加]** ダイアログで、**[新しいデプロイ名を作成する]** を選択し、「**production**」と入力します。

5. モデルで**Clock**を選択し、デプロイをクリックします。

6. モデルが展開されたら、 **[モデルのテスト中]** ページで **Clock** モデルを選びます。

6. **[モデルのテスト中]** ページで、 **[デプロイ名]** ドロップダウンリストから **[production]** を選択します。  

7. 次のテキストを入力し、**[テストを実行する]** をクリックします。

    `what's the time now?`

    返された結果を確認し、予測されたインテント と、予測されたインテントに対してモデルが計算した確率を示す信頼度スコアが含まれていることに注目しましょう。JSONタブには、それぞれの潜在的なインテントに対する比較信頼度が表示されます（信頼度のスコアが最も高いものが予測されるインテントです）。

8. テキスト ボックスをクリアし、次のテキストを使って別のテストを実行します。

    `tell me the time`

    もう一度、予測された意図と信頼スコアを確認します。

9. 次のテキストを試します。

    `what's the day today?`

    うまくいけば、モデルは **GetDay** 意図を予測します。

## <a name="add-entities"></a>複数エンティティの追加

ここまでで、インテントに対応するいくつかの簡単な発話を定義しました。実際のアプリケーションの多くは、より複雑な発話を含んでおり、そこから特定のデータエンティティを抽出して、インテントのコンテキストをより深く理解する必要があります。

### <a name="add-a-learned-entity"></a>*学習済み* エンティティを追加する

最も一般的な種類のエンティティは *学習済み* エンティティであり、モデルは例に基づいてエンティティ値を識別することを学習します。

1. Language Studio の **[スキーマ定義]** ページに戻り、 **[エンティティ]** タブで **[&#65291; 追加]** を選んで新しいエンティティを追加します。

2. **「エンティティの作成」** ダイアログボックスで、**Location** という名前の **学習済み(Learned)** エンティティを作成します。

3. **Location** エンティティが作成されたら、 **[スキーマ定義]** ページに戻り、 **[意図]** タブで **GetTime** 意図を選びます。

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

1. Language Studio の **[スキーマ定義]** ページに戻り、 **[エンティティ]** タブで **[&#65291; 追加]** を選んで新しいエンティティを追加します。

2. Add an entity ダイアログボックスで、エンティティ名に **Weekday** と入力し、リストタイプを選択し、追加します。

3. Weekday エンティティのページで、リストセクションの 新しいリストの追加 をクリックします。次に、次の値と類義語を入力し、[保存]をクリックします。

    | キーの一覧表示 | シノニム|
    |-------------------|---------|
    | Sunday | Sun |

4. 前の手順を繰り返して、次のリスト コンポーネントを追加します。

    | 値 | 類義語 |
    |-------------------|---------|
    | Monday | Mon |
    | Tuesday | Tue、Tues |
    | Wednesday | Wed、Weds |
    | Thursday | Thur、Thurs |
    | Friday | Fri |
    | Saturday | Sat |

5. **[スキーマ定義]** ページに戻り、 **[意図]** タブで **GetDate** 意図を選びます。

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

1. Language Studio の **[スキーマ定義]** ページに戻り、 **[エンティティ]** タブで **[&#65291; 追加]** を選んで新しいエンティティを追加します。

2. **[エンティティの追加]** ダイアログ ボックスで、エンティティ名 **Date** を入力し、エンティティの種類として **[事前構築済み]** を選びます。 次に、 **[エンティティの追加]** をクリックします。

3. **Date** エンティティのページの **[事前構築済み]** セクションをクリックします。

4. エンティティ名に **Date** と入力し、 追加します。

4. **Date**エンティティのページで、**新しい事前構築の追加**をクリックし、**DateTime**を選択し、保存します。

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

1. **[トレーニング ジョブ]** ページで、 **[トレーニング ジョブの開始]** を選択します。

1. [トレーニング ジョブの開始] ダイアログで、既存のモデルを上書きするオプションを選択し、Clockモデルを指定します。テストセットの自動分割オプションが選択されていることを確認し、[トレーニング] をクリックしてモデルをトレーニングします。既存のモデルを上書きすることを確認します。

2. トレーニングが完了した場合、ジョブの **[状態]** が " **トレーニング成功**" に更新されます。 

2. **[モデルのパフォーマンス]** ページを選択し、**Clock** モデルを選択します。 次に、全体、エンティティごと、意図ごとの評価メトリック ( *"精度"* 、 *"再現率"* 、 *"F1 スコア"* ) と、トレーニング時に行った評価で生成された *"混同行列"* を確認します (サンプル発話数が少ないため、すべての意図が結果に含まれていない可能性がある点に注意してください)。

3. **[モデルのデプロイ]** ページで、 **[デプロイの追加]** を選択します。

3. **[デプロイの追加]** ダイアログで、**[既存のデプロイ名を上書きする]** を選択し、「**production**」を選択します。

3. Clockモデルを選択し、デプロイをクリックするとデプロイされます。これには多少時間がかかる場合があります。

4. モデルが展開されたら、 **[デプロイのテスト中]** ページで **production** デプロイを選択して、次のテキストでテストします。

    `what's the time in Edinburgh?`

5. 返された結果を確認します。**GetTime** 意図と、テキスト値が "Edinburgh" の **Location** エンティティが予測されるはずです。

6. 次の発話をテストしてみてください。

    `what time is it in Tokyo?`
    
    `what date is it on Friday?`

    `what's the date on Weds?`

    `what day was 01/01/2020?`

    `what day will Mar 7th 2030 be?`

## <a name="use-the-model-from-a-client-app"></a>クライアント アプリからモデルを使う

実際のプロジェクトでは、予測性能に満足できるまで、インテントとエンティティを繰り返し改良し、再トレーニング、再テストを行うことになります。そして、テストして予測性能に満足したら、そのRESTインターフェースを呼び出して、クライアントアプリで使用することができます。この演習では、curlユーティリティを使用して、モデルの REST エンドポイントを呼び出すことにします。

1. Language Studio の[モデルのデプロイ]ページで、productionを選択します。次に、[予測 URL を取得] をクリックします。

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

Language Studio を使って言語理解モデルを開発およびテストできますが、DevOps のソフトウェア開発プロセスでは、継続的インテグレーションとデリバリー (CI/CD) パイプラインに含めることができるプロジェクトのソース制御定義を維持する必要があります。 コード スクリプトで Language REST API を使ってモデルを作成およびトレーニング *できます* が、より簡単な方法は、ポータルを使ってモデルを作成し、それを *.json* ファイルとしてエクスポートし、別の言語サービス インスタンスにインポートして再トレーニングできるようにすることです。 このアプローチにより、モデルの移植性と再現性を維持しながら、Language Studio ビジュアル インターフェイスの生産性のメリットを活用できます。

1. Language Studio の **[プロジェクト]** ページで **Clock** プロジェクトを選びます。 **Clock** をクリックしないで、円アイコンを選択して Clock プロジェクトを選択します。

2. **[&#x2913; エクスポート]** ボタンをクリックします。

3. 生成された **Clock.json** ファイルを (任意の場所に) 保存します。

4. ダウンロードしたファイルを好みのコード エディター (Visual Studio Code など) で開き、プロジェクトの JSON 定義を確認します。

## <a name="more-information"></a>詳細情報

**言語**サービスを使って会話言語理解ソリューションを作成する方法の詳細については、[言語サービスのドキュメント](https://docs.microsoft.com/azure/cognitive-services/language-service/conversational-language-understanding/overview)を参照してください。
