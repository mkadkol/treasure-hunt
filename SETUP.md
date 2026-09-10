# eGain Treasure Hunt — setup

Two things to do: put `index.html` on GitHub Pages (5 min), then wire up the live tracker (10 min).
The hunt already works after step 1 — the tracker just won't aggregate across phones until step 2.

---

## Step 1 — Publish the site

1. Go to <https://github.com/new>
   - **Repository name:** `treasure-hunt`
   - **Public** (Pages is free only on public repos — nothing sensitive is readable, see *Why this is safe* below)
   - Don't add a README
   - **Create repository**
2. On the empty repo page, click **uploading an existing file**, drag in `index.html`, then **Commit changes**.
3. Go to **Settings → Pages**.
   - **Source:** Deploy from a branch
   - **Branch:** `main`, folder `/ (root)` → **Save**
4. Wait about a minute, then open:

   **https://mkadkol.github.io/treasure-hunt/**

   You should see the "Star Chart Hunt" page. Test it with the Canteen link at the bottom of this file
   and Team 1's code `X7K9P2M4Q8R1T6VW`.

The repo name must be exactly `treasure-hunt` — every QR code in the print pack points at that URL.
If you want a different name, tell me and I'll regenerate the QR codes.

---

## Step 2 — Turn on the live tracker

GitHub Pages can't run a server, so team progress goes to a free Supabase project.

1. Sign up at <https://supabase.com> (email is fine, no Google account needed) and create a project.
   Any region near India — Mumbai or Singapore — keeps it snappy.
2. In the project, open **SQL Editor → New query**, paste all of this, and **Run**:

```sql
create table public.progress (
  team_id     text primary key,
  team_name   text,
  stops       integer default 0,
  last_room   text,
  last_at      timestamptz,
  finished_at  timestamptz,
  visits      jsonb default '[]'::jsonb,
  updated_at  timestamptz default now()
);

alter table public.progress enable row level security;

create policy "hunt read"   on public.progress for select to anon using (true);
create policy "hunt insert" on public.progress for insert to anon with check (true);
create policy "hunt update" on public.progress for update to anon using (true) with check (true);
create policy "hunt delete" on public.progress for delete to anon using (true);

-- run state: start/stop, the winner, and the team names
create table public.config (
  id          text primary key,
  state       text default 'idle',
  started_at  timestamptz,
  stopped_at  timestamptz,
  winner_team text,
  winner_name text,
  won_at      timestamptz,
  names       jsonb default '{}'::jsonb,
  updated_at  timestamptz default now()
);

alter table public.config enable row level security;

create policy "cfg read"   on public.config for select to anon using (true);
create policy "cfg insert" on public.config for insert to anon with check (true);
create policy "cfg update" on public.config for update to anon using (true) with check (true);

insert into public.config (id, state) values ('hunt', 'idle') on conflict (id) do nothing;
```

Both tables are already created on the live project — this block is only for rebuilding from scratch.

3. Open **Project Settings → API** (or **Data API**) and copy two values:
   - **Project URL** — looks like `https://abcdefghijkl.supabase.co`
   - **anon** / **publishable** key — the long one clearly labelled public

4. Edit `index.html` (GitHub's web editor is fine: open the file → pencil icon). Near the top you'll see:

```js
var SUPABASE_URL      = "";
var SUPABASE_ANON_KEY = "";
```

Paste your two values between the quotes — no trailing slash on the URL — and commit.

5. Reload the site, open **Organizer access**, passphrase `EGAIN2026`. The badge at the top should
   read **Live** instead of *This device only*.

If it still says *This device only*, the two values didn't save. If it says *Backend unreachable*,
one of them has a typo or the SQL didn't run.

---

## Running it from the tracker

Open the site, tap **Organizer access**, passphrase `EGAIN2026`.

- **Start the hunt** — until you press this, every room code shows "The hunt has not started yet".
- **Stop the hunt** — every code switches to a paused message. Press Start again to resume; nothing is lost.
- **Team names** — rename all ten teams, then **Save names & make slips** to download their code slips
  as a PDF. Names appear on the tracker, on each team's phone and on the slips. The codes never change,
  so posters already on the walls stay valid.
- **Clear all progress** — wipes progress, the winner and the clock, and sets the hunt back to not started.

## The treasure poster

Page 16 of the print pack is the winning code, in gold. Attach it to the prize itself, hidden in the
room the final clue names. A team must have unlocked all fourteen clues before a claim counts, so
finding the prize early wins nothing — the page tells them how far round they are and sends them
back. The first team to complete its route and claim the treasure wins: the page shows them
they won, the tracker records it, and every other code in the building immediately shows "hunt over"
with the winning team's name. The database picks the winner, so two teams scanning together cannot
both be told they won.

## On the day

- **Print the pack**, cut the 10 team slips apart, post the 14 room posters, and attach the gold
  treasure poster to the prize. Poster order is on page 1.
- **Do a practice run** with one phone, then open the tracker and hit **Clear all progress**.
  Ask your test phone to tap *Not your team?* so it forgets the practice code.
- **Hand each team only its own slip.** A borrowed code shows that team's route, not a shortcut.
- **Release all teams together** from the Canteen. Routes start in different rooms, so they spread out.
- **Watch the tracker.** It refreshes every few seconds and shows clues unlocked, the last room each
  team confirmed, the room they're due at next, and finish order once teams close the loop.
- **Press Start** when everyone is ready. Press **Stop** if you need to pause the floor.
- **Winner:** the first team to scan the gold treasure code and enter their team code. That ends the
  hunt for everyone and puts the winning team at the top of the tracker.

---

## Why publishing this publicly is safe

The repo is public, so anyone can read `index.html`. It contains no clues, no answers, no team codes
and no room addresses — only encrypted blocks.

- Every clue is sealed with AES-256-GCM. Its key is derived (150,000-round PBKDF2) from the team's
  code **plus the random tag in that room's QR code**. That tag exists nowhere except the printed
  poster, so a clue cannot be opened without physically standing in the room.
- Each team code opens exactly one clue per room — its own.
- The route key and team codes on the tracker are sealed under the organizer passphrase, so the
  passphrase itself isn't stored anywhere in the file either.
- Clues arrive in strict route order. A team holding 3 clues cannot log a 5th stop.

The one open door is deliberate: the Supabase public key lets any visitor write progress rows, which
is what makes a serverless tracker possible. Worst case someone fakes a team's position; nobody can
read a clue with it. Fine for an office game, and you can delete the Supabase project afterwards.

**To change the organizer passphrase** I need to rebuild the file — the passphrase is a decryption
key, not a stored string. Just ask.

---

## Station links (organizers only)

These are what the QR codes contain. Keep this list away from players — anyone with a link can open
that room's clue with their own team code, without walking there.

| # | Room | Link |
|---|------|------|
| 1 | Canteen (start + finish line) | https://mkadkol.github.io/treasure-hunt/#/canteen-hsekzysa |
| 2 | Luna | https://mkadkol.github.io/treasure-hunt/#/luna-fmmi3zf9 |
| 3 | Cygnus | https://mkadkol.github.io/treasure-hunt/#/cygnus-sjthqy5p |
| 4 | Andromeda | https://mkadkol.github.io/treasure-hunt/#/andromeda-98k6twad |
| 5 | Apus | https://mkadkol.github.io/treasure-hunt/#/apus-2s5pj7ex |
| 6 | Triton | https://mkadkol.github.io/treasure-hunt/#/triton-r8qsb25q |
| 7 | Hercules | https://mkadkol.github.io/treasure-hunt/#/hercules-rwh6qwuq |
| 8 | Phoenix | https://mkadkol.github.io/treasure-hunt/#/phoenix-miituict |
| 9 | Phoebe | https://mkadkol.github.io/treasure-hunt/#/phoebe-rrxe97ww |
| 10 | Lynx | https://mkadkol.github.io/treasure-hunt/#/lynx-k9dgbzdz |
| 11 | Pandora | https://mkadkol.github.io/treasure-hunt/#/pandora-pqw8ng8r |
| 12 | Europa | https://mkadkol.github.io/treasure-hunt/#/europa-yqriyy4j |
| 13 | Pegasus | https://mkadkol.github.io/treasure-hunt/#/pegasus-2vmvcaw5 |
| 14 | Atlas | https://mkadkol.github.io/treasure-hunt/#/atlas-cf8vekxq |
| — | **The Treasure** (winning code) | https://mkadkol.github.io/treasure-hunt/#/treasure-6ktgtvzt |

The same list is inside the tracker under **Station links**, so you don't need this file at the venue.
