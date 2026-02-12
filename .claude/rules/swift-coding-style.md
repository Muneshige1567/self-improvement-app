# Swift Coding Style Rules

## Architecture
- Follow DDD: Domain layer has ZERO framework dependencies
- MVVM + Repository pattern
- 4 Bounded Contexts: Check, Feedback, Character, Notification
- Domain events for cross-context communication

## Swift Style
- Use Swift concurrency (async/await) over completion handlers
- Prefer value types (struct/enum) over reference types (class) in Domain
- Use @Observable for ViewModels (iOS 17+)
- Use @Model for SwiftData entities (Infrastructure layer only)
- Mark UI-bound code with @MainActor
- Use Result type or throws for error handling, never force unwrap

## Naming
- Types: PascalCase (FeedbackEngine, DailyRecord)
- Properties/functions: camelCase (totalAverage, evaluate())
- Protocols: adjective or -able suffix (Renderable) or noun (Repository)
- Enum cases: camelCase (perceive, think, execute)
- Constants: camelCase (maxStreakDays)

## File Organization
- One primary type per file
- File name matches primary type name
- Extensions in separate files if >30 lines
- Keep files under 400 lines

## SwiftUI
- Views should be small (<100 lines of body)
- Extract subviews as separate structs
- Use @State for local, @Binding for passed, @Environment for global
- Prefer .task over .onAppear for async work

## Error Handling
- Use typed errors (enum conforming to Error)
- Always handle errors explicitly
- Log errors but don't crash (no fatalError in production)
- AI failures fallback to templates silently

## Comments
- Only add comments where logic isn't self-evident
- No TODO without linked task
- Japanese comments are OK for domain-specific terms (武士道呼称 etc.)
