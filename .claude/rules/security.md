# Security Rules

## API Keys
- MUST be stored in Keychain Services only
- NEVER in UserDefaults, plist, or source code
- NEVER log API keys

## AI Communication (Claude API)
- Send: category scores (numbers), category names, trend direction, streak count
- NEVER send: memo text, user name, device info, full history
- Use AIRequestContext struct to enforce what's sent

## Data Privacy
- All user data stays on device (SwiftData)
- No analytics or tracking
- No network calls except Claude API
- iCloud backup follows iOS standard behavior

## Input Validation
- Score values: validate 1-5 range
- Date values: validate with LogicalDate
- API responses: validate JSON structure before parsing
- Widget data: validate shared App Group data integrity
