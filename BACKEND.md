# Openitify — backend, OAuth and owner console

Everything below is **already applied** to your Supabase project. This file is the map.

| | |
|---|---|
| Project ref | `dmoxkwtifnwymcalzbie` |
| API URL | `https://dmoxkwtifnwymcalzbie.supabase.co` |
| Publishable key | `sb_publishable_secKTwXm6CJ4GaTOho_7OA_wG6S5Ut8` (safe to ship in the HTML) |
| Region / Postgres | `ap-south-1` · 17.6.1.155 |
| Migrations applied | `openitify_player_roles_moderation`, `openitify_ensure_profile`, `openitify_harden_function_grants` |

All Openitify objects are prefixed `op_` so they never collide with the tables that were
already in this project (those were left untouched).

---

## 1. Schema

**Tables**

- `public.op_profiles` — `id` (→ `auth.users`), `username`, `display_name`, `avatar_url`, `bio`,
  `role` (`listener | artist | moderator | admin | owner`), `verified`, `banned`, `ban_reason`,
  `banned_at`, timestamps.
- `public.op_tracks` — `owner_id`, `title`, `artist`, `album`, `tags[]`, `audio_url`, `art_url`,
  `duration`, `featured`, `status` (`published | hidden | removed`), `plays`, `created_at`.
- `public.op_likes` — `(user_id, track_id)`.
- `public.op_reports` — `track_id`, `reporter_id`, `reason`, `note`,
  `status` (`open | resolved | dismissed`), `resolved_by`, `resolved_at`.
- `public.op_audit` — append-only log: `actor_id`, `actor_label`, `action`, `target_type`,
  `target_id`, `meta jsonb`, `created_at`.
- `openitify_private.app_secrets` — console username + **bcrypt** password hash. Not exposed
  through the API at all.

**Storage buckets** — `tracks` (public, 75 MB/file) and `art` (public, 8 MB/file).

**Row-level security** is on for every table. Listeners read published tracks, artists write only
their own rows, staff (`moderator | admin | owner`) get the wider policies.

## 2. Owner / admin functions

Every privileged action is a `SECURITY DEFINER` function that re-checks the caller's role inside
Postgres, then writes an audit row. Grants were tightened so `anon` cannot call any of them.

| Capability | Function |
|---|---|
| Claim the owner seat | `op_claim_owner(p_username, p_password)` |
| Rotate console credentials | `op_rotate_owner_credentials(p_username, p_password)` |
| Rename an account | `op_admin_rename_user(p_user, p_display_name, p_username)` |
| Change role | `op_admin_set_role(p_user, p_role)` |
| Verify / unverify | `op_admin_set_verified(p_user, p_verified)` |
| Ban / unban | `op_admin_ban_user(p_user, p_banned, p_reason)` |
| Delete an account | `op_admin_delete_user(p_user)` |
| Upload music as anyone | `op_admin_upload_track(..., p_owner)` |
| Hide / remove / restore a track | `op_admin_set_track_status(p_track, p_status)` |
| Feature on Highlights | `op_admin_set_track_featured(p_track, p_featured)` |
| Delete a track | `op_admin_delete_track(p_track)` |
| Report a track | `op_report_track(p_track, p_reason, p_note)` |
| Resolve / dismiss a report | `op_admin_resolve_report(p_report, p_status)` |
| Console data | `op_admin_stats()`, `op_admin_list_users()`, `op_admin_queue()` |
| Profile bootstrap after OAuth | `op_ensure_profile()` |

A trigger on `auth.users` (`on_auth_user_created_openitify`) creates a profile for every new
sign-up; `op_ensure_profile()` back-fills accounts that existed before the migration.
A second trigger (`op_guard_profile_update`) stops anyone from editing their own `role`,
`verified` or `banned` columns directly — only the functions above can.

## 3. Claiming the owner seat

1. Open the app → **Sign in** (GitHub, Google, or email + password).
2. Go to **Admin console** in the sidebar.
3. Enter the console credentials in *Claim the owner seat* and press **Claim owner seat**.
4. Your profile flips to `owner`; the Moderation tab and every control unlock.

The username/password pair is checked against a bcrypt hash inside Postgres — it is never
stored in the HTML file, and the check cannot be bypassed from the browser console.

**Rotate them** as soon as you have claimed the seat (the pair was typed into a chat window):
Admin console → **Rotate console credentials**, or from SQL:

```sql
select public.op_rotate_owner_credentials('new-console-user', 'new-console-password');
```

## 4. Turning on GitHub / Google OAuth

Providers can only be enabled from the dashboard — the API does not expose that toggle.

1. **Authentication → Sign In / Providers → GitHub** → enable, paste Client ID + Secret.
   In GitHub → *Settings → Developer settings → OAuth Apps*, set the callback to:
   `https://dmoxkwtifnwymcalzbie.supabase.co/auth/v1/callback`
2. **Authentication → Sign In / Providers → Google** → enable, paste Client ID + Secret.
   In Google Cloud Console, add the same authorised redirect URI.
3. **Authentication → URL configuration → Redirect URLs** → add the URL the app is served from,
   e.g. `http://localhost:8000/Openitify.html` or your Netlify/Vercel/Pages URL.
4. Optional but recommended: **Authentication → Policies → enable leaked-password protection.**

### OAuth needs http(s)

A page opened from `file://` cannot receive an OAuth redirect, so the app blocks those buttons
with an explanation. Serve it instead:

```bash
python3 -m http.server 8000        # then open http://localhost:8000/Openitify.html
```

Email + password sign-in works even from `file://`.

## 5. What runs without any of this

With no session, the console falls back to **local demo mode**: six seeded accounts, one open
report, and a working audit trail stored in `localStorage`. Every control behaves the same so you
can try the flows offline. The seeded owner claim also works there, gated by a SHA-256 check —
that one is cosmetic, which is exactly why the real enforcement lives in RLS.

## 6. Honest security notes

- The publishable key in the HTML is meant to be public. Authority comes from RLS plus the
  role checks inside each function, not from the client.
- Hiding a button in the UI protects nobody; the reason the console is safe is that
  `op_admin_*` raises `insufficient_privilege` when a non-staff JWT calls it.
- `admin:///6767+` is a **console claim code**, not a Supabase login. You always sign in with a
  real identity first, then claim — so every privileged action is attributable in `op_audit`.
- Pre-existing tables in this project (`posts`, `profiles`, `films`, `roms`, …) were not modified.
