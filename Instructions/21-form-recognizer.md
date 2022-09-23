---
lab:
  title: Forms からのデータの抽出
  module: Module 11 - Reading Text in Images and Documents
---

# <a name="extract-data-from-forms"></a>Forms からのデータの抽出 

Suppose a company currently requires employees to manually purchase order sheets and enter the data into a database. They would like you to utilize AI services to improve the data entry process. You decide to build a machine learning model that will read the form and produce structured data that can be used to automatically update a database.

<bpt id="p1">**</bpt>Form Recognizer<ept id="p1">**</ept> is a cognitive service that enables users to build automated data processing software. This software can extract text, key/value pairs, and tables from form documents using optical character recognition (OCR). Form Recognizer has pre-built models for recognizing invoices, receipts, and business cards. The service also provides the capability to train custom models. In this exercise, we will focus on building custom models.

## <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

まだ行っていない場合は、このコースのコード リポジトリを複製する必要があります。

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git:Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるダイアログが表示された場合は、 **[今はしない]** を選択します。

## <a name="create-a-form-recognizer-resource"></a>Form Recognizer リソースを作成する

To use the Form Recognizer service, you need a Form Recognizer or Cognitive Services resource in your Azure subscription. You'll use the Azure portal to create a resource.

1.  Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。

2. **[&#65291;リソースの作成]** ボタンを選択し、*Form Recognizer* を検索して、次の設定で **Form Recognizer** リソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)*
    - **[リージョン]**: 使用できるリージョンを選択します**
    - **[名前]**: *一意の名前を入力します*
    - **[価格レベル]**: F0

    > **注**: サブスクリプションに既に F0 Form Recognizer サービスがある場合は、このサービスに **S0** を選択してください。

3. ある会社が現在、手動で注文シートを購入し、データベースにデータを入力することを社員に求めているとします。 

## <a name="gather-documents-for-training"></a>トレーニング用のドキュメントを収集する

![請求書の画像。](../21-custom-form/sample-forms/Form_1.jpg)  

このリポジトリの **21-custom-form/sample-forms** フォルダーのサンプル フォームを使用します。このサンプル フォームには、モデルをトレーニングし、テストするために必要なすべてのファイルが含まれています。

1. 会社は AI サービスを活用してデータ入力プロセスを改善したいと考えています。

    **.jpg** ファイルを使用し、モデルをトレーニングします。  

    あなたは、フォームを読み取り、(データベースを自動的に更新するために使用できる) 構造化データを生成する機械学習モデルを構築することに決めます。 

2. Azure portal ([https://portal.azure.com](https://portal.azure.com)) に戻ります。

3. 以前に Form Recognizer リソースを作成した**リソース グループ**を表示します。

4. On the <bpt id="p1">**</bpt>Overview<ept id="p1">**</ept> page for your resource group, note the <bpt id="p2">**</bpt>Subscription ID<ept id="p2">**</ept> and <bpt id="p3">**</bpt>Location<ept id="p3">**</ept>. You will need these values, along with your <bpt id="p1">**</bpt>resource group<ept id="p1">**</ept> name in subsequent steps.

![リソース グループ ページの例。](./images/resource_group_variables.png)

5. Visual Studio Code のエクスプローラー ペインで、**21-custom-form** フォルダーを右クリックし、 **[Open in Integrated Terminal]\(統合ターミナルで開く\)** を選択します。

6. ターミナル ペインで、次のコマンドを入力して、お使いの Azure サブスクリプションへの認証された接続を確立します。
    
```
az login --output none
```

7. **Form Recognizer** は、ユーザーが自動データ処理ソフトウェアを構築できるようにする Cognitive Services です。

8. 次のコマンドを実行して、Azure の場所を一覧表示します。

```
az account list-locations -o table
```

9. 出力で、リソース グループの場所に対応する **Name** の値を見つけます (たとえば、*米国東部* の場合、対応する名前は *eastus* です)。

    > **重要**: **Name** の値を記録しておき、手順 12 で使用します。

10. このソフトウェアは、光学式文字認識 (OCR) を使用して、フォーム ドキュメントからテキスト、キーと値のペア、およびテーブルを抽出できます。

11. Form Recognizer には、請求書、領収書、名刺を認識するためのモデルがあらかじめ構築されています。 
    - リソース グループにストレージ アカウントを作成する
    - ローカルの _sampleforms_ フォルダーからストレージ アカウントの _sampleforms_ というコンテナーにファイルをアップロードする
    - 共有アクセス署名 URI を印刷する

12. このサービスは、カスタム モデルをトレーニングする機能も提供します。

    この演習では、カスタム モデルの構築に焦点を当てます。  

13. **21-custom-form** フォルダーのターミナルで、次のコマンドを入力してスクリプトを実行します。

```
setup
```

14. スクリプトが完了したら、表示された出力を確認し、Azure リソースの SAS URI をメモします。

> **重要**: 進む前に、後で再度取得できる場所に SAS URI を貼り付けてください (たとえば、Visual Studio Code の新しいテキスト ファイルに)。

15. In the Azure portal, refresh the resource group and verify that it contains the Azure Storage account just created. Open the storage account and in the pane on the left, select <bpt id="p1">**</bpt>Storage Browser (preview)<ept id="p1">**</ept>. Then in Storage Browser, expand <bpt id="p1">**</bpt>BLOB CONTAINERS<ept id="p1">**</ept> and select the <bpt id="p2">**</bpt>sampleforms<ept id="p2">**</ept> container to verify that the files have been uploaded from your local <bpt id="p3">**</bpt>21-custom-form/sample-forms<ept id="p3">**</ept> folder.

## <a name="train-a-model-using-the-form-recognizer-sdk"></a>Form Recognizer SDK を使用してモデルをトレーニングする

次に、 **.jpg** ファイルと **.json** ファイルを使用してモデルをトレーニングします。

1. In Visual Studio Code, in the <bpt id="p1">**</bpt>21-custom-form/sample-forms<ept id="p1">**</ept> folder, open <bpt id="p2">**</bpt>fields.json<ept id="p2">**</ept> and review the JSON document it contains. This file defines the fields that you will train a model to extract from the forms.
2. Open <bpt id="p1">**</bpt>Form_1.jpg.labels.json<ept id="p1">**</ept> and review the JSON it contains. This file identifies the location and values for named fields in the <bpt id="p1">**</bpt>Form_1.jpg<ept id="p1">**</ept> training document.
3. Open <bpt id="p1">**</bpt>Form_1.jpg.ocr.json<ept id="p1">**</ept> and review the JSON it contains. This file contains a JSOn representation of the text layout of <bpt id="p1">**</bpt>Form_1.jpg<ept id="p1">**</ept>, including the location of all text areas found in the form.

    *この演習では、フィールド情報ファイルが提供されています。独自のプロジェクトの場合、[Form Recognizer Studio](https://formrecognizer.appliedai.azure.com/studio) を利用し、これらのファイルを作成できます。ツールを使用すると、フィールド情報ファイルが自動的に作成され、接続しているストレージ アカウントに保存されます。*

4. Visual Studio Code の **21-custom-form** フォルダーで、言語の設定に応じて **C-Sharp** フォルダーまたは **Python** フォルダーを展開します。
5. **train-model** フォルダーを右クリックして、統合ターミナルを開きます。

6. お使いの言語設定のための適切なコマンドを実行して、フォームの認識器のパッケージをインストールします

**C#**

```
dotnet add package Azure.AI.FormRecognizer --version 3.0.0 
```

**Python**

```
pip install azure-ai-formrecognizer==3.0.0
```

7. **train-model** フォルダーの内容を表示し、構成設定用のファイルが含まれていることに注意してください
    - **C#** : appsettings.json
    - **Python**: .env

8. 構成ファイルを編集し、以下を反映するように設定を変更します
    - Form Recognizer リソースの**エンドポイント**。
    - Form Recognizer リソースの**キー**。
    - BLOB コンテナーの **SAS URI**。

9. **train-model** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることに注意してください

    - **C#** : Program.cs
    - **Python**: train-model.py

    コード ファイルを開き、含まれているコードを確認して、次の詳細に注意してください。
    - インストールしたパッケージの名前空間インポートされます
    - **Main** 関数は構成設定を取得し、キーとエンドポイントを使用して認証済みの **Client** を作成します。
    - このコードは、トレーニングクライアントを使用して、生成した SAS URI を使用してアクセスされる BLOB ストレージ コンテナー内の画像を使用してモデルをトレーニングします。

10. **train-model** フォルダーで、トレーニング アプリケーションのコード ファイルを開きます。

    - **C#** : Program.cs
    - **Python**: train-model.py

11. **train-model** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

**C#**

```
dotnet run
```

**Python**

```
python train-model.py
```

12. 最後に、プログラムの待ち、その後、モデル出力を確認します。
13. Write down the Model ID in the terminal output. You will need it for the next part of the lab. 

## <a name="test-your-custom-form-recognizer-model"></a>カスタム Form Recognizer モデルをテストする 

1. **21-custom-form** フォルダーで、使用する言語のサブフォルダー (**C-Sharp** または **Python**) の **test-model** フォルダーを展開します。

2. **test-model** フォルダーを右クリックし、**統合ターミナル**を選択します。

3. **test-model** フォルダーのターミナルで、言語設定に適したコマンドを実行して、Form Recognizer パッケージをインストールします。

**C#**

```
dotnet add package Azure.AI.FormRecognizer --version 3.0.0 
```

**Python**

```
pip install azure-ai-formrecognizer==3.0.0
```

*以前に pip を使用してパッケージを Python 環境にインストールしたことがある場合、これは厳密には必要ありません。しかし、それがインストールされていることを確認しても害はありません。*

4. In the same terminal for the <bpt id="p1">**</bpt>test-model<ept id="p1">**</ept> folder, install the Tabulate library. This will provide your output in a table:

**C#**

```
dotnet add package Tabulate.NET --version 1.0.5
```

**Python**

```
pip install tabulate
```

5. **test-model** フォルダーで、構成ファイル (言語設定に応じて **appsettings.json** または **.env**) を編集して、次の値を追加します。
    - Form Recognizer エンドポイント。
    - Form Recognizer キー。
    - The Model ID generated when you trained the model (you can find this by switching the terminal back to the <bpt id="p1">**</bpt>cmd<ept id="p1">**</ept> console for the <bpt id="p2">**</bpt>train-model<ept id="p2">**</ept> folder). <bpt id="p1">**</bpt>Save<ept id="p1">**</ept> your changes.

6. **test-model** フォルダーで、クライアント アプリケーションのコード ファイル (C# の場合は *Program.cs*、Python の場合は *test-model.py*) を開き、含まれているコードを確認して、次の詳細に注意してください。
    - インストールしたパッケージの名前空間インポートされます
    - **Main** 関数は構成設定を取得し、キーとエンドポイントを使用して認証済みの **Client** を作成します。
    - 次に、クライアントを使用して、**test1.jpg** 画像からフォーム フィールドと値を抽出します。
    

7. **test-model** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

**C#**

```
dotnet run
```

**Python**

```
python test-model.py
```
    
8. 出力を表示し、"CompanyPhoneNumber" や "DatedAs" などのフィールド名がモデルの出力に含まれる様子を観察します。   

## <a name="more-information"></a>詳細情報

Form Recognizer サービスの詳細については、[Form Recognizer のドキュメント](https://docs.microsoft.com/azure/cognitive-services/form-recognizer/)を参照してください。
