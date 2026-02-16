---
name: rn-testing-and-debugging
description: Covers E2E testing framework comparison (Detox vs Maestro) and API failure simulation using MITMProxy for React Native and web apps. Use when choosing an E2E testing framework, setting up Maestro on CI, or simulating API failures during development.
license: MIT
metadata:
  author: Callstack
  tags: react-native, testing, e2e, detox, maestro, mitmproxy, debugging, api, ci
---

# React Native Testing & Debugging

## Overview

Practical guidance for E2E testing framework selection and API failure simulation. Based on real-world project experience with Detox and Maestro, plus a MITMProxy-based solution for testing API error resilience.

## When to Apply

Reference these guidelines when:
- Choosing between Detox and Maestro for E2E testing
- Setting up Maestro tests on CI (GitHub Actions)
- Simulating API failures or error responses during development
- Debugging API error handling in React Native or web apps
- Evaluating E2E testing costs for OSS or budget-constrained projects

## Quick Reference

| File | Description |
|------|-------------|
| [detox-vs-maestro.md][detox-vs-maestro] | Real-world Detox vs Maestro comparison with CI setup |
| [api-failure-simulation.md][api-failure-simulation] | MITMProxy + Redis setup for API response overriding |

## Problem → Skill Mapping

| Problem | Start With |
|---------|------------|
| Choosing E2E testing framework | [detox-vs-maestro.md][detox-vs-maestro] |
| Detox flaky on CI | [detox-vs-maestro.md][detox-vs-maestro] |
| Setting up Maestro on GitHub Actions | [detox-vs-maestro.md][detox-vs-maestro] |
| Need to simulate API errors | [api-failure-simulation.md][api-failure-simulation] |
| Test error handling without backend changes | [api-failure-simulation.md][api-failure-simulation] |

[detox-vs-maestro]: references/detox-vs-maestro.md
[api-failure-simulation]: references/api-failure-simulation.md
