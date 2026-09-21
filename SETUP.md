# Wedding invitation: setup guide

`index.html` is a single self-contained file (photos are embedded). It needs no build step.
All settings live in the `CONFIG` block near the top of the `<script>` tag.

## 1. Turn on the RSVP database (Supabase, free tier)

1. Create a project at https://supabase.com.
2. Open **SQL Editor**, paste and run:

```sql
create table public.rsvps (
  id bigint generated always as identity primary key,
  client_id uuid not null unique,               -- stops double-saves of one submission
  created_at timestamptz not null default now(),
  name text not null check (char_length(name) between 2 and 100),
  email text check (email is null or char_length(email) <= 200),
  phone text check (phone is null or char_length(phone) <= 30),
  attending boolean not null,
  guests int check (guests is null or guests between 1 and 10),
  celebration text check (celebration in ('wedding','reception','both')),
  message text check (message is null or char_length(message) <= 500),
  check (not attending or (guests is not null and celebration is not null))
);

alter table public.rsvps enable row level security;

-- Visitors may ADD a response. There is deliberately no read policy,
-- so nobody can list other guests' details from the website.
create policy "guests can submit an rsvp"
  on public.rsvps for insert to anon with check (true);
```

3. Go to **Project Settings > API**. Copy the **Project URL** and the **anon / publishable** key.
   Never use the `service_role` key in this file.
4. In `index.html`, fill in:

```js
rsvp: {
  supabaseUrl: "https://YOUR-PROJECT.supabase.co",
  supabaseAnonKey: "YOUR-ANON-KEY",
  table: "rsvps",
  ...
}
```

5. Read responses in Supabase under **Table Editor > rsvps** (you can export to CSV).

The anon key is designed to be public. It can only insert rows, because of the row-level-security policy above.

### No-database alternative
Set `whatsappNumber` (digits with country code, e.g. `919XXXXXXXXX`) or `email`.
The form then opens WhatsApp or the guest's mail app with the RSVP pre-written. The guest must press Send, so this is less reliable than the database and the page tells guests so.

If none of the three is set, the form says "RSVP is not connected yet" and sends nothing.

## 2. Host it (free)

Any static host works: Netlify (drag the folder onto app.netlify.com/drop), Vercel, Cloudflare Pages or GitHub Pages.
Supabase calls will not work inside Claude's preview page; they work on your own domain.

## 3. Before sharing

- **Maps**: test the three links in `CONFIG.maps`. They open Google Maps searches by venue name. For exact pins, use Google Maps > Share > Copy link. The QR on the printed invitation decodes to `https://b-d.app/wd-bgkay`, which you may prefer for the reception.
- **Link preview (WhatsApp/Facebook)**: save a landscape photo as `share.jpg` (about 1200x630) next to `index.html`, then uncomment and edit the `og:image` / `og:url` line in `<head>`.
- **Add photos**: put files in a `photos/` folder and add `{ src: "photos/name.jpg", alt: "...", w: 1200, h: 1600 }` to the `GALLERY` array.
- **Music**: a soft tanpura-style drone generated in the browser (no recordings, no licensing). It only plays when a guest taps the button.
