---
lab:
  title: Computer Vision を使用する画像の分析
  module: Module 8 - Getting Started with Computer Vision
---

# <a name="analyze-images-with-computer-vision"></a>Computer Vision を使用する画像の分析

Computer vision is an artificial intelligence capability that enables software systems to interpret visual input by analyzing images. In Microsoft Azure, the <bpt id="p1">**</bpt>Computer Vision<ept id="p1">**</ept> cognitive service provides pre-built models for common computer vision tasks, including analysis of images to suggest captions and tags, detection of common objects, landmarks, celebrities, brands, and the presence of adult content. You can also use the Computer Vision service to analyze image color and formats, and to generate "smart-cropped" thumbnail images.

## <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

If you have not already cloned <bpt id="p1">**</bpt>AI-102-AIEngineer<ept id="p1">**</ept> code repository to the environment where you're working on this lab, follow these steps to do so. Otherwise, open the cloned folder in Visual Studio Code.

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

この演習では、Computer VisionSDKを使用して画像を分析する部分的に実装されたクライアントアプリケーションを完成させます。

> Computer Vision は、ソフトウェアシステムが画像を分析することで視覚入力を解釈できるようにする人工知能機能です。

1. Visual Studio Code の**エクスプローラー** ペインで、**15-computer-vision** フォルダーを参照し、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. Microsoft Azure では、**Computer Vision** Cognitive Services は、キャプションとタグを提案する画像の分析、一般的なオブジェクト、ランドマーク、有名人、ブランドの検出、アダルトコンテンツの存在など、一般的な Computer Vision 用の構築済みモデルを提供します。

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-computervision==0.7.0
```
    
3. **image-analysis** フォルダーの内容を表示し、構成設定用のファイルが含まれていることに注意してください。
    - **C#** : appsettings.json
    - **Python**: .env

    Computer Vision サービスを使用して、画像の色と形式を分析し、"スマートにトリミングされた" サムネイル画像を生成することもできます。
4. **image-analysis** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることに注意してください。

    - **C#** : Program.cs
    - **Python**: image-analysis.py

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
from azure.cognitiveservices.vision.computervision.models import VisualFeatureTypes
from msrest.authentication import CognitiveServicesCredentials
```
    
## <a name="view-the-images-you-will-analyze"></a>ビューの画像は、あなたが分析します

この演習では、Computer Vision サービスを使用して複数の画像を分析します。

1. Visual Studio Code で、**image-analysis** フォルダーとそれに含まれる **images** フォルダーを展開します。
2. 各画像ファイルを順番に選択して表示し、Visual Studio Code で表示します。

## <a name="analyze-an-image-to-suggest-a-caption"></a>画像を分析してキャプションを提案する

これで、SDK を使用して Computer Vision サービスを呼び出す準備が整いました。

1. In the code file for your client application (<bpt id="p1">**</bpt>Program.cs<ept id="p1">**</ept> or <bpt id="p2">**</bpt>image-analysis.py<ept id="p2">**</ept>), in the <bpt id="p3">**</bpt>Main<ept id="p3">**</ept> function, note that the code to load the configuration settings has been provided. Then find the comment <bpt id="p1">**</bpt>Authenticate Computer Vision client<ept id="p1">**</ept>. Then, under this comment, add the following language-specific code to create and authenticate a Computer Vision client object:

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

2. このラボで作業している環境に **AI-102-AIEngineer** コードのリポジトリをまだクローンしていない場合は、次の手順に従ってクローンします。

3. **AnalyzeImage** 関数のコメント "**Specify features to be retrieved**" の下に、次のコードを追加します。

**C#**

```C#
// Specify features to be retrieved
List<VisualFeatureTypes?> features = new List<VisualFeatureTypes?>()
{
    VisualFeatureTypes.Description,
    VisualFeatureTypes.Tags,
    VisualFeatureTypes.Categories,
    VisualFeatureTypes.Brands,
    VisualFeatureTypes.Objects,
    VisualFeatureTypes.Adult
};
```

**Python**

```Python
# Specify features to be retrieved
features = [VisualFeatureTypes.description,
            VisualFeatureTypes.tags,
            VisualFeatureTypes.categories,
            VisualFeatureTypes.brands,
            VisualFeatureTypes.objects,
            VisualFeatureTypes.adult]
```
    
4. **AnalyzeImage** 関数のコメント "**Get image analysis**" の下に、次のコードを追加します (後でコードを追加する場所を示すコメントを含む)。

**C#**

```C
// Get image analysis
using (var imageData = File.OpenRead(imageFile))
{    
    var analysis = await cvClient.AnalyzeImageInStreamAsync(imageData, features);

    // get image captions
    foreach (var caption in analysis.Description.Captions)
    {
        Console.WriteLine($"Description: {caption.Text} (confidence: {caption.Confidence.ToString("P")})");
    }

    // Get image tags


    // Get image categories


    // Get brands in the image


    // Get objects in the image


    // Get moderation ratings
    

}            
```

**Python**

```Python
# Get image analysis
with open(image_file, mode="rb") as image_data:
    analysis = cv_client.analyze_image_in_stream(image_data , features)

# Get image description
for caption in analysis.description.captions:
    print("Description: '{}' (confidence: {:.2f}%)".format(caption.text, caption.confidence * 100))

# Get image tags


# Get image categories 


# Get brands in the image


# Get objects in the image


# Get moderation ratings

```
    
5. 変更を保存して、**image-analysis** フォルダーの統合ターミナルに戻り、引数 **images/street.jpg** でプログラムを実行するには、次のコマンドを入力します。

**C#**

```
dotnet run images/street.jpg
```

**Python**

```
python image-analysis.py images/street.jpg
```
    
6. 出力を観察します。これには、**street.jpg** 画像の推奨キャプションが含まれている必要があります。
7. 今度は引数 **images/building.jpg** を使用してプログラムを再度実行し、**building.jpg** 画像に対して生成されるキャプションを確認します。
8. 前の手順を繰り返して、**images/person.jpg** ファイルのキャプションを生成します。

## <a name="get-suggested-tags-for-an-image"></a>画像の推奨タグを取得

画像の内容に関する手がかりを提供する関連*タグ*を特定すると役立つ場合があります。

1. **AnalyzeImage** 関数のコメント "**Get image tags**" の下に、次のコードを追加します。

**C#**

```C
// Get image tags
if (analysis.Tags.Count > 0)
{
    Console.WriteLine("Tags:");
    foreach (var tag in analysis.Tags)
    {
        Console.WriteLine($" -{tag.Name} (confidence: {tag.Confidence.ToString("P")})");
    }
}
```

**Python**

```Python
# Get image tags
if (len(analysis.tags) > 0):
    print("Tags: ")
    for tag in analysis.tags:
        print(" -'{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
```

2. 変更を保存し、**images** フォルダー内の画像ファイルごとにプログラムを 1 回実行し、画像のキャプションに加えて、提案されたタグのリストが表示されることを確認します。

## <a name="get-image-categories"></a>Get image categories

Computer Vision サービスは、画像の *"カテゴリ"* を提案でき、各カテゴリ内で有名なランドマークを識別できます。

1. **AnalyzeImage** 関数のコメント "**Get image categories**" の下に、次のコードを追加します。

**C#**

```C
// Get image categories
List<LandmarksModel> landmarks = new List<LandmarksModel> {};
Console.WriteLine("Categories:");
foreach (var category in analysis.Categories)
{
    // Print the category
    Console.WriteLine($" -{category.Name} (confidence: {category.Score.ToString("P")})");

    // Get landmarks in this category
    if (category.Detail?.Landmarks != null)
    {
        foreach (LandmarksModel landmark in category.Detail.Landmarks)
        {
            if (!landmarks.Any(item => item.Name == landmark.Name))
            {
                landmarks.Add(landmark);
            }
        }
    }
}

// If there were landmarks, list them
if (landmarks.Count > 0)
{
    Console.WriteLine("Landmarks:");
    foreach(LandmarksModel landmark in landmarks)
    {
        Console.WriteLine($" -{landmark.Name} (confidence: {landmark.Confidence.ToString("P")})");
    }
}

```

**Python**

```Python
# Get image categories
if (len(analysis.categories) > 0):
    print("Categories:")
    landmarks = []
    for category in analysis.categories:
        # Print the category
        print(" -'{}' (confidence: {:.2f}%)".format(category.name, category.score * 100))
        if category.detail:
            # Get landmarks in this category
            if category.detail.landmarks:
                for landmark in category.detail.landmarks:
                    if landmark not in landmarks:
                        landmarks.append(landmark)

    # If there were landmarks, list them
    if len(landmarks) > 0:
        print("Landmarks:")
        for landmark in landmarks:
            print(" -'{}' (confidence: {:.2f}%)".format(landmark.name, landmark.confidence * 100))

```
    
2. 変更を保存し、**images** フォルダー内の画像ファイルごとにプログラムを 1 回実行し、画像のキャプションとタグに加えて、推奨されるカテゴリのリストが、認識されているランドマーク (具体的には **building.jpg** の画像) とともに表示されることを確認します。

## <a name="get-brands-in-an-image"></a>画像でブランドを取得する

それ以外の場合は、複製されたフォルダーを Visual Studio Code で開きます。

1. **AnalyzeImage** 関数で、コメント "**Get brands in the image**" 下に、次のコードを追加します。

**C#**

```C
// Get brands in the image
if (analysis.Brands.Count > 0)
{
    Console.WriteLine("Brands:");
    foreach (var brand in analysis.Brands)
    {
        Console.WriteLine($" -{brand.Name} (confidence: {brand.Confidence.ToString("P")})");
    }
}
```

**Python**

```Python
# Get brands in the image
if (len(analysis.brands) > 0):
    print("Brands: ")
    for brand in analysis.brands:
        print(" -'{}' (confidence: {:.2f}%)".format(brand.name, brand.confidence * 100))
```
    
2. 変更を保存し、**images** フォルダー内の画像ファイルごとにプログラムを 1 回実行し、識別されたブランド (具体的には、**person.jpg** 画像) を確認します。

## <a name="detect-and-locate-objects-in-an-image"></a>画像内のオブジェクトを検出して特定する

*物体検出*は、画像内の個々のオブジェクトが識別され、その場所が境界ボックスで示される特定の形式の Computer Vision です。

1. **AnalyzeImage** 関数のコメント "**Get objects in the image**" の下に、次のコードを追加します。

**C#**

```C
// Get objects in the image
if (analysis.Objects.Count > 0)
{
    Console.WriteLine("Objects in image:");

    // Prepare image for drawing
    Image image = Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.Black);

    foreach (var detectedObject in analysis.Objects)
    {
        // Print object name
        Console.WriteLine($" -{detectedObject.ObjectProperty} (confidence: {detectedObject.Confidence.ToString("P")})");

        // Draw object bounding box
        var r = detectedObject.Rectangle;
        Rectangle rect = new Rectangle(r.X, r.Y, r.W, r.H);
        graphics.DrawRectangle(pen, rect);
        graphics.DrawString(detectedObject.ObjectProperty,font,brush,r.X, r.Y);

    }
    // Save annotated image
    String output_file = "objects.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file);   
}
```

**Python**

```Python
# Get objects in the image
if len(analysis.objects) > 0:
    print("Objects in image:")

    # Prepare image for drawing
    fig = plt.figure(figsize=(8, 8))
    plt.axis('off')
    image = Image.open(image_file)
    draw = ImageDraw.Draw(image)
    color = 'cyan'
    for detected_object in analysis.objects:
        # Print object name
        print(" -{} (confidence: {:.2f}%)".format(detected_object.object_property, detected_object.confidence * 100))
        
        # Draw object bounding box
        r = detected_object.rectangle
        bounding_box = ((r.x, r.y), (r.x + r.w, r.y + r.h))
        draw.rectangle(bounding_box, outline=color, width=3)
        plt.annotate(detected_object.object_property,(r.x, r.y), backgroundcolor=color)
    # Save annotated image
    plt.imshow(image)
    outputfile = 'objects.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```
    
2. Save your changes and run the program once for each of the image files in the <bpt id="p1">**</bpt>images<ept id="p1">**</ept> folder, observing any objects that are detected. After each run, view the <bpt id="p1">**</bpt>objects.jpg<ept id="p1">**</ept> file that is generated in the same folder as your code file to see the annotated objects.

## <a name="get-moderation-ratings-for-an-image"></a>画像のモデレート評価を取得する

一部の画像はすべての視聴者に適しているとは限らないため、成人向けまたは暴力的な性質の画像を特定するためにモデレートを適用する必要がある場合があります。

1. **AnalyzeImage** 関数のコメント "**Get moderation ratings**" の下に、次のコードを追加します。

**C#**

```C
// Get moderation ratings
string ratings = $"Ratings:\n -Adult: {analysis.Adult.IsAdultContent}\n -Racy: {analysis.Adult.IsRacyContent}\n -Gore: {analysis.Adult.IsGoryContent}";
Console.WriteLine(ratings);
```

**Python**

```Python
# Get moderation ratings
ratings = 'Ratings:\n -Adult: {}\n -Racy: {}\n -Gore: {}'.format(analysis.adult.is_adult_content,
                                                                    analysis.adult.is_racy_content,
                                                                    analysis.adult.is_gory_content)
print(ratings)
```
    
2. 変更を保存し、**images** フォルダー内の画像ファイルごとにプログラムを 1 回実行し、各画像の評価を確認します。

> <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: In the preceding tasks, you used a single method to analyze the image, and then incrementally added code to parse and display the results. The SDK also provides individual methods for suggesting captions, identifying tags, detecting objects, and so on - meaning that you can use the most appropriate method to return only the information you need, reducing the size of the data payload that needs to be returned. See the <bpt id="p1">[</bpt>.NET SDK documentation<ept id="p1">](https://docs.microsoft.com/dotnet/api/overview/azure/cognitiveservices/client/computervision?view=azure-dotnet)</ept> or <bpt id="p2">[</bpt>Python SDK documentation<ept id="p2">](https://docs.microsoft.com/python/api/overview/azure/cognitiveservices/computervision?view=azure-python)</ept> for more details.

## <a name="generate-a-thumbnail-image"></a>サムネイル画像の生成

場合によっては、*サムネイル*という名前の画像の小さいバージョンを作成し、それをトリミングして、新しい画像の寸法内に主要な視覚的主題を含める必要があります。

1. コード ファイルで、**GetThumbnail** 関数を見つけます。コメント "**Generate a thumbnail**" の下に、次のコードを追加します。

**C#**

```C
// Generate a thumbnail
using (var imageData = File.OpenRead(imageFile))
{
    // Get thumbnail data
    var thumbnailStream = await cvClient.GenerateThumbnailInStreamAsync(100, 100,imageData, true);

    // Save thumbnail image
    string thumbnailFileName = "thumbnail.png";
    using (Stream thumbnailFile = File.Create(thumbnailFileName))
    {
        thumbnailStream.CopyTo(thumbnailFile);
    }

    Console.WriteLine($"Thumbnail saved in {thumbnailFileName}");
}
```

**Python**

```Python
# Generate a thumbnail
with open(image_file, mode="rb") as image_data:
    # Get thumbnail data
    thumbnail_stream = cv_client.generate_thumbnail_in_stream(100, 100, image_data, True)

# Save thumbnail image
thumbnail_file_name = 'thumbnail.png'
with open(thumbnail_file_name, "wb") as thumbnail_file:
    for chunk in thumbnail_stream:
        thumbnail_file.write(chunk)

print('Thumbnail saved in.', thumbnail_file_name)
```
    
2. 変更を保存し、**images** フォルダー内の画像ファイルごとにプログラムを 1 回実行し、各画像のコード ファイルと同じフォルダーに生成された **thumbnail.jpg** ファイルを開きます。

## <a name="more-information"></a>詳細情報

In this exercise, you explored some of the image analysis and manipulation capabilities of the Computer Vision service. The service also includes capabilities for reading text, detecting faces, and other computer vision tasks.

**Computer Vision** サービスの使用の詳細については、[Computer Vision のドキュメント](https://docs.microsoft.com/azure/cognitive-services/computer-vision/)を参照してください。
