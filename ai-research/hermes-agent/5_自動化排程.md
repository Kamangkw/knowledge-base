---
title: Hermes Agent Cron Jobs Automation: Design, Chaining and Monitoring
date: 2026-05-02
source: Self-research (hermes-agent SKILL.md, Cron docs)
tags: [Hermes-Agent, Cron, Automation, Scheduling]
description: Deep dive into Hermes Agent Cron Jobs system - automated workflows, job chaining, monitoring and maintenance best practices.
---

# Hermes Agent Cron Jobs Automation

## Overview

Cron Jobs allow you to schedule tasks to run automatically at specific times. Ideal for daily research, weekly knowledge base maintenance, and periodic workflows.

Current Kama setup:
- Daily 04:00: Knowledge base research (weekly topic-based)
- Daily 07:30: Signal/Noise music recommendations (Telegram push)

## Basic Operations

hermes cron create "0 9 * * *"   # Every day at 9am
hermes cron list
hermes cron run job-id
hermes cron pause job-id
hermes cron resume job-id
hermes cron remove job-id

## Cron Expression Syntax

Standard cron format (5 fields): minute hour day month weekday

| Expression | Meaning |
|------------|---------|
| 0 4 * * * | Daily at 4:00 AM |
| 0 7 * * * | Daily at 7:00 AM |
| 30 7 * * * | Daily at 7:30 AM |
| 0 9 * * 1 | Monday at 9:00 AM |
| 0 */2 * * * | Every 2 hours |
| 0 0 * * 0 | Sunday midnight |

Kama Signal/Noise at Macau time 07:30 = 30 7 * * * (UTC+8).

## Job Chaining

Multiple Cron Jobs can be chained with output feeding into the next:

context_from: research-job-id

Example Signal/Noise chain:
Job 1: Music research (07:30) -> outputs song list
Job 2: Write to history file (07:35)
Job 3: Send to Telegram (07:40)

## Design Principles

### Single Responsibility
Each Cron Job should focus on one thing:
- One job for research
- One job for writing to history
- One job for sending to Telegram

### Idempotency
Design: what happens if the same job runs twice?
- Should be harmless (no duplicate adds, no errors)
- Multiple runs should produce consistent results

### Failure Handling
1. Clear logging on failure
2. Retry mechanism (built-in but may need config)
3. Partial failure handling

## Monitoring

hermes cron list --all    # Including paused
hermes cron status        # Scheduler status

View execution logs:
grep "job-id" ~/.hermes/logs/agent.log

### Common Issues

Job did not execute:
1. Check schedule is correct
2. Check scheduler is running
3. Check network/environment
4. Manually trigger to confirm

Job executed but result wrong:
1. Check prompt is clear
2. Check context_from connections
3. Check upstream job succeeded

## Delivery Options

| Target | deliver value |
|--------|--------------|
| Current session | origin |
| Telegram | telegram:8680253639 |
| Local file | local |
| Discord | discord:#channel-name |

## Skills Integration

Cron Jobs can preload Skills to ensure job has workflow guidance:

skills: knowledge-base-daily-research

---

*Last updated: 2026-05-02*
