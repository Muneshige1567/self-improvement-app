# 8. 技術要件

> **対象**: 最強社会人チェックリスト iOS アプリ
> **最小サポートOS**: iOS 17.0+
> **アーキテクチャ**: MVVM + Repository + DDD

---

## 8.1 技術スタック

| レイヤー | 技術 | 選定理由 |
|---|---|---|
| **UI** | SwiftUI (iOS 17.0+) | 宣言的UI、SwiftData との統合、@Observable マクロ対応 |
| **データ永続化** | SwiftData | ローカルファースト。オフライン完全動作。Core Data 後継で API がシンプル |
| **チャート** | Canvas + Path（カスタム実装） | Swift Charts はレーダーチャート（Spider Chart）非対応。5角形チャートはカスタム Shape + Canvas で実装 |
| **通知** | UserNotifications | ローカル通知。リッチ通知: アクションボタン（Quick開始/Deep開始/後で）、キャラクターアイコン |
| **ウィジェット** | WidgetKit | ホーム画面ウィジェット + ロック画面ウィジェット。App Group 経由でデータ共有 |
| **AI** | Claude API (Anthropic) | フィードバック生成。キャラクターの口調でコンテキスト依存の助言を生成 |
| **ネットワーク** | URLSession | AI API 呼び出し専用。オフライン時はテンプレートベースにフォールバック |
| **APIキー管理** | Keychain Services | APIキーの暗号化保存。平文でのディスク保存は禁止 |
| **アーキテクチャ** | MVVM + Repository + DDD | SwiftUI との親和性 + データ層の抽象化 + ドメイン層の独立性 |

---

## 8.2 レイヤー構成

```
┌──────────────────────────────────┐
│           Presentation           │
│   SwiftUI Views + ViewModels     │
│   (@Observable, @Environment)    │
└────────────┬─────────────────────┘
             │
┌────────────▼─────────────────────┐
│          Domain Layer            │
│   Entities, Value Objects,       │
│   Domain Services, Events,       │
│   Repository Protocols           │
│   (UIフレームワーク非依存)         │
└────────────┬─────────────────────┘
             │
┌────────────▼─────────────────────┐
│        Infrastructure            │
│   SwiftData Repositories,        │
│   URLSession (Claude API),       │
│   Keychain, UserDefaults,        │
│   UserNotifications, WidgetKit   │
└──────────────────────────────────┘
```

### 型定義

```swift
// MARK: - ViewModel 基盤

@Observable
final class CheckInputViewModel {
    let dailyRecordRepository: DailyRecordRepository
    let feedbackEngine: FeedbackEngine
    let streakCalculator: StreakCalculator

    var currentDate: LogicalDate
    var scores: [CheckItemID: Int]              // 入力中のスコア
    var memos: [Category: String]               // 入力中のメモ
    var isSaving: Bool
}

@Observable
final class DashboardViewModel {
    let dailyRecordRepository: DailyRecordRepository

    var selectedPeriod: DashboardPeriod
    var categoryAverages: [Category: Float]
    var trends: [Category: Trend]
    var streakState: StreakState
}

enum DashboardPeriod {
    case week
    case month
    case threeMonths
    case sixMonths
    case year
}

// MARK: - Repository 実装

final class SwiftDataDailyRecordRepository: DailyRecordRepository {
    let modelContext: ModelContext
    // ... DailyRecordRepository プロトコルの実装
}

final class SwiftDataFeedbackRepository: FeedbackRepository {
    let modelContext: ModelContext
    // ... FeedbackRepository プロトコルの実装
}
```

---

## 8.3 パフォーマンス要件

| 項目 | 基準 | 実現方針 |
|---|---|---|
| **アプリ起動** | 2秒以内にチェック画面表示 | LazyVStack + 最小限の初期クエリ。ModelContext は非同期初期化 |
| **スコア入力** | タップから UI 反映まで 100ms 以内 | ViewModel のインメモリ操作。SwiftData 書き込みは入力完了時にバッチ実行 |
| **ダッシュボード描画** | 1年分データで 1秒以内 | DailyRecord の非正規化キャッシュ（`cachedCategoryAverages`, `cachedTotalAverage`）を使用。個別 Score への JOIN を回避 |
| **ストレージ** | 1年分のデータで 10MB 以下 | 見積り: Score 約450KB + Memo 約550KB + Cache 約100KB + SQLite OH 約2MB = 約3MB |

### パフォーマンス計測方針

- ダッシュボード描画の「1秒以内」は、キャッシュ利用時の計測値。初回起動時にキャッシュ未構築の場合、バックグラウンドで構築し、完了まではスケルトン UI を表示
- 計測ポイント: `signpost` API で ViewDidAppear 〜 データ描画完了を計測

### 非正規化キャッシュの設計

```swift
// DailyRecord 保存時に自動計算するキャッシュ値:
//
// cachedCategoryAverages:
//   入力済み Score を Category ごとにグループ化し、各グループの平均を算出。
//   例: {"perceive": 3.8, "think": 4.2, "execute": 3.0, "engage": 4.0, "sustain": 3.5}
//   カテゴリ内の全項目が未入力の場合、そのカテゴリのキーは含まない。
//
// cachedTotalAverage:
//   入力済み全 Score の平均。
//
// cachedInputCount:
//   入力済み Score の件数。
//
// ダッシュボードの集計クエリは DailyRecord のキャッシュフィールドのみを参照し、
// Score テーブルへの JOIN を回避する。
```

---

## 8.4 セキュリティ・プライバシー

| 項目 | 仕様 |
|---|---|
| **データ保存** | 端末ローカル（SwiftData / SQLite）。外部サーバーへの同期なし |
| **AI通信時の送信データ** | スコア数値 + カテゴリ名のみ。メモ本文は送信しない |
| **APIキー** | Keychain Services で暗号化保存。平文でのディスク/UserDefaults 保存は禁止 |
| **バックアップ** | iCloud バックアップに含まれる（iOS 標準動作） |
| **認証** | なし（個人端末前提）。v2.0 で Face ID / Touch ID ロックをオプション追加可能 |
| **分析・トラッキング** | 一切なし。外部 SDK 不使用 |
| **ネットワーク通信** | Claude API へのリクエスト時のみ。TLS 1.2+ 必須 |

### AI 通信のデータポリシー

```swift
// Claude API に送信するコンテキスト（許可されたデータのみ）:
struct AIRequestContext {
    let categoryAverages: [String: Float]   // {"perceive": 3.8, ...}
    let totalAverage: Float
    let streakDays: Int
    let daysUsed: Int
    let trendDirections: [String: String]   // {"perceive": "rising", ...}
    // メモ本文（CategoryMemo.text）は含めない
}
```

---

## 8.5 オフライン・フォールバック戦略

| 状態 | 挙動 |
|---|---|
| **完全オフライン** | チェック入力・ダッシュボード・ストリークは全て動作。フィードバックはテンプレートベースで生成 |
| **API キー未設定** | テンプレートベースで生成。設定画面に「AI フィードバックを有効にする」案内を表示 |
| **API エラー（タイムアウト・5xx）** | テンプレートベースにフォールバック。リトライは行わない（次回入力時に再試行） |
| **API レート制限（429）** | テンプレートベースにフォールバック。次回入力まで待機 |

---

## 8.6 フォントフォールバック

### 武将セリフ（明朝体）

```swift
// 優先順位:
// 1. YuMincho Demibold  — iOS 標準搭載。第一候補
// 2. YuMincho Medium    — Demibold が利用不可の場合
// 3. HiraMinProN-W6     — ヒラギノ明朝 Pro N W6
// 4. .system(.serif)    — システムセリフ体（最終フォールバック）

// SwiftUI での実装:
// Font.custom("YuMincho-Demibold", size: 17)
//   ?? Font.custom("YuMincho-Medium", size: 17)
//   ?? Font.custom("HiraMinProN-W6", size: 17)
//   ?? Font.system(size: 17, design: .serif)
```

### UI テキスト（ゴシック体）

```swift
// 優先順位:
// 1. ヒラギノ角ゴシック（HiraginoSans-W3 / W6）— iOS 標準搭載
// 2. .system(.default)  — システムデフォルト（San Francisco）

// 見出し: HiraginoSans-W6, 20pt
// 本文:   HiraginoSans-W3, 15pt
// キャプション: HiraginoSans-W3, 12pt
```

### スコア数値

```swift
// SF Pro Display Bold, 36pt
// iOS 標準搭載のため フォールバック不要
// Font.system(size: 36, weight: .bold, design: .default)
```

---

## 8.7 通知アーキテクチャ

### 通知カテゴリとアクション

```swift
// MARK: - 通知カテゴリ定義

enum NotificationCategoryID: String {
    case dailyReminder = "dailyReminder"   // 夜のリマインド
    case feedback      = "feedback"        // 即時・翌朝フィードバック
    case milestone     = "milestone"       // マイルストーン祝福
    case morning       = "morning"         // 朝の檄文
    case noon          = "noon"            // 昼の陣中見舞い
}

enum NotificationActionID: String {
    case startQuick = "startQuick"         // 「Quick開始」
    case startDeep  = "startDeep"          // 「Deep開始」
    case later      = "later"              // 「後で」
}

// dailyReminder カテゴリのアクション:
//   - startQuick: Quick チェック画面を直接開く
//   - startDeep:  Deep チェック画面を直接開く
//   - later:      通知を閉じる（何もしない）
```

### 通知スケジュール

| 通知 | デフォルト時刻 | トリガー | サムネイル |
|---|---|---|---|
| 朝の檄文 | 07:00 | `UNCalendarNotificationTrigger` | 道雪アイコン |
| 陣中見舞い | 12:30 | `UNCalendarNotificationTrigger` | 家臣アイコン |
| 夕べの下知 | 22:00 | `UNCalendarNotificationTrigger` | 誾千代アイコン |
| 即時講評 | チェック完了直後 | アプリ内表示（通知不要） | -- |
| 翌朝総括 | 翌07:00 | `UNCalendarNotificationTrigger` | 紹運アイコン |
| マイルストーン | ストリーク到達時 | `UNTimeIntervalNotificationTrigger` | 全員アイコン |

全通知時刻はユーザー設定で変更可能。各通知は個別にオン/オフ設定可能。

### ディープリンク設計

```swift
// 通知タップ時の遷移先を URL スキームで定義
enum DeepLink {
    case checkInput(mode: CheckMode?)   // checklist://check?mode=quick
    case dailySummary(date: LogicalDate) // checklist://summary?date=2026-02-13
    case weeklySummary                   // checklist://weekly
    case dashboard                       // checklist://dashboard
}

enum CheckMode: String {
    case quick
    case deep
}
```

---

## 8.8 WidgetKit 連携

### App Group によるデータ共有

```swift
// App Group ID: group.com.shibu.checklist
//
// Widget が参照するデータ:
//   - 当日の cachedTotalAverage
//   - 当日の cachedCategoryAverages
//   - StreakState（UserDefaults キャッシュ）
//   - 最新の RenderedMessage（notification チャネル用テキスト）
//
// データ共有方式:
//   - SwiftData の ModelContainer を App Group の共有ディレクトリに配置
//   - StreakState と最新フィードバックは UserDefaults(suiteName: appGroupID) に保存
```

### ウィジェットの種類

| ウィジェット | サイズ | 表示内容 |
|---|---|---|
| **スコアウィジェット** | Small / Medium | 当日の総合平均 + トレンド矢印（↑↓→）+ 武将の一言 |
| **ストリークウィジェット** | Small | 連続日数 + 松明アイコン |
| **カテゴリウィジェット** | Medium | 5カテゴリの当日スコアバー |
| **ロック画面ウィジェット** | Inline / Circular | 連続日数 or 総合スコア |

---

## 8.9 AI フィードバック生成（Claude API）

### リクエスト・レスポンス設計

```swift
// MARK: - Claude API 連携

struct ClaudeAPIClient {
    let baseURL: URL         // https://api.anthropic.com/v1/messages
    let session: URLSession
    let keychainService: KeychainService

    /// フィードバックメッセージを生成
    func generateFeedback(
        context: AIRequestContext,
        feedbackType: FeedbackType,
        character: CharacterID,
        channel: Channel
    ) async throws -> String
}

// プロンプト構成:
//   - System: キャラクター設定（口調・性格・呼びかけ）
//   - User: スコアデータ + フィードバックタイプ + チャネル制約（文字数等）
//
// レスポンス: 生成されたフィードバックテキスト（String）
//
// タイムアウト: 10秒。超過時はテンプレートフォールバック。
```

### コスト管理

```
- 1回のフィードバック生成: 約500-1000 tokens（入力 + 出力）
- 1日あたりの最大 API コール: 3回（即時 + 翌朝 + 週次）
- 月間見積り: 約90コール × 750 tokens = 約67,500 tokens/月
- Claude API のコストは個人利用レベルで十分に低い
```

---

## 8.10 テスト戦略

| テスト種別 | 対象 | ツール |
|---|---|---|
| **単体テスト** | FeedbackEngine の判定ロジック、移動平均計算、StreakCalculator、LogicalDate | XCTest |
| **コールドスタートテスト** | 1日目/7日目/30日目の各段階でのフィードバック検証 | XCTest |
| **エッジケーステスト** | 全項目未入力、1項目のみ入力、04:00境界、同日再入力 | XCTest |
| **スナップショットテスト** | レーダーチャート、フィードバック表示、ウィジェット | Swift Snapshot Testing |
| **パフォーマンステスト** | ダッシュボード描画（1年分データ）| XCTest + measure {} |
| **UI テスト** | チェック入力フロー、モード切り替え | XCUITest |

### テスト用の依存注入

```swift
// Repository プロトコルにより、テスト時はモックを注入可能。
// 例:
final class MockDailyRecordRepository: DailyRecordRepository {
    var records: [DailyRecord] = []
    // ... テスト用の実装
}
```

---

## 8.11 ビルド・デプロイ

| 項目 | 仕様 |
|---|---|
| **開発言語** | Swift 5.9+ |
| **最小デプロイターゲット** | iOS 17.0 |
| **Xcode** | 15.0+ |
| **パッケージマネージャー** | Swift Package Manager |
| **コード署名** | Personal Team（個人利用。App Store 公開は想定しないが将来の公開は排除しない） |
| **CI/CD** | なし（個人開発。必要に応じて GitHub Actions を追加） |
| **外部依存** | 最小限。標準フレームワークのみで構成。AI 連携は URLSession で直接実装 |
