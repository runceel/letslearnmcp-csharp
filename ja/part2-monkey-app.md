# パート2: MCPサーバーの使用 - MyMonkey アプリの構築

> **ナビゲーション**: [← パート1: セットアップ](part1-setup.md) | [次: パート3 - Lunch サーバー →](part3-lunch-server.md)

> **ベース**: [MyMonkeyAppMCP リポジトリ](https://github.com/jamesmontemagno/MyMonkeyAppMCP)

## 概要
このセクションでは、GitHubと統合する楽しいサルをテーマにしたアプリケーションを構築することで、既存のMCPサーバーの使用方法を学習します。この実践チュートリアルでは、独自のMCPサーバーを構築する前に、コンシューマーの観点からMCPサーバーがどのように動作するかを教えます。

## ステップバイステップの説明

### 2.1 プロジェクトのセットアップ

#### 新しい.NETプロジェクトの作成
- **MonkeyApp**という新しいフォルダーを作成し、そのフォルダ内に**MyMonkeyApp**という名前の新しいC#コンソールアプリを作成します。
-- または、**MonkeyApp**という名前の新しいフォルダーでターミナルで以下のコマンドを実行します：

```bash
dotnet new console -n MyMonkeyApp
```

- リポジトリから不要なファイルを除外するために、以下のコマンドを実行して新しい.gitignoreファイルを追加します：

```bash 
dotnet new gitignore
```

#### GitHubにプッシュ
- 新しいGitHubリポジトリを作成
- ローカルプロジェクトに接続
- 最初のコミットを作成
- これはVS CodeのSource Controlパネルから実行できます
- Copilotがアクセスできるよう、リポジトリがパブリックであることを確認

### 2.2 MCPサーバーの設定と初期セットアップ

#### MonkeyMCPの設定
- VS CodeでMonkeyMCPサーバーをインストールして設定
- MonkeyMCP APIエンドポイントの理解
- MCPサーバー接続のテスト
- Copilot経由でサルのデータを取得できることを確認

**.vscode**という新しいフォルダーを作成し、以下の内容で**mcp.json**というファイルを作成します：

```json
{
    "inputs": [],
    "servers": {
        "monkeymcp": {
            "command": "docker",
            "args": [
                "run",
                "-i",
                "--rm",
                "jamesmontemagno/monkeymcp"
            ],
            "env": {}
        }
    }
}
```

#### GitHub MCPの設定
- VS CodeでGitHub MCPサーバーをインストール
- GitHub認証トークンの設定
- リポジトリアクセス権限の設定
- 上記のイシューを作成してGitHub MCP機能を確認

**mcp.json**を新しいサーバーで更新します。今度はGitHub用のリモートサーバーです：

```json
"github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    }
```

最終的な**mcp.json**は次のようになります：

```json
{
    "inputs": [],
    "servers": {
        "monkeymcp": {
            "command": "docker",
            "args": [
                "run",
                "-i",
                "--rm",
                "jamesmontemagno/monkeymcp"
            ],
            "env": {}
        },
        "github": {
            "type": "http",
            "url": "https://api.githubcopilot.com/mcp/"
        }
    }
}
```

GitHub MCPサーバーが実行されると、GitHubアカウントでの認証を求められます。指示に従って認証プロセスを完了してください。

#### Copilot Instructions の設定
リポジトリを作成し、初期コミットをプッシュした後、プロジェクト固有のコンテキストでCopilotを設定しましょう。

**タスク**: `.github/copilot-instructions.md`ファイルの作成
**手順**:
1. リポジトリルートに`.github`フォルダーを作成
2. 以下の内容で`copilot-instructions.md`ファイルを追加：

```markdown
このプロジェクトは.NET 9を使用し、C# 13を使用しています。

生成されるすべてのコードは、メインフォルダーのサブフォルダーである可能性があるMyMonkeyAppプロジェクト内に確実に配置してください。

これは https://github.com/YOUR_USERNAME/YOUR_REPO_NAME のGitHubにあります

## プロジェクトコンテキスト
これはサルの種データを管理し、MCPサーバー経由でGitHubと統合するコンソールアプリケーションです。

## C#コーディング標準
- クラス名、メソッド名、プロパティにはPascalCaseを使用
- ローカル変数とパラメーターにはcamelCaseを使用
- 目的を明確に示す説明的な名前を使用
- パブリックメソッドとクラスにはXMLドキュメントコメントを追加
- 型が明らかな場合はローカル変数に`var`を使用
- 可読性が向上する場合は明示的な型を使用
- 非同期操作にはasync/awaitを使用
- データアクセスにはリポジトリパターンに従う
- try-catchブロックで適切な例外処理を使用
- リソース管理時にはIDisposableを実装
- null参照例外を避けるためにnull許容参照型を使用
- よりクリーンなコード構成のためにファイルスコープの名前空間を使用

## 命名規則
- クラス: `MonkeyHelper`、`Monkey`、`Program`
- メソッド: `GetMonkeys()`、`GetRandomMonkey()`、`GetMonkeyByName()`
- プロパティ: `Name`、`Location`、`Population`
- 変数: `selectedMonkey`、`monkeyCount`、`userInput`
- 定数: `MAX_MONKEYS`、`DEFAULT_POPULATION`

## アーキテクチャ
- インタラクティブメニュー付きコンソールアプリケーション
- データ管理用静的ヘルパークラス
- データ表現用モデルクラス
- UIとビジネスロジックの関心の分離
```

**重要**: `YOUR_USERNAME`と`YOUR_REPO_NAME`を実際のGitHubユーザー名とリポジトリ名に置き換えてください。

#### プロジェクト計画用のGitHubイシューの作成
コーディングを始める前に、GitHub MCPサーバーを使用してプロジェクト目標を概説するイシューを作成しましょう。

**タスク**: GitHub MCPでVS Codeを使用して新しいイシューを作成
**サンプルCopilotコマンド**:
> "私のリポジトリに「Monkey Console Applicationの実装」というタイトルで新しいGitHubイシューを作成してください。以下の要件を含める：利用可能なすべてのサルをリストし、名前で特定のサルの詳細を取得し、ランダムなサルを選択できるC#コンソールアプリを作成する。アプリはMonkeyモデルクラスを使用し、視覚的魅力のためにASCIIアートを含める必要があります。'enhancement'や'good first issue'などの適切なラベルを追加してください。実装方法の詳細と必要な作業のチェックリストを追加してください。"

**作成される可能性のあるイシューサンプル内容**:
```markdown
# Monkey Console Applicationの実装

## 説明
サルの種データを探索するための楽しくインタラクティブなコンソールアプリケーションを作成する。

## 要件
- [ ] Name、Location、Details、Image、Population、Latitude、Longitudeプロパティを持つ`Monkey.cs`モデルクラスの作成
- [ ] データ管理用の`MonkeyHelper.cs`静的クラスの実装
- [ ] 以下オプション付きインタラクティブコンソールメニューの構築：
  - すべてのサルをリスト
  - 特定のサルの詳細を取得
  - ランダムなサルを選択
  - アプリケーション終了
- [ ] 詳細表示時のASCIIサルアートの含有
- [ ] 各サルのアクセス数の追跡
- [ ] エラーケースの適切な処理

## 技術詳細
- .NET 9 コンソールアプリケーション
- C# 13機能
- C#命名規則に従う
- XMLドキュメントコメントを含む
```

### 2.3 Copilotとのインタラクティブ開発

#### サルデータの取得
- **タスク**: CopilotにMonkeyMCPからサルのリストを取得するよう依頼
- CopilotがMCPサーバーとどのように統合するかを学習
- 返されるデータ構造の理解
- APIレスポンスとエラーケースの処理

#### サルモデルの生成
- **タスク**: CopilotにMonkeyMCPからのサルデータに基づいてC#モデルを作成するよう依頼
- **期待される出力**: 以下の構造を持つ`Monkey.cs`クラスファイル：

```csharp
namespace MonkeyApp
{
    public class Monkey
    {
        public string Name { get; set; }
        public string Location { get; set; }
        public string Details { get; set; }
        public string Image { get; set; }
        public int Population { get; set; }
        public double Latitude { get; set; }
        public double Longitude { get; set; }
    }
}
```

**主要プロパティの説明**:
- `Name` - サルの種名（例："テングザル"）
- `Location` - 地理的生息地（例："ボルネオ"）
- `Details` - 種に関する説明情報
- `Image` - サルの画像のURLまたはパス
- `Population` - 推定個体数
- `Latitude` / `Longitude` - マッピング用の地理座標

#### ランダムサルアプリの構築
- **タスク**: コレクションからランダムなサルを選択するアプリケーションを作成
- **期待される機能**:
  - ランダムなサルを表示するコンソールアプリケーション
  - サルの詳細を表示（名前、場所、個体数など）
  - "別のサルを選ぶ"オプション
  - 地理座標の表示
  - 空のサルコレクションを適切に処理

**サンプル出力**:
```
🐒 ランダムサル選択器！
========================
名前: テングザル
場所: ボルネオ
個体数: 7,000
詳細: 特徴的な大きな鼻と赤い毛で知られる...
座標: 1.5°N, 110.0°E

別のサルを選ぶには'R'、終了するには'Q'を押してください...
```

**実装ノート**:
- 選択に`Random`クラスを使用
- ユーザーインタラクションループを実装
- より良いUXのために絵文字と書式設定を追加
- サポートされている場合はサルの画像の表示を検討

#### MonkeyHelperサービスの作成
- **タスク**: Copilotにサルデータを管理する`MonkeyHelper.cs`静的クラスを作成するよう依頼
- **期待される出力**: 以下の機能を持つ静的ヘルパークラス：

**主要メソッド**:
- `GetMonkeys()` - 12種以上の異なるサルの種を含むList<Monkey>を返す
- `GetRandomMonkey()` - ランダムなサルを選択し、アクセスを追跡
- `GetMonkeyByName(string name)` - 名前で特定のサルを検索
- `GetMonkeyAccessCount(string name)` - サルがアクセスされた回数を返す

**サンプルCopilotコマンド**:
> "サルデータのコレクションを管理するMonkeyHelper静的クラスを作成してください。すべてのサルを取得、ランダムなサルを取得、名前で検索、アクセス数を追跡するメソッドを含めてください。サルデータには monkey mcp server からのサルを含めてください。"

**期待されるデータ**:
ヘルパーには以下のようなサルが含まれる必要があります：
- ヒヒ（アフリカ & アジア） - 個体数: 10,000
- オマキザル（中央・南アメリカ） - 個体数: 23,000
- ブルーモンキー（中央・東アフリカ） - 個体数: 12,000
- テングザル（ボルネオ） - 個体数: 15,000
- ニホンザル（日本） - 個体数: 1,000
- その他現実的なデータを持つ7種以上

#### コンソールアプリケーションの構築
- **タスク**: Copilotにインタラクティブメニューシステムを持つメイン`Program.cs`を作成するよう依頼
- **期待される出力**: 以下のメニュー構造を持つコンソールアプリケーション：

**サンプルCopilotコマンド**:
> "monkey appのインタラクティブメニューを表示するProgram.csファイルを更新してください。メニューには次のオプションが必要です：1) すべてのサルをリスト、2) 名前で特定のサルの詳細を取得、3) ランダムなサルを取得、4) 終了。サルの詳細を表示する際にサルの顔のASCIIアートを含めてください。MonkeyHelperクラスのメソッドを使用し、各サルのアクセス数を表示してください。"

**期待されるコンソール出力**:
```
Monkey App メニュー:
1. すべてのサルをリスト
2. 特定のサルの詳細を取得
3. ランダムなサルを取得
4. 終了
オプションを選択してください: 3

  //\ 
 (o o)
(  V  )
--m-m--

ランダムサル: テングザル
場所: ボルネオ
詳細: テングザルまたは長鼻サル、マレー語でbekantan、東南アジアのボルネオ島固有の赤茶色の樹上性旧世界サル。
個体数: 15000
緯度/経度: 0.961883, 114.55485
画像: https://raw.githubusercontent.com/jamesmontemagno/app-monkeys/master/borneo.jpg
アクセス回数: 1 回
```

**主要機能**:
- ユーザーが終了するまでのインタラクティブメニューループ
- 視覚的魅力のためのASCIIサルアート
- 詳細なサル情報の表示
- アクセス追跡と表示
- 無効な入力と見つからないサルのエラーハンドリング

### 2.4 GitHub統合

#### GitHubイシューの作成
- **タスク**: Copilotを使用してGitHubイシューをプログラム的に作成
- リポジトリ名を指定
- "新しいサル機能を追加"のイシューを作成するか、いくつかの新しいアイデアを作成
- MCP経由のGitHub API統合を理解

#### GitHub.comでの確認
- リポジトリに移動
- 自動作成されたイシューを確認
- イシューの詳細と書式設定を調査
- MCP → GitHubワークフローを理解

## 学習成果
- 既存のMCPサーバーの設定と使用方法
- AIアシスタントとの統合パターン
- MCP経由での実世界API相互作用
- GitHub自動化の可能性

---

> **次のステップ**: MCPサーバーの使用方法を学んだので、ゼロから独自のものを構築しましょう！

**続きはこちら**: [パート3 - 独自のMCPサーバーの構築 →](part3-lunch-server.md)
