# Routine Rivals — PostgreSQL Database Documentation

## Overview

Routine Rivals uses **Supabase PostgreSQL** for challenges, participants, recurring tasks, and daily check-ins. Supabase Authentication provides user identities; PostgreSQL **Row Level Security (RLS)** and database functions govern access to challenge data.

This documentation is based on the project's supplied **database setup SQL**. It describes the original two-participant version of the schema; any later production changes should be checked against the current Supabase database before this document is presented as an exact representation of production.

## Database tables

| Table | Purpose | Key columns and constraints |
| --- | --- | --- |
| `public.challenges` | Challenge metadata | `id` (UUID PK), `name` (1–80 characters), unique `invite_code`, `created_by` → `auth.users(id)`, `created_at` |
| `public.challenge_members` | Membership and roles | `id` (UUID PK), `challenge_id` → `challenges(id)`, `user_id` → `auth.users(id)`, `display_name` (1–40 characters), `role` (`owner` or `member`), `joined_at` |
| `public.routine_tasks` | Recurring tasks assigned to members | `id` (UUID PK), `challenge_id`, `owner_member_id`, `title` (1–120 characters), optional `category`, `weekdays` (`0`=Mon through `6`=Sun), `active`, `created_by`, `created_at` |
| `public.task_checkins` | Daily task completion records | `id` (UUID PK), `challenge_id`, `task_id` → `routine_tasks(id)`, `member_id`, `checkin_date`, `completed`, `updated_at` |

### Relationships and data integrity

- **Challenge → members:** one-to-many, with foreign-key cascade deletion.
- **Challenge member → routine tasks:** one-to-many, enforced by composite foreign key `(challenge_id, owner_member_id)` to `challenge_members(challenge_id, id)`.
- **Routine task → check-ins:** one-to-many; deleting a task cascades to its check-ins.
- **Challenge member → check-ins:** one-to-many, using composite foreign key `(challenge_id, member_id)`.
- `unique(challenge_id, user_id)` prevents duplicate membership within a challenge.
- The `one_challenge_per_user` unique index restricts an account to one challenge in this schema.
- `unique(task_id, member_id, checkin_date)` prevents duplicate check-ins for the same task, person, and date.
- `weekdays` is constrained to 1–7 values drawn from integers 0–6. The supplied constraint does not independently prohibit repeated day numbers.
- Indexes support lookups on `routine_tasks(challenge_id)` and `task_checkins(challenge_id, checkin_date)`.

## Row Level Security (RLS)

The setup script enables RLS on all four tables.

| Table | Operation | Policy behavior in supplied SQL |
| --- | --- | --- |
| `challenges` | SELECT | Only authenticated members of the challenge can read it. |
| `challenge_members` | SELECT | Challenge participants can view memberships in their challenge. |
| `routine_tasks` | SELECT | Challenge participants can read active and inactive tasks in their challenge, subject to query filters. |
| `routine_tasks` | INSERT | Authenticated members may create tasks in their challenge with `created_by = auth.uid()`. **The original INSERT policy does not restrict `owner_member_id` to the caller**, so a participant could assign a new task to another participant. |
| `routine_tasks` | UPDATE | The task owner may update their task. The application deactivates tasks by setting `active = false`. |
| `task_checkins` | SELECT | Members can read check-ins from their challenge. |
| `task_checkins` | INSERT | Members can record check-ins for their own active tasks. |
| `task_checkins` | UPDATE | Members can update their own check-ins, subject to the policy's task ownership test. |

**Security helpers:**

- `public.is_challenge_member(p_challenge_id uuid)` returns whether `auth.uid()` is associated with the challenge.
- `public.current_member_id(p_challenge_id uuid)` returns the current user's member ID in that challenge.

Both functions are `STABLE`, `SECURITY DEFINER` SQL functions with `search_path = ''`, and execution is explicitly granted to the `authenticated` role in the setup script. `SECURITY DEFINER` functions must be reviewed carefully because they execute with the function owner's privileges.

## Database functions

### `public.create_challenge(p_name text, p_display_name text)`

Authenticates the caller, validates challenge and display names, checks that they do not already belong to a challenge, creates a challenge, then creates their membership with role `owner`. Returns the new challenge UUID.

### `public.join_challenge(p_invite_code text, p_display_name text)`

Authenticates and validates the caller; locates a challenge by invite code; locks its row; checks current membership; then inserts a `member` record. **The supplied setup script limits each challenge to two participants** using `if v_member_count >= 2`. If the current app supports five participants, the current database function must be different; verify and update the documentation accordingly.

### `public.remove_challenge_participant(p_member_id uuid)` — separately supplied function

This function was provided in a separate SQL snippet, not the setup script above. It:

1. Requires an authenticated caller and an existing target membership.
2. Checks that the caller belongs to the same challenge with role `owner`, `creator`, or `admin`, and is not removing themselves.
3. Deletes the target member's check-ins and check-ins related to their tasks.
4. Deletes the target member's routine tasks and membership.
5. Does **not** delete the user's Supabase Authentication account.

The setup schema allows only `owner` and `member`, so `creator` and `admin` are accepted by this function but are not valid roles under the original table constraint. Execution is revoked from `public` and `anon`, and granted to `authenticated`.

## Permissions in the supplied setup script

- Authenticated users receive SELECT privileges on `challenges` and `challenge_members`.
- Authenticated users receive SELECT/INSERT/UPDATE on `routine_tasks` and `task_checkins`; RLS adds row-level restrictions.
- Direct INSERT/UPDATE/DELETE on `challenges` and `challenge_members` is revoked from `anon` and `authenticated`.
- Challenge creation and joining are exposed through explicitly granted functions rather than direct table insertion.

## Analytics calculated by the frontend

The supplied website code calculates daily completion percentage, successful days (at least **80%** of scheduled tasks completed), streaks, weekly completion, and leaderboard rankings using task and check-in records. **These calculations are implemented in JavaScript in the supplied frontend; they are not SQL views or stored analytics procedures in the provided database setup script.**

## Security and deployment notes

- This repository documentation contains **no production participant records, passwords, Supabase project URL, or API credentials**.
- Review the RLS task-insert policy if tasks should always belong to their creator.
- Validate `SECURITY DEFINER` function ownership and access grants in the live Supabase project.
- Test same-challenge checks when inserting/updating check-ins and verify that no inconsistent cross-challenge references can be created.
- Before claiming five-person participation or identical production behavior, compare this historical setup SQL to the currently deployed schema and functions.

## Source and reproducibility

The descriptions above were derived from the project's supplied Supabase SQL setup script and separate participant-removal PL/pgSQL function. They document the code's intended database behavior, not results of a live security audit or verified deployment test.
