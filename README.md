# FootyHornet

FootyHornet is a mobile-first offline PWA for tracking youth soccer stats during games. It uses plain HTML, CSS, and JavaScript. Player settings, active games, saved games, reviews, tags, and season totals are stored on the device with `localStorage`. Cloud sync, team rosters, approvals, and Live Share are intentionally future/optional features that should use a dedicated FootyHornet backend.

## Features

- Home, Track, Review, Season Dashboard, Players, More, and Live Game Tracker screens
- Local multi-player setup with active-player switching
- Big one-handed live stat buttons for game-day use
- Game format selector for Quarters or Halves, with OT support
- 50/50 Won and 50/50 Lost tracking with 50/50 win percentage
- Grouped live stat buttons with high-frequency events first and specialty stats lower on the screen
- Undo last event, save game, end game, review, and delete game actions
- 0-100 Game Impact score for each game, with season Average Impact
- Per-player season totals and averages from saved games
- Offline-ready `manifest.json` and service worker
- Future-ready user profiles, approved team access, shared rosters, and Live Share once a dedicated FootyHornet cloud backend is connected

## Local Setup

No install step is required.

```bash
python -m http.server 5173
```

Then open:

```text
http://localhost:5173
```

The root page is the public landing page. Open the working app directly at:

```text
http://localhost:5173/app.html
```

You can also use any static file server. Serving over `http://localhost` is recommended so the service worker can register during testing.

## GitHub Pages Deployment

1. Push this repository to GitHub.
2. In the GitHub repository, open **Settings**.
3. Go to **Pages**.
4. Under **Build and deployment**, choose **Deploy from a branch**.
5. Select your branch, usually `main`, and the root folder `/`.
6. Save. GitHub Pages will publish the static PWA.

The site is configured for:

```text
https://footyhornet.mybranford.com/
```

The public landing page lives at `/`. The PWA app lives at `/app.html`, and the manifest opens installed home-screen icons directly into the app.

## Launch Kit

The `launch-kit/` folder includes a QR code, printable parent handout, PDF handout, and message templates for sharing FootyHornet with teams and families.

## Future Cloud Multi-User Setup

FootyHornet starts with cloud settings intentionally blank in `app.js` so it does not write into the existing LaxHornet backend. Keep beta testing local-first until the soccer tracking workflow feels right. When ready, create a separate Supabase project for FootyHornet, then add that project URL and publishable key to `SUPABASE_CONFIG` in `app.js`.

To create or update the database tables:

1. Create or open the FootyHornet Supabase project dashboard.
2. Go to **SQL Editor**.
3. Open `supabase-schema.sql` from this repo.
4. Paste the full SQL into Supabase and run it.
5. In Supabase, open **Authentication > Providers** and make sure Email is enabled.
6. Open FootyHornet and create a team from the platform reviewer account.
7. Add rostered players with jersey numbers.
8. Share the team code with approved parents.
9. Parents create an account, submit their team code and child jersey number, and wait for approval.
10. Use **Copy Share Link** from the Live Game Tracker when you want a read-only share link.

Games and events are private to the signed-in user by default. Team roster games are visible to signed-in parents who are approved for the same team and claimed to one rostered player. Parent Tracker accounts can enter shared team stats only after admin approval. Copying a share link marks that game as shared so family can watch it read-only from another iPhone.

Team creation and roster administration are limited to the platform reviewer account: `degrassed@gmail.com`.

### Request And Approval Emails

`supabase-schema.sql` creates a `notification_queue` table for account request and approval email events. A static GitHub Pages app cannot send private transactional email by itself, so connect this queue to a Supabase Edge Function, Database Webhook, or Resend worker to deliver the queued messages.

## Future Shared Teams

Use **Team** when multiple parents need to track or view stats for the same approved rostered player.

1. Sign in with a User Profile.
2. Create a team to generate team access codes.
3. Add rostered players by name and jersey number.
4. Give team access codes only to parents who should request access.
5. The parent creates an account and submits the team code plus their child's jersey number.
6. The platform reviewer approves the request; approval automatically claims the matching rostered player.
7. The parent signs in and sees only that rostered player.

Best practice: choose one official Parent Tracker for each player/game. Multiple parents can sync and review the same stats, but two Parent Tracker accounts logging the same player at the same time can create duplicate events.

## Game Impact Scoring

Game Impact is a 0-100 score that estimates how much a player helped create possessions, convert possessions, protect possessions, and prevent opponent scoring chances. Game Review shows the Game Impact score for one game. Season Dashboard shows Average Impact across saved games.

Game Impact is position-weighted:

- Forward: higher scoring weight; medium possession and hustle; lower defense.
- Midfield: higher possession and hustle; medium scoring and defense.
- Defense: higher defense and hustle; medium possession; lower scoring.
- Goalkeeper: very high goalkeeper weight; medium possession and hustle; lower defense; scoring is not graded.

The raw event values behind the score are:

- Goal: +5
- Assist: +3
- Save: +3
- Shot on Goal: +1
- Missed Shot: -0.5
- Ball Recovery: +2
- Tackle Won: +3
- Interception: +3
- Clearance: +1
- Pressing Win: +2
- Hustle Play: +1
- Smart Play: +1
- Turnover: -2
- Failed Clearance: -2
- Goal Allowed: -1
- Foul Committed: -2
- Note: 0

For dashboard percentages, total shots are `Missed Shot` plus `Shot on Goal`.

## Possession Impact

Possession Impact separates two related ideas:

- Extra Possessions: the count of additional chances created or protected.
- Possession Value: the weighted value of those possession-changing plays.

Current possession rules:

- 50/50 won: +1 extra possession, +1.0 value
- Ball recovery: +1, +1.2
- Tackle won: +1, +1.8
- Save retained by the team: +1, +2.5
- Clearance: +0.5, +0.8
- Pressing Win: +1, +1.5
- Turnover: -1, -1.5
- Failed clearance: -1, -2.0
- Foul committed: 0, -1.5
