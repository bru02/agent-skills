---
name: rn-testing-and-debugging
description: Covers API failure simulation using MITMProxy for React Native and web apps. Use when simulating API failures during development.
license: MIT
metadata:
  author: Callstack
  tags: react-native, testing, mitmproxy, debugging, api
---

# React Native Testing & Debugging

## Overview

Practical guidance for API failure simulation. A MITMProxy-based solution for testing API error resilience.

## When to Apply

Reference these guidelines when:
- Simulating API failures or error responses during development
- Debugging API error handling in React Native or web apps

## Quick Reference

| File | Description |
|------|-------------|
| [api-failure-simulation.md][api-failure-simulation] | MITMProxy + Redis setup for API response overriding |

## Problem → Skill Mapping

| Problem | Start With |
|---------|------------|
| Need to simulate API errors | [api-failure-simulation.md][api-failure-simulation] |
| Test error handling without backend changes | [api-failure-simulation.md][api-failure-simulation] |

[api-failure-simulation]: references/api-failure-simulation.md
