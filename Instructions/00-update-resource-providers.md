---
lab:
  title: リソース プロバイダーを有効にする
  module: Setup
---

# <a name="enable-resource-providers"></a>リソース プロバイダーを有効にする

There are some resource providers that must be registered in your Azure subscription. Follow these steps to ensure that they're registered.

1. ご利用の Azure サブスクリプションに関連付けられている Microsoft 資格情報を使用して、Azure portal `https://portal.azure.com` にサインインします。
2. **[ホーム]** ページの **[サブスクリプション]** を選択します (または **[&#8801;]** メニューを展開し、 **[すべてのサービス]** を選択し、 **[全般]** カテゴリの **[サブスクリプション]** を選択します)。
3. Azure サブスクリプションを選択します (複数のサブスクリプションがある場合は、Azure Pass を利用して作成したサブスクリプションを選択します)。
4. サブスクリプションのブレードの左側のウィンドウにある **[設定]** セクションで、 **[リソース プロバイダー]** を選択します。
5. リソース プロバイダーのリストで、次のプロバイダーが登録されていることを確認します (登録されていない場合は、それらを選択して **[登録]** をクリックします)。
    - Microsoft.BotService
    - Microsoft.Web
    - Microsoft.ManagedIdentity
    - Microsoft.Search
    - Microsoft.Storage
    - Microsoft.CognitiveServices
    - Microsoft.AlertsManagement
    - microsoft.insights
    - Microsoft.KeyVault
    - Microsoft.ContainerInstance
