---
lab:
  title: 質問応答ソリューションを作成する
  module: Module 6 - Building a QnA Solution
---

# <a name="create-a-question-answering-solution"></a>質問応答ソリューションを作成する

One of the most common conversational scenarios is providing support through a knowledge base of frequently asked questions (FAQs). Many organizations publish FAQs as documents or web pages, which works well for a small set of question and answer pairs, but large documents can be difficult and time-consuming to search.

**Language** サービスには、*質問応答* 機能が含まれており、これを使用すると、自然言語で入力してクエリを実行できる、質問と回答のペアで構成されるナレッジ ベースを作成でき、ボットがユーザーから送信された質問への回答を検索するために使用できるリソースとして最も一般的に使用されます。

> **注**:言語サービスの質問応答機能は新しいバージョンの QnA Maker サービスであり、個別のサービスとして引き続き利用できます。

## <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

If you have not already cloned <bpt id="p1">**</bpt>AI-102-AIEngineer<ept id="p1">**</ept> code repository to the environment where you're working on this lab, follow these steps to do so. Otherwise, open the cloned folder in Visual Studio Code.

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git:Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるダイアログが表示された場合は、 **[今はしない]** を選択します。

## <a name="create-a-language-resource"></a>"言語" リソースを作成する

質問に答えるナレッジ ベースを作成してホストするには、Azure サブスクリプションに **Language** サービス リソースが必要です。

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. **[&#65291;リソースの作成]** ボタンを選択し、*Language* を検索して、**Language サービス** リソースを作成します。
3. Click <bpt id="p1">**</bpt>Select<ept id="p1">**</ept> on the <bpt id="p2">**</bpt>Custom question answering<ept id="p2">**</ept> block. Then click <bpt id="p1">**</bpt>Continue to create your resource<ept id="p1">**</ept>. You will need to enter the following settings:
    
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)*
    - **[リージョン]** : "*利用可能な場所を選択する*"
    - **[名前]**: *一意の名前を入力します*
    - **[価格レベル]**: Standard 5
    - **Azure 検索の場所**\*:"*言語リソースと同じグローバル リージョン内の場所を選択します*"。
    - **Azure Search の価格レベル**：無料 (F) ("*このレベルが利用できない場合は、[基本 (B)] を選択してください*")
    - **法的条項**: "同意"__ 
    - **責任ある AI 通知**: "同意"__
    
    \*カスタム質問応答は、Azure Search を使用して、質問と回答のナレッジ ベースにインデックスを付けてクエリを実行します。

4. デプロイが完了するまで待ち、デプロイの詳細を表示します。

## <a name="create-a-question-answering-project"></a>質問応答プロジェクトを作成する

最も一般的な会話シナリオの 1 つは、よくある質問 (FAQ) のナレッジ ベースを通じてサポートを提供することです。

1. 新しいブラウザー タブで *Language Studio* ポータル (`https://language.azure.com`) に移動し、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使ってサインインします。
2. 言語リソースの選択を求めるメッセージが表示されたら、次の設定を選択します。
    - **Azure ディレクトリ**: ご利用のサブスクリプションを含む Azure ディレクトリ
    - **Azure サブスクリプション**: ご利用の Azure サブスクリプション
    - **言語リソース**: 先ほど作成した言語リソース。
3. 言語リソースの選択を求めるメッセージが表示<u>されない</u>場合、原因として、お使いのサブスクリプションに複数の言語リソースが存在していることが考えられます。その場合は、次の操作を行います。
    1. ページの上部にあるバーで、**[設定] (&#9881;)** ボタンをクリックします。
    2. **[設定]** ページで、**[リソース]** タブを表示します。
    3. 作成したばかりの言語リソースを選択し、**[リソースの切り替え]** をクリックします。
    4. ページの上部で、**[Language Studio]** をクリックして、Language Studio のホーム ページに戻ります。
3. ポータルの上部にある **[新規作成]** メニューで、 **[カスタム質問応答]** を選択します。
4. 多くの組織は、FAQ をドキュメントまたは Web ページとして公開しています。これは、質問と回答のペアの小さなセットに適していますが、大きなドキュメントは検索が難しく、時間がかかる場合があります。
5. **[基本情報の入力]** ページで、次の詳細を入力したら、 **[次へ]** をクリックします。
    - **名前** LearnFAQ
    - **[説明]** : Microsoft Learn のよくあるご質問
    - **回答が返されない場合の既定の回答**: 申し訳ございません。質問がよくわかりませんでした
6. **[確認と終了]** ページで、**[プロジェクトの作成]** をクリックします。

## <a name="add-a-sources-to-the-knowledge-base"></a>ナレッジ ベースにソースを追加する

You can create a knowledge base from scratch, but it's common to start by importing questions and answers from an existing FAQ page or document. In this case, you'll import data from an existing FAQ web page for Microsoft learn, and you'll also import some pre-defined "chit chat" questions and answers to support common conversational exchanges.

1. On the <bpt id="p1">**</bpt>Manage sources<ept id="p1">**</ept> page for your question answering project, in the <bpt id="p2">**</bpt>&amp;#9547; Add source<ept id="p2">**</ept> list, select <bpt id="p3">**</bpt>URLs<ept id="p3">**</ept>. Then in the <bpt id="p1">**</bpt>Add URLs<ept id="p1">**</ept> dialog box, click <bpt id="p2">**</bpt>&amp;#9547; Add url<ept id="p2">**</ept> and add the following URL, and then click <bpt id="p3">**</bpt>Add all<ept id="p3">**</ept> to add it to the knowledge base:
    - **名前**: `Learn FAQ Page`
    - **URL**: `https://docs.microsoft.com/en-us/learn/support/faq`
2. On the <bpt id="p1">**</bpt>Manage sources<ept id="p1">**</ept> page for your question answering project, in the <bpt id="p2">**</bpt>&amp;#9547; Add source<ept id="p2">**</ept> list, select <bpt id="p3">**</bpt>Chitchat<ept id="p3">**</ept>. The in the <bpt id="p1">**</bpt>Add chit chat<ept id="p1">**</ept> dialog box, select <bpt id="p2">**</bpt>Friendly<ept id="p2">**</ept> and click <bpt id="p3">**</bpt>Add chit chat<ept id="p3">**</ept>.

## <a name="edit-the-knowledge-base"></a>ナレッジ ベースを編集する

Your knowledge base has been populated with question and answer pairs from the Microsoft Learn FAQ, supplemented with a set of conversational <bpt id="p1">*</bpt>chit-chat<ept id="p1">*</ept> question  and answer pairs. You can extend the knowledge base by adding additional question and answer pairs.

1. Language Studio の **[LearnFAQ]** プロジェクトで、 **[ナレッジ ベースの編集]** ページを選択して既存の質問と回答のペアを表示します (複数のヒントが表示されている場合は、それを読み、 **[了解]** をクリックして閉じるか、 **[すべてスキップ]** をクリックします)
2. ナレッジ ベースで、 **[&#65291; 質問のペアの追加]** を選択します。
3. **[質問]** ボックスに「`What is Microsoft certification?`」と入力し、**Enter**** キーを押します。
4. **[&#65291; 代替語句の追加]** を選択し、「`How can I demonstrate my Microsoft technology skills?`」と入力して **Enter** キーを押します。
5. **[回答]** ボックスに「`The Microsoft Certified Professional program enables you to validate and prove your skills with Microsoft technologies.`」と入力します。次に、 **[送信]** を押して質問 (代替フレージングを含む) と回答をナレッジ ベースに追加します。

    場合によっては、ユーザーが回答をフォローアップして、*複数ターン* 会話を作成できるようにすることが理にかなっています。ユーザーは、質問を繰り返し絞り込んで、必要な回答者にたどり着きます。

6. 認定の質問に入力した回答の下で、 **[&#65291; フォローアップ プロンプトの追加]** を選択します。
7. **[フォローアップ プロンプト]** ダイアログ ボックスで、次の設定を入力してから、 **[プロンプトの追加]** をクリックします。
    - **ユーザーへのプロンプトに表示されるテキスト**: `Learn more about certification`。
    - **[新しいペアへのリンクの作成]** を選択し、このテキストを入力します: `You can learn more about certification on the [Microsoft certification page](https://docs.microsoft.com/learn/certifications/).`
    - このラボで作業している環境に **AI-102-AIEngineer** コードのリポジトリをまだクローンしていない場合は、次の手順に従ってクローンします。

## <a name="train-and-test-the-knowledge-base"></a>ナレッジ ベースのトレーニングとテスト

ナレッジベースができたので、Language Studio でテストできます。

1. 次に、ページの右上にある **[変更の保存]** をクリックします。
2. 変更が保存されたら、 **[テスト]** をクリックしてテスト ペインを開きます。
3. それ以外の場合は、複製されたフォルダーを Visual Studio Code で開きます。
4. In the test pane, at the bottom enter the message <ph id="ph1">`What is Microsoft Learn?`</ph>. An appropriate response from the FAQ should be returned.
5. 「`Thanks!`」というメッセージを入力します。適切なおしゃべりの応答が返されます。
6. Enter the message <ph id="ph1">`Tell me about certification`</ph>. The answer you created should be returned along with a follow-up prompt link.
7. Select the <bpt id="p1">**</bpt>Learn more about certification<ept id="p1">**</ept> follow-up link. The follow-up answer with a link to the certification page should be returned.
8. ナレッジ ベースのテストが完了したら、テスト ペインを閉じます。

## <a name="deploy-and-test-the-knowledge-base"></a>ナレッジ ベースのデプロイとテスト

The knowledge base provides a back-end service that client applications can use to answer questions. Now you are ready to publish your knowledge base and access its REST interface from a client.

1. Language Studio で、**LearnFAQ** プロジェクトの **[ナレッジ ベースのデプロイ]** ページを選択します。
2. At the top of the page, click <bpt id="p1">**</bpt>Deploy<ept id="p1">**</ept>. Then click <bpt id="p1">**</bpt>Deploy<ept id="p1">**</ept> to confirm you want to deploy the knowledge base.
3. デプロイが完了したら、 **[予測 URL の取得]** をクリックしてナレッジ ベースの REST エンドポイントを表示し、それをクリップボードにコピーします (ただし、ダイアログ ボックスはまだ閉じないでください)。
4. In Visual Studio Code, in the <bpt id="p1">**</bpt>12-qna<ept id="p1">**</ept> folder, open <bpt id="p2">**</bpt>ask-question.cmd<ept id="p2">**</ept>. This script uses <bpt id="p1">*</bpt>Curl<ept id="p1">*</ept> to call the REST interface of a question answering endpoint.
5. スクリプトで、*YOUR_PREDICTION_ENDPOINT* をコピーした予測エンドポイントに置き換えます (必ず引用符で囲んでください)。
6. Return to the browser and in the <bpt id="p1">**</bpt>Get prediction URL<ept id="p1">**</ept> dialog box, note that the sample request includes a value for the <bpt id="p2">**</bpt>Ocp-Apim-Subscription-Key<ept id="p2">**</ept> parameter, which looks similar to <bpt id="p3">*</bpt>ab12c345de678fg9hijk01lmno2pqrs34<ept id="p3">*</ept>. This is the authorization key for your resource. Copy it to the clipboard, and then click <bpt id="p1">**</bpt>Got it<ept id="p1">**</ept> to close the dialog box.
7. Visual Studio Code に戻り、**ask-question.cmd** スクリプトの *YOUR_KEY* をコピーしたキーに置き換えます (必ず引用符で囲んでください)。
8. スクリプトの Curl コマンドによって、**What is a Learning Path?** (ラーニング パスとは何ですか) という値を持つ **question** パラメーターが送信されることに注意してください。
9. スクリプト全体が次のコードのようになっていることを確認し、ファイルを保存します。

    ```
    @echo off
    SETLOCAL ENABLEDELAYEDEXPANSION

    rem Set variables
    set prediction_url="https://some-name.cognitiveservices.azure.com/language/......."
    set key="ab12c345de678fg9hijk01lmno2pqrs34"

    curl -X POST !prediction_url! -H "Ocp-Apim-Subscription-Key: !key!" -H "Content-Type: application/json" -d "{'question': 'What is a learning Path?' }"
    ```

10. Visual Studio Code のエクスプローラー ペインで、**ask-question.cmd** スクリプトを右クリックし、 **[Open in Integrated Terminal]\(統合ターミナルで開く\)** を選択します。
11. ターミナル ペインで、スクリプトを実行するコマンド `ask-question.cmd` を入力し、サービスによって返された JSON 応答を表示します。これには、*What is a learning path?* という質問に対する適切な回答が含まれているはずです。

## <a name="create-a-bot-for-the-knowledge-base"></a>ナレッジ ベース用のボットを作成する

最も一般的には、ナレッジ ベースから回答を取得するために使用されるクライアント アプリケーションはボットです。

1. Return to Language Studio in the browser, and in the <bpt id="p1">**</bpt>Deploy knowledge base<ept id="p1">**</ept> page, select <bpt id="p2">**</bpt>Create Bot<ept id="p2">**</ept>. This opens the Azure portal in a new browser tab so you can create a Web App Bot in your Azure subscription (if prompted, sign in).
2. Azure portal で、次の設定で Web アプリ ボットを作成します (ほとんどの設定は事前入力されています)。

    *一部の値が欠落している場合は、ブラウザーを更新します。*  

  - **[ボット ハンドル]**: *ボットの一意の名前*
  - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
  - **[リソース グループ]** : "*言語リソースを含むリソース グループ*"
  - **[場所]** : *Text Analytics サービスと同じ場所*。
  - **[価格レベル]**: F0
  - **[アプリ名]** : ***ボット ハンドル** と同じで、一意の ID と *.azurewebsites.net* が自動的に追加されます
  - **[SDK 言語]**: *C# または Node.js を選択します*
  - **[QnA 認証キー]**: *これは、QnA ナレッジ ベースの認証キーに自動的に設定されます*
  - **アプリ サービス プラン/場所**:"*これは、適切なプランと場所が存在する場合は、自動的に設定されることがあります。それ以外の場合は、新しいプランを作成します*"
  - **[Application Insights]**: オフ
  - **[Microsoft アプリ ID とパスワード]**: アプリ ID とパスワードを自動作成します。
3. Wait for your bot to be created . Then click <bpt id="p1">**</bpt>Go to resource<ept id="p1">**</ept> (or alternatively, on the home page, click <bpt id="p2">**</bpt>Resource groups<ept id="p2">**</ept>, open the resource group where you created the web app bot, and click it.)
4. In the blade for your bot, view the <bpt id="p1">**</bpt>Test in Web Chat<ept id="p1">**</ept> page, and wait until the bot displays the message <bpt id="p2">**</bpt>Hello and welcome!<ept id="p2">**</ept> (it may take a few seconds to initialize).
5. **[カスタム質問応答]** ブロックで **[選択]** をクリックします。

## <a name="more-information"></a>詳細情報

言語サービスの質問応答の詳細については、[言語サービスのドキュメント](https://docs.microsoft.com/en-us/azure/cognitive-services/language-service/question-answering/overview)に関するページを参照してください。
