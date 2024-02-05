---
title: "ページネーション、フィルタリング、ソートを備えた動的テーブルを作成するためのタブレータを備えたASP.NET Core"
date: 2019-10-09
author: "Aleksandr Agapitov"
draft: false
description: |
  動的テーブルの実装 
  ASP.NET Coreと
  EntityFrameworkコアとページネーション

tags: ["ASP.NET", "EntityFramework", "Javascript"]
categories: ["チュートリアル"]
---

<figure>
  <img src="/images/2019/oct/dotnet-tabulator-filtering.png" height="250" alt="Dotnet-Tabulator-Filtering"/>
  <figcaption>これが、この記事で私たちが作っているものだ。</figcaption>
</figure>

<!--more-->

[Tabulator](http://tabulator.info)は、データ駆動型テーブルを作成するためのJavascriptフレームワークで、ソート、フィルタリング、エクスポート、その他多くの素晴らしい機能など、サーバーサイドにもフロントエンドにも多くの機能を提供しています。以下のようなツールを使ってきました： [Handsontable](https://www.handsontable.com)、[Datatables](https://www.datatables.net)、そしてカスタムの[Razor Pages](https://docs.microsoft.com/en-us/aspnet/core/data/ef-rp/sort-filter-page?view=aspnetcore-3.0)のようなツールを使ってきましたが、Tabulatorを試してみることにしました。そして、私が望むすべての機能を無料で提供し、柔軟性に富み、比較的少ないフロントエンドのコードで済む最適なソリューションであることがわかりました。この記事では、この素晴らしいJavascriptフレームワークがASP.NET CoreとEntityFrameworkでどのように効果的に動作するかを紹介する。

この記事のコードはすべて、[こちら](https://github.com/aleksvagapitov/DotnetTabulatorFiltering)で見ることができる。

<!--more-->

## この例のモデル

この例では、以下のモデルを持つデータベース・スキーマを使用します：

{{< highlight csharp >}}
public class Contact
{
    [Key]
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public int Age { get; set; }
    public string Gender { get; set; }
    public string Email { get; set; }
    public int ZipCode { get; set; }
    public string About { get; set; }
}
{{< / highlight >}}


## データベースコンテキストの構造

この例では、Startup.csファイルで依存性注入を使用したインメモリ・ストレージを使用します：

{{< highlight csharp >}}
public class TabulatorContext : DbContext
{
    public DbSet<Contact> Contacts { get; set; }

    public TabulatorContext
      (DbContextOptions<TabulatorContext> options) 
        : base (options) { }

    public void EnsureSeedData (){}
}
{{< / highlight >}}

## リポジトリの構造

リポジトリを見る前にインストールする必要がある重要なパッケージの1つは、[EntityFrameworkPaginateCore](https://www.nuget.org/packages/EntityFrameworkPaginateCore)です。これは、バックエンド側でフィルタリングとソートを行うために必要です。Dotnet CLIを使って、以下のコマンドでインストールできます：

| dotnet add package EntityFrameworkPaginateCore --version 1.1.0

*Github上のサンプル・プロジェクトには必要なパッケージがすべて含まれており、プロジェクトのREADMEの指示に従うだけです。*

リポジトリ構造は、以下のメソッドシグネチャに依存しています：


{{< highlight csharp >}}
Task<TabulatorViewModel> GetFilteredData (int page, int size,
  List<Dictionary<string, string>> filters, List<Dictionary<string, string>> sorters)
{{< / highlight >}}

ここで重要なのは、「フィルター」と「ソーター」のパラメーターの種類である：

{{< highlight csharp >}}
List<Dictionary<string, string>>
{{< / highlight >}}

というのも、以下のような形式でフィルターを取得することになるからだ：
{{< highlight json >}}
[{field:"age", type:">", value:52}, {field:"height", type:"<", value:142}]
{{< / highlight >}}

とソーター用：

{{< highlight json >}}
[{field: "age", dir: "asc"}]
{{< / highlight >}}

さらに重要なのは、Tabulatorにデータを戻すために使用するViewModelです：

{{< highlight csharp >}}
public class TabulatorViewModel
{
    public IEnumerable<dynamic> Data { get; set; }
    public int Last_page { get; set; }
}
{{< / highlight >}}

メソッドの全容は[こちら](https://github.com/aleksvagapitov/DotnetTabulatorFiltering/blob/master/Models/TabulatorRespository.cs)で説明されており、ほとんどの場合、辞書項目のリストを反復処理し、ブログの冒頭で言及したモデルで説明した各カラムのフィールドと値を設定することで構成されている。

## フロントエンド・タブレーター・コード 
{{< highlight csharp >}}
var table = new Tabulator("#example-table", {
    pagination: "remote",
    ajaxURL: baseUrl + apiEndpoint,
    paginationSize: 10,
    paginationSizeSelector: [10, 20, 30, 50],
    ajaxFiltering: true,
    ajaxSorting: true,
    columns: [
        { title: "FirstName", field: "firstName", sorter: "string", headerFilter: "input" },
        { title: "LastName", field: "lastName", sorter: "string", headerFilter: "input" },
        { title: "Age", field: "age", sorter: "number", headerFilter: "input" },
        { title: "Gender", field: "gender", sorter: "string", headerFilter: "input" },
        { title: "Email", field: "email", sorter: "string", headerFilter: "input" },
        { title: "ZipCode", field: "zipCode", sorter: "number", headerFilter: "input" },
        { title: "About", field: "about", sorter: "string", headerFilter: "input", width: 500 }
    ]
});
{{< / highlight >}}

上記の構造により、Ajaxを使ってデータにフィルタをかけたり、ページ分割したりすることができる。この構造でとても便利なのは、バックエンドを調整することなくフロントエンドからカラムを削除できることだ。**しかし**、フロントエンドに渡すオブジェクトには注意し、APIが何を返すか確認してください。データが漏れないようにする方法は、ViewModelsと[AutoMapper](https://automapper.org)を使うか、通常のオブジェクト合成を使って、関連するデータだけをフロントエンドに渡すことです。このブログでは、あまり多くの概念を紹介しないように、Repositoryメソッドの実装の中で、AutoMapper節が入る箇所をコメントアウトしておきました。最後に、[Postman](https://www.getpostman.com)のようなツールを使ってAPIを照会し、それが返すデータを見ることができる。

繰り返しますが、すべてのコードは[こちら](https://github.com/aleksvagapitov/DotnetTabulatorFiltering)に、Tabulatorに関する完全なドキュメントは[こちら](http://tabulator.info)にあります。

お楽しみください。バグや問題があれば、遠慮なく報告ください： [GitHub issues](https://github.com/aleksvagapitov/DotnetTabulatorFiltering/issues) :)
