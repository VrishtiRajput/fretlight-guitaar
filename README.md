# 🎸 Fretlight — Play Guitar With Your Hands

Fretlight turns your webcam into a virtual guitar. Using real-time hand tracking right in the browser, you fret and pluck strings in the air — no instrument, no hardware, just your hands and a camera.

**[Live Demo →](https://fretlight-guitar.netlify.app)**

---

## ✨ Features

- **Hand-tracked virtual fretboard** — real-time hand/finger detection maps your fingertips onto a 6-string fretboard overlaid on your live camera feed.
- **Natural pluck detection** — strings sound only when a fingertip actually crosses the string line, like a real strum or pick, not just from hovering nearby.
- **Free-play & chord modes** — play individual frets freely, or snap to full chord shapes.
- **Karplus-Strong string synthesis** — plucked strings are synthesized in real time via the Web Audio API for a natural, decaying guitar tone.
- **Record your sessions** — capture the fretboard view plus your guitar and singing, mixed together, as downloadable video takes.
- **Google Sign-In** — authenticate with your Google account.
- **Cloud-synced recordings** — signed-in users' takes are automatically uploaded and persist across sessions and devices, backed by Supabase (Postgres + Storage + Auth).
- **Fully client-side** — a single HTML file, no backend server to run yourself.

---

## 🛠️ Tech Stack

| Layer            | Technology |
|-------------------|------------|
| Hand tracking      | [MediaPipe Tasks Vision](https://developers.google.com/mediapipe) (HandLandmarker) |
| Audio synthesis    | Web Audio API (Karplus-Strong plucked-string model) |
| Recording          | MediaRecorder API |
| Auth / Database / Storage | [Supabase](https://supabase.com) (free tier) + Google OAuth |
| Hosting            | Static hosting (Netlify) |

---

## 🚀 Getting Started (Run It Yourself)

Fretlight is a single self-contained HTML file. To run your own copy with your own cloud backend:

### 1. Clone this repo
```bash
git clone https://github.com/VrishtiRajput/fretlight.git
cd fretlight
```

### 2. Create a free Supabase project
1. Go to [supabase.com](https://supabase.com) → New project (no credit card required).
2. **Authentication → Providers → Google** — enable it (you'll need a free Google Cloud OAuth Client ID/Secret — see below).
3. **Storage** → create a new bucket named `takes`, set to **Public**.
4. **SQL Editor** → run:
    ```sql
    create table takes (
      id uuid primary key default gen_random_uuid(),
      user_id uuid references auth.users not null,
      name text not null,
      url text not null,
      storage_path text not null,
      created_at timestamptz default now()
    );

    alter table takes enable row level security;

    create policy "Users manage their own takes"
    on takes for all
    using (auth.uid() = user_id)
    with check (auth.uid() = user_id);

    create policy "Users can upload their own takes"
    on storage.objects for insert
    to authenticated
    with check (
      bucket_id = 'takes' and (storage.foldername(name))[1] = auth.uid()::text
    );

    create policy "Users can view their own takes"
    on storage.objects for select
    to authenticated
    using (
      bucket_id = 'takes' and (storage.foldername(name))[1] = auth.uid()::text
    );

    create policy "Users can delete their own takes"
    on storage.objects for delete
    to authenticated
    using (
      bucket_id = 'takes' and (storage.foldername(name))[1] = auth.uid()::text
    );
    ```
5. **Project Settings → API** — copy your **Project URL** and **Publishable (anon) key**.

### 3. Set up Google Sign-In
1. In [Google Cloud Console](https://console.cloud.google.com/apis/credentials), create a project and an **OAuth 2.0 Client ID** (type: Web application).
2. Add your Supabase callback URL (`https://<your-project>.supabase.co/auth/v1/callback`) as an **Authorized redirect URI**.
3. Add your app's domain(s) (e.g. `http://localhost:8000`, your live domain) as **Authorized JavaScript origins**.
4. Paste the Client ID/Secret into Supabase's Google provider settings and save.

### 4. Add your keys to the app
Open `fretlight.html` and fill in your own values near the top of the `<script type="module">` block:
```js
const supabaseConfig = {
  url: "https://YOUR_PROJECT.supabase.co",
  anonKey: "YOUR_SUPABASE_PUBLISHABLE_KEY"
};
```

### 5. Run locally
Browsers block camera access and OAuth on `file://` pages, so serve it:
```bash
python3 -m http.server 8000
```
Then open `http://localhost:8000/fretlight.html`.

### 6. Deploy
Drag the file into [Netlify Drop](https://app.netlify.com/drop) (or any static host) for a free live HTTPS URL. Remember to add the deployed domain to both Google OAuth and Supabase's redirect URL settings.

---

## 🔒 Privacy & Data

- Camera and microphone access happen entirely in-browser; frames are never uploaded anywhere.
- Recordings are only uploaded to the cloud if you're signed in — otherwise they stay local to your browser tab.
- Row Level Security ensures each signed-in user can only see and manage their own recordings.

---

## 📄 License

MIT — feel free to fork, modify, and build on this.
