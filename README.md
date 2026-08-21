# simple-qr-cli

A simple but flexible CLI for making QR codes.

Point it at a URL (or any text), pick colors and error correction, optionally drop a logo in the center, and get a `.png`, `.jpg`, or `.svg` — the format comes from the filename you give it.

## Requirements

- Python 3.9+
- `pip` (bundled with Python)
- `libcairo2` on Linux (needed by `cairosvg` for rasterizing SVG logos into PNG/JPG output). Install with `sudo apt install libcairo2` on Debian/Ubuntu. macOS ships this; Windows works out of the box with the `cairocffi` wheels.

## Setup

Clone the repo and set up a virtual environment:

```bash
git clone https://github.com/maryland-state-innovation-team/simple-qr-cli.git
cd simple-qr-cli

python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

The `.venv/` directory is git-ignored.

## Usage

```
qr.py [-h] -o OUTPUT [-e {L,M,Q,H}] [-c COLOR] [-b BACKGROUND]
      [--logo PATH] [--logo-scale FLOAT] [--logo-backing COLOR] [--square-backing]
      data
```

### Arguments

| Argument | Description | Default |
| --- | --- | --- |
| `data` (positional) | Text to encode. Typically a URL, but any string works. Quote it if it contains spaces or shell metacharacters. | — (required) |
| `-o`, `--output` | Output filename. The format is inferred from the extension: `.png`, `.jpg`/`.jpeg`, or `.svg`. | — (required) |
| `-e`, `--error-correction` | Error correction level. Higher levels tolerate more damage but produce denser codes. `L` ≈ 7%, `M` ≈ 15%, `Q` ≈ 25%, `H` ≈ 30%. | `M`, auto-upgraded to `H` when `--logo` is used |
| `-c`, `--color` | Foreground (module) color. Named color (`black`, `navy`, `red`) or hex (`#1a1a1a`). | `black` |
| `-b`, `--background` | Background color. Named color, hex, or `transparent` / `none` for an alpha-channel background (PNG and SVG only — JPEG has no alpha). | `white` |
| `--logo` | Path to a logo image (PNG, JPG, or SVG) to overlay in the center. | none |
| `--logo-scale` | Logo size as a fraction of the QR width. Values > 0 and < 1. Above ~0.28 the code often stops scanning. | `0.22` |
| `--logo-backing` | Solid color drawn behind the logo. Any named color, hex, or `transparent`. Useful when the logo has transparent pixels — a small opaque backing keeps QR modules from bleeding through. | `white` |
| `--square-backing` | Flag. By default the backing rect matches the logo's aspect ratio (a wide logo gets a wide backing). Pass this to force a square backing — good for near-square logos where a square looks tidier than a slightly-off rectangle. | off |

### Examples

Basic QR code to a URL:

```bash
python qr.py "https://maryland.gov" -o maryland.png
```

Higher error correction (useful when the code will be printed small or partially covered by a logo):

```bash
python qr.py "https://maryland.gov" -o maryland.png -e H
```

Custom colors — Maryland state red on cream, as PNG:

```bash
python qr.py "https://maryland.gov" -o maryland-brand.png -c "#c8102e" -b "#f7f4ec"
```

Vector output for print material:

```bash
python qr.py "https://maryland.gov" -o maryland.svg -c navy -b white
```

Transparent background — PNG or SVG only, so the code can sit on any colored surface:

```bash
python qr.py "https://maryland.gov" -o maryland.png -b transparent
python qr.py "https://maryland.gov" -o maryland.svg -b transparent
```

JPEG (smaller file, but lossy — prefer PNG or SVG for crisp scanning):

```bash
python qr.py "mailto:hello@example.com" -o contact.jpg
```

Encoding non-URL text works too:

```bash
python qr.py "WIFI:T:WPA;S:GuestNet;P:hunter2;;" -o wifi.png -e Q
```

Add a logo in the center. Error correction auto-bumps to `H` so the code stays scannable:

```bash
python qr.py "https://maryland.gov" -o branded.png --logo input/maryland-logo.svg
```

SVG logo embedded in SVG output stays fully vector — good for print:

```bash
python qr.py "https://maryland.gov" -o poster.svg --logo input/maryland-logo.svg
```

Larger logo on a transparent QR, with a matching transparent backing (only works if your logo already has opaque pixels covering the QR modules underneath — otherwise leave `--logo-backing` at its default):

```bash
python qr.py "https://maryland.gov" -o hero.svg --logo input/maryland-logo.svg \
    --logo-scale 0.26 -b transparent --logo-backing transparent
```

## How logos work on a QR code

QR codes have no native "logo hole" in the spec. Every logo QR you've seen exploits Reed-Solomon error correction — the decoder treats logo-covered modules as damage and recovers them, provided the total damaged area stays under the error correction budget (~30% at level `H`). The three big square finder patterns in the corners aren't error-correctable, which is why logos always go in the middle. Test your generated codes with a real phone camera before printing — anything above `--logo-scale 0.28` tends to fail.

## Choosing an error correction level

- **L** — smallest / lowest density. Fine for large, clean prints.
- **M** *(default)* — good general-purpose balance.
- **Q** — resilient against smudging and moderate damage.
- **H** — use when overlaying a logo or when the QR will live on stickers, packaging, or anywhere it might get scuffed.

Higher error correction means more modules, so the code becomes denser at the same physical size — scan it back after generation to confirm your camera picks it up.

## Choosing an output format

- **PNG** — best default. Lossless, small, universally supported.
- **SVG** — vector. Scales to any print size without pixelation. Ideal for posters, signage, and embedding in other vector artwork.
- **JPG / JPEG** — lossy compression can introduce artifacts around the module edges. Use only when a downstream system specifically requires JPEG.

## Colors

Colors accept anything Pillow accepts for raster output (CSS named colors, `#rgb`, `#rrggbb`) and anything valid as an SVG `fill` attribute for SVG output. In practice, named colors and 6-digit hex codes work in both.

Keep contrast high. Very light foreground or very dark background will make the code hard to scan — aim for at least a 4.5:1 contrast ratio between foreground and background.

## Exit codes

- `0` — success
- `1` — a runtime error occurred (e.g. couldn't write the output file, invalid color)
- `2` — bad arguments (unknown/missing extension, unknown flag, etc.)

## Project layout

```
simple-qr-cli/
├── qr.py              # the CLI
├── requirements.txt   # pinned dependencies
├── README.md
├── LICENSE
└── .gitignore
```

## License

MIT — see [LICENSE](LICENSE).
