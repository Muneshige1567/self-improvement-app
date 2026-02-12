# 最強社会人チェックリスト — Claude Code Project Guide

## Project Overview

iOS self-improvement app where the user IS Tachibana Muneshige (立花宗茂).
Sengoku-era characters provide feedback on daily self-check scores.

**Core business logic**: 25 evaluation items across 5 categories (眼/知/剣/人/道) that measure professional growth.
**Expression layer**: Historical characters (道雪/誾千代/紹運/家臣団) deliver feedback in authentic voice.

## Tech Stack

- **Platform**: iOS 17.0+ (iPhone)
- **UI**: SwiftUI
- **Data**: SwiftData (local-first)
- **AI**: Claude API (Anthropic) for feedback generation
- **Widgets**: WidgetKit
- **Notifications**: UserNotifications (rich notifications)
- **Architecture**: DDD + MVVM + Repository pattern

## Project Structure

```
SaikouChecklist/
├── Sources/
│   ├── App/                    # App entry point, DI container
│   ├── Domain/                 # ZERO framework dependencies
│   │   ├── Check/              # DailyRecord, Score, CheckItem, LogicalDate
│   │   ├── Feedback/           # FeedbackEngine, FeedbackResult, templates
│   │   ├── Character/          # CharacterOrchestrator, Character, messages
│   │   └── Notification/       # NotificationPayload, scheduling rules
│   ├── Infrastructure/         # Framework implementations
│   │   ├── Persistence/        # SwiftData repositories
│   │   ├── AI/                 # Claude API client
│   │   └── Notification/       # UNUserNotification implementation
│   └── Presentation/           # SwiftUI views + ViewModels
│       ├── Home/
│       ├── CheckInput/
│       ├── Summary/
│       ├── Dashboard/
│       ├── Settings/
│       └── Shared/             # Reusable components (RadarChart, QuoteCard)
├── Resources/                  # Assets, fonts, localization
├── Widget/                     # WidgetKit extension
└── Tests/
    ├── Domain/                 # 90%+ coverage required
    ├── Infrastructure/
    └── Presentation/
```

## Key Domain Concepts

- **LogicalDate**: Day boundary at 04:00 (not midnight). 03:59 = yesterday.
- **FeedbackEngine**: Priority 0-7 evaluation. One type per feedback.
- **CharacterOrchestrator**: Selects character based on feedback type + unlock state.
- **StreakState**: Consecutive days tracked. Milestones map to Tachibana history.
- **Null Score**: Excluded from averages (not treated as 0).

## Development Workflow (MUST FOLLOW)

全ての実装作業は以下のフローに従うこと。プロンプトで特に指示がなくても自動的にこの順序で進める。

### 1. 仕様確認 (Spec First)
- `docs/sections/` から該当する要件定義を読む
- 実装対象の仕様・制約・エッジケースを把握する
- 不明点があればユーザーに確認する（勝手に解釈しない）

### 2. テスト作成 (RED)
- 仕様に基づいてテストを先に書く（Swift Testing `@Test` + `#expect`）
- テスト命名: `test_[unit]_[scenario]_[expected]`
- エッジケース（LogicalDate 04:00境界、null score、cold start 等）も含める
- この時点でテストは失敗する状態であること

### 3. 最小実装 (GREEN)
- テストを通すための最小限のコードを書く
- 過剰設計しない。動くことを最優先

### 4. リファクタリング (REFACTOR)
- テストが通った状態を維持しながら改善
- DDD 原則（Domain層のフレームワーク非依存）を確認
- 命名・構造の整理

### 5. レビュー (VERIFY)
- DDD 違反がないか確認（Domain に `import SwiftUI` 等がないか）
- セキュリティルール違反がないか
- テストカバレッジが Domain 90%+ を維持しているか

**省略禁止**: 「テスト後で書きます」は不可。必ずテストから書くこと。

## Critical Rules

1. **Domain layer = NO framework imports**. No SwiftUI, no SwiftData in Domain/.
2. **User IS Muneshige**. Characters address the user as 宗茂/あなた/殿.
3. **Characters are authentically harsh**. パワハラ上等. Don't soften for UX.
4. **Offline-first**. AI is enhancement. App works fully with templates.
5. **API keys in Keychain ONLY**. Never UserDefaults, never plist.
6. **Memos never sent to AI**. Only scores, categories, and trends.

## Requirements Documents

Unified requirements: `docs/requirements.md` (consolidated from docs/sections/)
Section sources: `docs/sections/01_overview.md` through `08_technical.md`
Character research: `docs/sections/characters/*.md`

## Useful Commands

```bash
# Build (on Mac)
xcodebuild build -scheme SaikouChecklist -destination 'platform=iOS Simulator,name=iPhone 15'

# Test (on Mac)
xcodebuild test -scheme SaikouChecklist -destination 'platform=iOS Simulator,name=iPhone 15'
```

## References

- everything-claude-code configs: `C:\Users\shibu\dev\everything-claude-code\`
- ECC agents/skills/hooks adapted for this project in `.claude/`
