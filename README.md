# Our Daily Bread · Church Ministry

Single-page app (`index.html`) with Supabase login, roles, and cloud data.

## Setup (about 10 minutes)

### 1. Supabase
1. Create a project at supabase.com.
2. **SQL Editor** -> New query -> paste all of `supabase/schema.sql` -> Run. Then run `private/create_host.sql` the same way. That creates the single Host account (it is git-ignored, so it won't be uploaded).
3. **Project Settings -> API**: copy the **Project URL** and the **anon public** key into `supabase-config.js`. (Never use the `service_role` key.)
4. **Authentication -> URL Configuration**: set *Site URL* to your GitHub Pages address (step 2).
5. Optional: **Authentication -> Providers -> Email**: turn off "Confirm email" if you don't want sign-ups to confirm by email.

### 1b. Google sign-in
1. Google Cloud Console -> APIs & Services -> **OAuth consent screen**: app name "Our Daily Bread", add your logo and support email, then **Publish app** (in "Testing" only listed test users can sign in).
2. **Credentials -> Create credentials -> OAuth client ID -> Web application**. Authorized redirect URI: `https://fanabedccdgwifiukmjc.supabase.co/auth/v1/callback`
3. Supabase **Authentication -> Sign In / Providers -> Google**: enable, paste the Client ID and Client Secret, save.
4. Supabase **Authentication -> URL Configuration**: add your GitHub Pages address (e.g. `https://<user>.github.io/<repo>/`) to **Redirect URLs** as well as Site URL.

Members now sign in only with Google. Anyone whose Google email matches an existing account is linked to that same account automatically. The Host still uses "Host sign-in" (username Host + password).

### 2. GitHub
1. Create a repo and upload everything in this folder, including the hidden `.github` folder (or push with git).
2. **Settings -> Pages -> Source: GitHub Actions**. Each push to `main` redeploys. Your site goes live at `https://<user>.github.io/<repo>/`.

### 3. Logging in
The Host signs in with username **Host** and password **12345678** (change it after first login). There is exactly one Host; the app and database both refuse to create or promote a second. Everyone else taps **Continue with Google**, then sets their name and picks Youth, Missionary or Evangelist. The role works right away with no approval; the Host can change a role or remove anyone who isn't a member from the Community screen. Youth and Missionary start right away; people who pick "Evangelist" appear in the Host's Community screen with an Approve button.

## Updating an existing install
Run `supabase/update_google_missionary.sql` once in the SQL Editor, then upload `index.html`, `supabase-sync.js` and `README.md` to GitHub.

## Roles
| | Youth / Missionary | Evangelist | Host |
|---|---|---|---|
| Own attendance, notes, prayer, journal | yes | yes | yes |
| Post in Community, edit own posts | yes | yes | yes |
| Post/delete announcements, delete any Community post | no | yes | yes |
| View all members' meditations | no | yes | yes |
| Manage songs library and Drive links, giving details | no | yes | yes |
| Change roles, remove members, see online list | no | no | yes |

## How data is handled
- **Login/roles:** Supabase Auth + `profiles` table. Passwords are never stored in the app data. Only the Host can change roles or remove members (enforced by the database, not the browser).
- **Personal data** (attendance, notes, prayer requests, student log, notifications...) is a private row per user; nobody else can read it.
- **Shared data** (community posts, announcements, events, giving details, meditation journals, plans, songs...) is visible to every signed-in member and updates live on other devices. Only Host/Evangelists can change announcements, giving details and the songs library (enforced by the database).
- Which keys are personal vs shared is listed at the top of `supabase-sync.js`.

## Known limits
- Meditation journals are shared rows; the app hides other people's drafts in the interface, but a technical member could read them through the API. Move `med` to the personal list in `supabase-sync.js` if you want strict privacy (leaders then won't see members' journals).
- Shared lists are saved per key (last writer wins). Fine for a church group; very heavy simultaneous editing of the same list can drop changes.
- Profile photos are stored in the database as small images. Keep them small.
- Leave `supabase-config.js` blank to run local-only (old localStorage mode).
