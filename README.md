# FAKE ン RARE — Living Plate

A Fake Rare poster that repaints itself. The banner turns every six hours through 501 places —
the artist's own line, then the 500 highest-valued Fake Rares — and comes home every 125.25 days.
The seal carries the Bitcoin block each name was issued in. The sweatshirt turns over every 24
hours between his F and his C.

Everything is in one file: `fake-rare-living-plate.json` (2.03 MB). It carries the painting, the
pigment, the 500-asset roster, the rotation order, and the dry-brush rendering engine itself —
base64'd, with a SHA-256 for each part. **Nothing is fetched at runtime.** Once assembled, the
plate runs offline forever.

---

## Files

| File | Size | What it is |
|---|---|---|
| `fake-rare-living-plate.json` | 2.03 MB | The artwork. Canonical, self-verifying. |
| `index.html` | 9 KB | Viewer — fetches the JSON, checks all 5 hashes, assembles, renders. |
| `plate.html` | 1.72 MB | Sealed standalone copy. Zero network. Open it anywhere. |
| `FAKENRARE.json` | 1 KB | Counterparty enhanced-asset-info metadata. **This is the URL that goes on-chain.** |
| `FAKENRARE.png` | 4.2 MB | The home plate, 1600×2240 lossless master. |
| `FAKENRARE.jpg` | 627 KB | Same still, display weight — this is what `image_large` points at. |
| `FAKENRARE-48.png` | 5 KB | 48×48 PNG — required exactly by the Counterparty spec. |
| `FAKENRARE-256.png` | 90 KB | Social / preview card. |
| `FAKENRARE-800.jpg` | 213 KB | Mid-size still, for anything that wants one. |

The stills are all **slot 0** — the home plate, his own `FAKE ン RARE` on the banner and his F on
the chest, no block seal. That is the plate as it was painted, and the right face for the asset.

### Verified

All five declared hashes match their payloads:

```
renderer     38,545 bytes   56ffa11005dd1193…   OK
base_plate  741,304 bytes   360ff9ebc44d6a84…   OK
fabric       86,130 bytes   284124be9f0d1467…   OK
letter_c    109,464 bytes   9c57dac5d6787172…   OK
home_plate  385,435 bytes   c3e3db3e88d0f5c7…   OK
```

The viewer re-checks every one of them in the browser via SubtleCrypto before it strikes the
plate. If a byte of the JSON is ever altered in transit or at rest, the page refuses it and falls
back to the sealed copy.

---

## Hosting

### Live: GitHub Pages (viewer) + jsDelivr (the JSON)

Both halves are up and verified:

| | URL | Verified headers |
|---|---|---|
| **Viewer** | https://aqua-019.github.io/fake-rare-living-plate/ | `200`, renders the plate |
| **Metadata** (on-chain) | `.../fake-rare-living-plate/FAKENRARE.json` | `application/json`, CORS `*` |
| **The artwork** | `.../fake-rare-living-plate/fake-rare-living-plate.json` | `application/json`, CORS `*`, 2,126,649 B |
| **Artwork, immutable** | `cdn.jsdelivr.net/gh/aqua-019/fake-rare-living-plate@ef06ba4.../fake-rare-living-plate.json` | `application/json`, CORS `*`, `max-age=31536000, immutable` |

First commit: `ef06ba46152b29710fbfcb3ec8bd65ba54fe1432`

The served JSON hashes to `d4b0334cc0cb1216d6228c1be22b4caa12b725ea41beea49a6570b807542c6cd`,
byte-identical to the source. The jsDelivr URL pinned to that **full commit SHA** is the one that
belongs on-chain: content-addressed by git, global CDN, and it can never change under the asset.

`.gitattributes` sets `* -text` so git can never rewrite a byte of this repo on any platform.

To cut a new version later:

```bash
git add -A && git commit -m "..." && git push
git rev-parse HEAD          # <- new SHA for a new immutable jsDelivr URL
```

### Why not `raw.githubusercontent.com`

It works, and it is what most XCP mints use, but it is the weakest option available:

- serves `content-type: text/plain`, not `application/json`
- `cache-control: max-age=300` — no CDN, five-minute cache
- ships a `content-security-policy: sandbox` header
- `refs/heads/main` is **mutable** — the art can change or vanish under the asset
- GitHub [rate-limits unauthenticated raw downloads](https://github.blog/changelog/2025-05-08-updated-rate-limits-for-unauthenticated-requests/), and Counterparty re-queries every asset's JSON **every 30–60 minutes**

Same repo, same files — just point the on-chain URL at jsDelivr instead of raw.

### Alternative: Vercel

`vercel.json` is included with the correct `Content-Type`, CORS, `Cross-Origin-Resource-Policy`
and cache headers already set. Use this if you want the plate on a custom domain
(`plate.fakeraredirectory.com`). Note that Vercel gives you no content-addressed URL — keep
jsDelivr as the canonical on-chain pointer even if the viewer lives here.

### If you want true permanence

For art that must outlive any host, put the JSON on **Arweave** (one-time fee, permanent) or
**IPFS** pinned through a service, and use the `ar://` / `ipfs://` hash on-chain with the
GitHub/jsDelivr URL as a gateway mirror. The JSON is already fully self-contained, so it needs
nothing else to survive — which is exactly what makes it a good candidate.

---

## Counterparty

`FAKENRARE.json` follows the [enhanced asset info spec](https://docs.counterparty.io/docs/basics/assets/enhanced-asset/).
Before broadcasting:

1. Set `asset` to your real asset name (must match exactly, ≤24 chars).
2. The URLs already point at this repo. Swap in the jsDelivr or a custom domain if you prefer.
3. Keep `image` pointing at a 48×48 PNG and the URL under 100 characters — both are hard
   requirements in the spec.
4. Broadcast the URL to that JSON as the asset description. **It must end in `.json`.**

`icon`, `image_large` and `animation_url` are not in the core spec — they are the extensions
Xchain and pepe.wtf read. Harmless to explorers that ignore them, and they are what make the
living plate show up as the animated piece rather than a still.

---

## Runtime

```
genesis     2026-09-22T00:00:00Z
banner      501 slots, one turn every 6h → full cycle 125.25 days
sweatshirt  flips every 24h between F and C
slot 0      the untouched original — his FAKE ン RARE, his F, block seal only
shuffle     0x1A5EFA11, xorshift32 Fisher-Yates
```

`slot = floor((now − genesis) / 6h) mod 501` — deterministic, identical in every browser, forever.
