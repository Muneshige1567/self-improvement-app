# 2. DDDドメインモデル

> **対象**: 最強社会人チェックリスト iOS アプリ
> **ドメイン設計方針**: MVVM + Repository + DDD。ドメイン層はUIフレームワークに依存しない。

---

## 2.1 境界づけられたコンテキスト（Bounded Contexts）

```
┌─────────────────┐    ┌─────────────────┐
│  Check Context  │───→│ Feedback Context │
│  (チェック入力)  │    │  (フィードバック) │
└────────┬────────┘    └────────┬────────┘
         │                      │
         │                      ↓
         │             ┌─────────────────┐
         │             │Character Context │
         │             │(キャラクター表現) │
         │             └────────┬────────┘
         │                      │
         ↓                      ↓
┌──────────────────────────────────────┐
│       Notification Context           │
│  (通知スケジュール・WidgetKit連携)     │
└──────────────────────────────────────┘
```

### Check Context（チェック入力）

責務: デイリーチェックの入力・保存・日次集計。

- `DailyRecord` 集約が中心
- スコアの UPSERT（同日再入力は上書き）
- 論理日付（04:00 区切り）の管理
- カテゴリ別平均の非正規化キャッシュ計算

### Feedback Context（フィードバック生成）

責務: スコアデータからフィードバックタイプ・対象を判定し、`FeedbackResult` を生成する。

- `FeedbackEngine` がドメインサービスとして判定ロジックを実行
- 即時・翌朝・週次の3種のフィードバックを生成
- 閾値判定、トレンド計算、コールドスタート制御
- キャラクターの口調には依存しない（純粋なビジネスロジック）

### Character Context（キャラクター表現）

責務: `FeedbackResult` を受け取り、キャラクターの口調・テンプレートで最終メッセージを生成する。

- キャラクター別のテンプレート選択
- 武士道語彙への変換（カテゴリ → 眼・知・剣・人・道）
- チャネル別（notification / summary / detail）の情報量調整
- Feedback Context のビジネスロジック変更なしにキャラクター差し替え可能

### Notification Context（通知・Widget連携）

責務: 通知スケジュール管理、WidgetKit データ共有。

- UserNotifications によるローカル通知のスケジューリング
- リッチ通知（アクションボタン・キャラアイコン）
- WidgetKit の Timeline Provider へのデータ提供
- 通知タップ時のディープリンク生成

---

## 2.2 集約・エンティティ・値オブジェクト

### 値オブジェクト

```swift
// MARK: - 論理日付（04:00区切り）

/// 日付の区切りを04:00とする論理日付。
/// 例: 2/14 03:30 の入力 → LogicalDate は 2/13。
struct LogicalDate: Hashable, Comparable, Codable {
    let value: Date  // yyyy-MM-dd の日付部分のみ（時刻は00:00:00固定）

    /// 現在時刻から論理日付を算出（04:00区切り）
    static func now(calendar: Calendar = .current) -> LogicalDate

    /// 任意の Date から論理日付を算出
    static func from(_ date: Date, calendar: Calendar = .current) -> LogicalDate

    static func < (lhs: LogicalDate, rhs: LogicalDate) -> Bool
}

// MARK: - カテゴリ

enum Category: String, CaseIterable, Codable {
    case perceive
    case think
    case execute
    case engage
    case sustain

    /// 武士道呼称
    var bushidoLabel: String {
        switch self {
        case .perceive: return "眼"
        case .think:    return "知"
        case .execute:  return "剣"
        case .engage:   return "人"
        case .sustain:  return "道"
        }
    }

    /// 対応する武士道の徳
    var virtue: String {
        switch self {
        case .perceive: return "義"
        case .think:    return "誠"
        case .execute:  return "勇"
        case .engage:   return "仁"
        case .sustain:  return "名誉"
        }
    }

    /// カテゴリに属する項目ID一覧
    var itemIDs: [String] {
        switch self {
        case .perceive: return ["P1", "P2", "P3", "P4", "P5"]
        case .think:    return ["T1", "T2", "T3", "T4", "T5"]
        case .execute:  return ["E1", "E2", "E3", "E4", "E5"]
        case .engage:   return ["G1", "G2", "G3", "G4", "G5"]
        case .sustain:  return ["S1", "S2", "S3", "S4", "S5"]
        }
    }
}

// MARK: - チェック項目ID

/// 全25項目のID。Engage は "G"（"E" は Execute で使用済みのため、enGage の G を採用）。
enum CheckItemID: String, CaseIterable, Codable {
    case P1, P2, P3, P4, P5
    case T1, T2, T3, T4, T5
    case E1, E2, E3, E4, E5
    case G1, G2, G3, G4, G5
    case S1, S2, S3, S4, S5

    var category: Category {
        switch self {
        case .P1, .P2, .P3, .P4, .P5: return .perceive
        case .T1, .T2, .T3, .T4, .T5: return .think
        case .E1, .E2, .E3, .E4, .E5: return .execute
        case .G1, .G2, .G3, .G4, .G5: return .engage
        case .S1, .S2, .S3, .S4, .S5: return .sustain
        }
    }
}

// MARK: - スコアスナップショット

/// DailyRecord から算出される読み取り専用の集計値。
struct ScoreSnapshot {
    let totalAverage: Float            // 入力済み項目のみの平均（1.0-5.0）
    let categoryAverages: [Category: Float]  // カテゴリ別平均（入力済み項目のみ）
    let highestCategory: Category
    let lowestCategory: Category
    let highestItem: (id: CheckItemID, value: Int)
    let lowestItem: (id: CheckItemID, value: Int)
    let inputRate: Float               // 入力率 = 入力済み項目数 / 25
    let inputCount: Int                // 入力済み項目数
}

// MARK: - トレンド

struct Trend {
    let direction: TrendDirection
    let delta: Float                   // 変化量（例: +0.3, -0.5）
    let period: TrendPeriod
    let comparedTo: Float              // 比較対象の平均値
}

enum TrendDirection {
    case rising    // delta >= +0.3
    case stable    // -0.3 < delta < +0.3
    case declining // delta <= -0.3
}

enum TrendPeriod {
    case week
    case month
    case allTime
}

// MARK: - ストリーク

struct StreakState {
    let currentStreak: Int             // 現在の連続記録日数
    let longestStreak: Int             // 過去最長
    let lastRecordedDate: LogicalDate? // 最終記録の論理日付
    let status: StreakStatus
    let milestone: Milestone?          // 到達マイルストーン（あれば）
}

enum StreakStatus {
    case active        // 継続中
    case broken        // 途切れた
    case newRecord     // 過去最長を更新中
}

struct Milestone: Equatable {
    let days: Int
    let name: MilestoneType
}

enum MilestoneType: Int, CaseIterable {
    case weekComplete  = 7     // 初陣（七日の修行）
    case monthComplete = 30    // 立花山城（三十日の鍛錬）
    case habitFormed   = 66    // 西国無双（六十六日の道）— Lally (2010) 中央値
    case centurion     = 100   // 碧蹄館（百日の誓い）
    case biCenturion   = 200   // 柳川帰還（二百日の境地）
    case yearComplete  = 365   // 天下無双（一年の大成）
}

// MARK: - フィードバック関連

enum FeedbackType: String, Codable {
    case praise     // 称賛（担当: 道雪）
    case rebuke     // 叱咤（担当: 誾千代）
    case encourage  // 激励（担当: 家臣団）
    case advise     // 助言（担当: 道雪）
    case celebrate  // 祝福（担当: 全員）
    case warn       // 警告（担当: 紹運）
}

enum Channel: String, CaseIterable, Codable {
    case notification  // プッシュ通知（40文字以内）
    case widget        // ホーム画面ウィジェット（スコア + 1行）
    case summary       // アプリ内サマリー（全体評価 + カテゴリ + 助言1つ）
    case detail        // アプリ内詳細分析（全カテゴリ分析 + グラフ + 推奨）
}

/// フィードバック判定の入力コンテキスト
struct FeedbackContext {
    let today: ScoreSnapshot
    let avg7d: ScoreSnapshot?               // 過去7日平均（データ不足時 nil）
    let avg30d: ScoreSnapshot?              // 過去30日平均（データ不足時 nil）
    let streak: StreakState
    let daysUsed: Int                       // 総利用日数
    let recentTrends: [Category: Trend]     // カテゴリ別7日トレンド
    let isFirstRecordEver: Bool             // 初回記録かどうか
}

/// フィードバック判定結果（ビジネスロジック層の出力）
struct FeedbackResult {
    let type: FeedbackType
    let target: Category?                   // 対象カテゴリ（全体向けは nil）
    let context: FeedbackContext
    let triggeredPriority: Int              // どの Priority で発火したか（デバッグ用）
}

/// チャネルに投影された最終出力
struct RenderedMessage {
    let text: String                        // 最終表示テキスト（キャラクター口調適用済み）
    let tone: Tone
    let categoryLabel: String?              // 「眼」「知」等
    let virtueLabel: String?                // 「義」「勇」等
    let character: CharacterID              // どのキャラクターが発話したか
}

enum Tone {
    case solemn    // 厳粛（rebuke, warn）
    case warm      // 温かい（praise, encourage）
    case radiant   // 輝かしい（celebrate）
    case guiding   // 導く（advise）
}

enum CharacterID: String, CaseIterable {
    case dosetsu   // 立花道雪（養父）— praise, advise 担当
    case joun      // 高橋紹運（実父）— warn 担当
    case ginchiyo  // 誾千代（妻）— rebuke 担当
    case retainers // 家臣団（由布惟信 等）— encourage 担当
}
```

### エンティティ・集約ルート（SwiftData）

```swift
import SwiftData

// MARK: - DailyRecord 集約ルート

/// 1日分のチェック記録。集約ルート。
/// date は LogicalDate（04:00区切り）で、ユニーク制約。
@Model
final class DailyRecord {
    @Attribute(.unique)
    var date: Date                            // LogicalDate.value を格納

    var createdAt: Date
    var updatedAt: Date

    // --- 非正規化キャッシュ（スコア入力時に自動計算） ---
    var cachedCategoryAverages: [String: Float]  // {"perceive": 3.8, "think": 4.2, ...}
    var cachedTotalAverage: Float                 // 入力済み項目のみの全体平均
    var cachedInputCount: Int                     // 入力済み項目数

    // --- リレーション ---
    @Relationship(deleteRule: .cascade, inverse: \Score.dailyRecord)
    var scores: [Score]

    @Relationship(deleteRule: .cascade, inverse: \CategoryMemo.dailyRecord)
    var memos: [CategoryMemo]

    // --- Computed ---

    /// Quick/Deep の判定。メモの有無で決定する（mode フィールドは廃止）。
    /// メモが1件以上存在すれば Deep、なければ Quick。
    var isDeepMode: Bool {
        !memos.isEmpty && memos.contains(where: { !$0.text.isEmpty })
    }

    /// 記録済みかどうか（ストリーク判定用）。Score が1件以上あれば記録済み。
    var isRecorded: Bool {
        !scores.isEmpty
    }
}

/// 個別項目のスコア。
@Model
final class Score {
    var itemId: String                  // "P1", "P2", ..., "S5"
    var value: Int                      // 1-5
    var dailyRecord: DailyRecord?
}

/// カテゴリ別メモ（Deep モード時に記入）。
@Model
final class CategoryMemo {
    var category: String                // "perceive", "think", ..., "sustain"
    var text: String
    var dailyRecord: DailyRecord?
}

// MARK: - FeedbackCache

/// 生成済みフィードバックのキャッシュ。同一コンテキストでの再生成を防ぐ。
@Model
final class FeedbackCache {
    @Attribute(.unique)
    var date: Date                      // LogicalDate.value

    var feedbackType: String            // FeedbackType.rawValue
    var targetCategory: String?         // Category.rawValue（nilable）
    var triggeredPriority: Int
    var renderedNotification: String?   // notification チャネルの生成済みテキスト
    var renderedSummary: String?        // summary チャネルの生成済みテキスト
    var createdAt: Date
}
```

### 静的データ（SwiftData 永続化不要）

```swift
// MARK: - CheckItem（マスターデータ / 静的配列）

/// チェック項目の定義。コード内に静的配列として保持。
struct CheckItem {
    let id: CheckItemID
    let category: Category
    let title: String                   // 「一次ソースに当たる」
    let question: String                // 「二次情報や伝聞で済ませず...」
    let evidenceSources: [String]       // ["McKinsey fact-based", "Bridgewater Radical Truth"]
    let evidenceSummary: String
    let sortOrder: Int                  // 表示順（1-25）
}

// MARK: - FeedbackTemplate（静的配列）

/// フィードバックテンプレート。SwiftData 永続化不要。
/// コード内に静的配列として定義し、ランダム選択で使用。
struct FeedbackTemplate {
    let id: String                      // "praise_highScore_notification_01"
    let type: FeedbackType
    let channel: Channel
    let character: CharacterID          // 発話キャラクター
    let condition: String               // テンプレート選択条件の説明
    let template: String                // "{best_category}の鍛錬、成果が出ておるぞ。"
    let variables: [String]             // ["best_category", "delta"]
}
```

### StreakState の算出

```swift
// StreakState は DailyRecord の履歴から都度計算する。
// パフォーマンス最適化として、最新の StreakState を UserDefaults に軽量キャッシュ。
// キャッシュは DailyRecord の保存時に更新。

// UserDefaults キー:
//   "streak_current": Int
//   "streak_longest": Int
//   "streak_lastDate": Date?  (LogicalDate.value)
```

---

## 2.3 ドメインイベント

ドメインイベントは集約の状態変更を通知し、コンテキスト間の疎結合を実現する。

```swift
// MARK: - ドメインイベント

protocol DomainEvent {
    var occurredAt: Date { get }
}

/// チェック入力完了時に発火。Feedback Context がリッスンしてフィードバック生成を開始。
struct CheckCompleted: DomainEvent {
    let occurredAt: Date
    let dailyRecord: DailyRecord
    let snapshot: ScoreSnapshot
}

/// ストリーク更新時に発火。
struct StreakUpdated: DomainEvent {
    let occurredAt: Date
    let streakState: StreakState
}

/// マイルストーン到達時に発火。celebrate フィードバックの最優先トリガー。
struct MilestoneReached: DomainEvent {
    let occurredAt: Date
    let milestone: Milestone
    let streakState: StreakState
}

/// ストリーク途切れ時に発火。encourage（責めない）フィードバックを生成。
struct StreakBroken: DomainEvent {
    let occurredAt: Date
    let brokenStreak: Int          // 途切れた時点の連続日数
    let longestStreak: Int         // 過去最長（保持される）
}

/// フィードバック生成完了時に発火。Notification Context / UI がリッスン。
struct FeedbackGenerated: DomainEvent {
    let occurredAt: Date
    let result: FeedbackResult
    let renderedMessages: [Channel: RenderedMessage]
}
```

### イベントフロー

```
ユーザーがチェック入力を完了
  → CheckCompleted 発火
    → Feedback Context: FeedbackEngine.evaluate() 実行
      → FeedbackResult 生成
        → Character Context: テンプレート選択 + 口調適用
          → FeedbackGenerated 発火
            → Notification Context: 必要に応じて通知をスケジュール
            → UI: サマリー画面にフィードバックを表示

  → StreakState を再計算
    → StreakUpdated 発火
    → (マイルストーン到達の場合) MilestoneReached 発火
    → (ストリーク途切れの場合) StreakBroken 発火
```

---

## 2.4 リポジトリインターフェース

```swift
// MARK: - DailyRecordRepository

protocol DailyRecordRepository {
    /// 論理日付で DailyRecord を取得。存在しなければ nil。
    func find(by date: LogicalDate) -> DailyRecord?

    /// 期間指定で DailyRecord 一覧を取得（ダッシュボード用）。
    func findAll(from: LogicalDate, to: LogicalDate) -> [DailyRecord]

    /// 直近 N 日分の DailyRecord を取得（移動平均計算用）。
    /// null の日は含まない（記録がある日のみ返す）。
    func findRecent(days: Int, from: LogicalDate) -> [DailyRecord]

    /// DailyRecord を保存（新規作成 or 更新）。
    /// Score は UPSERT（同一 itemId は上書き）。
    /// 保存時に cachedCategoryAverages / cachedTotalAverage を自動計算。
    func save(_ record: DailyRecord) throws

    /// 記録済み日数の合計（コールドスタート判定用）。
    func countRecordedDays() -> Int

    /// ストリーク計算用: 指定日から過去に遡って連続記録日を数える。
    func countConsecutiveDays(from: LogicalDate) -> Int
}

// MARK: - FeedbackRepository

protocol FeedbackRepository {
    /// 論理日付でキャッシュ済みフィードバックを取得。
    func find(by date: LogicalDate) -> FeedbackCache?

    /// フィードバックキャッシュを保存。
    func save(_ cache: FeedbackCache) throws

    /// 直近で使用したテンプレートID一覧を取得（飽き防止用）。
    func recentTemplateIDs(limit: Int) -> [String]
}
```

---

## 2.5 ドメインサービス

```swift
// MARK: - FeedbackEngine（ドメインサービス）

protocol FeedbackEngine {
    /// チェック完了時の即時フィードバック判定。
    func evaluateImmediate(context: FeedbackContext) -> FeedbackResult

    /// 翌朝フィードバック判定。
    func evaluateMorning(context: FeedbackContext) -> FeedbackResult

    /// 週次フィードバック判定（複合タイプ）。
    func evaluateWeekly(context: FeedbackContext) -> [FeedbackResult]
}

// MARK: - CharacterRenderer（ドメインサービス）

protocol CharacterRenderer {
    var characterId: CharacterID { get }

    /// FeedbackResult をチャネルに合わせてレンダリング。
    func render(feedback: FeedbackResult, channel: Channel) -> RenderedMessage
}

// MARK: - StreakCalculator（ドメインサービス）

protocol StreakCalculator {
    /// 現在の StreakState を算出。
    func calculate(from repository: DailyRecordRepository, at date: LogicalDate) -> StreakState
}
```

---

## 2.6 Score の null 扱いルール

レビュー指摘（P0）を解消した統一ルール:

| ルール | 定義 |
|---|---|
| **平均計算** | 入力済み項目のみで平均を算出。null 項目は分母・分子に含めない |
| **inputRate** | `入力済み項目数 / 25` |
| **フィードバック抑制** | `inputRate < 0.2`（5項目未満の入力）の場合、praise / rebuke のトリガーを抑制。encourage のみ発火可能 |
| **カテゴリ平均** | カテゴリ内で入力済み項目のみの平均。カテゴリ内全項目が未入力の場合は nil |
| **ストリーク判定** | `DailyRecord` が存在し、かつ紐付く `Score` が1件以上ある場合のみ「記録済み」 |

---

## 2.7 日付境界ルール（04:00 区切り）

| 入力時刻 | 論理日付 | 説明 |
|---|---|---|
| 2/14 22:00 | 2/14 | 通常の夜入力 |
| 2/15 01:30 | 2/14 | 深夜の遅い入力（前日扱い） |
| 2/15 03:59 | 2/14 | 境界直前（前日扱い） |
| 2/15 04:00 | 2/15 | 境界（当日扱い） |
| 2/15 04:01 | 2/15 | 早朝（当日扱い） |

入力開始時点の論理日付を採用する。入力中に04:00を跨いでも、開始時に確定した日付を使用。
