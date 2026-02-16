---
title: Detox vs Maestro
impact: HIGH
tags: e2e, testing, detox, maestro, ci, github-actions, yaml, mobile-testing
---

# Skill: Detox vs Maestro

Real-world comparison of Detox and Maestro for React Native E2E testing, based on production project experience.

## Quick Reference

| Aspect | Detox | Maestro |
|--------|-------|---------|
| Test format | JavaScript/TypeScript | YAML |
| CI reliability | Flaky, often disabled | Stable with proper setup |
| Test authoring | Developers only | Manual testers can contribute |
| Mocking support | Built-in network mocking | None (young ecosystem) |
| Setup complexity | High (native build deps) | Low (standalone binary) |
| Maintenance | High | Low (YAML tests are simple) |

## When to Use

- Evaluating E2E testing frameworks for a React Native project
- Experiencing CI flakiness with Detox
- Need non-developers to write E2E tests
- Considering Maestro for an open-source project

## The Case for Maestro

### Detox Pain Points (Real-World)

On a production project, Detox tests were:
- **Disabled on CI** due to persistent unreliability
- Flaky across different CI environments
- Difficult to debug when failures occurred
- Required developer involvement for every test change

### Maestro Advantages

**YAML-based tests** — Non-developers can author tests:

```yaml
appId: com.example.myapp
---
- launchApp
- tapOn: "Login"
- inputText:
    id: "email-input"
    text: "test@example.com"
- inputText:
    id: "password-input"
    text: "password123"
- tapOn: "Submit"
- assertVisible: "Welcome"
```

**Maestro Studio** — Visual test recorder:
```bash
maestro studio
```

Opens a browser UI for recording interactions and generating YAML tests.

**Scale**: On one project a manual tester wrote 100+ YAML tests without developer assistance.

### CI Setup with GitHub Actions

No official Maestro CI guide exists. Use a self-hosted approach based on Stripe's GitHub Actions setup:

```yaml
name: E2E Tests
on: [pull_request]

jobs:
  maestro-tests:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Maestro
        run: |
          curl -Ls "https://get.maestro.mobile.dev" | bash
          echo "$HOME/.maestro/bin" >> $GITHUB_PATH

      - name: Build app
        run: |
          # Build the iOS app for simulator
          npx react-native build-ios --mode Release --simulator "iPhone 16"

      - name: Boot simulator
        run: |
          xcrun simctl boot "iPhone 16"

      - name: Install app
        run: |
          xcrun simctl install booted build/Build/Products/Release-iphonesimulator/MyApp.app

      - name: Run Maestro tests
        run: |
          maestro test .maestro/
```

### Cost Considerations

- **Maestro Cloud**: Pricing too high for OSS / budget-constrained projects
- **Self-hosted GitHub Actions**: Free for public repos, cost-effective for private
- **Local development**: Maestro CLI is free and open-source

## Maestro Limitations

- **No mocking**: Cannot mock network requests or native modules natively
- **Young ecosystem**: Fewer community resources and plugins than Detox
- **Limited assertions**: Assertion API is simpler than Detox's
- **Platform gaps**: Some platform-specific interactions may not be supported

### Workarounds for Missing Mocking

- Use environment variables to switch API endpoints in the app
- Set up a mock API server (MSW, json-server) running alongside tests
- Use build flavors/schemes to inject test configurations

## Decision Matrix

| Scenario | Recommendation |
|----------|---------------|
| Non-developers write tests | Maestro |
| Need network mocking | Detox |
| CI reliability is critical | Maestro |
| Large test suite (100+ tests) | Maestro (easier maintenance) |
| Deep native interaction testing | Detox |
| OSS project (budget-sensitive) | Maestro (self-hosted) |
| Existing Detox suite working well | Keep Detox |

## Common Pitfalls

- **Assuming Maestro Cloud is required**: Self-hosted runners work well and are cost-effective
- **Missing CI setup docs**: No official guide exists — use community examples (Stripe's GH Actions)
- **Expecting Detox-level mocking**: Plan alternative mocking strategies when using Maestro
- **Not using Maestro Studio**: Dramatically speeds up test creation for non-developers

## Related Skills

- [api-failure-simulation.md](./api-failure-simulation.md) — Simulate API failures for testing error handling
