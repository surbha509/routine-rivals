Database integration notes
The frontend references the following Supabase tables:
Table	Selected/used fields from the frontend
`challenges`	`id`, `name`, `invite_code`
`challenge_members`	`id`, `challenge_id`, `user_id`, `display_name`, `role`
`routine_tasks`	`id`, `challenge_id`, `owner_member_id`, `title`, `category`, `weekdays`, `active`, `created_at`, `created_by`
`task_checkins`	`id`, `challenge_id`, `task_id`, `member_id`, `checkin_date`, `completed`
The frontend also invokes PostgreSQL RPC functions `create_challenge(p_name, p_display_name)` and `join_challenge(p_invite_code, p_display_name)`.
The frontend upserts task check-ins using the conflict columns (`task_id`, `member_id`, `checkin_date`). It subscribes to optional database change notifications for members, tasks and check-ins.
Not included: the original SQL schema, SQL function definitions, grants, RLS policies, or database records. These cannot be reliably reconstructed or certified as secure from the HTML alone. Before publishing any real SQL, export and review its definition separately, without exporting production data.
