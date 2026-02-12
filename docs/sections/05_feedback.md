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
