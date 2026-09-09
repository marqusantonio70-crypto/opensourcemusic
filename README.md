<h1 align="center">Openitify</h1>

<p align="center">
  <b>A single-file, offline-capable music player for the open web.</b><br>
  Liquid glass UI · CD full-screen mode · karaoke with an auto lyrics generator · 108 built-in tracks · artist uploads · admin console
</p>

<p align="center">
  <a href="#quick-start">Quick start</a> ·
  <a href="#features">Features</a> ·
  <a href="#host-it-yourself">Host it</a> ·
  <a href="#optional-backend-supabase">Backend</a> ·
  <a href="#contributing">Contributing</a>
</p>

---

## What is this?

**Openitify is one HTML file.** No build step, no bundler, no `npm install`, no server. Open it and music plays — because the tracks are *synthesised in your browser* by a small Web Audio engine ("Open Node"), not streamed from anywhere. That makes it a legal, zero-cost, zero-dependency player you can fork, host, restyle and ship in five minutes.

It is a real player, not a mockup: seek, volume, speed, shuffle, repeat, likes, queue, search, uploads, playlists in `localStorage`, a full-screen spinning-CD cinema mode, and a karaoke view that will write and time lyrics for you.

```
1 file · 0 dependencies · 0 network requests · works from file://
```

## Features

### Player
- **108 pre-generated tracks** across 9 genre profiles (OPM R&B, synthwave, lo-fi, house, ambient, neo-soul, techno, cinematic, hyperpop) — imported automatically on first run, so the library is never empty.
- **CD / full-screen cinema mode** — press `F` or the disc button: album art becomes a spinning vinyl-CD with sheen, rings and spindle, synced to play/pause.
- **Karaoke mode** — press `K`: word-by-word wipe highlighting, click any line to seek, `±0.5s` offset nudges, `.lrc` import/export, and an **auto lyrics generator** that writes a verse/chorus/bridge structure from the track title and hooks it to the tempo.
- **Bulk import** — paste 100+ links or JSON, or drop local audio files; everything lands in the active playlist.
- **Artist uploads** — modal form for title, artist, album, tags, cover art and audio (file or URL), with instant play.
- **Dynamic ambient background** that re-tints the whole UI from the current album art.
- Animated frequency bars, hi-fi/normal quality switch, reduced-motion and calm modes, keyboard shortcuts, mobile layout.

### Admin & moderation (optional)
- Owner/admin/moderator/artist/listener roles.
- Rename, verify, ban, delete users; hide, feature, remove, restore tracks; report queue; audit log; live stats.
- Owner account is claimed with a one-time credential pair, then rotated from inside the console.

## Quick start

**Just listen:** download `index.html`, double-click it. That's it.

**Serve it locally:**
```bash
git clone https://github.com/<you>/openitify.git
cd openitify
python3 -m http.server 8080
# open http://localhost:8080
```

**Keyboard**
| Key | Action |
| --- | --- |
| `Space` | Play / pause |
| `←` `→` | Seek ∓5s |
| `↑` `↓` | Volume |
| `F` | Full-screen CD mode |
| `K` | Karaoke |
| `S` / `R` / `L` | Shuffle / repeat / like |
| `Esc` | Close overlay |

## Host it yourself

Any static host works, because there is nothing to build.

| Host | How |
| --- | --- |
| **GitHub Pages** | Included workflow (`.github/workflows/pages.yml`) publishes the repo root on every push to `main` and switches Pages on for you. |
| Netlify / Vercel / Cloudflare Pages | Drag the folder in, or connect the repo. Build command: *none*. Publish directory: `/`. |
| Any web server | Copy `index.html` into the document root. |

> Pages URL will be `https://<you>.github.io/openitify/`.

## Optional backend (Supabase)

Openitify runs fully offline. If you want accounts, real uploads, roles and moderation that persist across devices, point it at a Supabase project from **Account Settings → Backend**: paste the project URL and publishable key, sign in with email/password or GitHub/Google OAuth, then claim the owner account.

See [`docs/BACKEND.md`](docs/BACKEND.md) for the schema, the RPC surface, the owner-claim flow and the OAuth redirect setup.

**Security notes if you fork this publicly**
- Publishable/anon keys are safe to ship; service-role keys are not — never commit one.
- The offline demo gate is cosmetic. Real enforcement is row-level security plus `SECURITY DEFINER` functions in the database.
- Rotate the demo owner credential before you invite anyone in.
- OAuth cannot complete from `file://`; serve over http(s) and add that origin to your Supabase redirect URLs.

## Why "Openitify"?

Because a player should be *openable*. Fork it, read all of it in one sitting, change the palette, swap the audio engine, keep the karaoke. The logo is a code bracket around an open node — that's the whole thesis.

## Roadmap

- [ ] Visualiser presets (spectrum, oscilloscope, particles)
- [ ] Playlist import/export as JSON + shareable links
- [ ] Crossfade and gapless queueing
- [ ] Community lyrics pack format
- [ ] PWA install + offline cache manifest
- [ ] Theme marketplace (single CSS custom-property block per theme)

## Contributing

Issues and PRs welcome — especially themes, genre profiles for the audio engine, and translations.

The app ships as one file, but it is developed as parts that are concatenated by `build.py`. Small fixes can be made directly in `index.html`; anything structural should describe the change in the PR so it can be folded back into the sources.

1. Fork, branch, change.
2. Open `index.html` and confirm the console is clean.
3. PR with a screenshot or short clip.

## License

[MIT](LICENSE) — do anything, just keep the notice.

**Audio:** every bundled track is generated by the built-in engine; no copyrighted masters are included. The "Pagsamo (Session Sketch)" preset is an original instrumental tribute arrangement, not a recording of the song.
