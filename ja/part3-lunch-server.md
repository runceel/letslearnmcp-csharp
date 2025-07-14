# パート3: 独自のMCPサーバーの構築 - Lunch Roulette

> **ナビゲーション**: [← パート2: Monkey アプリ](part2-monkey-app.md) | [概要に戻る](README.md)

> **ベース**: [LunchRouletteMCP リポジトリ](https://github.com/jamesmontemagno/LunchRouletteMCP)

## 概要
ランチ推薦システムを作成することで、ゼロからMCPサーバーを構築する方法を学習します。このチュートリアルでは、C#を使用してMCPサーバーの設計、実装、デプロイの完全なプロセスを教えます。

## 3.1 プロジェクト基盤

### コンソールアプリの作成

**LunchTime**という新しいフォルダーを作成し、そのフォルダ内に**LunchTimeMCP**という名前の新しいC#コンソールアプリを作成します。

### NuGetパッケージのインストール
MCPサーバー開発に必要なパッケージ：
```xml
<ItemGroup>
    <PackageReference Include="Microsoft.Extensions.Hosting" Version="9.0.6" />
    <PackageReference Include="ModelContextProtocol" Version="0.3.0-preview.1" />
    <PackageReference Include="System.Text.Json" Version="9.0.6" />
</ItemGroup>
```

### Program.csのセットアップ
- ホストビルダーの設定
- 依存性注入のセットアップ
- サービスとMCPツールの登録
- MCPサーバーコンポーネントの初期化
- アプリケーションライフサイクルの処理

```csharp
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using ModelContextProtocol.Server;

var builder = Host.CreateEmptyApplicationBuilder(settings: null);

builder.Services
    .AddMcpServer()
    .WithStdioServerTransport();

await builder.Build().RunAsync();
```

**重要なポイント**:
- MCPライブラリが残りのサーバーセットアップとプロトコル実装を処理
- サービスとツールを登録するために戻ってきます

## 3.2 コアコンポーネント

### RestaurantServiceの実装
以下の完全な**RestaurantService**を**RestaurantService.cs**という新しいファイルにコピーします。このサービスはコアビジネスロジックを提供し、以下を含みます：

**主要機能：**
- **データ永続化**: ユーザーのAppDataディレクトリでのJSONファイルストレージ
- **レストラン管理**: レストランの追加、取得、追跡
- **訪問追跡**: レストラン選択の回数と統計
- **シードデータ**: トレンディな品川レストランで事前入力
- **非同期操作**: すべてのメソッドでasync/awaitパターンを使用
- **エラーハンドリング**: ファイル操作用のtry-catchブロック

**コアメソッド：**
- `GetRestaurantsAsync()` - すべてのレストランを取得
- `AddRestaurantAsync()` - 検証付きで新しいレストランを追加
- `PickRandomRestaurantAsync()` - 訪問追跡付きランダム選択
- `GetVisitStatsAsync()` - 生の訪問統計
- `GetFormattedVisitStatsAsync()` - ユーザーフレンドリーな書式設定済み統計

**データモデル：**
- `Restaurant` - Id、Name、Location、FoodType、DateAddedを持つコアレストランエンティティ
- `RestaurantVisitInfo` - 回数と最終訪問日による訪問追跡
- `RestaurantData` - レストランと訪問数のJSONシリアル化用コンテナ
- `FormattedRestaurantStats` - 統計表示用プレゼンテーション層

**技術アーキテクチャ：**
- **ファイルストレージ**: `%AppData%/LunchTimeMCP/restaurants.json`でのJSON永続化
- **JSONシリアル化**: ソースジェネレーター（`RestaurantContext`）でSystem.Text.Jsonを使用
- **メモリ管理**: 定期的なファイル保存付きインメモリコレクション
- **初期化**: 空の場合は10のトレンディな品川レストランで自動シード

```csharp
using System.Text.Json;
using System.Text.Json.Serialization;

namespace LunchTimeMCP;

public class RestaurantService
{
    private readonly string dataFilePath;
    private List<Restaurant> restaurants = new();
    private Dictionary<string, int> visitCounts = new();

    public RestaurantService()
    {
        // ユーザーのアプリデータディレクトリにデータを保存
        var appDataPath = Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData);
        var appDir = Path.Combine(appDataPath, "LunchTimeMCP");
        Directory.CreateDirectory(appDir);
        
        dataFilePath = Path.Combine(appDir, "restaurants.json");
        LoadData();
        
        // 空の場合はトレンディな品川のレストランで初期化
        if (restaurants.Count == 0)
        {
            InitializeWithTrendyRestaurants();
            SaveData();
        }
    }

    public async Task<List<Restaurant>> GetRestaurantsAsync()
    {
        return await Task.FromResult(restaurants.ToList());
    }

    public async Task<Restaurant> AddRestaurantAsync(string name, string location, string foodType)
    {
        var restaurant = new Restaurant
        {
            Id = Guid.NewGuid().ToString(),
            Name = name,
            Location = location,
            FoodType = foodType,
            DateAdded = DateTime.UtcNow
        };

        restaurants.Add(restaurant);
        SaveData();
        
        return await Task.FromResult(restaurant);
    }

    public async Task<Restaurant?> PickRandomRestaurantAsync()
    {
        if (restaurants.Count == 0)
            return null;

        var random = new Random();
        var selectedRestaurant = restaurants[random.Next(restaurants.Count)];
        
        // 訪問を追跡
        if (visitCounts.ContainsKey(selectedRestaurant.Id))
            visitCounts[selectedRestaurant.Id]++;
        else
            visitCounts[selectedRestaurant.Id] = 1;

        SaveData();
        
        return await Task.FromResult(selectedRestaurant);
    }

    public async Task<Dictionary<string, RestaurantVisitInfo>> GetVisitStatsAsync()
    {
        var stats = new Dictionary<string, RestaurantVisitInfo>();
        
        foreach (var restaurant in restaurants)
        {
            var visitCount = visitCounts.GetValueOrDefault(restaurant.Id, 0);
            stats[restaurant.Name] = new RestaurantVisitInfo
            {
                Restaurant = restaurant,
                VisitCount = visitCount,
                LastVisited = visitCount > 0 ? DateTime.UtcNow : null // 実際のアプリでは実際の訪問日を追跡
            };
        }

        return await Task.FromResult(stats);
    }

    public async Task<FormattedRestaurantStats> GetFormattedVisitStatsAsync()
    {
        var stats = await GetVisitStatsAsync();
        
        var formattedStats = stats.Values
            .OrderByDescending(x => x.VisitCount)
            .Select(stat => new FormattedRestaurantStat
            {
                Restaurant = stat.Restaurant.Name,
                Location = stat.Restaurant.Location,
                FoodType = stat.Restaurant.FoodType,
                VisitCount = stat.VisitCount,
                TimesEaten = stat.VisitCount == 0 ? "なし" : 
                            stat.VisitCount == 1 ? "1回" : 
                            $"{stat.VisitCount}回"
            })
            .ToList();

        return new FormattedRestaurantStats
        {
            Message = "レストラン訪問統計：",
            Statistics = formattedStats,
            TotalRestaurants = stats.Count,
            TotalVisits = stats.Values.Sum(x => x.VisitCount)
        };
    }

    private void LoadData()
    {
        if (!File.Exists(dataFilePath))
            return;

        try
        {
            var json = File.ReadAllText(dataFilePath);
            var data = JsonSerializer.Deserialize<RestaurantData>(json, RestaurantContext.Default.RestaurantData);
            
            if (data != null)
            {
                restaurants = data.Restaurants ?? new List<Restaurant>();
                visitCounts = data.VisitCounts ?? new Dictionary<string, int>();
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine($"データ読み込みエラー: {ex.Message}");
        }
    }

    private void SaveData()
    {
        try
        {
            var data = new RestaurantData
            {
                Restaurants = restaurants,
                VisitCounts = visitCounts
            };
            
            var json = JsonSerializer.Serialize(data, RestaurantContext.Default.RestaurantData);
            File.WriteAllText(dataFilePath, json);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"データ保存エラー: {ex.Message}");
        }
    }

    private void InitializeWithTrendyRestaurants()
    {
        var trendyRestaurants = new List<Restaurant>
        {
            new() { Id = Guid.NewGuid().ToString(), Name = "和楽", Location = "品川区東五反田2-3-5", FoodType = "和食", DateAdded = DateTime.UtcNow },
            new() { Id = Guid.NewGuid().ToString(), Name = "ブルー・オーシャン", Location = "品川区港南1-2-10", FoodType = "フレンチ", DateAdded = DateTime.UtcNow },
            new() { Id = Guid.NewGuid().ToString(), Name = "タイ・ガーデン", Location = "品川区大崎3-1-8", FoodType = "タイ料理", DateAdded = DateTime.UtcNow },
            new() { Id = Guid.NewGuid().ToString(), Name = "グリーン・テーブル", Location = "品川区西五反田4-2-15", FoodType = "ビーガン", DateAdded = DateTime.UtcNow },
            new() { Id = Guid.NewGuid().ToString(), Name = "桜坂テラス", Location = "品川区東品川1-6-3", FoodType = "創作料理", DateAdded = DateTime.UtcNow },
            new() { Id = Guid.NewGuid().ToString(), Name = "オーシャン・ハーベスト", Location = "品川区南品川2-4-7", FoodType = "シーフード", DateAdded = DateTime.UtcNow },
            new() { Id = Guid.NewGuid().ToString(), Name = "ベラ・ナポリ", Location = "品川区北品川3-5-12", FoodType = "イタリアン", DateAdded = DateTime.UtcNow },
            new() { Id = Guid.NewGuid().ToString(), Name = "グローバル・キッチン", Location = "品川区大井1-3-9", FoodType = "インターナショナル", DateAdded = DateTime.UtcNow },
            new() { Id = Guid.NewGuid().ToString(), Name = "地中海の風", Location = "品川区戸越2-1-6", FoodType = "地中海料理", DateAdded = DateTime.UtcNow },
            new() { Id = Guid.NewGuid().ToString(), Name = "モダン品川", Location = "品川区上大崎1-4-11", FoodType = "コンテンポラリー", DateAdded = DateTime.UtcNow }
        };

        restaurants.AddRange(trendyRestaurants);
    }
}

public partial class Restaurant
{
    public string Id { get; set; } = string.Empty;
    public string Name { get; set; } = string.Empty;
    public string Location { get; set; } = string.Empty;
    public string FoodType { get; set; } = string.Empty;
    public DateTime DateAdded { get; set; }
}

public partial class RestaurantVisitInfo
{
    public Restaurant Restaurant { get; set; } = new();
    public int VisitCount { get; set; }
    public DateTime? LastVisited { get; set; }
}

public partial class RestaurantData
{
    public List<Restaurant> Restaurants { get; set; } = new();
    public Dictionary<string, int> VisitCounts { get; set; } = new();
}

public class FormattedRestaurantStat
{
    public string Restaurant { get; set; } = string.Empty;
    public string Location { get; set; } = string.Empty;
    public string FoodType { get; set; } = string.Empty;
    public int VisitCount { get; set; }
    public string TimesEaten { get; set; } = string.Empty;
}

public class FormattedRestaurantStats
{
    public string Message { get; set; } = string.Empty;
    public List<FormattedRestaurantStat> Statistics { get; set; } = new();
    public int TotalRestaurants { get; set; }
    public int TotalVisits { get; set; }
}

[JsonSerializable(typeof(List<Restaurant>))]
[JsonSerializable(typeof(Restaurant))]
[JsonSerializable(typeof(RestaurantVisitInfo))]
[JsonSerializable(typeof(Dictionary<string, RestaurantVisitInfo>))]
[JsonSerializable(typeof(RestaurantData))]
[JsonSerializable(typeof(FormattedRestaurantStat))]
[JsonSerializable(typeof(FormattedRestaurantStats))]
internal sealed partial class RestaurantContext : JsonSerializerContext
{
}
```

### サービスアーキテクチャ
RestaurantServiceは以下の重要なパターンに従います：

**依存性注入対応**
- 簡単なDI登録のためのパラメーターなしコンストラクター
- 外部依存関係なし、自己完結サービス
- 適切な状態管理によるシングルトンセーフ

**データ層の分離**
- プライベートメソッドがファイルI/O（`LoadData()`、`SaveData()`）を処理
- パブリック非同期メソッドがビジネス操作を提供
- 永続化とビジネスロジックの明確な分離

**エラー耐性**
- 欠損または破損したデータファイルの適切な処理
- ファイル操作デバッグ用のコンソールログ
- シードデータ初期化による自動復旧

**パフォーマンス考慮事項**
- 高速アクセスのためのインメモリ操作
- ファイルシステムからの遅延読み込み
- バッチ保存による最小限のファイルI/O

**JSONソースジェネレーション**
- `RestaurantContext`がAOTフレンドリーなシリアル化を提供
- 最適なパフォーマンスのためにすべてのモデル型を登録
- コンパイル時検証による型安全なシリアル化

## 3.3 MCPツール開発

次に、レストラン機能をAIアシスタントに公開するMCPツールを実装します。**RestaurantTools.cs**という新しいファイルを作成し、各ツールを段階的に実装します。

### RestaurantTools.csファイル構造

ツールの基本構造を作成することから始めましょう。

```csharp
using System.ComponentModel;
using System.Text.Json;
using ModelContextProtocol.Server;

namespace LunchTimeMCP;

[McpServerToolType]
public sealed class RestaurantTools
{
    private readonly RestaurantService restaurantService;

    public RestaurantTools(RestaurantService restaurantService)
    {
        this.restaurantService = restaurantService;
    }

    // すべてのツールをここに実装...
}
```

**アーキテクチャノート**:
- `[McpServerToolType]`がクラスをMCPツールを含むものとしてマーク
- コンストラクター注入が`RestaurantService`へのアクセスを提供

### ツール実装概要
各ツールは同じパターンに従います：
1. `[McpServerTool]`属性を使用してMCPに登録
2. AIがツールの目的を理解するための説明的な`[Description]`属性を追加
3. 依存性注入を使用して`RestaurantService`にアクセス
4. `RestaurantContext`を使用してJSONシリアル化されたレスポンスを返す

### ツール1: すべてのレストランを取得
**目的**: 利用可能なすべてのレストランのリストを取得
**パラメーター**: なし
**実装**:

```csharp
[McpServerTool, Description("ランチに利用可能なすべてのレストランのリストを取得します。")]
public async Task<string> GetRestaurants()
{
    var restaurants = await restaurantService.GetRestaurantsAsync();
    return JsonSerializer.Serialize(restaurants, RestaurantContext.Default.ListRestaurant);
}
```

**重要なポイント**:
- パラメーターなしのシンプルなメソッド
- JSONシリアル化されたレストランのリストを返す
- 最適化されたシリアル化のために`RestaurantContext.Default.ListRestaurant`を使用

### ツール2: 新しいレストランを追加
**目的**: 利用可能なオプションに新しいレストランを追加
**パラメーター**: `name`、`location`、`foodType`
**実装**:

```csharp
[McpServerTool, Description("ランチオプションに新しいレストランを追加します。")]
public async Task<string> AddRestaurant(
    [Description("レストランの名前")] string name,
    [Description("レストランの場所/住所")] string location,
    [Description("提供される料理の種類（例：イタリアン、メキシカン、タイ等）")] string foodType)
{
    var restaurant = await restaurantService.AddRestaurantAsync(name, location, foodType);
    return JsonSerializer.Serialize(restaurant, RestaurantContext.Default.Restaurant);
}
```

**重要なポイント**:
- 各パラメーターに説明的な`[Description]`属性
- レストランを追加するためのサービスメソッドを呼び出し
- 新しく作成されたレストランオブジェクトをJSONとして返す

### ツール3: ランダムなレストランを選択
**目的**: ランダムなレストランを選択し、訪問を追跡
**パラメーター**: なし
**実装**:

```csharp
[McpServerTool, Description("ランチ用に利用可能なオプションからランダムなレストランを選択します。")]
public async Task<string> PickRandomRestaurant()
{
    var selectedRestaurant = await restaurantService.PickRandomRestaurantAsync();
    
    if (selectedRestaurant == null)
    {
        return JsonSerializer.Serialize(new { 
            message = "利用可能なレストランがありません。まずレストランを追加してください！" 
        });
    }

    return JsonSerializer.Serialize(new { 
        message = $"🍽️ {selectedRestaurant.Name}でランチの時間です！",
        restaurant = selectedRestaurant 
    });
}
```

**重要なポイント**:
- レストランが利用できない場合を処理
- 絵文字付きの親しみやすいメッセージを返す
- メッセージと選択されたレストランデータの両方を含む
- サービス内で自動的に訪問を追跡

### ツール4: 訪問統計を取得
**目的**: レストラン訪問に関する書式設定済み統計を取得
**パラメーター**: なし
**実装**:

```csharp
[McpServerTool, Description("各レストランが訪問された回数の統計を取得します。")]
public async Task<string> GetVisitStatistics()
{
    var formattedStats = await restaurantService.GetFormattedVisitStatsAsync();
    return JsonSerializer.Serialize(formattedStats, RestaurantContext.Default.FormattedRestaurantStats);
}
```

**重要なポイント**:
- ユーザーフレンドリーな出力のために書式設定済み統計メソッドを使用
- 訪問数と合計を含む包括的な統計を返す
- サービス内の事前構築された書式設定ロジックを活用

### Program.csでのツールとサービスの登録
最後に、`Program.cs`ファイルでツールとサービスを登録する必要があります。`Program.cs`を以下を含むように更新します：

```csharp
builder.Services.AddSingleton<RestaurantService>();
```

サーバー設定にツール登録行を追加：

```csharp
builder.Services
    .AddMcpServer()
    .WithStdioServerTransport()
    .WithTools<RestaurantTools>();
```

完成したファイルは以下のようになります：

```csharp
using LunchTimeMCP;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using ModelContextProtocol.Server;

var builder = Host.CreateEmptyApplicationBuilder(settings: null);

builder.Services
    .AddMcpServer()
    .WithStdioServerTransport()
    .WithTools<RestaurantTools>();

builder.Services.AddSingleton<RestaurantService>();

await builder.Build().RunAsync();
```

## 3.4 MCPサーバーのテスト

### MCP Inspectorによるローカルテスト

**ステップ1: プロジェクトに移動**
```bash
cd LunchTimeMCP
```

**ステップ2: サーバーのビルドとテスト**
```bash
dotnet build
```

**ステップ3: MCP Inspectorのインストールと実行**

**npmでMCP Inspectorをインストール**
```bash
npm install -g @modelcontextprotocol/inspector
```

**npxでMCP Inspectorを実行**
```bash
npx @modelcontextprotocol/inspector dotnet run
```

実行すると、ブラウザで開くための事前入力されたトークン付きURLが提供されます。

**ステップ4: MCPサーバーに接続**
1. MCP Inspector Webインターフェースで：
   - **Transport Type**: "STDIO"を選択
   - **Command**: `dotnet`
   - **Arguments**: `run`
2. "Connect"をクリックして接続を確立
3. **List Tools**をクリック
3. インスペクターに4つのツールが表示されるはずです：
   - `GetRestaurants`
   - `AddRestaurant` 
   - `PickRandomRestaurant`
   - `GetVisitStatistics`

**ステップ5: 各ツールのテスト**
- **GetRestaurants**: シードされた10のレストランを返すはず
- **AddRestaurant**: 名前、場所、料理タイプで新しいレストランを追加してみる
- **PickRandomRestaurant**: ランダムなレストランを選択して追跡するはず
- **GetVisitStatistics**: 訪問されたレストランのアクセス数を表示するはず

**トラブルシューティング**:
- 正しいプロジェクトディレクトリにいることを確認
- MCP Inspectorを使用する前に`dotnet run`が単独で動作することを確認
- エラーメッセージがないかコンソール出力を確認
- stdio transportが選択されていることを確認（TCPや他のオプションではない）

### AIアシスタントとの統合

#### VS CodeとGitHub Copilotの設定

MCPサーバーをVS CodeとGitHub Copilotと統合するには、VS Codeにサーバーへの接続方法を伝える設定ファイルを作成する必要があります。

**ステップ1: MCP設定ファイルの作成**

ワークスペースルートに`.vscode`フォルダーを作成し（存在しない場合）、`mcp.json`ファイルを追加：

```json
{
    "inputs": [],
    "servers": {
        "lunchroulette": {
            "type": "stdio",
            "command": "dotnet",
            "args": [
                "run",
                "--project",
                "PATH_TO_YOUR_PROJECT\\LunchTimeMCP.csproj"
            ],
            "env": {}
        }
    }
}
```

**重要**: `args`配列のプロジェクトパスを実際のプロジェクトの場所に合わせて更新してください。パスは`.csproj`ファイルを指している必要があります。VS Codeでプロジェクトを右クリックして"Copy Path"を選択すると正しいパスを取得できます。Windowsではパスに二重バックスラッシュ（`\\`）、Unix系システムでは単一フォワードスラッシュ（`/`）を使用してください。

**ステップ2: 設定の詳細**

- **`lunchroulette`**: MCPサーバーの一意識別子
- **`type`**: 標準入出力通信のために"stdio"に設定
- **`command`**: 実行可能コマンド（`dotnet`）
- **`args`**: プロジェクトパスを含む、コマンドに渡される引数
- **`env`**: 環境変数（デフォルト環境の場合は空オブジェクト）

**ステップ3: 設定の確認**

1. **mcp.json**ファイルで、サーバー名の上に**Start**ボタンが利用可能になります。
2. **Start**ボタンをクリックしてMCPサーバーを起動します。
3. すべてが正しく設定されていれば、MCPサーバーが開始され、GitHub Copilot chatからのコマンドを受け入れる準備ができます。
4. VS CodeでGitHub Copilot chatを開き、MCPサーバーを認識することを確認します。
5. エージェントモードでツールアイコンをクリックして、利用可能なすべてのツールを確認します。

**ステップ4: VS CodeでのMCPサーバーの使用**

設定が完了すると、GitHub Copilot chat経由でランチルーレットサーバーと対話できます：

- **"ランチ用にランダムなレストランを選んで"**
- **"Beverly HillsのSpagというカリフォルニア料理の新しいレストランを追加して"**
- **"利用可能なすべてのレストランを表示して"**
- **"レストラン訪問の統計を取得して"**

チャットで`#GetRestaurants`や`#PickRandomRestaurant`のように`#`プレフィックスを使ってツールを直接呼び出すこともできます。

---

> **おめでとうございます！** C#でのMCPチュートリアルシリーズを完了しました。既存のMCPサーバーの使用とゼロからの独自構築の両方の知識を得ました。

**戻る**: [チュートリアル概要](README.md)
