---
title: Hermes Agent Skills System: Writing, Classification and Lifecycle
date: 2026-05-02
source: Self-research (hermes-agent SKILL.md, hermes-agent-skill-authoring SKILL.md)
tags: [Hermes-Agent, Skills, Authoring, Best-Practices]
description: Deep dive into Hermes Agent Skills system - how to write effective SKILL.md, skill categories, lifecycle management.
---

# Hermes Agent Skills System

## Overview

Hermes Agent Skills system is its most distinctive design. Unlike traditional AI agents where each session starts fresh, Skills allow you to persist entire workflows as reusable documents that any future session can load directly.

Memory tells the agent who you are. Skills tell the agent what to do when X happens.

## Skills vs Memory

| Dimension | Memory | Skills |
|-----------|--------|--------|
| Content | Facts, preferences, config | Workflows, procedures |
| Granularity | Atomic | Structured |
| Trigger | Passive (injected each session) | Active (load when needed) |
| Use case | Long-term preferences | Complex repeatable tasks |

## SKILL.md Structure

### Required Frontmatter

- name: lowercase + hyphens, less than 64 chars
- description: Must start with "Use when ..." - exact trigger condition, less than 1024 chars
- version, author, license, metadata all recommended

### The Golden Rule for Description

Description determines when to trigger. Must start with "Use when ..." describing exact trigger condition.

Correct: Use when debugging memory issues in Hermes Agent.
Wrong:   Debug memory issues. (too vague)
Wrong:   Fix a bug. (too broad)

## Skill Categories

1. Tool Skills: encapsulate specific tool usage
2. Knowledge Skills: store domain expertise
3. Workflow Skills: define standard execution procedures
4. Integration Skills: connect external systems
5. Conversational Skills: guide specific dialogue patterns

## Lifecycle

1. Create: After completing complex task, ask to save as Skill
2. Test: Load with /skill and verify it works
3. Publish: hermes skills publish
4. Maintain: Update when APIs change or feedback indicates issues

## Common Pitfalls

1. Description too abstract - unclear when to trigger
2. Scope too large - violates single responsibility
3. Never tested - unknown if working
4. Duplicates existing Skill - did not check first

## Useful Commands

hermes skills list
hermes skills search query
hermes skills install id
hermes skills check

---

*Last updated: 2026-05-02*
