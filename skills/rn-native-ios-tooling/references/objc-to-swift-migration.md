---
title: Objective-C to Swift Migration
impact: HIGH
tags: objc, swift, migration, ios, bridging-header, refactoring, native
---

# Skill: Objective-C to Swift Migration

Plan and execute an Objective-C to Swift migration for iOS native code in React Native projects.

## Quick Reference

| Phase | Key Actions |
|-------|------------|
| Before | Investigate code, understand flows, simplify ObjC first |
| During | Migrate by module, use bridging headers, rewrite when needed |
| After | Migrate unit tests, update docs, apply SOLID principles |

## When to Use

- Planning migration of native iOS modules from ObjC to Swift
- React Native native module needs Swift rewrite
- Brownfield app has legacy ObjC code
- Team wants to leverage Swift-only features (async/await, structured concurrency)

## Prerequisites

- Xcode 15+ with Swift 5.9+
- Understanding of both Objective-C and Swift
- Existing unit test coverage (or willingness to add it)
- Bridging header knowledge

## Before Migration

### 1. Investigate the Code

- Map all ObjC files and their dependencies
- Identify interconnected modules and isolated components
- Document threading model, queuing, and dispatch patterns
- Note any C/C++ interop (Swift's C interop is more limited)

### 2. Understand Flows and Threading

- Trace execution paths through ObjC code
- Document GCD/NSOperationQueue usage patterns
- Identify main thread vs background thread boundaries
- Note any `@synchronized` blocks or locking mechanisms

### 3. Simplify First

Before migrating, clean up the ObjC:
- Remove dead code
- Break large classes into smaller ones
- Reduce coupling between modules
- Fix existing bugs (don't carry them into Swift)

### 4. Plan the Migration Order

Start with the **smallest-impact, most-isolated** module:
- Utility classes with no dependencies
- Data models / value objects
- Standalone helpers

Work outward toward more connected code.

## During Migration

### 5. Migrate Module by Module

**Do not attempt a "big bang" rewrite.** Migrate one module at a time:

1. Create Swift file(s) for the module
2. Set up bridging header for ObjC → Swift interop
3. Migrate the implementation
4. Update callers to use the Swift version
5. Remove old ObjC files
6. Verify tests pass

### 6. Use Bridging Headers

**ObjC calling Swift:**
```objc
// Import the auto-generated Swift header
#import "MyApp-Swift.h"

// Use Swift class
MySwiftClass *obj = [[MySwiftClass alloc] init];
```

**Swift calling ObjC:**
```swift
// In MyApp-Bridging-Header.h, add:
// #import "MyObjCClass.h"

// Then use directly in Swift
let obj = MyObjCClass()
```

**Tip**: Expose only what's needed via `@objc` and `@objcMembers`. Don't make everything visible.

### 7. Don't Hesitate to Rewrite

- Translating ObjC patterns 1:1 into Swift often produces poor Swift code
- Use Swift idioms: optionals instead of nil checks, enums with associated values, structs for value types
- Rewrite delegate patterns as closures where appropriate
- Replace NSNotificationCenter with Combine or async/await where practical

### 8. Leverage Swift Features

| ObjC Pattern | Swift Replacement |
|-------------|-------------------|
| Delegate + protocol | Closure / async-await |
| NSError** | throws / Result type |
| GCD dispatch_async | Task { } / structured concurrency |
| KVO | @Published + Combine |
| NSMutableArray | [Element] (value type) |
| Category | Extension |
| typedef enum | enum with raw values |

### 9. Migrate Unit Tests Early

- Migrate tests for a module **alongside** the module itself
- Swift tests can call both ObjC and Swift code
- Use XCTest's Swift API directly
- Don't leave test migration for "later" — it rarely happens

### 10. Apply SOLID Principles

Migration is an opportunity to improve architecture:
- **Single Responsibility**: Split classes that do too much
- **Open/Closed**: Use protocols for extensibility
- **Liskov Substitution**: Ensure protocol conformances are correct
- **Interface Segregation**: Prefer small, focused protocols
- **Dependency Inversion**: Inject dependencies rather than hard-coding

## Documentation

Keep documentation updated throughout:
- Update API docs as interfaces change
- Document any behavior changes (even if intentional improvements)
- Note breaking changes for consumers of the migrated code
- Update README with new Swift requirements

## React Native Specific Considerations

### Native Modules

When migrating a RN native module from ObjC to Swift:

```swift
import React

@objc(MyModule)
class MyModule: NSObject {
    @objc static func requiresMainQueueSetup() -> Bool {
        return false
    }

    @objc func doSomething(
        _ resolve: @escaping RCTPromiseResolveBlock,
        reject: @escaping RCTPromiseRejectBlock
    ) {
        resolve(["result": "success"])
    }
}
```

The `RCT_EXTERN_MODULE` macro still requires an ObjC file:

```objc
// MyModule.m
#import <React/RCTBridgeModule.h>

@interface RCT_EXTERN_MODULE(MyModule, NSObject)
RCT_EXTERN_METHOD(doSomething:
                  (RCTPromiseResolveBlock)resolve
                  reject:(RCTPromiseRejectBlock)reject)
@end
```

### Turbo Modules

For New Architecture Turbo Modules, prefer Swift with `@objc` annotations. The codegen bridge handles the interop layer.

## Common Pitfalls

- **"Big bang" migration**: Migrating everything at once introduces too many bugs simultaneously
- **1:1 translation**: Produces un-idiomatic Swift code. Rewrite to use Swift patterns.
- **Ignoring threading differences**: Swift concurrency model differs from GCD patterns
- **Skipping test migration**: Leads to untested Swift code
- **Making everything @objcMembers**: Only expose what ObjC callers actually need
- **Not simplifying ObjC first**: Migrating messy ObjC produces messy Swift

## Related Skills

- [ios-dependency-managers.md](./ios-dependency-managers.md) — Dependencies may change during migration
- [oss-licensing-tools.md](./oss-licensing-tools.md) — Verify license compliance after dependency changes
