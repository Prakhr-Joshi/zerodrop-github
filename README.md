# ZeroDrop

Private, browser-based peer-to-peer file transfer with an offline file vault.
One HTML file, no build step, no backend of your own.

- **P2P transfer** – pair two devices with an 8-digit code (or QR / link) and send files directly between browsers over WebRTC, split across 4 parallel data channels.
- **Encrypted chunks** – every chunk is encrypted with AES-GCM using a key derived from the pairing code (PBKDF2-SHA256).
- **Offline File Vault** – lock/unlock files with a passphrase (AES-256) entirely in your browser; locked files get the `.zkvault` extension.

## Deploy on GitHub Pages (free HTTPS hosting)

1. Create a new repository on GitHub (public).
2. Upload everything in this folder (keep `index.html` at the repo root).
3. Go to **Settings → Pages → Build and deployment**, set **Source** to *Deploy from a branch*, branch **main**, folder **/ (root)**, and save.
4. After a minute your app is live at `https://<your-username>.github.io/<repo-name>/`.

Command-line version:

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/<your-username>/<repo-name>.git
git push -u origin main
```

> GitHub Pages is served over **HTTPS**. That is why the app defaults to the public PeerJS signaling server (also HTTPS). A plain `http://` signaling server is blocked by browsers on an HTTPS page; the app ignores such a setting and falls back to the public server.

## How to use

1. Open the site on **both** devices.
2. Device A shows an 8-digit code and a QR code.
3. On device B type the code and press **Connect** (or scan the QR / open the shared link).
4. When the badge says **Connected** and 4 dots are green, drop a file on either side.

Each page load creates a **new** code. If the host reloads, read the new code from its screen.

## Status badge cheat-sheet

| Badge | Meaning | What to do |
|---|---|---|
| Signaling… / Retrying… | Reaching the signaling server | Wait; it retries 3 times on its own |
| Offline | Signaling server unreachable | Check internet, VPN, ad-blocker; press **New** or **Connect** to retry |
| Waiting | Ready, nobody joined yet | Share the code |
| Not found | No device is using that code | Re-read the code; make sure the host page is still open |
| Blocked | Devices found each other but no direct link could be opened | Same Wi-Fi, disable VPN, or configure a TURN server (below) |
| Connected | 4 channels open | Send files |

## Configuration

Everything configurable is at the top of the main `<script>` in `index.html`:

| Constant | Default | Purpose |
|---|---|---|
| `CODE_LENGTH` | `8` | Digits in the pairing code (use 6 for easier typing, less security) |
| `PBKDF2_ITERATIONS` | `250000` | Key-derivation work factor |
| `DEFAULT_SIGNAL_URL` | `""` | Empty = public PeerJS cloud. Set an `https://` URL to use your own server |
| `SIGNAL_TIMEOUT_MS` / `SIGNAL_MAX_RETRIES` | `12000` / `3` | How long to wait for signaling and how often to retry |
| `LINK_TIMEOUT_MS` | `20000` | Time before reporting a blocked direct link |

### Your own signaling server (optional)

```bash
npx peerjs --port 9000
```

Put it behind HTTPS (reverse proxy such as Caddy or nginx), then paste its URL into **Advanced → signaling** in the app, or set `DEFAULT_SIGNAL_URL`. For pure LAN use you can run it over `http://` and open the page from `http://localhost` or a LAN address instead of GitHub Pages.

### TURN server (recommended for different networks / mobile data)

Direct connections fail on some NATs and carrier networks. `getPeerOptions()` in `index.html` ships with the free public Open Relay TURN servers for testing. Their shared credentials are rate-limited and may change, so for real use create your own (for example a free Metered account, or self-host coturn) and replace the three `turn:` entries. Do not commit secrets you want to keep private; this is a static site, so anything in `index.html` is public.

## Privacy and security notes

- File data travels directly between the two browsers (WebRTC, DTLS-encrypted) and is additionally AES-GCM encrypted by the app. Nothing is uploaded to a server of yours.
- The signaling server only brokers the connection. Because the pairing code is both the room ID and the seed for the encryption key, treat it as a short-lived pairing secret, and prefer your own signaling server if you do not want a third party to see room IDs.
- If TURN is used, the relay forwards (already encrypted) traffic.
- An 8-digit code is convenient, not strong: this is a lightweight tool and has not had a formal security audit.

## Third-party scripts loaded from CDNs

Tailwind CSS (cdn.tailwindcss.com), Font Awesome and qrcode-generator (cdnjs), Google Fonts, and PeerJS 1.5.4 (jsDelivr / cdnjs / unpkg, tried in that order). The page needs internet access to load these. To run fully offline, download them and reference local copies.

## Repository layout

```
index.html   the whole app
README.md    this file
LICENSE      MIT (put your name in it)
.nojekyll    tells GitHub Pages to serve files as-is
.gitignore
```

## License

MIT, see [LICENSE](LICENSE).
