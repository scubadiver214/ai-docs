---
title: Implementation Planning Artifacts Workflow
description: Required workflow for creating and maintaining feature-scoped docs/specs plan and task-tracker markdown artifacts during implementation work.
author: GitHub Copilot
ms.date: 2026-07-30
ms.topic: how-to
keywords:
  - implementation planning
  - docs specs
  - task tracker
  - copilot guidance
estimated_reading_time: 4
---

## Requirement

Before implementation begins for a planned change, create or confirm two feature-scoped markdown artifacts under `docs/specs`:

* A saved plan markdown file
* A living task-tracker markdown file

If either artifact is missing, create both before the first code edit for that implementation slice.

## Purpose

These artifacts give us a durable execution record inside the repo. They make the plan visible to future contributors, keep multi-step work synchronized, and preserve verification results next to the feature work instead of leaving them only in chat history.

## Location and Naming

Use a feature-scoped folder under `docs/specs`.

Examples:

* `docs/specs/StoreController/STORE_SUMMARY_COUNTS_PLAN.md`
* `docs/specs/StoreController/STORE_SUMMARY_COUNTS_TASKS.md`
* `docs/specs/OrganizationsController/ORGANIZATIONS_SORTING_PLAN.md`
* `docs/specs/OrganizationsController/ORGANIZATIONS_SORTING_TODO.md`

Match the naming pattern already used by the feature area when one exists. Prefer staying within an existing feature folder instead of creating a parallel folder for the same surface.

## Minimum Plan Content

Include these sections in the plan file:

* Execution tracker reference
* Goal
* Scope
* Endpoint or behavior shape when relevant
* Technical constraints
* Ordered implementation steps
* Relevant files
* Verification

## Minimum Task Tracker Content

Include these sections in the task tracker file:

* Current status
* Tasks
* Update log
* Verification notes
* Maintenance rule

The tracker is a living artifact. Update it during execution, not only at the end.

## Maintenance Rules

When implementation is in progress:

* Update `Current Status` when the active phase changes
* Mark completed checklist items with `[x]`
* Append one short `Update Log` entry whenever a task completes or becomes blocked
* Record verification commands and outcomes in `Verification Notes`

## Workflow

1. Identify the implementation surface and choose the matching `docs/specs` folder.
2. Create the plan and task tracker if either is missing.
3. Record the intended file map and verification approach in the plan.
4. Start implementation.
5. Update the task tracker as each task completes.
6. Record targeted test commands and outcomes in the tracker.
7. Finish only after the tracker reflects the final state of the work.

## Application

Apply this workflow to implementation work in the API repo whenever the work is driven by a plan, whether the plan came from chat, a prompt, a spec artifact, or an earlier saved document.

This requirement covers new feature work, behavioral fixes, contract changes, and multi-file implementation slices. Small one-file edits can still benefit from the workflow when the task expands beyond a trivial change.