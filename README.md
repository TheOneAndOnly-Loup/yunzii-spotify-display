# Yunzii B75 Pro Max — Spotify Now Playing Display

Displays your current Spotify track (album art, song name, artist) on the built-in screen of the **Yunzii B75 Pro Max** keyboard, running entirely in the browser using WebHID. No native app, no drivers, no backend.

**Hosted at:** [https://theoneandonly-loup.github.io/yunzii-spotify-display/](https://theoneandonly-loup.github.io/yunzii-spotify-display/)

---

## Requirements

- **Keyboard:** Yunzii B75 Pro Max (the HID protocol is specific to this model)
- **Browser:** Google Chrome (or any Chromium-based browser) — WebHID is not supported in Firefox or Safari
- **Spotify account** (free or premium)

---

## Setup

### 1. Create a Spotify Developer App

1. Go to [developer.spotify.com/dashboard](https://developer.spotify.com/dashboard)
2. Log in and click **Create app**
3. Fill in any name and description
4. Under **Redirect URIs**, add:
   ```
   https://theoneandonly-loup.github.io/yunzii-spotify-display/
   ```
   Then click **Add** and **Save**
5. Go to **Settings** and copy your **Client ID**

### 2. Open the tool

Using Chrome, go to **https://theoneandonly-loup.github.io/yunzii-spotify-display/**

### 3. Paste your Client ID

In Step 1 on the page, paste your Client ID into the input field. It is saved in your browser and never sent anywhere.

### 4. Log in to Spotify

Click **Login with Spotify** and authorise the app. The page will redirect back automatically.

### 5. Connect your keyboard

Plug in your Yunzii B75 Pro Max via USB, then click **Connect Keyboard** in Step 2. Select the keyboard from the browser prompt.

That's it — the page checks Spotify every second and updates the screen whenever the song changes.

> **Order matters:** always log in to Spotify before connecting the keyboard. The OAuth redirect resets any HID connection.

---

## Options

| Control | Description |
|---|---|
| **Auto-update** | Toggle watching Spotify on/off |
| **Update interval** | How often Spotify is checked: **1s** (default) / 2s / 5s / 10s. Checking is cheap — the keyboard is only pushed when something changes |
| **Layout** | *Clean* (big art left) · *Cover top* (art fills top) · *Progress* (progress bar + timestamps) · *Ambient* (Clean on a gradient themed to the album art) |
| **Art fit** *(Cover top only)* | *Zoom* (crop to fill) · *Blur sides* (blurred bg + crisp centre) · *Stretch* |
| **Background** *(Ambient only)* | *Gradient* (fades between two colours from the cover) · *Solid* (one colour from the cover, matched exactly to a colour the screen can show — perfectly clean, no steps or dots) |
| **Dithering** *(Ambient only)* | How in-between shades are faked on gradients and album art: *Pattern* (regular dot pattern) · *Smooth* (fine, irregular grain) · *Off* (no dots, but gradients show visible steps). Other layouts use *Pattern* |
| **Rounded art corners** | Applies to all layouts |
| **Push Now** | Fetches the current track and pushes it immediately, even if nothing changed |
| **Push Custom Image** | Sends any image file to the screen (stays until the song changes) |
| **Debug messages** *(Log card)* | Shows every Spotify check (timing, what's playing, pushed or skipped and why), every push with a running count and how long it took, and album art downloads. Remembered between visits. The log keeps the last 500 lines |

---

## How it works

The keyboard exposes a 160×96 RGB565 framebuffer over WebHID. The protocol (reverse-engineered from the official yunzii-game.com web app) is:

1. **`0x40`** — init packet (signals start of frame transfer)
2. **`0x41`** — metadata packet
3. **`0x41` × 549** — pixel data chunks (56 bytes of RGB565 each, with offset + checksum header)
4. **`0x42`** — commit (display the frame)

The app draws the current track onto a hidden `<canvas>`, converts it to RGB565, and sends the full sequence.

A few details:

- **Push on change:** the screen keeps the last frame it received (in RAM — it's not saved, and replugging restores the default screen), so a frame is only sent when what it shows changes: a new song, play/pause, nothing playing, or a display setting. The *Progress* layout is the exception, since its bar moves every second. If any packet times out, the frame is resent on the next check so a half-sent image never sticks.
- **Double send:** the keyboard only displays every other frame it receives, and a single frame after a pause shows up white. Each frame is therefore sent twice back to back, which displays reliably.
- **Push timing:** a push takes about 1.5 s. A frame is 30 KB, sent as ~551 packets of 64 bytes, and the keyboard's USB connection accepts roughly one packet per millisecond — then it's sent twice. So expect the screen to update 1–2.5 s after a song change.
- **One push at a time:** drawing and pushing are queued, so a setting change and a Spotify check happening at the same moment can't interleave their frames.
- **Keyboard replies:** the keyboard echoes every packet back with an OK byte (`0x55`). These are instant receipts, not a "ready" signal, so the app doesn't wait for them.
- **Album art** uses Spotify's 300px cover, downscaled to 160px through the [weserv.nl](https://images.weserv.nl) image proxy (needed so the canvas can read the pixels).
- **Caching:** each cover is downloaded once per track and kept in memory (last 8 covers) along with its colour palette, so refreshes don't re-download it. The cache lives only while the page is open.
- **Ambient colours** are picked by sampling the cover at 32×32, grouping similar colours, and favouring common, saturated ones. They're then darkened for the gradient so white text stays readable.
- **Colours and dithering:** the screen uses RGB565 — 32 levels of red and blue, 64 of green (~65,000 colours). Dark, slow gradients fall between those levels, so they show either steps or dither dots; that's why Ambient offers a *Solid* background and a choice of dithering (ordered 4×4 pattern, Floyd–Steinberg error diffusion, or none). With the *Solid* background, colours the screen can show exactly are left undithered so it stays one clean colour.
- **Preview:** the on-page preview shows what the keyboard actually receives (reduced colours + dithering), not the full-colour drawing.

---

## Troubleshooting

Turn on **Debug messages** in the Log card first — it shows exactly what the page is doing.

- **Screen is white or didn't update:** click **Push Now**. If it keeps happening, look in the log for `packet(s) timed out` or pushes marked `(incomplete)`.
- **"Could not find screen interface":** unplug the keyboard, plug it back in, and click **Connect Keyboard** again.
- **Screen shows the keyboard's default image after replugging:** expected — frames live in the keyboard's memory only and aren't saved. Click **Connect Keyboard** again.
- **"Album art failed to load":** the image proxy didn't respond; it retries on the next check. The screen shows a grey square meanwhile.
- **Logged out / "Refresh failed":** click **Login with Spotify** again, then reconnect the keyboard (the login redirect drops the keyboard connection).

---

## Privacy

- Your Client ID is stored only in your browser's `localStorage` — it never leaves your machine
- Spotify tokens are also stored in `localStorage` and only used to call Spotify's API
- Album cover URLs (public Spotify image links, nothing tied to your account) are fetched through the images.weserv.nl proxy
- No analytics, no tracking, no server

---

## Limitations

- Chrome only — WebHID is not available in Firefox or Safari
- The page must stay open in a window (minimising doesn't work, but you can keep the the window open in the background and use another window to browse the internet)
- The keyboard must ALWAYS be connected using a usb cable. 

---

## Acknowledgements

Protocol reverse-engineered from [yunzii-game.com](https://yunzii-game.com) using Chrome DevTools WebHID capture.
