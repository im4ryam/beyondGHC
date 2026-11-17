<!-- LOVABLE:BEGIN -->
> [!IMPORTANT]
> This project is connected to [Lovable](https://lovable.dev). Avoid rewriting
> published git history — force pushing, or rebasing/amending/squashing commits
> that are already pushed — as it rewrites history on Lovable's side and the
> user will likely lose their project history.
>
> Commits you push to the connected branch sync back to Lovable and show up in
> the editor, so keep the branch in a working state.

## What Lovable manages

Rules outside this block are the project's own. Change the pieces below in
Lovable, not by hand. With the [Lovable MCP](https://mcp.lovable.dev)
connected, `send_message` asks Lovable to do it; otherwise ask the user to
use the editor.

- **Database.** Do not add or edit `supabase/migrations/` or `drizzle/`.
  Ask Lovable to change the schema.
- **Generated files.** Do not edit a file whose header says it is automatically
  generated.
- **AI gateway.** Keep model calls on Lovable's gateway, and keep
  `LOVABLE_API_KEY` out of browser code.
- **Connectors.** Lovable provides native connectors. `list_connectors` shows
  which are enabled; `add_connector` only returns the dashboard link. When a
  connector is enabled, calls go through the connector gateway and need
  `LOVABLE_API_KEY`. When Lovable does not provide one, users can bring their
  own key and store it in Secrets.
- **Secrets.** Lovable manages them. Do not put secret values in the repo.
<!-- LOVABLE:END -->

## Project rules

- Roles: org-level roles (`owner`, `admin`) live only in `public.user_roles`; team lead is `team_members.is_lead`. Why: keeps privilege data out of user-editable rows.
- The Primary Owner is granted only by the `claim_owner_if_eligible` trigger after email confirmation of the email in `integration_settings.primary_owner`, and moved only via `transfer_ownership()`. Why: owner must never be assignable through normal user management.
- All authorization is enforced in Postgres (RLS + security-definer helpers like `is_admin`, `can_manage_team`, `has_dataset`); the UI only hides controls. Why: brief requires backend-enforced permissions.
- Field-level limits (assignees can only change status/review flag; non-admins can't change account status) are enforced by BEFORE UPDATE guard triggers, not the client. Why: RLS can't restrict columns.
- Contact info (email, phone) lives in `profile_private`, separate from `profiles`. Why: teammates can see profiles without seeing contact details.
- Activity log and in-app notifications are written by database triggers. Why: every change path gets logged consistently.
- Interest-form rows upsert through `upsert_submission` (dedupe by lowercased email, history kept). Used by the spreadsheet import RPC and the `/api/public/form-intake` route (secret key, hash stored in `integration_settings`). Submissions never create logins.
- Privileged actions needing the service role (inviting accounts, rotating intake key) live in `src/lib/admin.functions.ts` and check `is_admin` first.
- Portal data is read client-side with React Query using shared query options in `src/lib/queries.ts`; mutations invalidate those keys. Why: the `_authenticated` subtree is client-only (`ssr: false`).
- Avatars are stored as small resized data URLs on `profiles.avatar_url`. Why: the workspace blocks public storage buckets.
- Signup links use hashed, expiring, single-use invitations consumed under a database lock by the signup lifecycle trigger; they never grant roles. Why: forwarded or replayed invitations must not bypass approval.
- Assignment requests are approved atomically through a database RPC; only the owner may include a co-admin grant. The shell checks assignment and refreshes account access periodically. Why: access changes must be enforced independently of the browser.
- Task creation is allowed for team members with a self-assignment constraint enforced by RLS and an insert trigger; managers retain team-scoped assignment. Why: changing form values must not allow volunteers to assign others.
