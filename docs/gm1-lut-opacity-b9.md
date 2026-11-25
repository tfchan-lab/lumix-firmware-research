# GM1 LUT Opacity — B9 Release

**Release date:** 2026-11-25
**Camera:** Panasonic DMC-GM1, official firmware 1.1, internal `0110`
**Status:** stable functional release
**Tag:** `gm1-lut-opacity-b9`

B9 is the release build for the native LUT opacity slider. It keeps the stock
profile switching and stock sliders working, and adds a working LUT blend,
bar and numeric readout.

The adjustment-row label on the LUT page still reads **Contrast**. Label
removal was attempted in B10, B11 and B12 and was abandoned: the first two
hooked the wrong table, and B12 changed the row value/layout and left the
label in place. B9 deliberately does not include those experiments.

## Tested behavior

| Area | Result |
|---|---|
| Stock profile switching | working |
| Stock adjustment sliders | working |
| LUT blend | working |
| LUT bar | working |
| LUT numeric readout | working, native `-5..+5` scale |
| Native Contrast decoupled from LUT opacity | yes |
| LUT row label | still `Contrast` (accepted limitation) |

## Package

| Item | Value |
|---|---|
| Package | `work/opacity-native/slider-package-20261125T103211Z/GM1__V11.bin` |
| Package SHA-256 | `85792aba1fbd283427dd89d1ba8b13d7f968f1de279f840bdbfc0f203f24bfd3` |
| Package size | 23,987,712 bytes |
| Program SHA-256 | `2c0beae279d9b4d07f40cba4869d11d79554db85c9ad4b1404df16816e1b462b` |
| guires SHA-256 | `9fa9f03ca4a2cc665d32d56537ed03cabd9094e623e8a978177be5fad594e321` |
| program2 SHA-256 | `21b5cdae4fd4ee66892028a611f345a7eab985f56c4ac30f27cd39c643479cbc` |
| Stock restore SHA-256 | `cad4ea2739407f3d59d919e87830fde4e1bad03cb3c26d87c53f7e734a45c007` |
| Stock restore size | 24,512,000 bytes |

The `preparation.json` inside the package was written before the release
decision and still says "test package, not a release" and "label pending".
The binary above is the tested B9 release binary; the note here supersedes
that stale package metadata.

## Install

1. Copy `GM1__V11.bin` to the root of the SD card.
2. Flash through the normal GM1 1.1 update flow.
3. After reboot, verify a stock profile first: switching and sliders must be
   normal.
4. Open a loaded LUT style's adjustment page and verify the bar, number and
   blend.

The card image used for the release verification had SHA-256
`85792aba1fbd283427dd89d1ba8b13d7f968f1de279f840bdbfc0f203f24bfd3`.

## Known limitations

- The LUT adjustment row label still reads `Contrast`.
- Power-cycle persistence is not yet proven.
- The stored opacity is a 0..100 strength in 10% steps; the numeric display
  uses the native `-5..+5` scale.
- B10/B11/B12 are experiments and must not be used as release builds:
  - B10 `d5f05e81...f920f`: label unchanged;
  - B11 `d1c9db78...edbe7`: label unchanged;
  - B12 `e8042d3e...0dab7`: label unchanged and the numeric readout stuck.

## Rollback

Use `stock-restore/GM1__V11.bin` from the package, or the copy on the PC with
SHA-256 `cad4ea2739407f3d59d919e87830fde4e1bad03cb3c26d87c53f7e734a45c007`.

## Source and artifacts

The module source, build scripts, package builder and test dumps live under
`work/opacity-native/slider/` and are kept out of Git by the repository
policy. The release identity is the package and component hashes above.
