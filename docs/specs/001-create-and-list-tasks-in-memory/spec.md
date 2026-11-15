---
spec_id: 001-create-and-list-tasks-in-memory
title: Create and list tasks in memory
status: approved
priority: P1
shape: service-python
---

# 001 — Create and list tasks in memory

## Problem

The Task API has no tasks yet: it answers only health, readiness, ping and metrics. Clients of the taskboard cannot record what needs doing or see what has been recorded. This first feature gives them the smallest useful core, creating a task with a title and an optional due date and listing all tasks. It keeps the data in memory so that the API contract can be used and tested before any storage is chosen.

## Stories

As a taskboard client, I want to create a task with a title and, if I like, a due date, so that I can record what needs doing.

As a taskboard client, I want to list all tasks, so that I can see what has been recorded.

## Acceptance criteria

- **AC-001.1** Given the service is running, when a client sends `POST /tasks` with JSON `{"title": "Buy milk", "due_date": "2026-12-01"}`, then the response is `201` with a JSON object holding a non-empty string `id`, `title` `"Buy milk"`, `due_date` `"2026-12-01"` and `created_at` as an ISO 8601 UTC timestamp. (smoke)
- **AC-001.2** Given the service is running, when a client sends `POST /tasks` with JSON `{"title": "Call Bob"}` (no `due_date`) or with `"due_date": null`, then the response is `201` and the returned task has `due_date` `null`.
- **AC-001.3** Given the service is running, when a client sends `POST /tasks` with `title` missing, not a string, empty, only whitespace, or longer than 200 characters after trimming, then the response is `422` with an error body naming the field `title`, and a following `GET /tasks` does not contain a new task.
- **AC-001.4** Given the service is running, when a client sends `POST /tasks` with a `due_date` that is not a valid calendar date in `YYYY-MM-DD` form (for example `"2026-13-01"`, `"01.12.2026"` or `"2026-12-01T10:00:00Z"`), then the response is `422` with an error body naming the field `due_date`, and no task is created.
- **AC-001.5** Given three tasks were created one after another, when a client sends `GET /tasks`, then the response is `200` with a JSON array of exactly those three tasks, each with the same `id`, `title`, `due_date` and `created_at` as returned on creation, ordered oldest first; two created tasks never share an `id`.
- **AC-001.6** Given no task has been created since the service started, when a client sends `GET /tasks`, then the response is `200` with an empty JSON array `[]`; tasks created before a restart are gone after it.

## Non-goals

- Persistent storage, or tasks shared between several running instances of the service.
- Reading a single task, updating, completing or deleting tasks.
- Pagination, filtering, sorting options or search on the list.
- Users, authentication or per-user task lists.
- Further task fields (description, status, priority, tags).

## Assumptions

- **A1** The endpoints are `POST /tasks` and `GET /tasks`, with JSON fields `id`, `title`, `due_date`, `created_at` (other option: other paths or camelCase field names; affects AC-001.1 to AC-001.6).
- **A2** The `id` is a server-assigned unique string (a UUID); a client-supplied `id` is ignored (other option: sequential integers; affects AC-001.1, AC-001.5).
- **A3** The title is trimmed of leading and trailing whitespace before it is checked and stored, and may be 1 to 200 characters long (other option: stored verbatim, other limit; affects AC-001.3).
- **A4** The due date is a calendar date without time (`YYYY-MM-DD`); dates in the past are accepted (other option: date-time with time zone, or reject past dates; affects AC-001.1, AC-001.4).
- **A5** Validation errors answer `422` in the framework's standard error body, which names the failing field (other option: `400` with a custom error body; affects AC-001.3, AC-001.4).
- **A6** The list returns all tasks, oldest first, without a limit (other option: newest first, or a page size; affects AC-001.5).
- **A7** Unknown fields in the create request are ignored (other option: reject them with `422`; affects AC-001.1).
- **A8** No limit on the number of tasks held in memory, since this is a dev-stage feature (other option: a cap with `507`/`429`; affects AC-001.1).

## Open questions

None.

## Changelog

- 2026-11-15 created
- 2026-11-15 approved
