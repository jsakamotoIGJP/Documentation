# Excel ファイル データ ソースの置き換え

ダッシュボードは、クラウドに保存されている Excel ファイルを表示形式のデータ ソースとして作成される場合があります。

アプリケーションに Reveal SDK を埋め込む場合、これらのクラウドベースのファイルを、実行時にサーバー上にあるローカル ディレクトリに保存されているファイルに置き換えることができます。

**手順 1** - ASP.NET Web API サーバー アプリケーションで、`IRVDataSourceProvider` を実装するクラスを作成します。このクラスは、Excel ファイルの実際の置換を実行します。

```cs
public class MyDataSourceProvider : IRVDataSourceProvider
{
    public Task<RVDataSourceItem> ChangeDataSourceItemAsync(IRVUserContext userContext, string dashboardId, RVDataSourceItem dataSourceItem)
    {
        throw new NotImplementedException();
    }
}
```

このクラスの `ChangeDataSourceItemAsync` メソッドは、表示形式がデータを取得するために使用する `RVDataSourceItem` を返します。`ChangeDataSourceItemAsync` メソッドで引数として提供される `RVDataSourceItem` 項目を変更することにより、データを取得する Excel ファイルを変更できます。

**手順 2** - `RevealSetupBuilder.AddDataSourceProvider` メソッドを使用して、作成した `IRVDataSourceProvider` を `RevealSetupBuilder` に追加するよう、`Program.cs` ファイルの `AddReveal` メソッドを更新します。

```cs
builder.Services.AddControllers().AddReveal( builder =>
{
    builder.AddDataSourceProvider<MyDataSourceProvider>();
});
```

## 例: Excel ファイル データ ソースの置き換え

この例では、「Sales CloudExcelFile」という名前のクラウドベースの Excel ファイルを使用しているデータ ソース項目を「SalesLocalExcelFile.xlsx」という名前のローカル Excel ファイルに置き換えています。

まず、引数に渡された `RVDataSourceItem` をチェックして、それが `RVExcelDataSourceItem` であるかどうかを確認します。そうである場合は、既存の `RVDataSourceItem.ResourceItem` を取得し、その `Title` プロパティを確認します。タイトルが「SalesCloudExcel File」の場合、新しい `RVLocalFileDataSourceItem` を作成し、`Uri` を新しいローカル Excel ファイルの場所に設定します。ローカル Excel ファイル データ ソース項目のタイトルを設定した後、`RVExcelDataSourceItem.ResourceItem` を新しく作成した `RVLocalFileDataSourceItem` に置き換えます。

```cs
public class MyDataSourceProvider : IRVDataSourceProvider
{
    public Task<RVDataSourceItem> ChangeDataSourceItemAsync(IRVUserContext userContext, string dashboardId, RVDataSourceItem dataSourceItem)
    {
        if (dataSourceItem is RVExcelDataSourceItem excelDataSourceItem)
        {
            var resourceItem = excelDataSourceItem.ResourceItem as RVDataSourceItem;
            if (resourceItem.Title == "Sales Cloud Excel File")
            {
                var localItem = new RVLocalFileDataSourceItem();
                localItem.Uri = "local:/SalesLocalExcelFile.xlsx";
                localItem.Title = resourceItem.Title;

                excelDataSourceItem.ResourceItem = localItem;
            }
        }

        return Task.FromResult(dataSourceItem);
    }
}
```