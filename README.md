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

That's it — the screen updates automatically every 2 seconds.

> **Order matters:** always log in to Spotify before connecting the keyboard. The OAuth redirect resets any HID connection.

---

## Options

| Control | Description |
|---|---|
| **Auto-update** | Toggle the refresh loop on/off |
| **Refresh interval** | 1s / 2s / 5s / 10s |
| **Layout** | *Clean* (big art left) · *Cover top* (art fills top) · *Progress* (progress bar + timestamps) · *Ambient* (Clean on a gradient themed to the album art) |
| **Art fit** *(Cover top only)* | *Zoom* (crop to fill) · *Blur sides* (blurred bg + crisp centre) · *Stretch* |
| **Rounded art corners** | Applies to all layouts |

---

## How it works

The keyboard exposes a 160×96 RGB565 framebuffer over WebHID. The protocol (reverse-engineered from the official yunzii-game.com web app) is:

1. **`0x40`** — init packet (signals start of frame transfer)
2. **`0x41`** — metadata packet
3. **`0x41` × 549** — pixel data chunks (56 bytes of RGB565 each, with offset + checksum header)
4. **`0x42`** — commit (display the frame)

The app draws the current track onto a hidden `<canvas>`, converts it to RGB565, and sends the full sequence every 2 seconds.

---

## Privacy

- Your Client ID is stored only in your browser's `localStorage` — it never leaves your machine
- Spotify tokens are also stored in `localStorage` and only used to call Spotify's API
- No analytics, no tracking, no server

---

## Limitations

- Chrome only — WebHID is not available in Firefox or Safari
- The page must stay open in a window (minimising doesn't work, but you can keep the the window open in the background and use another window to browse the internet)
- The keyboard screen resets after a few seconds without a push — hence the 2s auto-refresh
- The keyboard must ALWAYS be connected using a usb cable. 

---

## Acknowledgements

Protocol reverse-engineered from [yunzii-game.com](https://yunzii-game.com) using Chrome DevTools WebHID capture.
