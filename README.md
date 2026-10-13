# Routine Rivals

A responsive multi-participant routine tracker that combines personal task scheduling with challenge-based progress analytics.

## Features

- Account registration and login with Supabase Auth
- Create or join a challenge using an invite code
- Recurring task scheduling by weekday and per-participant task ownership
- Daily completion tracking with an 80% success threshold
- Streak calculations and weekly progress summaries
- Weekly leaderboard ranked by successful days, streak, and completion percentage
- Responsive dashboard and optional Realtime refresh

## Technology

HTML, CSS, JavaScript, Supabase Auth, PostgreSQL (Supabase), Supabase Realtime, and Netlify for the private deployment.

## Repository contents

- `index.html`: sanitized frontend source
- `docs/database-notes.md`: backend entities and operations referenced by the code
- `.gitignore`: excludes local credentials and exported data

## Running locally

This repository is a **portfolio source-code copy**, not a standalone ready-to-deploy demo. The original app depends on a Supabase backend that is deliberately **not** published here.

1. Provision a separate Supabase project and create appropriate database tables, functions, grants, and Row Level Security policies.
2. In your **local copy only**, replace `YOUR_PROJECT_ID` and `YOUR_SUPABASE_PUBLISHABLE_KEY` in `index.html` with credentials for that separate project.
3. Serve `index.html` from a local HTTP server.
4. Do not commit the configured copy, database exports, authentication credentials, user data, or invite codes.

The database notes describe the *interface expected by the frontend*, **not** a complete or tested database schema or policy set. The original production database is not bundled here.

## Privacy and security

Real user data, production endpoints, publishable keys, private credentials, invite codes, and deployment links are intentionally omitted. Real authorization must be enforced by Supabase Row Level Security and securely designed database functions, not just frontend checks.

## Portfolio note

The publicly shared source demonstrates frontend implementation and backend integration. A working private deployment exists, but its URL is not disclosed to protect participant privacy.
