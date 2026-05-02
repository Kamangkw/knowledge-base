---
title: Hermes Agent Memory System: Configuration, Backends and Best Practices
date: 2026-05-02
source: Self-research (hermes-agent SKILL.md, Memory Tool docs)
tags: [Hermes-Agent, Memory, Configuration, Persistence]
description: Deep dive into Hermes Agent Memory system - multiple memory backends, configuration, cross-session memory optimization.
---

# Hermes Agent Memory System

## Overview

Memory allows Hermes Agent to maintain awareness of user preferences, project context, and long-term goals across sessions without requiring background information to be re-entered each time.

## Memory Content Types

### User Profile (USER.md)
- Basic user info and preferences
- Communication style
- Device and platform constraints
- Professional background

### System Memory (MEMORY.md)
- Current project tech stack
- Recent task objectives
- Environment configuration
- API key locations

## Backend Options

| Backend | Features | Best For |
|---------|----------|----------|
| Built-in (default) | Basic KV store | General users |
| Honcho | Structured profiles | Strict user profiles |
| Mem0 | Professional long-term memory | Advanced users |
| Custom | Any compatible backend | Developers |

## Backend Configuration

hermes memory setup
hermes memory status
hermes memory off

Or in config.yaml:
memory:
  provider: mem0
  memory_enabled: true
  user_profile_enabled: true

## Known Limitation: Memory Tool Matching

Memory Tool does NOT store entries as plain text. memory remove and memory replace require exact internal format match.

Workarounds:
1. Use replace instead of remove (more likely to succeed)
2. Edit MEMORY.md file directly if using disk backend
3. Verify actual stored content via session_search before operating
4. Backup important preferences to Skill files (Skills are more reliable)

## Best Practices

### Should Remember
- User preferences (dislikes complex solutions, iPhone only)
- Environment facts (NAS on Z2 Pro, database is Neon PostgreSQL)
- Project context (this project uses Flask + SQLite)
- API locations (Spotify Client ID in .env)

### Should NOT Remember
- Task intermediate state (changes each session)
- Already documented content
- Temporary debug info
- Passwords (use .env instead)

## Memory Inflation

Memory accumulates over time. Periodic cleanup:
1. Quarterly review of MEMORY.md for outdated info
2. Active deletion when errors found
3. Organized sections for clarity

## Viewing Current Memory

hermes memory status
Or ask: What do you remember about my projects?

---

*Last updated: 2026-05-02*
