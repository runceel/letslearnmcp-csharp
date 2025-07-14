# C#でMCPを学ぼう - チュートリアルシリーズ

C#開発者向けのModel Context Protocol（MCP）サーバーの理解と構築に関する包括的なガイド。

## 何を構築するか

このチュートリアルシリーズを完了すると、以下が完成します：

1. **🐒 MyMonkey アプリ** - 既存のMCPサーバーを使用してサルのデータを管理し、GitHubイシューを作成する楽しいコンソールアプリケーション
2. **🍽️ Lunch Roulette MCPサーバー** - AIアシスタントがレストランを選択し、食事の好みを追跡するのに役立つ独自のカスタムMCPサーバー

## チュートリアル構造

### [パート1: 前提条件とセットアップ](part1-setup.md)
**⏱️ 所要時間: 15-20分**

開発環境をセットアップし、MCPの基礎を理解します：
- VS Code、.NET 9 SDK、C# Dev Kitのインストール
- Model Context Protocolとは何か、なぜ重要なのかを学習
- クライアント・サーバーアーキテクチャの理解
- 開発環境の確認

**ここから始める**: [パート1: 前提条件とセットアップ →](part1-setup.md)

---

### [パート2: MCPサーバーの使用 - MyMonkey アプリ](part2-monkey-app.md)
**⏱️ 所要時間: 15-30分**

サルをテーマにしたアプリケーションを構築して、既存のMCPサーバーの使用方法を学習します：
- MonkeyMCPとGitHub MCPサーバーの設定
- サルのデータ用のC#モデルの作成
- ASCIIアートを使ったインタラクティブなコンソールアプリケーションの構築
- GitHubとの統合によるプログラム的なイシュー作成
- AIアシスタントがMCPサーバーとどのように相互作用するかの理解

**作成するもの**:
```
🐒 ランダムサル選択器！
========================
名前: テングザル
場所: ボルネオ
個体数: 15,000
詳細: 特徴的な大きな鼻で知られる...
```

**続きはこちら**: [パート2: MyMonkey アプリ →](part2-monkey-app.md)

---

### [パート3: 独自のMCPサーバーの構築 - Lunch Roulette](part3-lunch-server.md)
**⏱️ 所要時間: 30-45分**

ゼロから完全なMCPサーバーを構築します：
- JSON永続化を使用したレストラン管理サービスの作成
- AIアシスタント統合用の4つのMCPツールの実装
- 依存性注入と非同期操作の処理
- サーバーのテストとデプロイ
- MCPプロトコル実装の詳細の理解

**構築するもの**:
- `GetRestaurants()` - 利用可能なすべてのレストランをリスト
- `AddRestaurant()` - 新しい食事オプションを追加
- `PickRandomRestaurant()` - AI活用のランチ選択
- `GetVisitStatistics()` - 食事パターンの追跡

**続きはこちら**: [パート3: Lunch Roulette サーバー →](part3-lunch-server.md)

---

## クイックスタート

すぐに始めたい場合：

1. **前提条件**: VS Code、.NET 9 SDK、C# Dev Kitがインストールされていることを確認
2. **パスを選択**:
   - 🆕 **MCPが初めて？** [パート1: セットアップ](part1-setup.md)から始める
   - 🔧 **MCPを使いたい？** [パート2: MyMonkey アプリ](part2-monkey-app.md)にジャンプ
   - 🏗️ **構築準備完了？** [パート3: Lunch サーバー](part3-lunch-server.md)に進む

## 学習成果

このチュートリアルシリーズを完了すると、以下を理解できます：

- ✅ **MCPの基礎** - MCPとは何か、AIツール統合をどのように可能にするか
- ✅ **コンシューマーパターン** - アプリケーションで既存のMCPサーバーを使用する方法
- ✅ **サーバー開発** - 独自のMCPサーバーを構築・デプロイする方法
- ✅ **C#のベストプラクティス** - 依存性注入、非同期パターン、JSONシリアル化
- ✅ **AI統合** - AIアシスタントがあなたのツールやサービスと相互作用する方法

## 技術スタック

このチュートリアルでは、最新のC#開発プラクティスを使用します：

- **.NET 9** - パフォーマンス向上を伴う最新の.NETランタイム
- **C# 13** - 最新の言語機能と構文
- **System.Text.Json** - ソースジェネレーターを使用した高性能JSONシリアル化
- **Microsoft.Extensions.Hosting** - 依存性注入とアプリケーションライフサイクル
- **ModelContextProtocol** - MCP サーバー開発用の公式C# SDK

## リポジトリ構造

```
letslearnmcp-csharp/
├── README.md                 # この概要
├── part1-setup.md           # 前提条件と環境セットアップ
├── part2-monkey-app.md      # 既存のMCPサーバーの使用
└── part3-lunch-server.md    # 独自のMCPサーバーの構築
```

## 追加リソース

- 📖 [MCP公式ドキュメント](https://modelcontextprotocol.io/)
- 🛠️ [C# MCP SDKリポジトリ](https://github.com/modelcontextprotocol/csharp-sdk)
- 🐒 [MyMonkey アプリリポジトリ](https://github.com/jamesmontemagno/MyMonkeyAppMCP)
- 🍽️ [Lunch Rouletteリポジトリ](https://github.com/jamesmontemagno/LunchRouletteMCP)

## 貢献

このチュートリアルはオープンソースです！お気軽に：
- 🐛 改善と修正を提出
- 💡 より多くの例とユースケースを追加
- 🤝 独自のMCPサーバー実装を共有
- 💬 ディスカッションで他の人を支援

---

**始める準備はできましたか？** [パート1: 前提条件とセットアップ →](part1-setup.md)から始めましょう

*楽しいコーディングを！ 🐒🍽️*
