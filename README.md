# ZeroDrop

Send files directly between two browsers, with no account, no upload and no server of your own. A single `index.html` file, no build step.

- **Peer-to-peer transfer:** pair two devices with an 8-digit code, QR code or link. Files travel over WebRTC on 4 parallel channels.
- **Encrypted:** each chunk is encrypted with AES-GCM, using a key derived from the pairing code (PBKDF2-SHA256).
- **Offline vault:** lock and unlock files with a passphrase (AES-256) entirely in your browser. Locked files use the `.zkvault` extension.

## Use it

1. Open the site on **both** devices.
2. Device A shows an 8-digit code and a QR code.
3. On device B, type the code and press **Connect** (or scan the QR code or open the shared link).
4. When the badge says **Connected**, drop a file on either side.

A new code is created every time the page loads. If the host reloads, read the new code from its screen.

## Deploy on GitHub Pages

1. Put `index.html` at the root of a **public** repository.
2. Go to **Settings → Pages**, set **Source** to *Deploy from a branch*, choose **main** and **/ (root)**, then save.
3. After 1-3 minutes the site is live at `https://<username>.github.io/<repo-name>/`.

Pages serves HTTPS, so the app uses the public PeerJS server (also HTTPS) by default. An `http://` signaling server is ignored on an HTTPS page.

## Troubleshooting

| Badge | Meaning | What to do |
|---|---|---|
| Signaling… / Retrying… | Reaching the signaling server | Wait; it retries 3 times |
| Offline | Signaling server unreachable | Check your internet, VPN and ad-blocker, then press **New** or **Connect** |
| Waiting | Ready, nobody has joined | Share the code |
| Not found | No device is using that code | Re-read the code; keep the host page open |
| Blocked | Devices found each other but cannot open a direct link | Use the same Wi-Fi, turn off VPN, or add a TURN server |
| Connected | All 4 channels open | Send files |

## Configuration

Settings are constants at the top of the main `<script>` in `index.html`.

| Constant | Default | Purpose |
|---|---|---|
| `CODE_LENGTH` | `8` | Digits in the pairing code |
| `PBKDF2_ITERATIONS` | `250000` | Key-derivation work factor |
| `DEFAULT_SIGNAL_URL` | `""` | Empty uses the public PeerJS server; set an `https://` URL to use your own |
| `SIGNAL_TIMEOUT_MS` / `SIGNAL_MAX_RETRIES` | `12000` / `3` | Signaling wait time and retry count |
| `LINK_TIMEOUT_MS` | `20000` | Time before reporting a blocked direct link |

**Own signaling server (optional):** run `npx peerjs --port 9000`, put it behind HTTPS, then enter its URL under **Advanced** in the app or set `DEFAULT_SIGNAL_URL`.

**TURN server (recommended across different networks or mobile data):** `getPeerOptions()` includes the free public Open Relay servers for testing. They are rate-limited and may change, so replace the three `turn:` entries with your own credentials (for example a Metered account or self-hosted coturn). This is a static site, so anything in `index.html` is public.

## Privacy and security

- File data goes directly between the browsers, encrypted by WebRTC (DTLS) and again by the app (AES-GCM). Nothing is uploaded to your server.
- The signaling server only helps the devices find each other. The pairing code is both the room ID and the encryption-key seed, so treat it as a short-lived secret and use your own signaling server if you don't want a third party to see room IDs.
- If a TURN relay is used, it forwards already-encrypted traffic.
- An 8-digit code is convenient rather than strong, and this tool has not been security-audited.

## Dependencies

The page loads Tailwind CSS, Font Awesome, a QR code library, Google Fonts and PeerJS 1.5.4 from CDNs (PeerJS falls back across unpkg, jsDelivr and cdnjs). It needs internet access; to work offline, download these and reference local copies.

## Files

```
index.html   the whole app
README.md    this file
LICENSE      MIT
.nojekyll    makes GitHub Pages serve files as-is
.gitignore
```

## License

MIT. See [LICENSE](LICENSE).
