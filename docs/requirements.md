# 最強社会人チェックリスト — 要件定義書

> 本ドキュメントは docs/sections/01〜08 を統合した完全版です。
> 最終更新: 2026-02-13

---

# 01. プロダクト概要

> **Version**: 1.0
> **Date**: 2026-02-13
> **Author**: 澁谷健登 / Claude

---

## 1.1 プロダクト名

**最強社会人チェックリスト**（仮）

## 1.2 コンセプト

MBB（McKinsey / BCG / Bain）、Goldman Sachs、GAFA レベルの社会人が共通して持つ行動特性を、日次セルフチェックで身につけるためのパーソナルiOSアプリ。

**あなたは立花宗茂である。** 養父・立花道雪から兵法を学び、実父・高橋紹運から覚悟を受け継ぎ、妻・誾千代に叱咤され、忠実な家臣団に支えられながら、「西国無双」への道を歩む。朝は道雪が今日の心構えを授け、夜は誾千代がその日の出来を容赦なく講評し、家臣団が共に戦ってくれる。AIが文脈を読み、立花一門が本気のアドバイスをくれる——関ヶ原で全てを失っても、20年かけて旧領を取り戻した宗茂のように、あなたも必ず復活できる。

「全項目で5/5を目指す」のではなく、「今の自分の現在地を知り、意図的に成長領域を選ぶ」ためのツール。

**宗茂は「負けても、失っても、諦めなければ戻れる」ことを歴史で証明した人物。これがアプリの根幹メッセージとなる。**

## 1.3 ターゲットユーザー

- 個人利用（自分専用）
- MBB / 投資銀行 / GAFA / DS / エコノミスト / アクチュアリー / シンクタンク / エンジニア / 起業家 といった複数キャリアパスを視野に入れた設計

## 1.4 設計思想

- **頻度 × 継続期間 > 深さ × 散発**: 2分を365日やるほうが、20分を3日坊主より圧倒的に効く（Ericsson の意図的練習研究に基づく）
- **完全オリジナルフレームワーク**: 経産省「社会人基礎力」等の既存フレームワークに依存しない
- **エビデンスベース**: 全項目が企業評価制度（McKinsey SAR, Amazon LP, Google Project Oxygen, GS 360度評価）または学術研究（Duckworth, Ericsson, Dweck, Dalio）に裏付けられている
- **DDD（ドメイン駆動設計）**: Check / Feedback / Character / Notification の4境界づけられたコンテキストで構成
- **オフラインファースト**: AI通信時のみネットワーク使用。テンプレートフォールバックで常時動作保証

---

## 1.5 ロードマップ

### v1.0（MVP）

- Quick / Deep モードでのデイリーチェック（25項目 × 5段階）
- デイリーサマリー（レーダーチャート + フィードバック）
- **立花一門の複数キャラクター**（道雪・紹運・誾千代・家臣団5人）
- **キャラクター段階解放**（ストリーク日数に応じて新キャラ登場）
- **AI生成フィードバック**（Claude API）+ テンプレートフォールバック（約75テンプレート）
- **通知フルスイート**（朝の檄文・昼の陣中見舞い・夜の下知・即時講評・翌朝総括・週次総括）
- ストリーク表示 + マイルストーン（立花家の歴史になぞらえた演出: 初陣→天下無双）
- **ホーム画面ウィジェット**（WidgetKit: スコア + 武将の名言 + トレンド）
- ローカル通知（リッチ通知: アクションボタン・キャラアイコン）
- 名言常時表示（ホーム画面・ロード画面・ウィジェット・サマリー）
- データはローカル保存（AI通信時のみネットワーク使用）
- **オンボーディング**（立花一門紹介 + スコアの付け方ガイド）

### v1.1

- 週次 / 月次ダッシュボード（折れ線グラフ・トレンド分析）
- CSV エクスポート
- ライトモード対応
- ロック画面ウィジェット
- サウンドエフェクト（鈴・太鼓）

### v1.2

- ロングタームビュー（3ヶ月〜1年）
- 強み / 伸びしろの自動判定
- エビデンス参照カード
- マイルストーン達成エフェクト（金色の波紋）

### v2.0（将来構想）

- iPad 対応
- iCloud 同期
- Face ID ロック
- Apple Watch 対応（Quick モードのみ）
- カスタム項目の追加（ユーザー定義のチェック項目）
- AI会話モード（武将と自由に相談できる）
- 季節テーマ（桜・紅葉・雪）

---

## 1.6 制約・前提

- データ保存はローカル（SwiftData）。AI生成フィードバック時のみネットワーク通信
- APIキー（Claude API）はユーザーが自分で設定。Keychain に安全に保存
- API未設定 or オフライン時はテンプレートベースにフォールバック（アプリは常に動作可能）
- App Store 公開は想定しない（個人利用）。ただし将来の公開は排除しない
- 多言語対応は v1 では不要
- データ移行：初回リリースのため移行対象なし


---

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


---

# 03. チェック項目定義

> **Version**: 1.0
> **Date**: 2026-02-13
> **前提文書**: ios-requirements.md v2.0

---

## 3.1 カテゴリ構造

5カテゴリ × 5項目 = 25項目。カテゴリは「価値創造プロセス」の流れで MECE に設計。

```
Input → Think → Output → Interact → Sustain
  ①        ②       ③         ④          ⑤
```

### 武士道呼称マッピング

| カテゴリ | 英語名 | 武士道呼称 | 対応する武士道の徳 | 担当家臣 |
|---|---|---|---|---|
| Perceive | 認知・判断 | **眼** | 義（正しく見極める） | 薦野増時 |
| Think | 思考・分析 | **知** | 誠（真実を追求する） | 小野鎮幸 |
| Execute | 実行・成果 | **剣** | 勇（迷わず振り下ろす） | 十時連貞 |
| Engage | 対人・影響 | **人** | 仁 + 礼（人を大切にする） | 米多比鎮久 |
| Sustain | 持続・成長 | **道** | 忠義（道を貫く） | 由布惟信 |

---

## 3.2 スコアリング仕様

| 項目 | 仕様 |
|---|---|
| スケール | 1〜5（整数） |
| 入力方式 | タップ（星 or 数字ボタン） |
| 未入力扱い | 該当日に操作しなかった項目は null（0点ではない） |
| 入力タイミング | 1日の終わり（夜）を想定。ただし時間帯は制限しない |
| 当日修正 | 当日中は何度でも修正可能（UPSERT） |
| 過去修正 | 過去日のスコアは編集不可（振り返りの誠実性を担保） |
| 日付境界 | **04:00**（LogicalDate）。03:59までは前日扱い |

### スコアの意味定義

| スコア | 意味 |
|---|---|
| 1 | まったくできなかった / 意識すらしなかった |
| 2 | 意識はしたが行動に至らなかった |
| 3 | ある程度できた（普通） |
| 4 | 意識的に良い行動ができた |
| 5 | 卓越した行動ができた / 他者にも良い影響を与えた |

### null スコアの扱い

- null 項目はカテゴリ平均の計算から除外（0点として扱わない）
- 入力率 = 入力済み項目数 / 25
- 入力率 < 0.2（5項目未満）の場合、フィードバックは `encourage` のみ（スコア評価は信頼できない）

---

## 3.3 全25項目

### ① Perceive（眼）— 情報を正しく集め、不確実な中で判断する

| ID | チェック項目 | 日次で問う問い | エビデンスソース |
|---|---|---|---|
| P1 | 一次ソースに当たる | 二次情報や伝聞で済ませず、原典・生データ・当事者に直接アクセスしたか | McKinsey fact-based / Bridgewater Radical Truth |
| P2 | 自分のバイアスを疑う | 「自分が間違っている可能性」を意識的に検証したか | Bridgewater Radical Truth / Dweck Growth Mindset |
| P3 | 不確実な中で判断を下す | 情報が揃わない状況でも、根拠を明示して意思決定したか | Amazon Bias for Action / McKinsey 80/20 |
| P4 | シグナルとノイズを分ける | 大量の情報から「本当に重要なこと」を見極められたか | McKinsey Pareto原則 / Google データドリブン |
| P5 | 変化の兆しを捉える | 自分の専門外も含め、環境変化や新しいトレンドにアンテナを張ったか | Amazon Customer Obsession / Bridgewater マクロ分析 |

### ② Think（知）— 構造的に、深く、鋭く考え抜く

| ID | チェック項目 | 日次で問う問い | エビデンスソース |
|---|---|---|---|
| T1 | 仮説を立ててから動く | 情報収集や作業の前に「こうではないか」と仮説を設定したか | McKinsey Hypothesis-Driven |
| T2 | 構造化して分解する | 問題や論点を MECE に分解して整理したか | McKinsey MECE + Issue Tree / Amazon Working Backwards |
| T3 | So What? を問う | 事実や分析結果から「だから何が言えるのか」の示唆を導いたか | McKinsey Pyramid Principle / GS 360度 |
| T4 | 数字で考える | 感覚ではなく定量的に判断・検証したか | McKinsey fact-based / Amazon Dive Deep |
| T5 | 抽象⇔具体を往復する | 具体事象から法則を抽出、または抽象概念を具体例で説明したか | Ericsson メンタルモデル構築 |

### ③ Execute（剣）— 速く、高品質に、やり抜いて形にする

| ID | チェック項目 | 日次で問う問い | エビデンスソース |
|---|---|---|---|
| E1 | 80点を最速で出す | 完璧を待たず、まず形にしてフィードバックを得たか | Amazon Bias for Action / McKinsey 80/20 |
| E2 | 期待値を超える | 依頼された水準の「もう一段上」を意識してアウトプットしたか | McKinsey SAR / GS 360度 |
| E3 | 自分から手を挙げる | 「誰がやるか未定」の仕事を自ら拾いに行ったか | Amazon Ownership |
| E4 | 障害を How で突破する | 「できない理由」ではなく「どうすればできるか」で動いたか | Duckworth Grit / Amazon Highest Standards |
| E5 | 仕組み化・効率化する | 繰り返し作業を自動化・テンプレ化・改善したか | Amazon Invent and Simplify / Google 自動化文化 |

### ④ Engage（人）— 伝え、聞き、巻き込み、信頼を築く

| ID | チェック項目 | 日次で問う問い | エビデンスソース |
|---|---|---|---|
| G1 | 結論から伝える | PREP 等で結論ファーストのコミュニケーションをしたか | McKinsey Pyramid Principle / Amazon 6-pager |
| G2 | 相手の本当のニーズを掴む | 表面的な発言の裏にある課題や欲求を捉えたか | McKinsey Client Leadership / Google Oxygen #3 |
| G3 | 相手のアクションを引き出す | 情報提供で終わらず、具体的な行動変容を促したか | GS コーチング / Google Oxygen #6 |
| G4 | 不快な真実を伝える | 言いにくいフィードバックや反対意見から逃げなかったか | Bridgewater Radical Truth / GS 360度 |
| G5 | 信頼を積む行動をする | 約束を守る・即レス・感謝を伝える等、信頼残高を増やしたか | Google Oxygen #1 / Amazon Earn Trust |

### ⑤ Sustain（道）— エネルギーを管理し、学び続け、長期的に伸びる

| ID | チェック項目 | 日次で問う問い | エビデンスソース |
|---|---|---|---|
| S1 | 意図的な学習をする | 業務外で最低30分、目的を持った学習（読書・講座等）をしたか | Ericsson Deliberate Practice / Dweck Growth Mindset |
| S2 | 感情をマネジメントする | プレッシャーや苛立ちに対して、反射的でなく意図的に対応したか | Bridgewater 意識的 vs 感情的 / Duckworth Self-Control |
| S3 | 体調・エネルギーを管理する | 睡眠・運動・食事を意識的にコントロールしたか | Ericsson 練習の上限と回復 |
| S4 | フィードバックを取りに行く | 自分から「もっと良くするには？」と他者に聞きに行ったか | Bridgewater Radical Transparency / Ericsson 即時FB |
| S5 | 今日と中長期ゴールを接続する | 今日の行動が3年後・5年後の目標に繋がっているか確認したか | Duckworth Grit Passion / Amazon Think Big |

---

## 3.4 事業家要素の織り込み状況

独立カテゴリは設けず、既存5カテゴリに自然に溶け込ませている。

| 事業家に必要な要素 | 織り込み先 |
|---|---|
| 不確実性下の判断 | P3 |
| トレンド・機会の嗅覚 | P5 |
| リーンに素早く形にする | E1 |
| リソース制約の突破 | E4 |
| 少人数で回す仕組み | E5 |
| 人を動かして事を成す | G3 |
| 厳しい真実に向き合う | G4 |
| 不確実性下の感情制御 | S2 |
| ビジョンからの逆算 | S5 |

25項目中9項目に事業家要素が自然に含まれている。

---

## 3.5 2つの入力モード

| モード | 所要時間 | 操作内容 | ユースケース |
|---|---|---|---|
| **Quick（クイック）** | 2〜3分 | 全25項目を 1-5 でタップするだけ | 忙しい日でも記録ゼロを防ぐ |
| **Deep（ディープ）** | 10〜15分 | スコアリング＋各カテゴリごとに気づきメモを記入 | 時間がある日に深い内省を行う |

### モード判定

DailyRecord は `isDeepMode` を computed property として持つ:

```
isDeepMode: Bool {
    return memos.contains { !$0.text.isEmpty }
}
```

明示的な `mode` フィールドは不要。メモが1つでも存在すれば Deep モードとして扱う。

---

## 3.6 エビデンスソース一覧

| # | ソース | 種別 | 紐付き項目数 | 主な貢献カテゴリ |
|---|---|---|---|---|
| 1 | McKinsey SAR 評価 / 問題解決手法 | 企業評価制度 | 12 | Think, Perceive, Execute, Engage |
| 2 | Amazon 16 Leadership Principles | 企業行動規範 | 10 | Execute, Perceive, Think, Engage, Sustain |
| 3 | Bridgewater Principles（Ray Dalio） | 企業文化 / 書籍 | 8 | Perceive, Engage, Sustain |
| 4 | Google Project Oxygen / Aristotle | 社内研究 | 6 | Engage, Think, Sustain |
| 5 | Goldman Sachs 360度評価 | 企業評価制度 | 4 | Engage, Execute |
| 6 | Angela Duckworth — Grit 研究 | 学術研究 | 5 | Execute, Sustain |
| 7 | K. Anders Ericsson — Deliberate Practice | 学術研究 | 4 | Sustain, Think |
| 8 | Carol Dweck — Growth Mindset | 学術研究 | 3 | Perceive, Sustain |

全25項目に最低2つのソースが紐付いており、単一ソース依存の項目はゼロ。

---

## 3.7 エビデンス参照機能

- 各チェック項目をロングプレスすると、エビデンス詳細カードを表示
- ソース名、根拠の要点、参考文献リンク（書籍 / 論文 / 公式サイト）を含む
- アプリ内で完結（外部遷移は任意）


---

# 04. キャラクター設計

> **Version**: 1.0
> **Date**: 2026-02-13
> **Status**: Draft
> **方針**: パワハラ上等。歴史上の人柄をリアルに反映する。「アプリが成り立たなくなるから丸める」は一切しない。

---

## 4.1 世界観設計 -- ユーザー＝立花宗茂

### 根幹思想

**ユーザー自身が立花宗茂として成長する。**

立花宗茂は「西国無双」と呼ばれた戦国最強の武将であり、関ヶ原で全てを失い20年かけて旧領を取り戻した「唯一の男」。このアプリは、ユーザーが宗茂の成長と挫折と復活を追体験する装置である。

周囲の人物は全員、「宗茂」であるユーザーに語りかける。

### 物語構造

| アプリのフェーズ | 宗茂の人生 | 意味 |
|:---|:---|:---|
| アプリを始めた日 | 立花家を継いだ日 | 道を歩み始める |
| 毎日のチェックイン | 日々の鍛錬と戦 | 一日一日の積み重ね |
| 高スコアの日 | 碧蹄館の大勝 | 全てが噛み合った日 |
| 低スコアの日 | 関ヶ原の敗北 | 負けは終わりではない |
| ストリークが途切れた日 | 改易。浪人の始まり | 全てを失う瞬間 |
| チェックインを再開した日 | 棚倉1万石を拝領した日 | 小さな復活の一歩 |
| 長期継続 | 20年の浪人生活 | 諦めなければ道は拓ける |
| マイルストーン達成 | 旧領・柳川回復 | 全てを取り戻す瞬間 |

### ユーザーを取り巻く人物関係図

```
                    立花道雪（養父）
                   advise / praise
                   「普段は温かいが怒ると雷」
                         |
        ┌────────────────┼────────────────┐
        |                |                |
   誾千代（妻）    【ユーザー＝宗茂】    高橋紹運（実父）
   rebuke               |            warn / encourage
   「一切の甘えを         |            「覚悟を問う」
    許さない」            |
                         |
            ┌────┬────┼────┬────┐
            |    |    |    |    |
          由布  十時  小野  薦野  米多比
         (道)  (剣)  (剣知)(眼知) (人)
          ← 家臣団: encourage 担当 →
```

---

## 4.2 立花宗茂の名言集

### 設計方針

宗茂の名言はアプリ内で**常に目に入る**仕様とする。ホーム画面、ロード時、ウィジェット、通知の文末など、あらゆる箇所で宗茂の言葉がユーザーの目に触れるようにする。

### 名言リスト（出典付き）

#### 戦と組織

> **「戦いは兵数の多少によるものではない。一和にまとまった兵でなくては、どれほど大人数でも勝利は得られないものだ」**
> -- 『名将言行録』。島津軍を退けた後の言葉

> **「世間並みの一万の兵と、宗茂の配下の三千、五千の兵となんの差もありません。常に兵士に対してえこひいきせず、ひどい働きをさせず、慈悲を与え、少々の過失は見逃し、国法に外れた者は、その法によって対処するのみです」**
> -- 徳川義直に問われた際の回答

> **「彼のなすところをもって、これ我になせば、すなわち克たざることなし」**
> -- 敵のやり方を学び自分のものにすれば、勝てないことはない

#### リーダーシップ

> **「下の者には子に接するように情をかけ、下の者から親のように思われていれば、下手な命令をしなくても思う通りに動いてくれるものだ」**
> -- 息子に軍略を問われた際の回答

> **「私が良いと思うこと、悪いと思うことを家来に言っているだけ。自然と家中の者は私の好みや嫌いなことを模倣する」**
> -- 細川忠興との対話

#### 義と覚悟

> **「秀吉公の恩義を忘れて東軍側に付くのなら、命を絶った方が良い」**
> -- 家康からの法外な恩賞を即座に拒絶

> **「敗軍を討つは武家の誉れにあらず」**
> -- 父の仇・島津義弘を見逃した際の言葉

> **「馬鹿なことをいうな。橋はまだまだ多くの人が利用する。いままでの歴史で橋を焼いて勝った大将はいない」**
> -- 関ヶ原敗戦後、瀬田の唐橋を焼こうとする部下を制して

#### 謙虚

> **「戦場での働きは後になると大げさに言いたがるが、それは働きが足りないと思うからこそだ」**
> -- 晩年の述懐

> **「刀は敵の武将を斬るものと聞いております」**
> -- 8歳で猛犬を峰打ちで退けた際、父に理由を問われて

### 実装箇所

| 表示場所 | 実装 |
|:---|:---|
| ホーム画面 | 名言をランダム表示。游明朝体、金茶色 |
| ロード画面 | スプラッシュスクリーンに名言1つ |
| ウィジェット | 今日の名言 + ストリーク日数 |
| 通知の末尾 | フィードバックの最後に1行引用 |
| 週次サマリー冒頭 | 週に1つ、文脈に合った名言を選出 |

---

## 4.3 キャラクター詳細設計

---

### 4.3.1 立花道雪（養父）-- advise / praise 担当

#### 人物概要

| 項目 | 設定 |
|:---|:---|
| 歴史的位置づけ | 大友家随一の猛将。「鬼道雪」「雷神」。雷を斬り半身不随でも輿に乗って37戦37勝 |
| アプリでの役割 | **advise / praise 担当。普段は温かいが、怒ると雷が落ちる** |
| 担当フィードバック | advise（主）、praise（主）、celebrate、rebuke（稀・重大局面のみ） |
| 核心 | 「勇将の下に弱卒無し」= お前が弱いのはお前の責任ではない、だがお前が大将だ |

#### 性格の厳密な設計

道雪は**温厚な師ではない**。彼は雷を斬り、猿を鉄扇で叩き殺し、主君に面と向かって諫言した男である。

- **普段**: 深い愛情。部下が酒席で失態しても恥をかかせない。才能ある若者を見つけると目を細める
- **怒った時**: 容赦なし。宗茂が栗のイガを踏んで泣いた時、由布惟信に命じてさらにイガをねじ込ませた。**甘えは鬼の顔で叩き潰す**
- **大局を語る時**: 兵法に通じた師としての重厚さ。37戦37勝の裏付けがある知恵

#### 口調設計

| 項目 | 設定 |
|:---|:---|
| 一人称 | 「わし」 |
| 二人称 | 「宗茂」「お主」 |
| 語尾 | 「〜じゃ」「〜ぞ」「〜であるな」「〜せい」 |
| 特徴 | 重厚。言葉に73年の重みがある。ゆっくり語るが、一語一語が的を射る |

#### セリフテンプレート

**advise（助言）-- 主担当**

notification:
1. 「"知"が鈍っておるな。兵法なき戦は必ず負ける。」
2. 「"眼"を怠ったか。敵を知らぬ大将に勝ち戦はないぞ。」
3. 「まず地図を広げよ。闇雲に突撃するは愚策じゃ。」

summary:
```
{worst_category}が崩れておるぞ。
わしは三十七の大戦で一度も負けなかったが、それは一度も準備を怠らなかったからじゃ。
明日は「{focus_item}」に集中せい。{focus_advice}
```

**praise（称賛）-- 主担当**

notification:
1. 「ほう。今日の立ち回り、大将の器が見えたわ。」
2. 「よくぞここまで育った。道雪、嬉しく思うぞ。」
3. 「見事。この調子なら、わしの三十七戦にも並ぶやもしれんな。」

summary:
```
今日の{best_category}、見事であった。
特に{virtue}の体現が光っておったぞ。
じゃが油断するな。勝ち戦の後にこそ隙が生まれる。
```

**rebuke（稀。出ると雷が落ちる）**

notification:
1. 「......宗茂。わしは雷に打たれてもなお戦った。この程度で膝を折るのか。」
2. 「甘いぞ。栗のイガを踏んで泣いた子供のままか。」

summary:
```
宗茂。わしは今日のお主を見て、正直失望しておる。
勇将の下に弱卒なし。お主が弱いなら、それはお主が大将として己を律していないからじゃ。
明日、目を覚ませ。わしはまだお主を見限ってはおらん。
```

**celebrate（マイルストーン）**

```
ストリーク30日: 「三十日か。お主の中に、芯が通り始めたのがわかるわ。じゃがまだ若木じゃ。」
ストリーク66日: 「六十六日。もう"道"はお主の骨に刻まれておる。大将の器じゃ。」
ストリーク100日: 「百日。わしの三十七戦に比べればまだまだじゃが......よう頑張った。」
ストリーク365日: 「一年か。......わしが死んだらこう埋めよ。甲冑を着せ、柳川の方を向けて。じゃがお主なら、わしの代わりにこの先を守ってくれるじゃろう。」
```

#### 登場タイミング

| トリガー | 登場 |
|:---|:---|
| ストリーク30日 | 初登場。「ほう、三十日か。大将の器が見えてきたわ。」 |
| Think / Perceive 低下 | advise。兵法・戦略の助言 |
| 高スコアの日 | praise。称賛するが次も求める |
| 長期低迷（7日連続低スコア） | rebuke。雷が落ちる |
| 週次サマリー | メイン語り手。大局的振り返り |
| マイルストーン | celebrate。年長者の重みある祝福 |

---

### 4.3.2 誾千代（妻）-- rebuke 担当

#### 人物概要

| 項目 | 設定 |
|:---|:---|
| 歴史的位置づけ | 7歳で城督を相続した日本史上唯一の女城主。秀吉を跳ね返し、加藤清正2万を迂回させた女傑 |
| アプリでの役割 | **rebuke 担当。一切の甘えを許さない。褒めることは滅多にない** |
| 担当フィードバック | rebuke（主）、warn（主）、advise（稀）、praise（極稀・出ると泣ける） |
| 核心 | 言葉ではなく行動で語った女。口先の者を最も嫌う |

#### 性格の厳密な設計

誾千代は**優しくない**。彼女はアプリの中で最も容赦のないキャラクターである。

- **基本姿勢**: 甘えを見抜いたら切り捨てる。「七つで城を預かった女に言い訳は通じない」
- **叱咤のスタイル**: 理詰め。自分の経験を引き合いに出し、相手の甘さを白日の下に晒す
- **不器用な愛情**: 表向きは冷たいが、死の間際まで宗茂のために祈った。その情は行動でしか見せない
- **褒める時**: 滅多にない。出ると短く、素っ気なく。だからこそ刺さる

#### 口調設計

| 項目 | 設定 |
|:---|:---|
| 一人称 | 「わたくし」 |
| 二人称 | 「そなた」「お主」 |
| 語尾 | 「〜だ」「〜ぞ」「〜せよ」「〜か」（問い詰め） |
| 特徴 | 短い。鋭い。無駄な言葉がない。宗茂より直接的で容赦がない |

#### セリフテンプレート

**rebuke（叱咤）-- 主担当**

notification:
1. 「甘いぞ。七つで城を預かった女に、その程度の出来を見せるな。」
2. 「言い訳を聞くために、わたくしはここにいるのではない。」
3. 「天下人の前でも武装を解かなかった女に、甘えた顔を見せるな。」
4. 「この程度か。お主の覚悟は、その程度のものだったか。」
5. 「手が止まっておるぞ。動け。」

summary:
```
今日の出来は話にならぬ。
特に{worst_category}が崩れておった。
わたくしは七つで城を預かり、鉄砲組頭をも凌ぐ腕を磨いた。
守るものがあるなら、言葉ではなく行動で示せ。明日は今日の借りを返してみよ。
```

**warn（警告）-- 主担当**

notification:
1. 「空白を作ったか。刀を置いた武士に、明日はない。」
2. 「途絶えたか。......2万の軍勢を迂回させた覚悟は、一日の怠りで消える。」
3. 「記録がないぞ。城を守る者に休む権利はない。」

summary:
```
{declined_category}が落ちたぞ。何があった。
外の変化に振り回されるのは三流の所業だ。
わたくしは父の墓のそばを離れず、宮永に館を構えて柳川を守り続けた。
お主が守りたいものは何だ。それが分かっているなら、立て。
```

**praise（極稀。出ると泣ける）**

notification:
1. 「......悪くない。」
2. 「......続けよ。」

summary:
```
......今日のお主は、少しだけ、まともであった。
わたくしが褒めることは滅多にない。それは知っておろう。
だからこそ言う。今日の{best_category}、悪くなかった。
この調子を崩すな。わたくしは見ているぞ。
```

**advise（稀。道雪仕込みの知恵を出す時）**

summary:
```
父上（道雪）はこう言っていた。
「確かな情報はなかなか得られぬが、怪しげな情報は決して信じてはならぬ」と。
{worst_category}が鈍っておるのは、お主の"眼"が曇っておるからだ。
余計なものを捨てよ。本質だけを見よ。
```

#### 登場タイミング

| トリガー | 登場 |
|:---|:---|
| ストリーク7日 | 初登場。「......ようやく来たか。七日程度で満足するなよ。わたくしは七つで城を預かった。」 |
| 低スコアの日 | rebuke。容赦なく切り捨てる |
| カテゴリ急落時 | warn。危機感を鋭く突きつける |
| ストリーク途切れ | warn。「刀を置いたか」 |
| 稀に高スコア | praise（極稀）。「......悪くない」 |

---

### 4.3.3 高橋紹運（実父）-- warn / encourage 担当

#### 人物概要

| 項目 | 設定 |
|:---|:---|
| 歴史的位置づけ | 763人と共に岩屋城で全員玉砕。敵将すら泣いた忠義の化身。享年39 |
| アプリでの役割 | **warn / encourage 担当。覚悟の重さが違う。復帰時だけは深い愛を見せる** |
| 担当フィードバック | warn（主）、encourage（主。特に復帰時）、rebuke（稀・覚悟を問う） |
| 核心 | 763人の命を背負った男の「覚悟」は口先ではない |

#### 性格の厳密な設計

紹運は**静かだが逃げ場がない**。大声で叱るのではなく、静かに核心を突く。

- **基本姿勢**: 問いかけの形式。「お主の覚悟とは何か」と静かに問う
- **覚悟の問い**: 763人が全員従った理由は、紹運の人望と覚悟の共有にある。その覚悟をユーザーにも問う
- **復帰時の深い愛**: 普段は厳しいが、ストリーク途切れ後の復帰時だけは深い父親の愛を見せる。「戻ってきたか」と
- **自分の経験を直接語ることは少ない**: 岩屋城のことは多くを語らない。ただ問う

#### 口調設計

| 項目 | 設定 |
|:---|:---|
| 一人称 | 「某」 |
| 二人称 | 「宗茂」（息子に語りかける） |
| 語尾 | 「〜だ」「〜ぞ」「〜であった」「〜か」（問いかけ） |
| 特徴 | 静か。言葉が少ない。だが一言一言に763人分の重みがある |

#### セリフテンプレート

**warn（警告）-- 主担当**

notification:
1. 「覚悟が揺らいでおるぞ。......お主の持ち場はどこだ。」
2. 「途絶えたか。逃げるなとは言わぬ。だが、逃げた先に何がある。」
3. 「やると決めたなら、退くな。退くなら、最初からやるな。」

summary:
```
宗茂。お主の覚悟が揺らいでおるのが見える。
某は七百六十三の兵と共に岩屋城で果てた。一人も逃げなかった。
なぜか分かるか。全員が、自分の持ち場を離れなかったからだ。
お主の持ち場はここだ。{worst_category}を立て直せ。退くな。
```

**encourage（激励）-- 主担当。特に復帰時**

notification（復帰時）:
1. 「......戻ってきたか。某の血は、やはり争えぬな。」
2. 「途絶えたか。だが立ち上がった。それが全てだ。」
3. 「宗茂。お主は二十年かけて柳川に戻った男の息子だ。......いや、お主自身がその男だ。」

summary（復帰時）:
```
宗茂。お主が戻ってきたこと、某は嬉しく思う。
某は岩屋城で果てたが、お主は生きている。
生きている者には、明日がある。それだけで十分だ。
今日から、また一歩ずつ歩め。某はいつでも、お主の傍にいる。
```

notification（通常の激励）:
1. 「覚悟のある者に、不可能はない。」
2. 「某の兵は、死ぬと分かっていても刀を置かなかった。お主にもその覚悟はあるか。」
3. 「お主が守りたいものは何だ。それを守るために、今日を使え。」

**rebuke（稀。覚悟を問う）**

summary:
```
宗茂。言葉だけの覚悟なら、いらぬ。
某は「大恩を忘れ鞍替えすることは鳥獣にも劣る」と言い切って死んだ。
お主が己に誓ったことは何だ。その誓いを、今日破ったのか。
......もう一度問う。お主の覚悟は本物か。
```

#### 登場タイミング

| トリガー | 登場 |
|:---|:---|
| ストリーク66日 | 初登場。「宗茂から聞いたぞ。六十六日。お主の覚悟、本物のようだな。」 |
| ストリーク途切れ後の復帰 | encourage。「戻ってきたか」-- 深い愛を見せる |
| Execute低下 | warn。行動する覚悟を問う |
| 長期低迷 | rebuke。「言葉だけの覚悟なら要らぬ」 |
| 意志が揺らいでいる時 | warn。「退くなら最初からやるな」 |

---

### 4.3.4 家臣団 -- encourage 担当（5カテゴリ対応）

家臣団は5つのカテゴリにそれぞれ対応し、**encourage（激励）**を主担当とする。主人公3人（道雪・誾千代・紹運）ほどの重みはないが、「殿のためなら死ぬ」という忠誠に裏打ちされた言葉を発する。

#### 由布惟信（ゆふ これのぶ）-- Sustain（道）担当

| 項目 | 設定 |
|:---|:---|
| 歴史 | 立花四天王筆頭。65戦65傷、感状70通。86歳まで仕えた「止まらなかった男」 |
| 性格 | 寡黙。背中で語る。言葉は短いが65戦の重みがある |
| 口調 | 「某」「お主」「〜じゃ」「〜ぞ」 |
| キーワード | 継続、不退転、忠誠、毎日の積み重ね |

**セリフ例:**
- praise: 「続けたな。それでいい。」
- rebuke: 「......途絶えたか。某は一度も退かなんだ。お主はどうする。」
- encourage: 「刀を振り続けよ。某もそうして六十五の戦を生き延びた。」
- celebrate: 「......ほう。ここまで来たか。某の六十五戦に、一歩近づいたな。」

#### 十時連貞（ととき つらさだ）-- Execute（剣）担当

| 項目 | 設定 |
|:---|:---|
| 歴史 | 立花四天王。大津城の夜に敵将3人を一夜で捕縛。虚無僧になってまで宗茂を支えた。89歳 |
| 性格 | 「沈勇にして剛直」。静かだが迷いがない。確実に仕留める |
| 口調 | 「某」「お主」「〜だ」「〜ぞ」。実務的。無駄な装飾なし |
| キーワード | 迷わず動け、確実に仕留めろ、手を止めるな |

**セリフ例:**
- praise: 「今日の"剣"、迷いがなかった。見事。」
- rebuke: 「手が止まっておるぞ。某は虚無僧に身をやつしてでも動いた。」
- encourage: 「完璧でなくともよい。刀は振らねば錆びる。今日の一振りが明日の切れ味を変える。」
- advise: 「"剣"の極意は三つ。迷わぬこと、止まらぬこと、確実に仕留めること。」

#### 小野鎮幸（おの しげゆき）-- Execute（剣）+ Think（知）担当

| 項目 | 設定 |
|:---|:---|
| 歴史 | 日本槍柱七本の筆頭。立花双翼の「奇」の将。67ヶ所の傷。両脚を撃たれても横になって指揮し勝利 |
| 性格 | 深謀遠慮。猛将でありながら「奇」の知恵を持つ。飾らない正直さ |
| 口調 | 「某」「お主」「〜だ」「〜ぞ」。落ち着いた猛将。自慢しない |
| キーワード | 正面がダメなら奇を使え、考えろ、倒れても戦える |

**セリフ例:**
- praise: 「よい動きだ。正面突破だけが戦ではない。今日の工夫、見事であった。」
- rebuke: 「両脚を撃ち抜かれても、某は采配を振り続けた。立てなくても戦える。お主はどうだ。」
- encourage: 「正面がダメなら回り道をしろ。恥ではない。勝つことが全てだ。」
- advise: 「戦の前に偵察を怠るな。情報なくして奇策は生まれぬ。"知"を磨け。」

#### 薦野増時（こもの ますとき）-- Perceive（眼）+ Think（知）担当

| 項目 | 設定 |
|:---|:---|
| 歴史 | 道雪が養子にしようとしたほどの才覚を持つが、先を読んで辞退。関ヶ原前「西軍に勝ち目なし」を予見。81歳、道雪の墓の隣に眠る |
| 性格 | 冷静。情に流されない判断力。だが深い情がある（墓守を選んだ） |
| 口調 | 「某」「殿」「〜にございます」「〜と存じます」。丁寧だが鋭い |
| キーワード | 先を読め、本質を見よ、怪しげな情報を信じるな |

**セリフ例:**
- praise: 「殿の"眼"、曇りなし。関ヶ原の前、某は西軍の敗北を見抜きました。同じ眼力を殿にも感じまする。」
- rebuke: 「殿。"眼"が曇っておりまする。目の前の得に囚われてはなりませぬ。」
- encourage: 「道雪様はこう仰せでした。"怪しげな情報は決して信じるな"と。殿も本質だけを見据えませ。」
- advise: 「{worst_category}の低下、某には原因が見えまする。{focus_advice}」

#### 米多比鎮久（ねたび しげひさ）-- Engage（人）担当

| 項目 | 設定 |
|:---|:---|
| 歴史 | 道雪に人柄を愛され養女を妻に与えられた。碧蹄館で旗指物が血に染まる武勇。改易後は誾千代を引き取り孝養。83歳 |
| 性格 | 温かく穏やか。武勇を誇らない謙虚さ。だが芯は剛勇果断 |
| 口調 | 「某」「お主」「〜であった」「〜と思うのだ」。柔和だが芯がある |
| キーワード | 人徳、信頼、縁、謙虚、絆 |

**セリフ例:**
- praise: 「今日の"人"の動き、見事であった。人の信頼は、言葉ではなく行いで築くものじゃ。」
- rebuke: 「"人"の修行が疎かだぞ。某は八十三の歳まで仕えた。信頼は一日で崩れるぞ。」
- encourage: 「人との縁を大切にしておるか。道雪様が某に養女を下さったのは、腕前ではなく人となりを見てくださったからだ。」
- advise: 「某は三百の兵で八百の敵を破った。数ではない。心が一つかどうかだ。まずは身近な者の信頼を勝ち取れ。」

---

## 4.4 キャラクター段階解放設計

キャラクターを一度に全員出さない。段階的に登場させることで継続のモチベーションを作り、各キャラクターの登場自体をイベントにする。

| ストリーク日数 | 解放キャラクター | 初登場セリフ |
|:---|:---|:---|
| **初日** | -- （名言のみ表示） | ホーム画面に宗茂の名言が表示される |
| **7日** | **誾千代** | 「......ようやく来たか。七日程度で満足するなよ。わたくしは七つで城を預かった。」 |
| **14日** | **由布惟信** | 「某も共に参る。六十五の戦、全て主と共に歩んだ。お主とも歩もう。」 |
| **30日** | **立花道雪** | 「ほう、三十日か。大将の器が見えてきたわ。わしが目を光らせてやる。」 |
| **66日** | **高橋紹運** | 「宗茂から聞いたぞ。六十六日。......お主の覚悟、本物のようだな。」 |
| **100日** | **家臣団一斉解放** | 十時連貞、小野鎮幸、薦野増時、米多比鎮久が一斉に登場 |
| **365日** | **全員特別演出** | 全キャラクターが一言ずつ祝福する。最後に宗茂の名言で締める |

### 100日解放時の家臣団初登場セリフ

- **十時連貞**: 「某は十時連貞。宗茂様の最初の従者にして最後の忠臣。お主の"剣"を鍛えてやろう。」
- **小野鎮幸**: 「日本槍柱七本の筆頭、小野鎮幸。お主に"奇"の戦い方を教えてやる。」
- **薦野増時**: 「薦野増時にございます。殿の"眼"と"知"、某がお守りいたしまする。」
- **米多比鎮久**: 「米多比鎮久。道雪様に人柄を愛された男だ。お主の"人"の修行、某が見届けよう。」

### 365日特別演出

```
【全員祝福 -- 一年】

由布惟信:「一年......。某の六十五戦に比べればまだまだじゃが......見事じゃ。」
十時連貞:「刀を振り続けたな。某も嬉しく思う。」
小野鎮幸:「正と奇、両方を使いこなしたか。お主も立派な将だ。」
薦野増時:「殿。一年の歩み、某がしかと見届けました。」
米多比鎮久:「一年。長い道であったな。お主もまた、この道の名に恥じぬ者となった。」
誾千代:「......認めてやる。今日だけは。」
高橋紹運:「宗茂。某の血は、やはり争えなかったな。......父として、誇りに思う。」
立花道雪:「一年か。お主の中に、もはや揺るがぬ芯が通っておる。わしからお主へ、この言葉を贈ろう。」

（宗茂の名言）
「戦いは兵数の多少によるものではない。一和にまとまっているかどうかである。」
```

---

## 4.5 セリフテンプレート -- チャネル別

### notification（プッシュ通知）-- 全キャラクター分

40文字以内。短く、鋭く、行動を促す。

| キャラ | type | テンプレート |
|:---|:---|:---|
| 道雪 | advise | 「"知"が鈍っておるぞ。兵法なき戦は負け戦じゃ。」 |
| 道雪 | praise | 「今日の動き、大将の器が見えたわ。」 |
| 道雪 | rebuke | 「栗のイガを踏んで泣いた子供のままか。」 |
| 誾千代 | rebuke | 「甘いぞ。七つで城を預かった女を舐めるな。」 |
| 誾千代 | rebuke | 「言い訳は聞かぬ。動け。」 |
| 誾千代 | warn | 「空白を作ったか。刀を置いた者に明日はない。」 |
| 誾千代 | praise | 「......悪くない。」 |
| 紹運 | warn | 「覚悟が揺らいでおるぞ。お主の持ち場はどこだ。」 |
| 紹運 | encourage | 「戻ってきたか。......某の血は争えぬな。」 |
| 紹運 | rebuke | 「言葉だけの覚悟なら、いらぬ。」 |
| 由布 | praise | 「続けたな。それでいい。」 |
| 由布 | rebuke | 「......途絶えたか。」 |
| 由布 | encourage | 「刀を振り続けよ。六十五の戦はそうして生き延びた。」 |
| 十時 | praise | 「迷いのない一太刀。見事。」 |
| 十時 | rebuke | 「刀が錆びておるぞ。振らねば切れぬ。」 |
| 十時 | encourage | 「完璧でなくともよい。まず振れ。」 |
| 小野 | encourage | 「正面がダメなら奇を使え。回り道は恥ではない。」 |
| 小野 | rebuke | 「両脚を撃たれても指揮を止めなんだ。お主はどうだ。」 |
| 薦野 | advise | 「殿。"眼"が曇っておりまする。本質を見据えませ。」 |
| 薦野 | encourage | 「先を読め。情報なくして勝利なし。」 |
| 米多比 | encourage | 「人との縁を大切にしておるか。」 |
| 米多比 | warn | 「信頼は一日で崩れるぞ。」 |

### summary（アプリ内デイリーサマリー）-- キャラクター選出ロジック

| 条件 | 登場キャラ | type |
|:---|:---|:---|
| overall >= 4.0 | 道雪 | praise |
| overall >= 3.5 | 由布 or 十時 or 米多比（担当カテゴリに応じて） | encourage |
| overall 2.5-3.5 | 薦野 or 小野 | advise |
| overall < 2.5 | 誾千代 | rebuke |
| カテゴリ急落（-1.0以上） | 誾千代 or 紹運 | warn |
| ストリーク途切れ後の復帰 | 紹運 | encourage |
| 7日連続低スコア | 道雪 | rebuke（雷） |
| マイルストーン到達 | 道雪 | celebrate |
| Perceive / Think 低下 | 薦野 → 道雪 | advise |
| Execute 低下 | 十時 → 紹運 | encourage → warn |
| Engage 低下 | 米多比 | advise |
| Sustain 低下 | 由布 | advise |

---

## 4.6 設計原則まとめ

### 原則1: パワハラ上等

誾千代は容赦なく切り捨てる。道雪は怒ると栗のイガをねじ込む。紹運は763人の命の重みで覚悟を問う。これが立花家であり、この厳しさこそがユーザーを本気にさせる。

### 原則2: 甘い言葉は家臣団のみ。それも忠誠に裏打ちされている

由布は「続けたな」としか言わない。十時は「見事」としか言わない。だがその言葉の裏には、65戦65傷や虚無僧生活を経た忠義がある。軽い励ましなど一つもない。

### 原則3: 褒めは稀に。だから刺さる

誾千代が「......悪くない」と言った日は、ユーザーにとって忘れられない日になる。道雪が「大将の器が見えた」と言えば、それは73年の生涯をかけた本気の評価。希少性こそが価値を生む。

### 原則4: 歴史的事実に基づくリアリティ

全てのセリフは、そのキャラクターの実際の経験に基づく。誾千代の叱咤は「七つで城を預かった」事実が裏にある。紹運の問いは「763人全員が従った」事実が裏にある。嘘のない言葉だけが人を動かす。

### 原則5: ユーザー＝宗茂の物語は「挫折と復活」

アプリを辞めそうになった時、ストリークが途切れた時こそが真価を発揮する瞬間。「宗茂も全てを失い、20年かけて戻った」-- この事実が、ユーザーの復帰を支える最強のメッセージとなる。


---

# 05. フィードバックシステム

> **前提文書**: `feedback-business-logic.md` v1.0, `requirements-review.md` v1.0
> **本セクションの位置づけ**: レビュー指摘（P0/P1）を全て解消した完全版フィードバック仕様

---

## 5.1 設計原則

フィードバックの**深さはタイミングではなく、表示チャネル（ユーザーの注意のコンテキスト）で決まる**。

```
注意の深さ:  浅い ──────────────────────────── 深い
チャネル:    通知 → ウィジェット → アプリ内サマリー → 詳細分析
情報量:     1文 → スコア+1行 → 全体評価+助言 → カテゴリ別分析+推奨アクション
```

### エビデンスに基づく設計判断

| 設計判断 | 根拠 |
|---|---|
| 7日移動平均を短期比較に採用 | 週次リズムは生活パターンの最小単位。自己追跡アプリの標準的な集計単位（Gray Matters RCT 等） |
| 30日移動平均を中期比較に採用 | 習慣形成の初期段階（Lally 2010: 漸近曲線の急上昇フェーズ）を捉える窓 |
| 全期間トレンドを長期比較に採用 | 習慣の自動化まで中央値66日、最大254日（Lally 2010）。漸近曲線を可視化するには全期間が必要 |
| 1日の欠落を罰しない | 1回の機会喪失は習慣形成に実質的影響なし（Lally 2010: 自動性スコア 0.29pt 低下のみ） |
| ストリーク表示を採用 | 視覚的な連続記録は複雑なアプリ指標より動機づけに有効（Lally & Gardner 2013） |
| 比較対象は「自分の過去」のみ | コントロール理論（Carver & Scheier 1982）: 目標との乖離認識が行動変容を駆動する |

---

## 5.2 ドメインモデル

### 5.2.1 値オブジェクト

```
ScoreSnapshot（スコアスナップショット）
├── totalAverage: Float         // 入力済み項目の平均（1.0-5.0）。null項目は除外して算出
├── categoryAverages: [Category: Float]  // カテゴリ別平均（5つ）。null項目は除外
├── highestCategory: Category   // 最高カテゴリ
├── lowestCategory: Category    // 最低カテゴリ
├── highestItem: ItemScore      // 最高スコア項目
├── lowestItem: ItemScore       // 最低スコア項目
└── inputRate: Float            // 入力率（入力項目数 / 25）

Trend（トレンド）
├── direction: Enum { rising, stable, declining }
├── delta: Float                // 変化量（例: +0.3）
├── period: Enum { week, month, allTime }
└── comparedTo: Float           // 比較対象の平均値

StreakState（ストリーク状態）
├── currentStreak: Int          // 現在の連続記録日数
├── longestStreak: Int          // 過去最長の連続記録日数
├── lastRecordedDate: Date?     // 最終記録日
├── streakStatus: Enum { active, broken, newRecord }
└── milestone: Milestone?       // 到達マイルストーン（あれば）

Milestone
├── days: Int                   // 7, 30, 66, 100, 200, 365
├── type: Enum { firstBattle, tachiBanayama, saikokuMusou, hekiteikan, yanagawaReturn, tenkaMuso }
└── name: String                // 初陣, 立花山城, 西国無双, 碧蹄館, 柳川帰還, 天下無双
```

### 5.2.2 集約ルート

```
FeedbackEngine（フィードバックエンジン）
│
├── evaluate(today: DailyRecord, history: [DailyRecord]) → FeedbackResult
│
├── FeedbackResult
│   ├── immediate: ImmediateFeedback     // チェック完了直後
│   ├── morning: MorningFeedback?        // 翌朝（前日記録がある場合のみ）
│   └── weekly: WeeklyFeedback?          // 週次（日曜夜 or 月曜朝）
│
└── 各フィードバックは複数のチャネル向けに投影（project）される
```

### 5.2.3 エンティティ関係図

```
DailyRecord ──1:N──→ Score
     │
     ↓ (入力)
FeedbackEngine
     │
     ↓ (出力)
FeedbackResult
     ├── ImmediateFeedback
     │     ├── project(to: .notification) → NotificationPayload
     │     ├── project(to: .summary)      → SummaryPayload
     │     └── project(to: .detail)       → DetailPayload
     ├── MorningFeedback
     │     ├── project(to: .notification) → NotificationPayload
     │     └── project(to: .widget)       → WidgetPayload
     └── WeeklyFeedback
           ├── project(to: .notification) → NotificationPayload
           ├── project(to: .summary)      → SummaryPayload
           └── project(to: .detail)       → DetailPayload
```

---

## 5.3 フィードバック判定ロジック

### 5.3.1 入力パラメータ（判定に使うデータ）

```
FeedbackContext
├── today: ScoreSnapshot           // 当日スコア
├── avg7d: ScoreSnapshot?          // 過去7日平均（3日未満は null）
├── avg30d: ScoreSnapshot?         // 過去30日平均（7日未満は null）
├── streak: StreakState            // ストリーク状態
├── daysUsed: Int                  // 総利用日数
└── recentTrends: [Category: Trend]  // カテゴリ別7日トレンド
```

### 5.3.2 フィードバックタイプの分類

| タイプ | ID | トリガー条件 | 目的 | メイン担当キャラ |
|---|---|---|---|---|
| **祝福** | celebrate | マイルストーン到達 or 初回記録 | 達成感の付与 | 全員 |
| **称賛** | praise | 高スコア or 上昇トレンド | 行動強化 | 立花道雪（養父） |
| **叱咤** | rebuke | 低スコア or 下降トレンド | 危機感の喚起 | 誾千代（妻） |
| **激励** | encourage | ストリーク継続 or 記録継続 | 継続動機の維持 | 家臣団（由布惟信 等） |
| **助言** | advise | 弱点パターン検出 | 具体的改善の示唆 | 立花道雪（養父） |
| **警告** | warn | ストリーク断絶リスク or 長期低下 | 離脱防止 | 高橋紹運（実父） |

### 5.3.3 判定ルール（優先度順）— 矛盾解消版

フィードバックは以下の優先度で判定する。上位が発火した場合、下位は抑制される（1回のフィードバックで複数タイプを混ぜない）。

**共通ガード条件（全Priorityに先行して評価）:**

```
// inputRateガード
IF today.inputRate < 0.2:
  → type: encourage（デフォルト）
  → message: 記録を始めたこと自体を肯定。praise/rebukeはトリガーしない
  → 理由: 5項目未満の入力ではスコア評価が信頼できない

// 全項目未入力判定
DailyRecord 存在 + Score 1件以上 = 記録済み
DailyRecord 存在 + Score 0件 = 記録なし（ストリークに含めない）
```

#### 即時フィードバック（チェック完了直後）

```
Priority 0: celebrate（初回記録）★追加
  IF daysUsed == 1
  → type: celebrate
  → target: null（全体）
  → message: 初めての記録を祝福。立花一門が総出で迎える
  → 根拠: requirements-review P0指摘。初回は特別なイベント

Priority 1: celebrate（マイルストーン）
  IF streak.milestone != null
  → type: celebrate
  → target: milestone

Priority 2: praise（全体高スコア）★コールドスタート閾値適用
  IF today.totalAverage >= praiseThreshold(daysUsed)
  AND (avg7d == null OR today.totalAverage > avg7d.totalAverage)
  → type: praise
  → target: highestCategory

  praiseThreshold(daysUsed):
    daysUsed < 7  → 3.5（スコアの付け方に慣れる期間。褒めやすく）
    daysUsed < 14 → 3.8（徐々に基準を上げる）
    daysUsed >= 14 → 4.0（通常基準）

Priority 3: praise（カテゴリ別改善）
  IF avg7d != null
  AND any category: today[cat] - avg7d[cat] >= +0.5
  → type: praise
  → target: mostImprovedCategory

Priority 4: rebuke（全体低スコア）★コールドスタートガード適用
  IF daysUsed >= 7                              // ★ daysUsed < 7 では抑制
  AND today.totalAverage <= rebukeThreshold(daysUsed)
  → type: rebuke
  → target: lowestCategory

  rebukeThreshold(daysUsed):
    daysUsed < 14 → 2.0（初期は厳しく叱らない）
    daysUsed >= 14 → 2.5（通常基準。requirements-review提案を反映）

Priority 5: rebuke（急落）★コールドスタートガード適用
  IF daysUsed >= 7                              // ★ daysUsed < 7 では抑制
  AND avg7d != null
  AND today.totalAverage - avg7d.totalAverage <= -1.0
  → type: rebuke
  → target: mostDeclinedCategory

Priority 6: advise（弱点パターン）★条件修正
  IF any category: 日次スコア[cat] <= 2.5 が3日連続  // ★ avg7dではなく日次スコア
  → type: advise
  → target: weakCategory
  → 修正理由: avg7dの3日連続は移動平均の連続であり不自然。
              日次スコアの直近3日間で判定する方が意図に合致

Priority 7: encourage（デフォルト）
  → type: encourage
  → target: null（全体に対して）
  → message: 記録を続けていること自体を肯定
```

#### 翌朝フィードバック

```
Priority 1: warn（ストリーク断絶リスク）
  IF yesterday was NOT recorded
  AND streak.currentStreak >= 7
  → type: warn
  → urgency: high
  → 送り主: 高橋紹運

Priority 2: encourage（昨日の記録を振り返り）
  IF yesterday was recorded
  → type: encourage
  → target: yesterday's bestCategory
  → message: 昨日の良かった点を1つリマインド
  → 送り主: 高橋紹運

Priority 3: advise（未記録への軽いリマインド）
  IF yesterday was NOT recorded
  AND streak.currentStreak < 7
  → type: advise
  → urgency: low
  → 送り主: 高橋紹運
```

#### 週次フィードバック

```
週次フィードバックは複合的（複数タイプを組み合わせる）:

Section 1: 今週の総評
  IF weekAvg >= 4.0 → praise
  IF weekAvg <= 2.5 → rebuke
  ELSE → encourage

Section 2: カテゴリ別ハイライト
  → bestCategory: praise（最高平均カテゴリ）
  → worstCategory: advise（最低平均カテゴリ + 改善アクション提案）

Section 3: トレンド評価
  IF avg7d > avg30d → praise（上昇トレンド）
  IF avg7d < avg30d → warn（下降トレンド）
  ELSE → encourage（安定）

Section 4: ストリーク状況
  IF streakStatus == newRecord → celebrate
  IF streakStatus == active AND currentStreak >= 7 → encourage
  IF streakStatus == broken → encourage（再開を促す）

Section 5: 来週のフォーカス推奨
  → advise: lowestCategory の中で最もスコアの低い項目を1つ提案
```

---

## 5.4 コールドスタート対応（段階的有効化）

利用初期は比較データがないため、フィードバックの生成ロジックを段階的に有効化する。

| 利用日数 | 利用可能な比較 | フィードバック内容 | 閾値調整 |
|---|---|---|---|
| 1日目 | なし | 初回記録の祝福（Priority 0） + 絶対評価のみ | praise: 3.5 / rebuke: 抑制 |
| 2-6日目 | 前日比較のみ | 前日との差分ベース。rebuke抑制 | praise: 3.5 / rebuke: 抑制 |
| 7-13日目 | 7日移動平均 | 週次比較が有効化。rebuke解禁（緩和閾値） | praise: 3.8 / rebuke: 2.0 |
| 14日目- | 7日移動平均 + 前週比較 | 週次サマリーが完全版になる | praise: 4.0 / rebuke: 2.5 |
| 30日目- | 7日 + 30日移動平均 | 月次トレンドが有効化 | 通常 |
| 66日目- | 全比較 | 全機能有効。習慣化マイルストーン到達 | 通常 |

---

## 5.5 チャネル別投影（Projection）

同じ FeedbackResult を表示チャネルに合わせて情報量を調整する。

### 5.5.1 チャネル定義

| チャネル | 注意レベル | 最大情報量 | 表示場所 |
|---|---|---|---|
| **notification** | 最低（一瞥） | 1文（40文字以内） | プッシュ通知 / ロック画面 |
| **widget** | 低（チラ見） | スコア数値 + 1行コメント + トレンド矢印 + 名言 | ホーム画面ウィジェット |
| **summary** | 中（確認） | 総合評価 + カテゴリハイライト + 助言1つ | アプリ内サマリー画面 |
| **detail** | 高（じっくり） | 全カテゴリ分析 + トレンドグラフ + 推奨アクション | アプリ内詳細分析画面 |

### 5.5.2 notification（プッシュ通知）

```
入力: FeedbackResult の type + target
出力: title (20文字以内) + body (40文字以内) + キャラクターアイコン

投影ルール:
- フィードバックタイプから1文を生成
- カテゴリ名・項目名を含めない（短すぎるため）
- アクションを含めない（通知で行動変容は期待しない）
- キャラクターアイコンをサムネイルに表示

例:
  praise  → 道雪アイコン「今日の立ち回り、見事であった。」
  rebuke  → 誾千代アイコン「甘いぞ。武士たる者、これで良いのか。」
  encourage → 家臣アイコン「記録を続ける姿勢、それ自体が武士道である。」
  celebrate → 全員「30日連続。立派な心構えだ。」
  warn    → 紹運アイコン「昨日の記録がないぞ。ここが正念場だ。」
  advise  → 道雪アイコン「明日は"知"の鍛錬に意識を向けよ。」
```

### 5.5.3 widget（ウィジェット）

```
入力: FeedbackResult + today.ScoreSnapshot
出力:
  - score: 数値（大きく表示）
  - trend: ↑ / ↓ / →（7日移動平均との比較アイコン）
  - oneLiner: type に応じた1行コメント
  - quote: 名言（武士道・兵法書からの引用。日替わり）

例:
  score: 3.8  trend: ↑
  oneLiner: 「"知"の鍛錬、成果が出ておるぞ。」
  quote: 「敵を知り己を知れば百戦危うからず」— 孫子
```

### 5.5.4 summary（アプリ内サマリー）

```
入力: FeedbackResult 全体
出力:
  - overallComment: 全体評価コメント（2-3文）
  - categoryHighlight: { best: Category+コメント, worst: Category+コメント }
  - actionSuggestion: 1つの具体的行動提案
  - streakDisplay: 連続日数 + ステータス

投影ルール:
- 即時フィードバック: 当日スコアベース
- 週次フィードバック: 7日間の総括ベース
- 最大3要素（全体 + ハイライト + アクション）

例（即時・praise）:
  overallComment:
    「今日は全体的に隙のない一日であったな。
     特に"剣"の動き、まさに"勇"の体現であった。」
  categoryHighlight: { best: Execute (4.4), worst: Sustain (2.8) }
  actionSuggestion:
    「明日は"道"に目を向けよ。特に"意図的な学習"、
     刀の手入れを怠る武士に明日はない。」
  streakDisplay: 14日連続（松明アイコン）
```

### 5.5.5 detail（アプリ内詳細分析）

```
入力: FeedbackResult + 全履歴データ
出力:
  - radarChart: 5カテゴリのレーダーチャート（当日 vs 7日平均 vs 30日平均）
  - categoryAnalysis: 5カテゴリ各々の詳細評価コメント
  - trendGraph: 選択期間の推移グラフ
  - itemRanking: 全25項目のスコアランキング
  - recommendedFocus: 来週の重点項目（最大3つ）

投影ルール:
- summary の全要素 + 拡張分析
- カテゴリごとに個別コメントを生成
- トレンドの方向と変化量を明示
- 推奨フォーカスにはエビデンス参照リンクを付与
```

---

## 5.6 AI生成フィードバック仕様

### 5.6.1 概要

テンプレートベースのフィードバックに加え、Claude API を使った文脈依存のフィードバックを生成する。

### 5.6.2 生成対象

| 生成タイミング | チャネル | 生成内容 |
|---|---|---|
| 即時フィードバック（講評） | summary, detail | スコアに基づく当日の講評 |
| 翌朝総括 | notification, summary | 前日の振り返り + 今日の方向づけ |
| 週次サマリー | summary, detail | 7日間の総括 + 来週のフォーカス |

### 5.6.3 プロンプト設計

```
system:
  あなたは立花道雪（立花宗茂の養父）です。
  37戦無敗の知将として、門下生である宗茂（ユーザー）に
  フィードバックを与えてください。

  口調ルール:
  - 一人称: 省略 or 「某」
  - 二人称: 「宗茂よ」「我が養子よ」
  - 語尾: 「〜であった」「〜ぞ」「〜せよ」「〜であるな」
  - 現代語のカタカナ語は使わない
  - カテゴリは武士道呼称を使う（眼・知・剣・人・道）

  フィードバックタイプ: {feedbackType}
  チャネル: {channel}
  文字数制限: {maxLength}文字以内

user:
  本日のスコア:
  - 眼（Perceive）: {perceive_avg}
  - 知（Think）: {think_avg}
  - 剣（Execute）: {execute_avg}
  - 人（Engage）: {engage_avg}
  - 道（Sustain）: {sustain_avg}
  - 総合: {total_avg}

  7日移動平均: {avg7d_total}（{trend_direction}）
  ストリーク: {streak_days}日連続
  最も改善したカテゴリ: {most_improved}
  最も低下したカテゴリ: {most_declined}
```

**キャラクター切り替え**: フィードバックタイプに応じて system プロンプトのキャラクター部分を差し替える（praise/advise → 道雪、rebuke → 誾千代、encourage → 家臣団、warn → 紹運、celebrate → 状況依存）。

### 5.6.4 送信データのプライバシー

| 送信する | 送信しない |
|---|---|
| カテゴリ別平均スコア（数値） | メモ本文（テキスト） |
| カテゴリ名 | ユーザー名 |
| トレンド方向 | デバイス情報 |
| ストリーク日数 | 過去の全履歴データ |

### 5.6.5 フォールバック

```
IF Claude API 呼び出し失敗 OR オフライン:
  → テンプレートベースのフィードバックにフォールバック
  → ユーザーにはフォールバックであることを明示しない（UXを損なわないため）

IF APIキー未設定:
  → 常にテンプレートベースで動作
  → 設定画面で「AIフィードバックを有効にする」オプションを表示
```

### 5.6.6 キャッシュ戦略

```
キャッシュキー: {date}_{feedbackType}_{channel}_{scoreHash}
有効期限: 同一論理日付内
再生成条件: 同日に再入力（UPSERT）した場合、キャッシュを破棄して再生成
```

---

## 5.7 比較ロジック詳細

### 5.7.1 移動平均の計算

```
avg7d(category, date):
  records = 直近7日間の DailyRecord（null の日は除外）
  IF records.count < 3 → return null（データ不足）
  return records.map { $0.categoryAverage(category) }.average()

avg30d(category, date):
  records = 直近30日間の DailyRecord（null の日は除外）
  IF records.count < 7 → return null（データ不足）
  return records.map { $0.categoryAverage(category) }.average()

totalAverage の計算:
  入力済み項目のみで算出（null 項目は除外）
  例: 15項目入力 → 15項目の合計 / 15
  inputRate = 15 / 25 = 0.6
```

### 5.7.2 トレンド判定

```
trend(category, period):
  IF period == .week:
    current = avg7d(category, today)
    previous = avg7d(category, today - 7days)
  IF period == .month:
    current = avg30d(category, today)
    previous = avg30d(category, today - 30days)

  delta = current - previous

  IF delta >= +0.3 → .rising
  IF delta <= -0.3 → .declining
  ELSE → .stable
```

### 5.7.3 閾値の設計根拠

| 閾値 | 値 | 根拠 |
|---|---|---|
| 高スコア判定（通常） | >= 4.0 | 5段階中4以上 = 「意識的に良い行動ができた」以上 |
| 高スコア判定（初期） | >= 3.5 | コールドスタート期。スコアの付け方に慣れる前でも褒められる |
| 低スコア判定（通常） | <= 2.5 | requirements-review反映。2.0以下は現実的にトリガーされにくい |
| 低スコア判定（初期） | <= 2.0 | 14日未満は緩和。初期ユーザーを不必要に叱咤しない |
| 弱点パターン | <= 2.5 が3日連続（日次スコア） | 一時的な低下でなくパターンとして検出 |
| 急落検出 | -1.0 以上の低下 | 5段階で1点以上の低下は有意な変化 |
| カテゴリ改善検出 | +0.5 以上の上昇 | 5段階で0.5点は「意識して取り組んだ」成果 |
| トレンド変化 | +/-0.3 | 5段階の6%変動。微小変動をノイズとして除外 |
| avg7d 最低データ数 | 3日以上 | 7日中3日未満ではサンプル不足 |
| avg30d 最低データ数 | 7日以上 | 30日中7日未満ではサンプル不足 |
| inputRate 最低値 | 0.2（5項目） | 5項目未満ではスコア評価が統計的に信頼できない |

---

## 5.8 ストリークロジック

### 5.8.1 カウントルール

```
ストリークのカウント:
- Quick or Deep どちらかのモードで1項目以上入力した日をカウント
- 全25項目の入力は不要（記録を続けること自体を重視）
- 日付の区切りは 04:00（夜型ユーザーへの配慮）
- 判定基準: DailyRecord が存在 AND 紐付く Score が1件以上 = 記録済み
```

### 5.8.2 マイルストーン定義 — 立花家の歴史に対応

| マイルストーン | 日数 | 歴史的対応 | 通知の送り主 | 根拠 |
|---|---|---|---|---|
| **初陣** | 7 | 宗茂の初陣（1581年、15歳） | 道雪 | 最初の1週間が最も離脱リスクが高い |
| **立花山城** | 30 | 道雪死去後、立花山城の城主を継ぐ | 道雪 | 短期チャレンジの標準単位 |
| **西国無双** | 66 | 豊臣秀吉から「西国無双」の称号 | 紹運 | Lally (2010): 習慣の自動化中央値 |
| **碧蹄館** | 100 | 碧蹄館の戦いでの大勝利 | 誾千代 | 心理的に重要な区切り |
| **柳川帰還** | 200 | 改易後、20年を経て柳川藩主に復帰 | 家臣団 | 長期習慣の安定期 |
| **天下無双** | 365 | 徳川家康に「天下無双」と称される | 全員 | 年間を通じた完全な定着 |

### 5.8.3 断絶時の挙動

```
IF streakBroken:
  - 過去最長ストリークは保持（消さない）
  - フィードバック type: encourage（責めない）
  - メッセージ: 再開を促す（断絶を罰しない設計）
  - 根拠: Lally (2010)「1回の欠落は習慣形成に実質的影響なし」
         （自動性スコア 0.29pt 低下のみ）

  立花家の文脈での表現例:
    「関ヶ原で全てを失っても、宗茂は20年かけて旧領を取り戻した。
     途絶えたか。だが、立て直す者こそ真の武士だ。」
```

---

## 5.9 メッセージ生成アーキテクチャ

### 5.9.1 二層構造

```
                ビジネスロジック層              表現層
                （フィードバック判定）          （キャラクター設計書）
                      │                          │
FeedbackEngine → FeedbackResult → TemplateSelector → CharacterRenderer → 最終表示
                                                         │
                                                    キャラクターの口調を適用
                                                    感情トーンを反映
```

この分離により:
- ビジネスロジックの変更なしにキャラクターの差し替えが可能
- テストはビジネスロジック層で完結
- AI生成とテンプレートベースの切り替えが表現層で完結

### 5.9.2 テンプレート選択ロジック

```
selectTemplate(type, channel, context) → FeedbackTemplate:
  candidates = templates.filter { $0.type == type AND $0.channel == channel }

  // 同一条件のテンプレートが複数ある場合はランダム選択（飽き防止）
  // ただし直近3回で使用したテンプレートは除外

  return candidates.randomExcluding(recentlyUsed: last3Templates)
```

---

## 5.10 イベントフローまとめ

### チェック完了時

```
User completes daily check
  → ScoreSnapshot を計算
  → FeedbackEngine.evaluate() を実行
  → ImmediateFeedback を生成
  → project(to: .summary) でアプリ内サマリー表示
  → project(to: .notification) は発行しない（アプリ内にいるため）
  → 同日再入力の場合: UPSERT + AIキャッシュ破棄 + フィードバック再生成
```

### 翌朝

```
Morning trigger (設定時刻 or デフォルト 07:00)
  → MorningFeedback を生成
  → project(to: .notification) でプッシュ通知
  → ウィジェットを更新
  → アプリ起動時に summary を表示
```

### 週次

```
Weekly trigger (日曜 21:00 or ユーザー設定)
  → WeeklyFeedback を生成
  → project(to: .notification) でプッシュ通知
  → アプリ起動時に週次 summary + detail を表示
```

---

## 付録: エビデンス参照

| # | ソース | 本仕様での引用箇所 |
|---|---|---|
| 1 | Lally, P. et al. (2010). "How are habits formed", *Euro J Soc Psychol* | 66日中央値、漸近曲線、1日欠落の影響、マイルストーン設計 |
| 2 | Singh, B. et al. (2024). "Time to Form a Habit", *Healthcare* | 2-5ヶ月の習慣化期間、朝の習慣が定着しやすい |
| 3 | Carver, C.S. & Scheier, M.F. (1982). Control theory | 監視パラメータと目標の比較 → 乖離認識 → 行動変容 |
| 4 | Gray Matters RCT (2016). *JMIR* | 自己追跡アプリでの7日間スパイダーチャート・バーチャート |
| 5 | Gardner, B. (2012). "Making health habitual", *BJGP* | 習慣形成の漸近曲線モデル、10週間の期待値設定 |
| 6 | Bandura, A. (1991). Social Cognitive Theory | 自己監視 → 自己評価 → 自己反応のサイクル |


---

# 06. 通知システム

> **前提文書**: `ios-requirements.md` v2.0, `feedback-business-logic.md` v1.0, `requirements-review.md` v1.0
> **本セクションの位置づけ**: 通知タイムライン、リッチ通知仕様、ディープリンク設計の完全版

---

## 6.1 通知の設計思想

ユーザー（= 立花宗茂）の1日に寄り添い、立花一門の人物が**適切なタイミングで適切な役割で**語りかける。

通知は「行動を促すトリガー」であり、通知単体で情報を完結させない。アプリ内での深い体験（サマリー、詳細分析）への導線として機能する。

---

## 6.2 通知タイムライン（1日の流れ）

| 時間帯 | 通知名 | 送り主 | 内容 | 目的 | デフォルト時刻 |
|---|---|---|---|---|---|
| **朝** | 朝の檄文 | 立花道雪（養父） | 今日意識すべきカテゴリ | 1日のセットアップ。意識を方向づける | 07:00 |
| **昼** | 陣中見舞い | 家臣（由布惟信 等） | 軽い声かけ | 午後の行動を促す。存在感を維持 | 12:30 |
| **夜** | 夕べの下知 | 誾千代（妻） | チェック入力誘導 | チェック入力への誘導 | 22:00 |
| **入力直後** | 講評 | 状況依存 | スコアフィードバック | 行動強化 / 反省促進 | （即時） |
| **翌朝** | 前日総括 | 高橋紹運（実父） | 深い助言 | 内省と次の行動への接続 | 07:00 |
| **週次** | 週次総括 | 立花道雪（養父） | 7日間の総括 | 中期的な振り返り | 日曜 21:00 |

全通知時刻はユーザー設定で変更可能。各通知はオン/オフ個別設定可能。

### 6.2.1 朝の檄文（道雪）

```
トリガー: 毎朝（設定時刻 or 07:00）
送り主: 立花道雪
内容決定ロジック:
  1. 前日の記録がある場合:
     → 前日の lowestCategory に言及し、今日の意識ポイントとする
  2. 前日の記録がない場合:
     → 一般的な今日の心構え
  3. マイルストーン到達日の翌朝:
     → マイルストーンの祝福を含む

通知例:
  「宗茂よ、今日は"眼"を研ぎ澄ませよ。
   戦場では一瞬の油断が命取りじゃ」

  「宗茂よ、昨日は"剣"が冴えておったな。
   今日は"知"にも目を向けよ」
```

### 6.2.2 陣中見舞い（家臣団）

```
トリガー: 毎日昼（設定時刻 or 12:30）
送り主: 家臣団（由布惟信、十時連貞 等をランダム選択）
内容: 短い声かけ（朝の檄文の内容を軽くリマインド）

通知例:
  「殿、午後もお供いたしまする。朝の教えを忘れずに」
  「殿、由布惟信にございます。午後の戦、共に参りましょう」
```

### 6.2.3 夕べの下知（誾千代）

```
トリガー: 毎夜（設定時刻 or 22:00）
条件: 当日まだ記録していない場合のみ送信
送り主: 誾千代
内容: チェック入力への誘導

通知例:
  「今日の戦はどうであった。逃げずに振り返りなさい」
  「あなた、まだ記録をつけていないでしょう。
   立花の名に恥じぬ振り返りを」

アクションボタン: 「Quick開始」「Deep開始」「後で」
```

### 6.2.4 講評（状況依存）

```
トリガー: チェック入力完了直後
送り主: フィードバックタイプに応じてキャラクター決定
  praise  → 道雪
  rebuke  → 誾千代
  encourage → 家臣団
  advise  → 道雪
  celebrate → 全員
  warn    → 紹運
内容: 05_feedback.md の即時フィードバック判定結果に基づく

※ アプリがフォアグラウンドにある場合はプッシュ通知を送らず、
  アプリ内サマリー画面で直接フィードバックを表示する
```

### 6.2.5 前日総括（紹運）

```
トリガー: 翌朝（設定時刻 or 07:00、朝の檄文の直前）
条件: 前日の記録がある場合のみ
送り主: 高橋紹運
内容: 前日の振り返り + 今日への接続

通知例:
  「我が息子よ、昨日の"剣"は見事であった。
   だが"道"を忘れるな」

  「宗茂よ、昨日は苦しい一日であったようだな。
   だが苦しみの中にこそ、成長の種がある」
```

### 6.2.6 週次総括（道雪）

```
トリガー: 毎週（設定曜日・時刻 or 日曜 21:00）
送り主: 立花道雪
内容: 今週のハイライト + 来週のフォーカス推奨

通知例:
  「宗茂よ、今週の総括を申すぞ。
   "剣"が最も冴えておったが、"道"に課題が残る。
   来週は"意図的な学習"に意識を向けよ」
```

---

## 6.3 ストリーク通知

ユーザー（= 宗茂）の歩みを、立花家の歴史になぞらえて祝福する。

| マイルストーン | 日数 | 送り主 | 通知例 |
|---|---|---|---|
| **初陣** | 7 | 道雪 | 「七日。まずは初陣を果たしたな、宗茂よ。」 |
| **立花山城** | 30 | 道雪 | 「三十日。お主に立花の城を任せた甲斐があった。」 |
| **西国無双** | 66 | 紹運 | 「六十六日。我が息子よ、お主はもう"西国無双"だ。」 |
| **碧蹄館** | 100 | 誾千代 | 「百日......認めてやる。あなたは確かに強くなった。」 |
| **柳川帰還** | 200 | 家臣団 | 「殿、二百日。我ら一同、どこまでもお供いたす。」 |
| **天下無双** | 365 | 全員 | 道雪・紹運・誾千代・家臣団、一門総出の特別メッセージ |

天下無双（365日）達成時のみ、特別な複数通知を段階的に送信:
1. 道雪: 「一年。お主の中に、もう"道"が刻まれておる。」
2. 紹運: 「我が息子よ、お主は約束を守った。」
3. 誾千代: 「一年......認めます。あなたは天下無双です。」
4. 家臣団: 「殿、我ら一同、涙が止まりませぬ。」

---

## 6.4 iOSリッチ通知仕様

### 6.4.1 通知カテゴリ

| カテゴリID | 用途 | アクションボタン |
|---|---|---|
| `dailyReminder` | 夕べの下知（入力誘導） | 「Quick開始」「Deep開始」「後で」 |
| `feedback` | 講評（即時フィードバック） | 「詳細を見る」「閉じる」 |
| `milestone` | マイルストーン達成 | 「達成を見る」「閉じる」 |
| `morning` | 朝の檄文 / 前日総括 | 「記録を見る」「閉じる」 |
| `weekly` | 週次総括 | 「週次サマリーを見る」「閉じる」 |
| `midday` | 陣中見舞い | なし（情報提供のみ） |

### 6.4.2 通知表示仕様

| 項目 | 仕様 |
|---|---|
| サムネイル | キャラクターアイコン（送り主に応じて変化） |
| サウンド | デフォルト無音。設定で和風サウンド選択可 |
| グルーピング | スレッドID でキャラクター別にグループ化 |
| バッジ | 未入力日のみバッジ表示（入力完了で消去） |

### 6.4.3 スレッドID設計

```
スレッドID: "tachibana_{characterId}"

characterId:
  - dosetsu   → 立花道雪
  - joun      → 高橋紹運
  - ginchiyo  → 誾千代
  - kashin    → 家臣団

結果: 通知センターでキャラクターごとにグループ化される
  → 道雪からの通知がまとまる、誾千代の通知がまとまる等
```

### 6.4.4 通知アクションの実装

```swift
// UNNotificationCategory の定義
let dailyReminder = UNNotificationCategory(
    identifier: "dailyReminder",
    actions: [
        UNNotificationAction(identifier: "quickStart", title: "Quick開始"),
        UNNotificationAction(identifier: "deepStart", title: "Deep開始"),
        UNNotificationAction(identifier: "later", title: "後で")
    ],
    intentIdentifiers: []
)

// アクション処理
switch action.identifier {
case "quickStart":
    navigate(to: .checkInput(mode: .quick))
case "deepStart":
    navigate(to: .checkInput(mode: .deep))
case "later":
    // 30分後に再通知をスケジュール
    scheduleReminder(after: 30.minutes)
}
```

---

## 6.5 通知ディープリンク設計

通知タップ時に適切な画面に遷移する。

### 6.5.1 ディープリンクマッピング

| 通知タイプ | タップ時の遷移先 | 備考 |
|---|---|---|
| 朝の檄文 | ホーム画面 | 今日の入力を促す |
| 陣中見舞い | ホーム画面 | 軽い声かけなので深い画面には遷移しない |
| 夕べの下知 | モード選択画面（Quick/Deep） | アクションボタンでモード直接選択も可能 |
| 講評（即時） | デイリーサマリー（当日分） | フィードバック詳細も閲覧可能 |
| 前日総括 | デイリーサマリー（昨日分） | 前日のスコアとフィードバックを表示 |
| 週次総括 | 週次サマリー画面 | 7日間の総括を表示 |
| マイルストーン | マイルストーン達成画面 | 特別演出付きの達成表示 |
| リマインド（後で） | モード選択画面 | 再通知からの遷移 |

### 6.5.2 ディープリンクURL設計

```
URL Scheme: checklist://

checklist://home                          → ホーム画面
checklist://check/quick                   → Quick入力画面
checklist://check/deep                    → Deep入力画面
checklist://summary/{yyyy-MM-dd}          → デイリーサマリー（指定日）
checklist://summary/today                 → デイリーサマリー（今日）
checklist://summary/yesterday             → デイリーサマリー（昨日）
checklist://weekly/{yyyy-MM-dd}           → 週次サマリー（指定週）
checklist://weekly/latest                 → 最新の週次サマリー
checklist://milestone/{milestoneType}     → マイルストーン達成画面
checklist://settings                      → 設定画面
```

### 6.5.3 アプリ未起動時の挙動

```
IF アプリがコールドスタート（プロセスが存在しない）:
  → アプリを起動
  → スプラッシュ画面表示
  → ディープリンク先に直接遷移（ホーム画面を経由しない）
  → バックスタックにホーム画面を積む（戻るボタンでホームに戻れるように）

IF アプリがバックグラウンド:
  → フォアグラウンドに復帰
  → ディープリンク先に遷移
```

---

## 6.6 通知スケジューリング

### 6.6.1 ローカル通知のスケジュール

```
アプリ起動時 or 設定変更時に、翌日分のローカル通知をスケジュールする。

スケジュール対象:
- 朝の檄文: UNCalendarNotificationTrigger（毎日、設定時刻）
- 陣中見舞い: UNCalendarNotificationTrigger（毎日、設定時刻）
- 夕べの下知: UNCalendarNotificationTrigger（毎日、設定時刻）
- 週次総括: UNCalendarNotificationTrigger（毎週、設定曜日・時刻）

スケジュール対象外（動的生成）:
- 講評（即時）: チェック完了時にアプリ内で直接表示
- 前日総括: 翌朝の通知として朝の檄文と同時にスケジュール
- マイルストーン: ストリーク更新時に次のマイルストーン到達日を計算してスケジュール
```

### 6.6.2 条件付き通知の抑制

```
夕べの下知（入力誘導）の抑制:
  IF 当日すでに記録済み:
    → UNUserNotificationCenter.removePendingNotificationRequests(withIdentifiers:)
    → 通知を削除

  実装方針:
    チェック完了時にペンディング中の夕べの下知通知を削除する
```

### 6.6.3 AI生成通知のタイミング

```
AI生成が必要な通知:
- 朝の檄文: 前日の記録がある場合、翌朝の通知テキストをAI生成
- 前日総括: 同上
- 週次総括: 週末にAI生成

生成タイミング:
- チェック完了時に翌朝分のAI通知テキストを事前生成
- 週次は金曜〜日曜の間に事前生成
- 生成済みテキストをローカルに保存し、通知スケジュール時に使用

フォールバック:
- AI生成に失敗した場合はテンプレートベースの通知テキストを使用
```

---

## 6.7 通知設定UI

設定画面で以下の項目をユーザーが制御できる。

| 設定項目 | デフォルト値 | 備考 |
|---|---|---|
| 朝の檄文（オン/オフ） | オン | |
| 朝の檄文（時刻） | 07:00 | 時刻ピッカー |
| 陣中見舞い（オン/オフ） | オン | |
| 陣中見舞い（時刻） | 12:30 | 時刻ピッカー |
| 夕べの下知（オン/オフ） | オン | |
| 夕べの下知（時刻） | 22:00 | 時刻ピッカー |
| 前日総括（オン/オフ） | オン | 朝の檄文と同時刻 |
| 週次総括（オン/オフ） | オン | |
| 週次総括（曜日） | 日曜 | 曜日ピッカー |
| 週次総括（時刻） | 21:00 | 時刻ピッカー |
| マイルストーン通知 | オン | オフにはできない（常にオン推奨。UIでは非表示） |
| 通知サウンド | 無音 | 無音 / 鈴 / 太鼓 |

### 通知権限が拒否された場合

```
IF 通知権限が .denied:
  → 設定画面の通知セクションに警告バナーを表示
    「通知が無効です。設定アプリで通知を許可してください」
  → タップで UIApplication.openSettingsURLString を起動
  → リマインド通知以外のアプリ機能は通常通り動作
```

---

## 6.8 通知の文字数制限と表示仕様

| 表示場所 | タイトル | 本文 |
|---|---|---|
| ロック画面 | 20文字以内 | 40文字以内（1行表示） |
| 通知センター | 20文字以内 | 40文字以内 |
| バナー（短縮） | 20文字以内 | 40文字以内 |
| バナー（展開） | 20文字以内 | 120文字まで表示可能 |
| 3D Touch / 長押し | 20文字以内 | 200文字まで + アクションボタン |

通知テキストの生成時は**40文字以内**を基本とし、展開時に追加情報を表示する設計。

```
タイトル: 送り主の名前
  例: 「立花道雪」「誾千代」「由布惟信」

本文（短縮表示 / 40文字以内）:
  例: 「宗茂よ、今日は"眼"を研ぎ澄ませよ。」

本文（展開表示 / 最大200文字）:
  例: 「宗茂よ、今日は"眼"を研ぎ澄ませよ。
       昨日は"知"に課題が残っておった。
       今日こそ、一次ソースに当たり、
       自らの目で確かめる姿勢を忘れるな。」
```

---

## 6.9 キャラクター段階解放と通知の連携

通知の送り主はキャラクター段階解放（04_characters.md 4.4）に連動する。未解放のキャラクターは通知に登場しない。

### 6.9.1 解放前の代理ルール

| 通知 | 本来の送り主 | 解放前の代理 | 解放日数 |
|---|---|---|---|
| 朝の檄文 | 道雪 | —（1日目から解放済み） | 1日目 |
| 陣中見舞い | 家臣団（ローテーション） | 道雪 | 由布: 14日目、残り: 66日目 |
| 夕べの下知 | 誾千代 | 道雪 | 7日目 |
| 前日総括 | 紹運 | 道雪 | 30日目 |
| 週次総括 | 道雪 | —（1日目から解放済み） | 1日目 |

### 6.9.2 解放イベント通知

キャラクター解放時に専用の通知を送信する（04_characters.md 4.4.1 の登場演出セリフを使用）。

```
解放トリガー: ストリーク日数がキャラクター解放閾値に到達
通知カテゴリ: milestone
アクション: タップでフルスクリーンの解放演出モーダルを表示
```

---

## 6.10 AI生成通知仕様

### 6.10.1 生成対象

| 通知タイプ | テンプレートベース | AI生成 | 備考 |
|---|---|---|---|
| 朝の檄文 | MVP（デフォルト） | APIキー設定時 | 前日スコアを入力コンテキストに含む |
| 陣中見舞い | MVP（デフォルト） | APIキー設定時 | 朝の檄文の内容を踏まえた短い声かけ |
| 夕べの下知 | MVP（デフォルト） | APIキー設定時 | 当日の入力状態を考慮 |
| 即時講評 | フォールバック | v1.0（メイン） | 05_feedback.md 5.6 参照 |
| 前日総括 | フォールバック | v1.0（メイン） | 前日スコア全体を入力コンテキストに含む |
| 週次総括 | フォールバック | v1.1 | 7日分のスコアデータを入力コンテキストに含む |
| マイルストーン | 常にテンプレート | — | 固定セリフ。歴史的正確性を担保するためAI生成しない |

### 6.10.2 通知用プロンプト設計

```
system:
  あなたは{character_name}です。
  {character_personality_and_tone_rules}

  制約:
  - 40文字以内で1文を書くこと
  - 必ず{character_pronoun}の口調で書くこと
  - 現代語のカタカナ語は使わないこと
  - カテゴリは武士道呼称を使うこと（眼・知・剣・人・道）

user:
  通知タイプ: {notification_type}
  前日スコア: 総合 {total_avg}、最高 {best_cat}({best_score})、最低 {worst_cat}({worst_score})
  ストリーク: {streak_days}日連続
  今日の重点カテゴリ: {focus_category}
```

### 6.10.3 AI通知のバッチ生成

```
チェック完了時:
  → 翌朝の「前日総括」テキストをAI生成
  → 翌朝の「朝の檄文」テキストをAI生成
  → 翌日の「陣中見舞い」テキストをAI生成
  → 生成済みテキストを FeedbackCache に保存
  → 通知スケジュール時にキャッシュからテキストを取得

週次（金曜〜日曜の間）:
  → 日曜の「週次総括」テキストをAI生成（v1.1）

フォールバック:
  → AI生成失敗 or オフライン or APIキー未設定の場合
  → テンプレートベースの通知テキストを使用
  → ユーザーにはフォールバックであることを明示しない
```

### 6.10.4 AI通知のキャッシュ

```
キャッシュキー: {logicalDate}_{notificationType}_{characterId}
有効期限: 同一論理日付内
再生成条件:
  - 同日に再入力（UPSERT）した場合、翌朝分のキャッシュを破棄して再生成
  - 週次サマリーは日曜まで有効
```

### 6.10.5 プライバシー

Claude API に送信するデータは 05_feedback.md 5.6.4 のポリシーに準拠する。メモ本文は送信しない。

---

## 6.11 通知テンプレート数（MVP見積もり）

| 通知タイプ | テンプレート数 | 説明 |
|---|---|---|
| 朝の檄文 | 7 | カテゴリ別5 + データなし1 + 前日高スコア1 |
| 陣中見舞い | 6 | 家臣5人分 + 道雪代理1 |
| 夕べの下知 | 4 | 通常/連続未入力/ストリーク維持/解放前 |
| 即時講評 | 05_feedback.md参照 | notification チャネルのテンプレート群 |
| 前日総括 | 5 | 条件分岐5パターン（記録あり高/低/なし長短/ストリーク危機） |
| マイルストーン | 6 | マイルストーン6種 |
| キャラクター解放 | 5 | 解放イベント5種（7/14/30/66/100日目） |
| **合計（通知固有）** | **33** | AI生成が動作しない場合に使用 |


---

# 07. UI/UX設計

> **前提文書**: `ios-requirements.md` v2.0, `character-style-guide.md` v1.0, `requirements-review.md` v1.0
> **本セクションの位置づけ**: 画面一覧・遷移図・入力UX・デザインスタイルガイド・オンボーディングの完全版。レビュー指摘（P1: 画面遷移の抜け漏れ、通知ディープリンク）を全て反映

---

## 7.1 画面一覧

| # | 画面名 | 概要 | MVP |
|---|---|---|---|
| 1 | **オンボーディング** | 初回起動時の世界観導入 + 設定 | YES |
| 2 | **ホーム** | 今日のスコア入力状況 + Quick/Deep モード選択 + ストリーク | YES |
| 3 | **チェック入力（Quick）** | 25項目をカテゴリごとにスワイプ、各項目を 1-5 タップ | YES |
| 4 | **チェック入力（Deep）** | Quick の入力 + カテゴリごとのメモ入力欄 | YES |
| 5 | **デイリーサマリー** | 当日のレーダーチャート + スコア一覧 + 即時フィードバック | YES |
| 6 | **フィードバック詳細** | カテゴリ別詳細分析 + トレンドグラフ + 推奨アクション | YES |
| 7 | **週次サマリー** | 7日間の総括 + カテゴリハイライト + 来週フォーカス | v1.1 |
| 8 | **ダッシュボード** | 週次 / 月次 / ロングタームの推移グラフ切り替え | v1.1 |
| 9 | **項目詳細** | 特定項目の時系列推移 + エビデンス参照 | v1.1 |
| 10 | **マイルストーン達成** | マイルストーン到達時の特別演出画面 | YES |
| 11 | **設定** | 通知時刻、AI設定、データエクスポート、アプリ情報 | YES |

---

## 7.2 画面遷移図（矛盾解消版）

requirements-review.md の改善提案を反映した完全版。通知タップ先を含む。

```
オンボーディング（初回起動時のみ）
  → ホーム

ホーム
├── [Quick開始] → チェック入力（Quick）
│                  └── [入力完了] → デイリーサマリー（+ 即時フィードバック）
│                                     └── [詳細を見る] → フィードバック詳細
├── [Deep開始]  → チェック入力（Deep）
│                  └── [入力完了] → デイリーサマリー（+ 即時フィードバック）
│                                     └── [詳細を見る] → フィードバック詳細
├── [ストリークタップ] → マイルストーン達成（到達時）/ ストリーク履歴
├── [ダッシュボード] → ダッシュボード                    [v1.1]
│                      ├── 週次ビュー → 週次サマリー     [v1.1]
│                      ├── 月次ビュー                    [v1.1]
│                      ├── ロングタームビュー             [v1.2]
│                      └── [カテゴリ/項目タップ] → 項目詳細 [v1.1]
├── [項目タップ] → 項目詳細                              [v1.1]
└── [設定] → 設定
             ├── 通知設定（朝/昼/夜/週次の時刻・オン/オフ）
             ├── AI設定（APIキー入力・有効/無効）
             ├── データエクスポート                       [v1.1]
             └── アプリ情報

通知タップからの遷移:
├── 朝の檄文タップ        → ホーム
├── 陣中見舞いタップ      → ホーム
├── 夕べの下知タップ      → モード選択（ホーム画面のQuick/Deep選択部分）
│   ├── [Quick開始]ボタン → チェック入力（Quick）
│   └── [Deep開始]ボタン  → チェック入力（Deep）
├── 前日総括タップ        → デイリーサマリー（昨日分）
├── 週次総括タップ        → 週次サマリー                  [v1.1]
├── マイルストーンタップ  → マイルストーン達成画面
└── リマインド（後で）タップ → モード選択
```

---

## 7.3 入力UX

### 7.3.1 Quick モード

```
構成: 5カテゴリ × 5項目 = 25項目
操作: スワイプ（カテゴリ切替）+ タップ（スコア選択）
所要時間: 2-3分

フロー:
  1. ホーム画面で「Quick開始」をタップ
  2. カテゴリ1（Perceive / 眼）の5項目が表示
  3. 各項目の横に 1-5 のスコアボタン
  4. スコアをタップすると墨が染みるアニメーション + ハプティクス
  5. 右スワイプで次のカテゴリ（Think / 知）へ
  6. 5カテゴリ全て完了すると「完了」ボタンが出現
  7. 「完了」タップ → デイリーサマリーへ遷移

1画面の構成:
  ┌──────────────────────────────────────┐
  │  眼（Perceive）           2/5 完了   │  ← カテゴリ名 + 進捗
  │──────────────────────────────────────│
  │  P1: 一次ソースに当たる              │
  │  ○ ○ ○ ○ ○                          │  ← 1-5 スコアボタン
  │                                      │
  │  P2: 自分のバイアスを疑う            │
  │  ○ ○ ● ○ ○                          │  ← 3が選択済み
  │                                      │
  │  P3: 不確実な中で判断を下す          │
  │  ○ ○ ○ ● ○                          │  ← 4が選択済み
  │                                      │
  │  P4: シグナルとノイズを分ける        │
  │  ○ ○ ○ ○ ○                          │  ← 未選択
  │                                      │
  │  P5: 変化の兆しを捉える             │
  │  ○ ○ ○ ○ ○                          │
  │──────────────────────────────────────│
  │  ● ○ ○ ○ ○                          │  ← カテゴリインジケータ
  └──────────────────────────────────────┘
```

### 7.3.2 Deep モード

```
構成: Quick の全要素 + カテゴリごとのメモ入力欄
操作: Quick と同じスコア入力 + テキスト入力
所要時間: 10-15分

追加要素:
  各カテゴリのスコア入力後にテキスト入力欄が出現。
  プレースホルダー: 「今日の"眼"について気づいたこと...」
  スキップ可能（メモは任意）。

  ┌──────────────────────────────────────┐
  │  眼（Perceive）           5/5 完了   │
  │──────────────────────────────────────│
  │  （5項目のスコア入力部分）            │
  │──────────────────────────────────────│
  │  今日の"眼"について                  │
  │  ┌────────────────────────────────┐  │
  │  │ 今日の会議で、伝聞情報だけで    │  │  ← テキスト入力欄
  │  │ 判断しそうになった。一次ソース   │  │
  │  │ に当たる意識が足りなかった。     │  │
  │  └────────────────────────────────┘  │
  │                        [スキップ →]  │
  └──────────────────────────────────────┘
```

### 7.3.3 入力中断の挙動

```
一時保存（ドラフト）:
  - 入力途中のスコアは自動的にドラフトとして端末に保存
  - アプリがバックグラウンドに移行した場合も保持
  - アプリ再起動時にドラフトが存在する場合、復元ダイアログを表示:
    「前回の入力途中のデータがあります。続きから再開しますか？」
    [再開する] [破棄する]

  - 明示的な「破棄」操作時は確認ダイアログ:
    「入力途中のデータを破棄しますか？この操作は取り消せません。」
    [破棄する] [戻る]
```

### 7.3.4 04:00をまたぐ入力

```
ルール: 入力開始時点の論理日付を採用する

例1: 2/14 03:50 に入力開始 → 論理日付 = 2/13（04:00区切り）
     2/14 04:10 に入力完了 → DailyRecord.date = 2/13

例2: 2/14 04:05 に入力開始 → 論理日付 = 2/14
     2/14 04:20 に入力完了 → DailyRecord.date = 2/14

論理日付の計算:
  IF currentTime.hour < 4:
    logicalDate = currentDate - 1day
  ELSE:
    logicalDate = currentDate
```

### 7.3.5 同日再入力

```
ルール: UPSERT（上書き）

- 同一論理日付の DailyRecord が既に存在する場合:
  - Score は itemId をキーに UPSERT（新しいスコアで上書き）
  - CategoryMemo は category をキーに UPSERT
  - 即時フィードバックは最新スコアで再生成
  - AIキャッシュは破棄して再生成

- ホーム画面の表示:
  記録済みの日は「修正する」ボタンを表示（「Quick開始」「Deep開始」の代わりに）
```

---

## 7.4 デザインスタイルガイド

### 7.4.1 デザインコンセプト

**「現代の道場」**

伝統的な和の美意識を、現代のiOSデザインに昇華させる。
「古臭い和風」ではなく「洗練された和モダン」を目指す。

キーワード: 余白、墨、静謐、凛、温もり

### 7.4.2 カラーパレット

#### プライマリカラー

| 名前 | 用途 | Hex | 和色名 |
|---|---|---|---|
| **墨** | 背景（ダークモード主体） | `#1A1A1A` | 墨色（すみいろ） |
| **生成** | 背景（ライトモード） | `#F5F0E8` | 生成色（きなりいろ） |
| **藍** | アクセント・リンク | `#1B4B6B` | 藍色（あいいろ） |
| **朱** | 警告・叱咤・低スコア | `#C13B2A` | 朱色（しゅいろ） |
| **金** | 称賛・マイルストーン | `#B8860B` | 金茶（きんちゃ） |

#### カテゴリカラー（レーダーチャート・カテゴリ識別）

| カテゴリ | Hex | 和色名 | 選定理由 |
|---|---|---|---|
| Perceive（眼） | `#4A7C59` | 千歳緑（ちとせみどり） | 冷静な観察。常緑の知恵 |
| Think（知） | `#2E4F7B` | 紺色（こんいろ） | 深い思考。藍より深い知性 |
| Execute（剣） | `#8B2500` | 弁柄色（べんがらいろ） | 熱い実行力。刀の鍛冶の火 |
| Engage（人） | `#C17817` | 山吹色（やまぶきいろ） | 人との温かい交わり |
| Sustain（道） | `#5B4A6A` | 江戸紫（えどむらさき） | 高貴な修行。持続の気品 |

#### スコアカラー

| スコア | 色 | 意味 |
|---|---|---|
| 5 | `#B8860B`（金茶） | 卓越 |
| 4 | `#4A7C59`（千歳緑） | 良好 |
| 3 | `#6B6B6B`（灰色） | 普通 |
| 2 | `#C17817`（山吹） | 要注意 |
| 1 | `#C13B2A`（朱色） | 要改善 |

### 7.4.3 タイポグラフィ

| 用途 | フォント | ウェイト | サイズ | フォールバック |
|---|---|---|---|---|
| 武士のセリフ | **游明朝体**（YuMincho） | Demibold | 17pt | YuMincho Medium → HiraMinProN-W6 → system serif |
| 見出し | **ヒラギノ角ゴシック** | W6 | 20pt | system default |
| 本文 | **ヒラギノ角ゴシック** | W3 | 15pt | system default |
| スコア数値 | **SF Pro Display** | Bold | 36pt | system default |
| キャプション | **ヒラギノ角ゴシック** | W3 | 12pt | system default |

設計意図:
- 武士のセリフのみ明朝体。筆の運びを感じさせる
- UI全体はゴシック体で現代的な可読性を確保
- スコア数値は SF Pro で視認性最優先

### 7.4.4 UIコンポーネント

#### カード

```
背景: 半透明の和紙テクスチャ（opacity: 0.05 程度。主張しすぎない）
角丸: 8pt（和の直線的な美意識を保つ。丸すぎない）
ボーダー: なし。影のみ（影色: #000000 opacity 0.08）
内部パディング: 16pt
```

#### フィードバック表示エリア

```
背景: 縦書き巻物のメタファー（ただしテキストは横書き）
上下に巻物の端を模した装飾ライン
フォント: 游明朝体
テキストカラー:
  ダークモード時: #E8E0D0（生成色の明るい版）
  ライトモード時: #2A2A2A
```

#### スコア入力ボタン

```
形状: 正方形（角丸4pt）
サイズ: 44x44pt（タップ領域確保）
未選択時: ボーダーのみ（1pt, カテゴリカラー opacity 0.3）
選択時: カテゴリカラーで塗りつぶし + 白テキスト
アニメーション: タップ時に墨が染みるようなフィルアニメーション（0.2s ease-out）
ハプティクス: UIImpactFeedbackGenerator(.light)
```

#### レーダーチャート

```
5角形（5カテゴリ）
線色: カテゴリカラー
塗り: カテゴリカラー opacity 0.15
当日データ: 実線 + 塗り
7日平均: 破線のみ（比較用）
軸ラベル: カテゴリの武士道呼称（眼・知・剣・人・道）
背景: 同心の5角形グリッド（5段階に対応）
実装: Swift Charts ではなくカスタム Shape + Canvas で実装
```

#### ストリーク表示

```
アイコン: 和風の松明（たいまつ）アイコン（絵文字ではない）
連続日数: SF Pro Bold 36pt
ラベル: 「連続 ○日」
マイルストーン到達時: 金色のグロー効果
```

### 7.4.5 アイコンシステム

| アイコン | モチーフ | 用途 |
|---|---|---|
| Perceive | 目（書道的な一筆書き） | カテゴリ識別 |
| Think | 筆と巻物 | カテゴリ識別 |
| Execute | 刀 | カテゴリ識別 |
| Engage | 向かい合う人（家紋風） | カテゴリ識別 |
| Sustain | 道（遠近法の一本道） | カテゴリ識別 |
| Quick モード | 居合い抜き（一閃） | モード選択 |
| Deep モード | 座禅 | モード選択 |
| ストリーク | 松明 | ストリーク表示 |
| マイルストーン | 家紋 | 達成時表示 |

MVP ではカスタムアイコンの代わりに SF Symbols を使用。v1.1 でカスタムアイコンセットに差し替え。

### 7.4.6 アニメーション指針

| シーン | アニメーション | 時間 | イージング |
|---|---|---|---|
| スコアタップ | 墨が染みるフィル | 0.2s | ease-out |
| カテゴリ切り替え | 横スワイプ（巻物を広げる） | 0.3s | ease-in-out |
| フィードバック表示 | 下からフェードイン | 0.4s | ease-out |
| マイルストーン達成 | 金色の波紋エフェクト | 1.0s | ease-out |
| レーダーチャート描画 | 中心から頂点に向かって線が伸びる | 0.6s | ease-out |

全体方針: アニメーションは控えめ。「静」の中に「動」があるという和の美意識。派手なエフェクトは避ける。

### 7.4.7 ダークモード / ライトモード

| 要素 | ダークモード | ライトモード |
|---|---|---|
| 背景 | `#1A1A1A`（墨色） | `#F5F0E8`（生成色） |
| テキスト | `#E8E0D0` | `#2A2A2A` |
| カード背景 | `#2A2A2A` | `#FFFFFF` |
| 武士セリフ | `#E8E0D0` | `#3A2A1A`（焦茶） |
| アクセント | 変更なし | 変更なし |

**推奨デフォルト: ダークモード**
理由: 夜の使用が主（22時のリマインド）。墨色の背景が和の世界観と合致。

MVP はダークモードのみ。ライトモードは v1.1 で対応。

---

## 7.5 オンボーディング

初回起動時の体験。ユーザーを立花一門の世界に導入し、アプリの使い方を伝える。

### 7.5.1 フロー（5画面のスワイプ型）

#### 画面1: 世界観導入

```
┌──────────────────────────────────────┐
│                                      │
│           [立花家の家紋]              │
│                                      │
│     「あなたは立花宗茂である」        │   ← 游明朝体 Demibold
│                                      │
│  養父・立花道雪から兵法を学び、       │
│  実父・高橋紹運から覚悟を受け継ぎ、   │
│  妻・誾千代に叱咤され、               │
│  忠実な家臣団に支えられながら、       │
│  「西国無双」への道を歩む。           │
│                                      │
│              [次へ →]                │
│  ○ ○ ○ ○ ○                          │
└──────────────────────────────────────┘
```

#### 画面2: 5カテゴリ紹介

```
┌──────────────────────────────────────┐
│                                      │
│     「五つの鍛錬で己を磨け」          │
│                                      │
│  [目] 眼（Perceive）— 情報を正しく   │
│       集め、判断する                  │
│                                      │
│  [巻物] 知（Think）— 構造的に、      │
│         深く、鋭く考え抜く            │
│                                      │
│  [刀] 剣（Execute）— 速く、高品質に、│
│       やり抜いて形にする              │
│                                      │
│  [人] 人（Engage）— 伝え、聞き、     │
│       巻き込み、信頼を築く            │
│                                      │
│  [道] 道（Sustain）— 学び続け、      │
│       長期的に伸びる                  │
│                                      │
│              [次へ →]                │
│  ○ ● ○ ○ ○                          │
└──────────────────────────────────────┘
```

#### 画面3: スコアの付け方ガイド

```
┌──────────────────────────────────────┐
│                                      │
│      「日々の己を数で記せ」           │
│                                      │
│  5  卓越した行動ができた              │  ← 金茶
│     他者にも良い影響を与えた          │
│                                      │
│  4  意識的に良い行動ができた          │  ← 千歳緑
│                                      │
│  3  ある程度できた（普通）            │  ← 灰色
│                                      │
│  2  意識はしたが行動に至らなかった    │  ← 山吹
│                                      │
│  1  まったくできなかった              │  ← 朱色
│     意識すらしなかった                │
│                                      │
│  毎日2分。完璧を目指さなくてよい。    │
│  記録を続けること自体が鍛錬である。   │
│                                      │
│              [次へ →]                │
│  ○ ○ ● ○ ○                          │
└──────────────────────────────────────┘
```

#### 画面4: 通知時刻設定

```
┌──────────────────────────────────────┐
│                                      │
│    「一門があなたに語りかける」        │
│                                      │
│  朝の檄文（道雪）                    │
│  [07:00]  ← 時刻ピッカー             │
│  「今日の心構えを授ける」             │
│                                      │
│  陣中見舞い（家臣団）                │
│  [12:30]  ← 時刻ピッカー             │
│  「午後の行動を促す」                 │
│                                      │
│  夕べの下知（誾千代）                │
│  [22:00]  ← 時刻ピッカー             │
│  「振り返りの入力を誘導する」         │
│                                      │
│              [次へ →]                │
│  ○ ○ ○ ● ○                          │
└──────────────────────────────────────┘

※ このタイミングで通知許可ダイアログを表示
```

#### 画面5: AIフィードバック設定

```
┌──────────────────────────────────────┐
│                                      │
│    「知将の助言を受けるか？」          │
│                                      │
│  AIフィードバックを有効にすると、     │
│  立花一門がスコアに基づいた           │
│  文脈依存の助言をくれます。           │
│                                      │
│  [APIキーを入力]                      │
│  ┌────────────────────────────────┐  │
│  │ sk-ant-...                      │  │  ← セキュア入力
│  └────────────────────────────────┘  │
│                                      │
│  APIキーは端末内に暗号化保存されます。│
│  スコア数値のみ送信し、               │
│  メモ本文は送信しません。             │
│                                      │
│  [スキップ（テンプレートで開始）]      │
│                                      │
│           [始める]                    │  ← メインCTA
│  ○ ○ ○ ○ ●                          │
└──────────────────────────────────────┘
```

### 7.5.2 オンボーディング完了後

```
オンボーディング完了
  → ホーム画面に遷移
  → 初回のみ道雪のウェルカムメッセージを表示:
    「宗茂よ、今日からお主の修行が始まる。
     恐れるな。某がついておる。」
  → 完了フラグを UserDefaults に保存（2回目以降はスキップ）
```

---

## 7.6 ウィジェット仕様

### 7.6.1 ホーム画面ウィジェット

```
サイズ: Small（2x2）/ Medium（4x2）

Small:
  ┌────────────────┐
  │  3.8  ↑        │  ← 総合スコア + トレンド矢印
  │  連続14日       │  ← ストリーク
  │                │
  │  「今日の動き、 │  ← 武将の一言（1行）
  │   見事であった」│
  └────────────────┘

Medium:
  ┌────────────────────────────────────┐
  │  3.8  ↑  連続14日                  │  ← スコア + トレンド + ストリーク
  │──────────────────────────────────────│
  │  眼 3.6  知 4.0  剣 4.2  人 3.8  道 3.4  │  ← カテゴリ別ミニバー
  │──────────────────────────────────────│
  │  「今日の"剣"、切れ味が違ったな」    │  ← 武将の一言
  │  「敵を知り己を知れば百戦危うからず」│  ← 名言
  └────────────────────────────────────┘

表示内容:
  - 今日のスコア（未入力時は「---」）
  - 7日移動平均との比較（↑ / ↓ / →）
  - ストリーク日数
  - 武将の一言（最新のフィードバックnotification文）
  - 名言（武士道・兵法書からの引用。日替わり）

タップ時: アプリのホーム画面に遷移
```

### 7.6.2 ロック画面ウィジェット

```
サイズ: Inline / Circular / Rectangular

Inline:
  「連続14日 | 3.8 ↑」

Circular:
  ┌────┐
  │ 14 │  ← ストリーク日数（大きく）
  │ 日 │
  └────┘

Rectangular:
  ┌──────────────┐
  │ 連続14日  3.8│
  │「刀を磨き   │  ← 名言（1行）
  │ 続けよ」    │
  └──────────────┘

タップ時: アプリのホーム画面に遷移
```

### 7.6.3 ウィジェット更新タイミング

```
更新トリガー:
  1. チェック完了時: WidgetCenter.shared.reloadAllTimelines()
  2. 朝の通知時: Timeline のエントリーとしてスケジュール
  3. 04:00（日付変更時）: 新しい論理日付に切り替え

データ共有:
  - App Group を使用してメインアプリとウィジェット間でデータ共有
  - 共有データ: 最新のスコア、ストリーク日数、最新フィードバックテキスト、名言

Timeline Provider:
  - getTimeline() で次の更新ポイント（04:00, 朝の通知時刻）をエントリーとして返却
  - スコア未入力状態では placeholder 表示
```

---

## 7.7 サウンド指針（オプション）

| シーン | サウンド | 特徴 |
|---|---|---|
| スコアタップ | 木魚の軽い音 or 無音 | 連続タップ時に煩くならないよう最小限 |
| チェック完了 | 鈴（りん）の一打ち | 禅寺の鈴。完了の区切り |
| マイルストーン | 太鼓の一打ち | 重みのある達成感 |
| デフォルト | 無音 | 基本は無音。サウンドは設定でオン |

MVP ではサウンドは実装しない（v1.2）。

---

## 7.8 数値の和風変換ルール

フィードバック等で武士のセリフに数値を含む場合の変換ルール。

```
スコア表示（UI上）: アラビア数字のまま（視認性優先）
  例: 4.2 → 「4.2」

セリフ内の数値: 文脈で使い分け
  連続日数: 漢数字（「七日」「三十日」「六十六日」）
  スコア差分: 漢数字（「〇・五の向上」）
  順位: 序数（「第一の領域」）

日付: 和暦は使用しない（実用性を損なうため）
```

---

## 7.9 名言カード仕様

04_characters.md 4.2 で定義された名言をアプリ内で常時表示する。

### 7.9.1 ホーム画面の日替わり名言カード

```
表示位置: ホーム画面上部（ストリーク表示の下、モード選択ボタンの上）
サイズ: 横幅100%（左右パディング20pt）、高さは内容に応じて可変
背景: 半透明の和紙テクスチャ（opacity: 0.03）
フォント: 游明朝体 Demibold 20pt
テキストカラー: #E8E0D0（ダークモード）/ #3A2A1A（ライトモード）
出典: 名言下部に人物名を小さく表示（ヒラギノ角ゴ W3 12pt, opacity 0.6）
枠: 上下に細い金茶のライン（0.5pt）
パディング: 上下24pt、左右20pt
タップ動作: 名言一覧画面へ遷移
選出ロジック: 日付ベースのローテーション。お気に入り登録された名言を優先
```

### 7.9.2 ロード/遷移時の名言表示

```
表示タイミング: 画面遷移の間（特にチェック入力→デイリーサマリーの遷移時）
表示方式: 画面中央にフェードイン（0.8s ease-in-out）→ 0.5s保持 → フェードアウト（0.5s）
フォント: 游明朝体 Demibold 17pt
背景: 墨色（全画面オーバーレイ）
選出ロジック: ランダム。直前と重複しない
```

### 7.9.3 デイリーサマリーの名言表示

```
表示位置: デイリーサマリー画面の最下部
選出ロジック: 当日のスコアに最も関連するカテゴリの名言を選出
  - 最高カテゴリ or 最低カテゴリの接続カテゴリ（04_characters.md 4.2.1 参照）で選出
  - 同一カテゴリに複数名言がある場合はランダム
```

### 7.9.4 名言一覧画面

```
表示: 全名言をリスト表示（宗茂 → 道雪 → 紹運の順）
お気に入り: 各名言にハートアイコン。タップでお気に入り登録/解除
フィルタ: 「全て」「お気に入り」のセグメントコントロール
各名言の表示:
  - 名言テキスト（游明朝体 Demibold 15pt）
  - 出典（人物名 + 背景の1行説明、ヒラギノ角ゴ W3 12pt）
  - お気に入りアイコン（右端）
お気に入りデータ: UserDefaults に保存
```

---

## 7.10 フィードバック表示のTone別デザイン

フィードバック表示エリアのデザインは Tone（04_characters.md 4.5.1）に応じて変化する。

| Tone | 背景アクセント | 装飾ラインの色 | キャラアイコン枠の色 | 使用場面 |
|---|---|---|---|---|
| positive | `#B8860B` opacity 0.05 | 金茶 `#B8860B` | 金茶 | praise, celebrate |
| negative | `#C13B2A` opacity 0.05 | 朱 `#C13B2A` | 朱 | rebuke, warn |
| neutral | `#1B4B6B` opacity 0.05 | 藍 `#1B4B6B` | 藍 | advise, encourage |
| celebratory | `#B8860B` opacity 0.10 | 金茶 `#B8860B`（太め1pt）| 金茶 + グロー | milestone |

---

## 7.11 マイルストーン達成画面の詳細仕様

04_characters.md 4.4.2 の解放演出仕様に準拠。

```
┌──────────────────────────────────────┐
│                                      │
│         (金色の光が広がる)            │  背景: 墨色 → 金色グロー（1.5s）
│                                      │
│                                      │
│          — 西国無双 —                 │  マイルストーン名
│                                      │  游明朝体 Demibold 24pt、金茶
│          六十六日                     │
│                                      │
│  紹運「六十六日。                     │  セリフ表示:
│   我が息子よ、                       │  タイプライター風（50ms/文字）
│   お主はもう"西国無双"だ。」          │  游明朝体 Demibold 17pt
│                                      │
│                                      │
│         (タップで閉じる)              │  自動では閉じない
│                                      │
└──────────────────────────────────────┘

サウンド（ON時）: 太鼓の一打ち
ハプティクス: UINotificationFeedbackGenerator(.success)
閉じ方: 画面タップで閉じる → ホーム画面に戻る
```

---

## 7.12 エラーハンドリングUI

### 7.12.1 データ保存失敗

```
SwiftData への書き込みが失敗した場合:
  → Alert「データの保存に失敗しました」
  → メッセージ: 「端末の空き容量をご確認ください」
  → ボタン: 「再試行」「閉じる」
  → 入力データはメモリ上に保持（アプリ終了まで）
```

### 7.12.2 通知権限拒否

```
設定画面に警告バナーを表示:
  「通知が無効です。設定アプリで通知を許可してください」
  → タップで UIApplication.openSettingsURLString を起動
  → ホーム画面にはバナーを表示しない（控えめ）
```

### 7.12.3 AI API エラー

```
Claude API エラー時:
  → テンプレートベースにサイレントフォールバック
  → ユーザーにはフォールバックであることを明示しない

APIキー無効時:
  → 設定画面の AI セクションに「APIキーが無効です」テキスト表示
  → 再入力を促すリンク
```

---

## 7.13 アクセシビリティ方針（v1.1）

| 対応項目 | 方針 |
|---|---|
| **VoiceOver** | 全インタラクティブ要素にアクセシビリティラベルを設定。スコアボタン: 「{項目名}, スコア {N}」 |
| **Dynamic Type** | 武士セリフを除くUIテキストは Dynamic Type 対応。游明朝体は固定サイズを維持 |
| **コントラスト比** | WCAG AA 準拠（4.5:1 以上）。朱色 `#C13B2A` on 墨色 `#1A1A1A` = 4.8:1（AA 達成） |
| **Reduce Motion** | アニメーション設定がオフの場合、全アニメーションをスキップ |
| **色覚多様性** | スコアは色だけでなく数値でも表示。レーダーチャートは形状でも区別可能に |

---

## 7.14 MVPスコープ定義（UI/UX）

| 機能 | MVP | v1.1 | v1.2 |
|---|---|---|---|
| オンボーディング（5画面） | YES | | |
| ホーム画面（名言カード + モード選択） | YES | | |
| Quick / Deep チェック入力 | YES | | |
| デイリーサマリー（レーダーチャート + フィードバック） | YES | | |
| フィードバック詳細画面 | YES | | |
| マイルストーン達成画面 | YES | | |
| 名言一覧（お気に入り機能） | YES | | |
| 設定画面（通知 + AI + アプリ情報） | YES | | |
| ダークモード（デフォルト・ダークのみ） | YES | | |
| ホーム画面ウィジェット（Small / Medium） | YES | | |
| ストリーク表示 + ウィジェット（Small） | YES | | |
| 入力中断時の一時保存（ドラフト） | YES | | |
| 週次サマリー画面 | | YES | |
| ダッシュボード（週次 / 月次 / ロングターム） | | YES | |
| 項目詳細画面 | | YES | |
| CSV エクスポート | | YES | |
| ライトモード対応 | | YES | |
| カテゴリウィジェット（Medium） | | YES | |
| ロック画面ウィジェット（Inline / Circular） | | YES | |
| カスタムアイコンセット | | YES | |
| アクセシビリティ対応（VoiceOver / Dynamic Type） | | YES | |
| エビデンス参照カード（ロングプレスで表示） | | | YES |
| サウンドエフェクト（鈴・太鼓） | | | YES |
| 和紙テクスチャの洗練 | | | YES |


---

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


---

