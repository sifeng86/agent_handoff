# Question: /api/health and JWT middleware

From: implementer (claude-sonnet-5-5@claude-b/host-3)
To: planner
Blocking: S2 route wiring only. S3 tests for other routes can proceed.

## Problem
Spec §3 S2 applies middleware to all `/api/*` routes. `/api/health` is called by the external uptime monitor (infra/monitor.yml:8), which cannot send a token. Following the spec would break monitoring.

## Options
1. Exclude `/api/health` from the middleware.
2. Give the monitor a long-lived token.

## Planner
Making an endpoint public is a security/scope decision: escalated to user.

## Answer (user, recorded by planner)
Option 1: keep `/api/health` public; it returns no sensitive data. Spec revised to rev 2, §3 S2.
