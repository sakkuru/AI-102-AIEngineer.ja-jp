---
lab:
  title: 顔の検出と分析
  module: 'Module 10 - Detecting, Analyzing, and Recognizing Faces'
---

# <a name="detect-and-analyze-faces"></a>顔の検出と分析

人間の顔を検出して分析する能力は、AIの中核となる能力です。この演習では、画像内の顔を扱うために使用できる2つのAzure Cognitive サービス、Computer VisionサービスとFaceサービスを調べます。

> 注：2022年6月21日から、個人を特定できる情報を返すコグニティブサービスの機能は、限定的なアクセスを許可されたお客様に制限されます。また、感情状態を推測する機能は使用できなくなります。これらの制限は、このラボ演習に影響を与える可能性があります。Microsoft はこの問題に対処していますが、それまでの間、以下の手順を実行するとエラーが発生する可能性があります。マイクロソフトが行った変更とその理由の詳細については、顔認識のための責任あるAI投資とセーフガードをご覧ください。

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
5. リソースがデプロイされたら、リソースに移動して、そのキーとエンドポイントのページを表示します。次の手順では、このページのエンドポイントとキーの 1 つが必要になります。


## <a name="prepare-to-use-the-computer-vision-sdk"></a>Computer Vision SDKを使用する準備をする

この演習では、Computer Vision SDK を使用して画像内の顔を分析する部分的に実装されたクライアント アプリケーションを完成させます。

> 人間の顔を検出して分析する機能は、AI のコア機能です。

1. Visual Studio Code の**エクスプローラー** ペインで、**19-face** フォルダーを参照し、言語の設定に応じて **C#** または **Python** フォルダーを展開します。
2. **computer-vision** フォルダーを右クリックして、統合ターミナルを開きます。 次に、言語設定に適したコマンドを実行して、Computer Vision SDK パッケージをインストールします。

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-computervision==0.7.0
    ```
    
3. **computer-vision** フォルダーの内容を表示し、構成設定用のファイルが含まれていることを確認してください。
    - **C#** : appsettings.json
    - **Python**: .env

4. 構成ファイルを開き、含まれている構成値を更新して、Cognitive Services リソースの**エンドポイント**と認証**キー**を反映します。変更を保存します。

5. **computer-vision** フォルダーに、クライアント アプリケーションの次のコード ファイルが含まれていることを確認してください。

    - **C#** : Program.cs
    - **Python**: detect-faces.py

6. コード ファイルを開き、上部の既存の名前空間参照の下で、「**Import namespaces**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Computer Vision SDK を使用するために必要な名前空間をインポートします

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
    from azure.cognitiveservices.vision.computervision.models import VisualFeatureTypes
    from msrest.authentication import CognitiveServicesCredentials
    ```

## <a name="view-the-image-you-will-analyze"></a>分析する画像の確認

この演習では、Computer Vision サービスを使用して、人の画像を分析します。

1. Visual Studio Code で、**computer-vision** フォルダーとそれに含まれる **images** フォルダーを展開します。
2. **people.jpg** 画像を選択して表示します。

## <a name="detect-faces-in-an-image"></a>画像内の顔を検出する

これで、SDK を使用して Computer Vision サービスを呼び出し、画像内の顔を検出する準備が整いました。

1. クライアント アプリケーションのコード ファイル (**Program.cs** または **detect-faces.py**) の **Main** 関数で、構成設定をロードするためのコードが提供されていることに注意してください。次に、コメント **「Authenticate Computer Vision client」** を見つけます。次に、このコメントの下に、次の言語固有のコードを追加して、Computer Vision クライアント オブジェクトを作成および認証します

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

2. **Main** 関数で、追加したコードの下で、コードが画像ファイルへのパスを指定し、**AnalyzeFaces** という名前の関数に画像パスを渡すことに注意してください。この関数はまだ完全には実装されていません。

3. **AnalyzeFaces** 関数のコメント **「Specify features to be retrieved (faces)」** の下に、次のコードを追加します。

    **C#**

    ```C#
    // Specify features to be retrieved (faces)
    List<VisualFeatureTypes?> features = new List<VisualFeatureTypes?>()
    {
        VisualFeatureTypes.Faces
    };
    ```

    **Python**

    ```Python
    # Specify features to be retrieved (faces)
    features = [VisualFeatureTypes.faces]
    ```

4. **AnalyzeFaces** 関数のコメント **「Get image analysis」** の下に、次のコードを追加します。

**C#**

```C
// Get image analysis
using (var imageData = File.OpenRead(imageFile))
{    
    var analysis = await cvClient.AnalyzeImageInStreamAsync(imageData, features);

    // Get faces
    if (analysis.Faces.Count > 0)
    {
        Console.WriteLine($"{analysis.Faces.Count} faces detected.");

        // Prepare image for drawing
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 3);
        SolidBrush brush = new SolidBrush(Color.LightGreen);

        // Draw and annotate each face
        foreach (var face in analysis.Faces)
        {
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Person at approximately {r.Left}, {r.Top}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // Save annotated image
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}        
```

**Python**

```Python
# Get image analysis
with open(image_file, mode="rb") as image_data:
    analysis = cv_client.analyze_image_in_stream(image_data , features)

    # Get faces
    if analysis.faces:
        print(len(analysis.faces), 'faces detected.')

        # Prepare image for drawing
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'

        # Draw and annotate each face
        for face in analysis.faces:
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Person at approximately {}, {}'.format(r.left, r.top)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # Save annotated image
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('Results saved in', outputfile)
```

5. 変更を保存し、**computer-vision** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python detect-faces.py
    ```

6. 検出された顔の数を示す出力を確認します。
7. コードファイルと同じフォルダーに生成された **detected_faces.jpg** ファイルを表示して、注釈付きの顔を確認します。

## <a name="prepare-to-use-the-face-sdk"></a>Face SDK を使用する準備をする

**Computer Vision** サービスは (他の多くの画像分析機能とともに) 基本的な顔検出を提供しますが、**Face** サービスは顔の分析と認識のためのより包括的な機能を提供します。

1. Visual Studio Code の**エクスプローラー** ペインで、**19-iface** フォルダーを参照し、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. **face-api** フォルダーを右クリックして、統合ターミナルを開きます。次に、言語設定に適したコマンドを実行して、Face SDK パッケージをインストールします。

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.Face --version 2.6.0-preview.1
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-face==0.4.1
    ```
    
3. **face-api** フォルダーの内容を表示し、構成設定用のファイルが含まれていることを確認してください。
    - **C#** : appsettings.json
    - **Python**: .env

4. 構成ファイルを開き、含まれている構成値を更新して、Cognitive Services リソースの**エンドポイント**と認証**キー**を反映します。変更を保存します

5. **face-api** フォルダーに、クライアント アプリケーションの次のコード ファイルが含まれていることを確認してください。

    - **C#** : Program.cs
    - **Python**: analyze-faces.py

6. コード ファイルを開き、上部の既存の名前空間参照の下で、**「Import namespaces」** というコメントを見つけます。次に、このコメントの下に、次の言語固有のコードを追加して、Computer Vision SDK を使用するために必要な名前空間をインポートします

    **C#**

    ```C#
    // Import namespaces
    using Microsoft.Azure.CognitiveServices.Vision.Face;
    using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.cognitiveservices.vision.face import FaceClient
    from azure.cognitiveservices.vision.face.models import FaceAttributeType
    from msrest.authentication import CognitiveServicesCredentials
    ```

7. **Main** 関数では、構成設定をロードするためのコードが提供されていることに注意してください。次に、コメント **「Authenticate Face client」** を見つけます。次に、このコメントの下に、次の言語固有のコードを追加して、**FaceClient** オブジェクトを作成および認証します

    **C#**

    ```C#
    // Authenticate Face client
    ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
    faceClient = new FaceClient(credentials)
    {
        Endpoint = cogSvcEndpoint
    };
    ```

    **Python**

    ```Python
    # Authenticate Face client
    credentials = CognitiveServicesCredentials(cog_key)
    face_client = FaceClient(cog_endpoint, credentials)
    ```

8. **Main** 関数で、追加したコードの下に、コード内の関数を呼び出して Face サービスの機能を調べることができるメニューがコードに表示されることに注意してください。この演習の残りの部分では、これらの関数を実装します。

## <a name="detect-and-analyze-faces"></a>顔を検出して分析する

Face サービスの最も基本的な機能の 1 つは、画像内の顔を検出し、頭部姿勢、ぼやけ、眼鏡の存在などの属性を決定することです。

1. アプリケーションのコードファイルの **Main** 関数で、ユーザーがメニュー オプション **1** を選択した場合に実行されるコードを調べます。このコードは **DetectFaces** 関数を呼び出し、パスを画像ファイルに渡します。
2. コード ファイルで **DetectFaces** 関数を見つけて、コメント "**取得する顔機能を指定する**" の下に、次のコードを追加します。

    **C#**

    ```C#
    // Specify facial features to be retrieved
    List<FaceAttributeType?> features = new List<FaceAttributeType?>
    {
        FaceAttributeType.Occlusion,
        FaceAttributeType.Blur,
        FaceAttributeType.Glasses
    };
    ```

    **Python**

    ```Python
    # Specify facial features to be retrieved
    features = [FaceAttributeType.occlusion,
                FaceAttributeType.blur,
                FaceAttributeType.glasses]
    ```

3. **DetectFaces** 関数で、追加したコードの下に、コメント「**Get faces**」を見つけて、次のコードを追加します。

**C#**

```C
// Get faces
using (var imageData = File.OpenRead(imageFile))
{    
    var detected_faces = await faceClient.Face.DetectWithStreamAsync(imageData, returnFaceAttributes: features, returnFaceId: false);

    if (detected_faces.Count > 0)
    {
        Console.WriteLine($"{detected_faces.Count} faces detected.");

        // Prepare image for drawing
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        SolidBrush brush = new SolidBrush(Color.Black);
        int faceCount=0;

        // Draw and annotate each face
        foreach (var face in detected_faces)
        {
            faceCount++;
            Console.WriteLine($"\nFace number {faceCount}");
            
            // Get face properties
            Console.WriteLine($" - Mouth Occluded: {face.FaceAttributes.Occlusion.MouthOccluded}");
            Console.WriteLine($" - Eye Occluded: {face.FaceAttributes.Occlusion.EyeOccluded}");
            Console.WriteLine($" - Blur: {face.FaceAttributes.Blur.BlurLevel}");
            Console.WriteLine($" - Glasses: {face.FaceAttributes.Glasses}");

            // Draw and annotate face
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
        }

        // Save annotated image
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}
```

**Python**

```Python
# Get faces
with open(image_file, mode="rb") as image_data:
    detected_faces = face_client.face.detect_with_stream(image=image_data,
                                                            return_face_attributes=features,                     return_face_id=False)

    if len(detected_faces) > 0:
        print(len(detected_faces), 'faces detected.')

        # Prepare image for drawing
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'
        face_count = 0

        # Draw and annotate each face
        for face in detected_faces:

            # Get face properties
            face_count += 1
            print('\nFace number {}'.format(face_count))

            detected_attributes = face.face_attributes.as_dict()
            if 'blur' in detected_attributes:
                print(' - Blur:')
                for blur_name in detected_attributes['blur']:
                    print('   - {}: {}'.format(blur_name, detected_attributes['blur'][blur_name]))
                    
            if 'occlusion' in detected_attributes:
                print(' - Occlusion:')
                for occlusion_name in detected_attributes['occlusion']:
                    print('   - {}: {}'.format(occlusion_name, detected_attributes['occlusion'][occlusion_name]))

            if 'glasses' in detected_attributes:
                print(' - Glasses:{}'.format(detected_attributes['glasses']))

            # Draw and annotate face
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)

        # Save annotated image
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('\nResults saved in', outputfile)
```

4. **DetectFaces** 関数に追加したコードを調べます。画像ファイルを分析し、年齢、感情、眼鏡の存在など、画像ファイルに含まれるすべての顔を検出します。各顔に割り当てられた一意の顔識別子を含む、各顔の詳細が表示されます。顔の位置は、境界ボックスを使用して画像に示されます。
5. 変更を保存して **face-api** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    *C# 出力に、**await** 演算子を使用している非同期関数に関する警告が表示される場合があります。これは無視してもかまいません。*

    **Python**

    ```
    python analyze-faces.py
    ```

6. プロンプトが表示されたら、「**1**」と入力し、出力を観察します。ここには、検出された各顔の ID と属性が含まれているはずです。
7. コード ファイルと同じフォルダーに生成された **detected_faces.jpg** ファイルを表示して、注釈付きの顔を確認します。

## <a name="more-information"></a>詳細情報

**Face** サービスにはいくつかの追加機能がありますが、[責任ある AI 標準](https://aka.ms/aah91ff)に従うと、制限付きアクセス ポリシーで制限されます。 これらの機能には、顔認識モデルの識別、検証、作成が含まれます。 詳細とアクセスの申請については、[Cognitive Services の制限付きアクセス](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-limited-access)に関するページを参照してください。

顔検出に **Computer Vision** サービスを使用する方法の詳細については、[Computer Vision のドキュメント](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces)を参照してください。

**Face** サービスの詳細については、[Face のドキュメント](https://docs.microsoft.com/azure/cognitive-services/face/)を参照してください。
