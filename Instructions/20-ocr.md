---
lab:
  title: 画像内のテキストの読み取り
  module: Module 11 - Reading Text in Images and Documents
---

# <a name="read-text-in-images"></a>画像内のテキストの読み取り

Optical character recognition (OCR) is a subset of computer vision that deals with reading text in images and documents. The <bpt id="p1">**</bpt>Computer Vision<ept id="p1">**</ept> service provides two APIs for reading text, which you'll explore in this exercise.

## <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

まだ行っていない場合は、このコースのコード リポジトリを複製する必要があります。

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

## <a name="prepare-to-use-the-computer-vision-sdk"></a>Computer Vision SDKを使用する準備をする

この演習では、Computer VisionSDK を使用してテキストを読み取る部分的に実装されたクライアント アプリケーションを完成させます。

> <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: You can choose to use the SDK for either <bpt id="p2">**</bpt>C#<ept id="p2">**</ept> or <bpt id="p3">**</bpt>Python<ept id="p3">**</ept>. In the steps below, perform the actions appropriate for your preferred language.

1. Visual Studio Code の**エクスプローラー** ペインで、**20-ocr** フォルダーを参照し、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. 光学式文字認識 (OCR) は、画像およびドキュメント内のテキストの読み取りを処理する Computer Vision のサブセットです。

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-computervision==0.7.0
```

3. **read-text** フォルダーの内容を表示し、構成設定用のファイルが含まれていることにご注意ください。
    - **C#** : appsettings.json
    - **Python**: .env

    **Computer Vision** サービスにより、テキストを読み取るための 2 つの API が提供されます。これについては、この演習で説明します。
4. **read-text** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることにご注意ください。

    - **C#** : Program.cs
    - **Python**: read-text.py

    Open the code file and at the top, under the existing namespace references, find the comment <bpt id="p1">**</bpt>Import namespaces<ept id="p1">**</ept>. Then, under this comment, add the following language-specific code to import the namespaces you will need to use the Computer Vision SDK:

**C#**

```C#
// import namespaces
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;
```

**Python**

```Python
# import namespaces
from azure.cognitiveservices.vision.computervision import ComputerVisionClient
from azure.cognitiveservices.vision.computervision.models import OperationStatusCodes
from msrest.authentication import CognitiveServicesCredentials
```

5. In the code file for your client application, in the <bpt id="p1">**</bpt>Main<ept id="p1">**</ept> function, note that the code to load the configuration settings has been provided. Then find the comment <bpt id="p1">**</bpt>Authenticate Computer Vision client<ept id="p1">**</ept>. Then, under this comment, add the following language-specific code to create and authenticate a Computer Vision client object:

**C#**

```C#
// Authenticate Computer Vision client
ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
cvClient = new ComputerVisionClient(credentials)
{
    Endpoint = cogSvcEndpoint
};
```

**Python**

```Python
# Authenticate Computer Vision client
credential = CognitiveServicesCredentials(cog_key) 
cv_client = ComputerVisionClient(cog_endpoint, credential)
```
    
## <a name="use-the-ocr-api"></a>OCR API を使用する

The <bpt id="p1">**</bpt>OCR<ept id="p1">**</ept> API is an optical character recognition API that is optimized for reading small to medium amounts of printed text in <bpt id="p2">*</bpt>.jpg<ept id="p2">*</ept>, <bpt id="p3">*</bpt>.png<ept id="p3">*</ept>, <bpt id="p4">*</bpt>.gif<ept id="p4">*</ept>, and <bpt id="p5">*</bpt>.bmp<ept id="p5">*</ept> format images. It supports a wide range of languages and in addition to reading text in the image it can determine the orientation of each text region and return information about the rotation angle of the text in relation to the image

1. In the code file for your application, in the <bpt id="p1">**</bpt>Main<ept id="p1">**</ept> function, examine the code that runs if the user selects menu option <bpt id="p2">**</bpt>1<ept id="p2">**</ept>. This code calls the <bpt id="p1">**</bpt>GetTextOcr<ept id="p1">**</ept> function, passing the path to an image file.
2. **read-text/images** フォルダーで、**Lincoln.jpg** を開いて、コードによって処理される画像を表示します。
3. コード ファイルに戻り、**GetTextOcr** 関数を見つけ、コンソールにメッセージを出力する既存のコードの下に、次のコードを追加します。

**C#**

```C#
// Use OCR API to read text in image
using (var imageData = File.OpenRead(imageFile))
{    
    var ocrResults = await cvClient.RecognizePrintedTextInStreamAsync(detectOrientation:false, image:imageData);

    // Prepare image for drawing
    Image image = Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Magenta, 3);

    foreach(var region in ocrResults.Regions)
    {
        foreach(var line in region.Lines)
        {
            // Show the position of the line of text
            int[] dims = line.BoundingBox.Split(",").Select(int.Parse).ToArray();
            Rectangle rect = new Rectangle(dims[0], dims[1], dims[2], dims[3]);
            graphics.DrawRectangle(pen, rect);

            // Read the words in the line of text
            string lineText = "";
            foreach(var word in line.Words)
            {
                lineText += word.Text + " ";
            }
            Console.WriteLine(lineText.Trim());
        }
    }

    // Save the image with the text locations highlighted
    String output_file = "ocr_results.jpg";
    image.Save(output_file);
    Console.WriteLine("Results saved in " + output_file);
}
```

**Python**

```Python
# Use OCR API to read text in image
with open(image_file, mode="rb") as image_data:
    ocr_results = cv_client.recognize_printed_text_in_stream(image_data)

# Prepare image for drawing
fig = plt.figure(figsize=(7, 7))
img = Image.open(image_file)
draw = ImageDraw.Draw(img)

# Process the text line by line
for region in ocr_results.regions:
    for line in region.lines:

        # Show the position of the line of text
        l,t,w,h = list(map(int, line.bounding_box.split(',')))
        draw.rectangle(((l,t), (l+w, t+h)), outline='magenta', width=5)

        # Read the words in the line of text
        line_text = ''
        for word in line.words:
            line_text += word.text + ' '
        print(line_text.rstrip())

# Save the image with the text locations highlighted
plt.axis('off')
plt.imshow(img)
outputfile = 'ocr_results.jpg'
fig.savefig(outputfile)
print('Results saved in', outputfile)
```

4. Examine the code you added to the <bpt id="p1">**</bpt>GetTextOcr<ept id="p1">**</ept> function. It detects regions of printed text from an image file, and for each region it extracts the lines of text and highlights there position on the image. It then extracts the words in each line and prints them.
5. 変更を保存し、**read-text** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

**C#**

```
dotnet run
```

*C# 出力に、**await** 演算子を使用している非同期関数に関する警告が表示される場合があります。これは無視してもかまいません。*

**Python**

```
python read-text.py
```

6. プロンプトが表示されたら、**1** を入力して、画像から抽出されたテキストである出力を確認します。
7. コード ファイルと同じフォルダーに生成された **ocr_results.jpg** ファイルを表示して、画像内の注釈付きのテキスト行を確認します。

## <a name="use-the-read-api"></a>Read API を使用する

The <bpt id="p1">**</bpt>Read<ept id="p1">**</ept> API uses a newer text recognition model than the OCR API, and performs better for larger images that contain a lot of text. It also supports text extraction from <bpt id="p1">*</bpt>.pdf<ept id="p1">*</ept> files, and can recognize both printed text and handwritten text in multiple languages.

**Read** API は、テキスト認識を開始する要求が送信される非同期操作モデルを使用します。その後、リクエストから返された操作 ID を使用して、進行状況を確認し、結果を取得できます。

1. In the code file for your application, in the <bpt id="p1">**</bpt>Main<ept id="p1">**</ept> function, examine the code that runs if the user selects menu option <bpt id="p2">**</bpt>2<ept id="p2">**</ept>. This code calls the <bpt id="p1">**</bpt>GetTextRead<ept id="p1">**</ept> function, passing the path to a PDF document file.
2. In the <bpt id="p1">**</bpt>read-text/images<ept id="p1">**</ept> folder, right-click <bpt id="p2">**</bpt>Rome.pdf<ept id="p2">**</ept> and select <bpt id="p3">**</bpt>Reveal in File Explorer<ept id="p3">**</ept>. Then in File Explorer, open the PDF file to view it.
3. Visual Studio Code のコードファイルに戻り、**GetTextRead** 関数を見つけ、コンソールにメッセージを出力する既存のコードの下に、次のコードを追加します。

**C#**

```C#
// Use Read API to read text in image
using (var imageData = File.OpenRead(imageFile))
{    
    var readOp = await cvClient.ReadInStreamAsync(imageData);

    // Get the async operation ID so we can check for the results
    string operationLocation = readOp.OperationLocation;
    string operationId = operationLocation.Substring(operationLocation.Length - 36);

    // Wait for the asynchronous operation to complete
    ReadOperationResult results;
    do
    {
        Thread.Sleep(1000);
        results = await cvClient.GetReadResultAsync(Guid.Parse(operationId));
    }
    while ((results.Status == OperationStatusCodes.Running ||
            results.Status == OperationStatusCodes.NotStarted));

    // If the operation was successfuly, process the text line by line
    if (results.Status == OperationStatusCodes.Succeeded)
    {
        var textUrlFileResults = results.AnalyzeResult.ReadResults;
        foreach (ReadResult page in textUrlFileResults)
        {
            foreach (Line line in page.Lines)
            {
                Console.WriteLine(line.Text);
            }
        }
    }
}  
```

**Python**

```Python
# Use Read API to read text in image
with open(image_file, mode="rb") as image_data:
    read_op = cv_client.read_in_stream(image_data, raw=True)

    # Get the async operation ID so we can check for the results
    operation_location = read_op.headers["Operation-Location"]
    operation_id = operation_location.split("/")[-1]

    # Wait for the asynchronous operation to complete
    while True:
        read_results = cv_client.get_read_result(operation_id)
        if read_results.status not in [OperationStatusCodes.running, OperationStatusCodes.not_started]:
            break
        time.sleep(1)

    # If the operation was successfuly, process the text line by line
    if read_results.status == OperationStatusCodes.succeeded:
        for page in read_results.analyze_result.read_results:
            for line in page.lines:
                print(line.text)
```
    
4. Examine the code you added to the <bpt id="p1">**</bpt>GetTextRead<ept id="p1">**</ept> function. It submits a request for a read operation, and then repeatedly checks status until the operation has completed. If it was successful, the code processes the results by iterating through each page, and then through each line.
5. 変更を保存し、**read-text** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

**C#**

```
dotnet run
```

**Python**

```
python read-text.py
```

6. ダイアログが表示されたら、**2** を入力し、ドキュメントから抽出されたテキストである出力を確認します。

## <a name="read-handwritten-text"></a>手書きテキストの読み取り

**Read** API は、印刷されたテキスト以外にも、英語の手書きのテキストを抽出できます。

1. In the code file for your application, in the <bpt id="p1">**</bpt>Main<ept id="p1">**</ept> function, examine the code that runs if the user selects menu option <bpt id="p2">**</bpt>3<ept id="p2">**</ept>. This code calls the <bpt id="p1">**</bpt>GetTextRead<ept id="p1">**</ept> function, passing the path to an image file.
2. **read-text/images** フォルダーで、**Note.jpg** を開いて、コードによって処理される画像を表示します。
3. **read-text** フォルダーの統合ターミナルで、次のコマンドを入力してプログラムを実行します。

**C#**

```
dotnet run
```

**Python**

```
python read-text.py
```

4. ダイアログが表示されたら、**3** を入力し、ドキュメントから抽出されたテキストである出力を確認します。

## <a name="more-information"></a>詳細情報

**Computer Vision** サービスを使用してテキストを読み取る方法の詳細については、[Computer Vision のドキュメント](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-recognizing-text)を参照してください。
