---
lab:
  title: Language Understanding アプリを作成する
  module: Module 5 - Creating Language Understanding Solutions
---

# <a name="create-a-language-understanding-app"></a>Language Understanding アプリを作成する

Language Understanding サービスを使用すると、アプリケーションがユーザーからの自然言語入力を解釈し、ユーザーの*意図* (達成したいこと) を予測し、意図を適用する必要がある*エンティティ*を特定するために使用できる言語モデルをカプセル化するアプリを定義できます。

たとえば、時計アプリケーション用の言語理解アプリは、次のような入力を処理することが期待される場合があります。

*What's the time in London?* (ロンドンの時刻は何時ですか?)

この種の入力は、*発話* (ユーザーが言うまたは入力する可能性のあるもの) の例です。*意図* は、特定の場所 (*エンティティ*) (この場合はロンドン) の時刻を得ることです。

> <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: The task of the language understanding app is to predict the user's intent, and identify any entities to which the intent applies. It is <bpt id="p1">&lt;u&gt;</bpt>not<ept id="p1">&lt;/u&gt;</ept> its job to actually perform the actions required to satisfy the intent. For example, the clock application can use a language app to discern that the user wants to know the time in London; but the client application itself must then implement the logic to determine the correct time and present it to the user.

## <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

If you have not already cloned <bpt id="p1">**</bpt>AI-102-AIEngineer<ept id="p1">**</ept> code repository to the environment where you're working on this lab, follow these steps to do so. Otherwise, open the cloned folder in Visual Studio Code.

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git:Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるダイアログが表示された場合は、 **[今はしない]** を選択します。

## <a name="create-language-understanding-resources"></a>Language Understanding リソースの作成

Language Understanding サービスを使用するには、次の 2 種類のリソースが必要です。

- An <bpt id="p1">*</bpt>authoring<ept id="p1">*</ept> resource: used to define, train, and test the language understanding app. This must be a <bpt id="p1">**</bpt>Language Understanding - Authoring<ept id="p1">**</ept> resource in your Azure subscription.
- A <bpt id="p1">*</bpt>prediction<ept id="p1">*</ept> resource: used to publish your language understanding app and handle requests from client applications that use it. This can be either a <bpt id="p1">**</bpt>Language Understanding<ept id="p1">**</ept> or <bpt id="p2">**</bpt>Cognitive Services<ept id="p2">**</ept> resource in your Azure subscription.

     > <bpt id="p1">**</bpt>Important<ept id="p1">**</ept>: Authoring resources must be created in one of three <bpt id="p2">*</bpt>regions<ept id="p2">*</ept> (Europe, Australia, or US). Language Understanding apps created in European or Australian authoring resources can only be deployed to prediction resources in Europe or Australia respectively; models created in US authoring resources can be deployed to prediction resources in any Azure location other than Europe and Australia. See the <bpt id="p1">[</bpt>authoring and publishing regions documentation<ept id="p1">](https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-regions)</ept> for details about matching authoring and prediction locations.

言語理解のオーサリングおよび予測リソースをまだ持っていない場合：

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. **[&#65291;リソースの作成]** ボタンを選択して、「*language understanding*」を検索し、次の設定を使用して **Language Understanding** リソースを作成します。

    *Language Understanding (Azure Cognitive Services) では<u>なく</u>、**Language Understanding** を選択していることを確認してください*

    - **[Create option](作成オプション)**: 両方
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)*
    - **[名前]**: *一意の名前を入力します*
    - **[作成場所]**: *希望の場所を選択します*
    - **[価格レベルを作成しています]**: F0
    - **予測の場所**: *作成場所と同じです*
    - **[予測価格レベル]**: F0
3. Wait for the resources to be created, and note that two Language Understanding resources are provisioned; one for authoring, and another for prediction. You can view both of these by navigating to the resource group where you created them. If you select <bpt id="p1">**</bpt>Go to resource<ept id="p1">**</ept>, it will open the <bpt id="p2">*</bpt>authoring<ept id="p2">*</ept> resource.

## <a name="create-a-language-understanding-app"></a>Language Understanding アプリを作成する

オーサリング リソースを作成したので、それを使用して Language Understanding アプリを作成できます。

1. ブラウザーの新しいタブで、`https://www.luis.ai` の Language Understanding ポータルを開きます。
2. Sign in using the Microsoft account associated with your Azure subscription. If this is the first time you have signed into the Language Understanding portal, you may need to grant the app some permissions to access your account details. Then complete the <bpt id="p1">*</bpt>Welcome<ept id="p1">*</ept> steps by selecting your Azure subscription and the authoring resource you just created.

    > **注**: アカウントが異なるディレクトリ内の複数のサブスクリプションに関連付けられている場合は、Language Understanding リソースをプロビジョニングしたサブスクリプションを含むディレクトリに切り替える必要がある場合があります。

3. **注**: 言語理解アプリのタスクは、ユーザーの意図を予測し、意図が適用されるエンティティを特定することです。
    - **名前**: Clock
    - **カルチャ**: 英語 (*このオプションが使用できない場合は空白のままにします*)
    - **説明**: Natural language clock (自然言語の時計)
    - **予測リソース**: *Language Understanding 予測リソース*

    **Clock** アプリが自動的に開かない場合は、それを開いてください。
    
    効果的な Language Understanding アプリを作成するためのヒントが表示されたパネルが表示された場合、そのパネルを閉じます。

## <a name="create-intents"></a>意図の作成

新しいアプリで最初に行うことは、いくつかの意図を定義することです。

1. **[意図]** ページで、 **[&#65291; 作成]** を選択し、**GetTime** という名前の新しい意図を作成します。
2. **GetTime** 意図で、ユーザー入力の例として次の発話を追加します。

    *what is the time?* (時刻は何時ですか)

    *what time is it?* (何時ですか)

3. これらの発話を追加したら、 **[意図]** ページに戻り、次の発話を含む **GetDay** という名前の別の新しい意図を追加します。

    *what is the day today?* (今日の曜日は何ですか)

    *what day is it?* (今日は何曜日ですか)

4. これらの発話を追加したら、 **[意図]** ページに戻り、次の発話を含む **GetDate** という名前の別の新しい意図を追加します。

    *what is the date today?* (今日の日付は何ですか)

    *what date is it?* (今日は何日ですか)

5. 意図を満たすために必要なアクションを実際に実行することは、その仕事では<u>ありません</u>。
6. 次の発話を **None** 意図に追加します。

    *hello*

    *goodbye*

## <a name="train-and-test-the-app"></a>アプリの訓練およびテスト

意図をいくつか追加したので、アプリをトレーニングして、ユーザー入力から正しく予測できるかどうかを確認しましょう。

1. ポータルの右上で、 **[トレーニング]** を選択してアプリをトレーニングします。
2. アプリがトレーニングされたら、 **[テスト]** を選択して [テスト] パネルを表示し、次のテスト発話を入力します。

    *what's the time now?* (今何時ですか)

    返された結果を確認します。予測された意図 (**GetTime** であるはずです) と、予測された意図に対してモデルが計算した確率を示す信頼スコアが含まれていることに注意してください。

3. 次のテスト発話を試してください。

    *tell me the time* (時刻を教えて下さ)

    もう一度、予測された意図と信頼スコアを確認します。

4. 次のテスト発話を試してください。

    *what's today?* (今日は何曜日ですか)

    うまくいけば、モデルは **GetDay** 意図を予測します。

5. 最後に、このテスト発話を試してください。

    *hi*

    これにより、**None** 意図が返されます。

6. [テスト] パネルを閉じます。

## <a name="add-entities"></a>複数エンティティの追加

たとえば、時計アプリケーションは言語アプリを使用して、ユーザーがロンドンの時刻を知りたいことを識別できます。ただし、クライアント アプリケーション自体は、正しい時刻を決定してユーザーに提示するロジックを実装する必要があります。

### <a name="add-a-machine-learned-entity"></a>*機械学習*エンティティを追加する

最も一般的な種類のエンティティは*機械学習エンティティ*であり、アプリは例に基づいてエンティティ値を識別することを学習します。

1. **[エンティティ]** ページで、 **[&#65291; 作成]** を選択し、新しいエンティティを作成します。
2. **[エンティティの作成]** ダイアログ ボックスで、**Location** という名前の**機械学習**エンティティを作成します。
3. **Location** エンティティが作成されたら、 **[意図]** ページに戻り、**GetTime** 意図を選択します。
4. 次の新しい発話例を入力します。

    *what time is it in London?* (ロンドンの時刻は何時ですか)

5. 発話が追加されたら、***london*** という単語を選び、表示されるドロップダウン リストで **Location** を選んで、"london" が場所の例であることを示します。
6. 別の発話例を追加します。

    *what is the current time in New York?* (ニューヨークの時刻は何時ですか)

7. 発話が追加されたら、***new york*** という単語を選択し、それらを **Location** エンティティにマップします。

### <a name="add-a-list-entity"></a>*リスト* エンティティを追加する

場合によっては、エンティティの有効な値を特定の用語と同義語のリストに制限できます。これは、アプリが発話内のエンティティのインスタンスを識別するのに役立ちます。

1. **[エンティティ]** ページで、 **[&#65291; 作成]** を選択し、新しいエンティティを作成します。
2. **[エンティティの作成]** ダイアログボックスで、**Weekday** という名前の**リスト** エンティティを作成します。
3. 次の**正規化**された値と**シノニム**を追加します。

    | 正規化された値 | シノニム|
    |-------------------|---------|
    | sunday | sun |
    | monday | mon |
    | tuesday | tue |
    | wednesday | wed |
    | thursday | thu |
    | friday | fri |
    | saturday | sat |

3. **Weekday** エンティティが作成されたら、 **[意図]** ページに戻り、**GetDate** 意図を選択します。
4. 次の新しい発話例を入力します。

    *what date was it on Saturday?* (土曜日は何日でしたか)

5. When the utterance has been added, verify that <bpt id="p1">**</bpt>saturday<ept id="p1">**</ept> has been automatically mapped to the <bpt id="p2">**</bpt>Weekday<ept id="p2">**</ept> entity. If not, select the word <bpt id="p1">***</bpt>saturday<ept id="p1">***</ept>, and in the drop-down list that appears, select <bpt id="p2">**</bpt>Weekday<ept id="p2">**</ept>.
6. 別の発話例を追加します。

    *what date will it be on Friday?* (金曜日は何日ですか)

7. 発話が追加されたら、**friday** が **Weekday** エンティティにマップされていることを確認します。

### <a name="add-a-regex-entity"></a>*正規表現エ*ンティティを追加する

Sometimes, entities have a specific format, such as a serial number, form code, or date. You can define a regular expression (<bpt id="p1">*</bpt>regex<ept id="p1">*</ept>) that describes an expected format to help your app identify matching entity values.

1. **[エンティティ]** ページで、 **[&#65291; 作成]** を選択し、新しいエンティティを作成します。
2. **[エンティティの作成]** ダイアログ ボックスで、次の正規表現を使用して **Date** という名前の**正規表現**エンティティを作成します。

    ```
    [0-9]{2}/[0-9]{2}/[0-9]{4}
    ```

    > このラボで作業している環境に **AI-102-AIEngineer** コードのリポジトリをまだクローンしていない場合は、次の手順に従ってクローンします。

3. **Date** エンティティが作成されたら、 **[意図]** ページに戻り、**GetDay** 意図を選択します。
4. 次の新しい発話例を入力します。

    *what day was 01/01/1901?* (1901 年 1 月 1 日は何曜日でしたか)

5. それ以外の場合は、複製されたフォルダーを Visual Studio Code で開きます。
6. 別の発話例を追加します。

    *what day will it be on 12/12/2099?* (2099 年 12 月 12 日は何曜日ですか)

7. 発話が追加されたら、**12/12/2099** が **Date** エンティティにマップされていることを確認します。

### <a name="retrain-the-app"></a>アプリを再トレーニングする

言語モデルを変更したので、アプリを再トレーニングして再テストする必要があります。

1. ポータルの右上で、 **[トレーニング]** を選択してアプリを再トレーニングします。
2. アプリがトレーニングされたら、 **[テスト]** を選択して [テスト] パネルを表示し、次のテスト発話を入力します。

    *what's the time in Edinburgh?* (エジンバラの時刻は何時ですか)

3. Review the result that is returned, which should hopefully predict the <bpt id="p1">**</bpt>GetTime<ept id="p1">**</ept> intent. Then select <bpt id="p1">**</bpt>Inspect<ept id="p1">**</ept> and in the additional inspection panel that is displayed, examine the <bpt id="p2">**</bpt>ML entities<ept id="p2">**</ept> section. The model should have predicted that "edinburgh" is an instance of a <bpt id="p1">**</bpt>Location<ept id="p1">**</ept> entity.
4. 次の発話をテストしてみてください。

    *what date is it on Friday?* (金曜日は何日ですか)

    *what's the date on Thu?* (木曜日は何日ですか)

    *what was the day on 01/01/2020?* (2020 年 1 月 1 日は何曜日ですか)

5. テストが終了したら、検査パネルを閉じますが、テスト パネルは開いたままにしておきます。

## <a name="perform-batch-testing"></a>バッチ テストを実行する

テスト ペインを使用して個々の発話をインタラクティブにテストできますが、より複雑な言語モデルの場合は、通常、*バッチ テスト*を実行する方が効率的です。

1. In Visual Studio Code, open the <bpt id="p1">**</bpt>batch-test.json<ept id="p1">**</ept> file in the <bpt id="p2">**</bpt>09-luis-app<ept id="p2">**</ept> folder. This file consists of a JSON document that contains multiple test cases for the clock language model you created.
2. In the Language Understanding portal, in the Test panel, select <bpt id="p1">**</bpt>Batch testing panel<ept id="p1">**</ept>. Then select <bpt id="p1">**</bpt>&amp;#65291; Import<ept id="p1">**</ept> and import the <bpt id="p2">**</bpt>batch-test.json<ept id="p2">**</ept> file, assigning the name <bpt id="p3">**</bpt>clock-test<ept id="p3">**</ept>.
3. [バッチ テスト] パネルで、**clock-test** テストを実行します。
4. テストが完了したら、 **[結果を表示]** を選択します。
5. On the results page, view the confusion matrix that represents the prediction results. It shows true positive, false positive, true negative, and false negative predictions for the intent or entity that is selected in the list on the right.

    ![Language Understanding バッチ テストの混同行列](./images/luis-confusion-matrix.jpg)

    > <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: Each utterance is scored as <bpt id="p2">*</bpt>positive<ept id="p2">*</ept> or <bpt id="p3">*</bpt>negative<ept id="p3">*</ept> for each intent - so for example "what time is it?" should be scored as <bpt id="p1">*</bpt>positive<ept id="p1">*</ept> for the <bpt id="p2">**</bpt>GetTime<ept id="p2">**</ept> intent, and <bpt id="p3">*</bpt>negative<ept id="p3">*</ept> for the <bpt id="p4">**</bpt>GetDate<ept id="p4">**</ept> intent. The points on the confusion matrix show which utterances were predicted correctly (<bpt id="p1">*</bpt>true<ept id="p1">*</ept>) and incorrectly (<bpt id="p2">*</bpt>false<ept id="p2">*</ept>) as <bpt id="p3">*</bpt>positive<ept id="p3">*</ept> and <bpt id="p4">*</bpt>negative<ept id="p4">*</ept> for the selected intent.

6. With the <bpt id="p1">**</bpt>GetDate<ept id="p1">**</ept> intent selected, select any of the points on the confusion matrix to see the details of the prediction - including the utterance and the confidence score for the prediction. Then select the <bpt id="p1">**</bpt>GetDay<ept id="p1">**</ept>, <bpt id="p2">**</bpt>GetTime<ept id="p2">**</ept> and <bpt id="p3">**</bpt>None<ept id="p3">**</ept> intents and view their prediction results. The app should have done well at predicting the intents correctly.

    > **注**: ユーザー インターフェイスでは、以前に選択したポイントがクリアされない場合があります。

7. Select the <bpt id="p1">**</bpt>Location<ept id="p1">**</ept> entity and view the prediction results in the confusion matrix. In particular, note the predictions that were <bpt id="p1">*</bpt>false negatives<ept id="p1">*</ept> - these were cases where the app failed to detect the specified location in the utterance, indicating that you may need to add more sample utterances to the intents and retrain the model.
8. [バッチ テスト] パネルを閉じます。

## <a name="publish-the-app"></a>アプリの発行

In a real project, you'd iteratively refine intents and entities, retrain, and retest until you are satisfied with the predictive performance. Then, you can publish the app for client applications to use.

1. Language Understanding ポータルの右上にある **[公開]** を選択します。
2. **[本番スロット]** を選択し、アプリを公開します。
3. 公開が完了したら、Language Understanding ポータルの上部にある **[管理]** を選択します。
4. *オーサリング* リソース: Language Understanding アプリの定義、トレーニング、およびテストに使用されます。
5. Azure サブスクリプションの **Language Understanding - オーサリング** リソースである必要があります。
6. In Visual Studio Code, in the <bpt id="p1">**</bpt>09-luis-app<ept id="p1">**</ept> folder, select the <bpt id="p2">**</bpt>GetIntent.cmd<ept id="p2">**</ept> batch file and view the code it contains. This command-line script uses cURL to call the Language Understanding REST API for the specified application and prediction endpoint.
7. スクリプト内のプレースホルダー値を、**アプリ ID**、**エンドポイント URL**、Language Understanding アプリの**プライマリ キー**または**セカンダリ キー**のいずれかに置き換えます。次に、更新したファイルを保存します。
8. *予測*リソース: Language Understanding アプリを公開し、それを使用するクライアント アプリケーションからのリクエストを処理するために使用されます。

    ```
    GetIntent "What's the time?"
    ```

9. アプリから返された JSON 応答を確認します。これは、入力に対して予測された最高スコアの意図 (**GetTime** のはずです) が表示されます。
10. 次のコマンドをお試しください。

    ```
    GetIntent "What's today's date?"
    ```

11. 応答を調べて、**GetDate** 意図を予測していることを確認します。
12. 次のコマンドをお試しください。

    ```
    GetIntent "What time is it in Sydney?"
    ```

13. 応答を調べて、**Location** エンティティが含まれていることを確認します。

14. 次のコマンドを試して、応答を調べてください。

    ```
    GetIntent "What time is it in Glasgow?"
    ```

    ```
    GetIntent "What's the time in Nairobi?"
    ```

    ```
    GetIntent "What's the UK time?"
    ```
15. さらにいくつかのバリエーションを試してください。目標は、**GetTime** 意図を正しく予測するが、**Location** エンティティの検出に失敗する少なくともいくつかの応答を生成することです。

    Azure サブスクリプションの **Language Understanding** リソースまたは **Cognitive Services** リソースのどちらでも構いません。

## <a name="apply-active-learning"></a>*アクティブ ラーニング*を適用する

You can improve a Language Understanding app based on historical utterances submitted to the endpoint. This practice is called <bpt id="p1">*</bpt>active learning<ept id="p1">*</ept>.

**重要**: オーサリング リソースは、3 つの*リージョン* (ヨーロッパ、オーストラリア、または米国) のいずれかで作成する必要があります。

1. ヨーロッパまたはオーストラリアのオーサリング リソースで作成された Language Understanding アプリは、それぞれヨーロッパまたはオーストラリアの予測リソースにのみデプロイできます。米国のオーサリング リソースで作成されたモデルは、ヨーロッパとオーストラリア以外の Azure の場所にある予測リソースにデプロイできます。
2. 意図と新しい Location エンティティ (元のトレーニング発話に含まれていなかった) が正しく予測された発話については、 **&#10003;** を選択してエンティティを確認し、 **&#10514;** アイコンを使用して、トレーニングの例として、発話を意図に追加します。
3. 一致するオーサリングと予測の場所の詳細については、[オーサリングとパブリッシング リージョンのドキュメント](https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-regions)を参照してください。
4. **[意図]** ページに移動し、**GetTime** 意図を開いて、提案された発話が追加されたことを確認します。
5. Language Understanding ポータルの上部にある **[トレーニング]** を選択して、アプリを再トレーニングします。
6. Language Understanding ポータルの右上にある **[公開]** を選択し、アプリを**本番スロット**に再公開します。
7. **09-luis-app** フォルダーのターミナルに戻り、**GetIntent** コマンドを使用して、アクティブ ラーニング中に追加および修正した発話を送信します。
8. Verify that the result now includes the <bpt id="p1">**</bpt>Location<ept id="p1">**</ept> entity. Then try another utterance that uses the same phrasing but specifies a different location (for example, <bpt id="p1">*</bpt>Berlin<ept id="p1">*</ept>).

## <a name="export-the-app"></a>アプリをエクスポートする

You can use the Language Understanding portal to develop and test your language app, but in a software development process for DevOps, you should maintain a source controlled definition of the app that can be included in continuous integration and delivery (CI/CD) pipelines. While you <bpt id="p1">*</bpt>can<ept id="p1">*</ept> use the Language Understanding SDK or REST API in code scripts to create and train the app, a simpler way is to use the portal to create the app, and export it as a <bpt id="p2">*</bpt>.lu<ept id="p2">*</ept> file that can be imported and retrained in another Language Understanding instance. This approach enables you to make use of the productivity benefits of the portal while maintaining portability and reproducibility for the app.

1. Language Understanding ポータルで、 **[管理]** を選択します。
2. **[バージョン]** ページで、アプリの現在のバージョンを選択します (1 つだけである必要があります)。
3. In the <bpt id="p1">**</bpt>Export<ept id="p1">**</ept> drop-down list, select <bpt id="p2">**</bpt>Export as LU<ept id="p2">**</ept>. Then, when prompted by your browser, save the file in the <bpt id="p1">**</bpt>09-luis-app<ept id="p1">**</ept> folder.
4. In Visual Studio Code, open the <bpt id="p1">**</bpt>.lu<ept id="p1">**</ept> file you just exported and downloaded (if you are prompted to search the marketplace for an extension that can read it, dismiss the prompt). Note that the LU format is human-readable, making it an effective way to document the definition of your Language Understanding app in a team development environment.

## <a name="more-information"></a>詳細情報

**Language Understanding** サービスの使用の詳細については、[Language Understanding のドキュメント](https://docs.microsoft.com/azure/cognitive-services/luis/)を参照してください。
