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
