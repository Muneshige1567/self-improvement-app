# Testing Rules

## Coverage Targets
- Domain layer: 90%+ (business logic is the core value)
- Infrastructure layer: 70%+
- Presentation layer: 60%+
- Overall: 80%+

## Test Framework
- Use Swift Testing (@Test, #expect) for new tests
- XCTest is acceptable for existing patterns
- No third-party test frameworks

## What to Test
- Every FeedbackEngine priority rule
- Every StreakCalculator scenario
- Every CharacterOrchestrator selection path
- LogicalDate boundary cases (03:59 vs 04:00)
- Null score handling in averages
- Cold start threshold variations

## Test Organization
```
SaikouChecklistTests/
├── Domain/
│   ├── FeedbackEngineTests.swift
│   ├── StreakCalculatorTests.swift
│   ├── CharacterOrchestratorTests.swift
│   ├── ScoreSnapshotTests.swift
│   └── LogicalDateTests.swift
├── Infrastructure/
│   ├── DailyRecordRepositoryTests.swift
│   └── ClaudeAPIClientTests.swift
└── Presentation/
    ├── HomeViewModelTests.swift
    └── CheckInputViewModelTests.swift
```

## Test Naming
- `test_[unit]_[scenario]_[expected]`
- Example: `test_feedbackEngine_highScoreOnDay1_returnsPraiseWithLowThreshold`

## Stubs and Mocks
- Create `.stub()` factory methods on domain types
- Use protocol-based dependency injection for mocking
- Never mock what you own in Domain layer
