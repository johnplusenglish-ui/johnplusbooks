# johnplusbooks.com — Book42

Static site, GitHub Pages (`CNAME` → johnplusbooks.com), deployed straight from `main` on push —
confirmed live: response headers show `server: GitHub.com` and a `last-modified` matching the
latest commit. **Every push to `main` goes live immediately**, no staging, no build step.

A year-by-year gallery of a Telegram book club's monthly picks ("Book42"), plus a Podcast tab.

## Architecture — much simpler than johnplusenglish.com, don't assume the same shape

This is genuinely a single-page site, not a multi-page one:

- **`index.html`** (~275KB) — the entire site. Inline `<style>` and one big inline `<script>` at the
  bottom; no separate CSS/JS files, no `-content.html` split, no sidebar/wrapper pattern. Year
  sections, the sideways-scrolling gallery, the podcast tab, and the Supabase-backed notes/activity
  feature are all in this one file. Read the specific area you need (grep for the relevant heading
  or function name) rather than the whole file — it's long.
- **`404.html`** — standalone, small.
- **`covers/`** — ~29MB of book cover `.jpg` files, one per pick, named
  `YYYY-month-book-title.jpg`. Sourced from an Open Library cover pipeline in an earlier session.
  Binary assets — don't open/grep, and don't add new covers without following that same naming
  convention.
- **`fonts/`** — `.woff2` files, binary, skip.
- No `-content.html`/iframe split, no shared `/assets/` folder, no `tools.html`-style legacy shim —
  none of that johnplusenglish.com-specific architecture applies here. Don't port patterns over
  from that repo without checking they actually fit this one.

## Backend: Supabase

`index.html` embeds a Supabase URL + anon key (`SUPABASE_URL` near the top of the `<script>` block)
for a small notes/activity feature (readers can leave notes, see a recent-activity bell). Also uses
`localStorage` for local-only state. **Known gotcha, already correctly applied — don't "fix" it
back**: inserts use `Prefer: 'resolution=merge-duplicates,return=minimal'`. Using
`return=representation` instead throws a misleading RLS-policy-shaped error on this project even
when the actual insert would succeed — keep `return=minimal` on any new insert call. The RLS policy
SQL itself (`supabase_setup.sql`, referenced in a comment near `SUPABASE_URL`) isn't in this repo —
it was run directly against the Supabase project, not checked in. Don't go looking for it here.

## Before shipping any change here

1. Local preview: `python3 -m http.server` + browser check — this repo has no CLAUDE.md-documented
   incident history yet, but the same rule applies as anywhere: verify in a browser, don't just read
   the code and assume it's right.
2. `git status --short` before staging; stage the files you actually touched, not `-A`/`-u` by
   habit, especially since this working directory may be shared with other concurrent sessions the
   same way johnplusenglish.com's is.
3. Given the whole site is one 275KB file, a "small" edit can still touch a lot of surrounding
   markup if you're not precise about the boundaries of what you're changing — grep for the
   surrounding structure (the enclosing year section, the enclosing function) before editing, not
   just the line that looks relevant.
