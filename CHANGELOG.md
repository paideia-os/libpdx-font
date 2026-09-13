# CHANGELOG — libpdx-font

Every libpdx-font release follows semver (design/graphics/r102-user-plan.md
in the paideia-os repo). The 0.5.0 line closes Milestones M1 and M2 of the
five-milestone plan in this repo's README; every subsequent entry adds one
line to the top under the same shape.

## 0.6.0 — 2026-09-13

Wave CC follow-up: M3 schema-registry integration (WEAK stub) + first
M4 smoke, closing libpdx-font#6 and #7.

### Landed

- **libpdx-font#6 (R102.M3-001) — schema-registry integration (WEAK
  stub).** `src/font_schema.pdx` (`Module FontSchema`) adds
  `font_schema_init() -> u64`, intended to register the
  `FontMetricsRecord@0.1` schema DDL with a real schema-registry client
  and store the returned handle in module-level
  `font_metrics_schema_id`. `deps.list` holds no linkable
  `libpdx-schema-registry` dependency at this landing, so the body is a
  WEAK stub returning a fixed, deterministic handle
  (`FS_SCHEMA_ID_STUB` = 1) rather than fabricating a real registration
  round-trip — same "declared dependency, unprovisioned resource"
  posture `pdxping`'s own `src/elevate_gate.pdx` / `src/audit_wire.pdx`
  document for the identical kind of gap. **Naming note**: this file
  registers under `FontMetricsRecord@0.1` (the name this issue and this
  repo's own README use); the M2-003 wire-shape module
  (`src/font_metrics.pdx`) names the identical 128-byte layout
  `MetricsRecord@0.1` internally — flagged in `font_schema.pdx`'s own
  header for a future cleanup pass to reconcile the two names, not
  fixed in this commit (out of scope).
- **libpdx-font#7 (R102.M4-001) — glyph equivalence smoke.**
  `tests/font_equivalence_smoke.pdx` (`Module FontEquivalenceSmoke`)
  adds `font_equivalence_smoke_run() -> u64`, comparing `Font8x16`'s 26
  ASCII-letter glyphs (`'A'..'Z'`, 0x41..0x5A) against the reference
  bytes paideia-os's `fb_font.pdx` currently holds for that range,
  returning the number of glyphs that differ (0 = byte-identical).
  **Byte-inspected at paideia-os HEAD (commit ae011e2)**: every glyph
  in `fb_font.pdx`'s printable-ASCII placeholder range is the identical
  flat 16-byte `0xAA`/`0x55` alternating-row stripe — confirmed for all
  26 letters, not merely asserted from prose — so the reference is
  encoded as that one 16-byte pattern rather than a copied binary
  asset. Because this library ships REAL vendored glyph pixels
  (`src/font_8x16.pdx`'s own Lat15-VGA16 provenance) while
  `fb_font.pdx` still ships a placeholder, running this smoke TODAY is
  *expected* to report all 26 glyphs mismatched
  (`FES_EXPECTED_MISMATCH_AT_THIS_LANDING` = 26) — not a bug. This
  assertion becomes load-bearing the moment paideia-os lands real
  glyph pixels in `fb_font.pdx` (its own header: "R23.M2 replaces the
  asset with actual... vgacon-8x16 glyph data"): a re-run returning 0
  at that point confirms kernel/user glyph parity; a nonzero return is
  the signal to open a real cross-repo divergence issue.

## 0.5.0 — 2026-09-13

Wave CC: single implementation pass landing the M1 scaffold + M1 API
stubs + M2 glyph store + M2 layout, closing libpdx-font#1 through #5 in
one commit.

### Landed

- **libpdx-font#1 (R102.M1-001) — repo scaffold.** README.md and
  LICENSE (MIT) were already present from repo creation. Added
  `caps.decl` (`requires: - KIND_USER`, modern list-form grammar
  matching the shell/libpdx-argv/rm 2026-09 convention rather than the
  older R90 `!KIND_FOO <mask>` line grammar), `tools/build.sh` (per-repo
  `paideia-as build --emit elf64` driver, copied from the libpdx-argv /
  rm shape: resolves `paideia-as` via `$PAIDEIA_AS` / sibling checkout /
  `$PATH`, requires >= 0.21.0), `manifest.pdxsig` in unsigned source-form
  (dual-sig placeholder record, same layout as rm's, sized for the
  author + paideia-root ML-DSA-65 re-sign), `VERSION` (`0.5.0`), and
  `deps.list` (no library dependencies at 0.5.0).
- **libpdx-font#2 (R102.M1-002) — caps.decl + public API.**
  `src/font_api.pdx` (`Module FontApi`) with `font_lookup_glyph`,
  `font_glyph_bitmap`, `font_measure_text`. Landed as REAL bodies
  rather than pure stubs (see M2 note below) because #2 through #5 land
  in the same commit and a stub-then-immediately-replace churn inside
  one commit adds no review value.
- **libpdx-font#3 (R102.M2-001) — 8x16 glyph store.**
  `src/font_8x16.pdx` (`Module Font8x16`) + `assets/fonts/glyphs_8x16.bin`
  (4096 bytes, `@include_bytes` + `@align(64)`, same shape as
  paideia-os's `fb_font.pdx`). **Provenance gap**: paideia-os's
  `src/kernel/core/drivers/fb_font.pdx` documents itself (see its own
  header) as importing `assets/fonts/vga8x16.bin`, and that asset is
  confirmed (byte-inspected at HEAD) to be a **placeholder** — mostly
  zero bytes, with an `0xAA`/`0x55` alternating-row stripe standing in
  for ASCII 0x21..0x7E — not real glyph pixel data. Bit-exact import
  from `fb_font.pdx` per the issue text is therefore impossible today;
  the kernel side has not landed real glyph pixels yet (its own header:
  "R23.M2 replaces the asset with actual... vgacon-8x16 glyph data").
  Rather than fabricate a synthetic ASCII-only placeholder per the wave
  fallback instruction, this repo vendors a **real, public-domain 8x16
  VGA console font**: `Lat15-VGA16.psf.gz` from Debian/Ubuntu's `kbd` /
  `console-setup` package (PSF1 format; confirmed "All console fonts
  are public domain by nature" per `/usr/share/doc/console-setup/
  copyright`). The 4096-byte glyph table (offset 4, 256 glyphs x 16
  bytes) was extracted verbatim — same 8x16, MSB-left, 256-glyph byte
  layout `fb_font.pdx` itself specifies, and visually verified (ASCII
  'A', '@', 'g' render correctly). Follow-up: when paideia-os lands
  real glyph pixels in `fb_font.pdx` (R23.M2), a future libpdx-font
  issue should re-sync onto that exact source for kernel/user glyph
  parity; until then this is real font data, not a placeholder, just
  not byte-identical to the (not-yet-real) kernel asset.
- **libpdx-font#4 (R102.M2-002) — fixed-advance text layout.**
  `src/font_layout.pdx` (`Module FontLayout`) with `font_layout_text`.
  Advances x by 8px per glyph; `\n` (0x0A) resets x and advances y by
  16px. Emits one `(x, y, glyph_id)` u64 triplet per non-newline byte
  into caller's `out_ops` buffer (24 bytes/op; no bound check — the
  issue's literal 5-arg signature carries no `out_max`, so the buffer-
  size contract is documented in the function's header comment: size
  for `len * 24` bytes as a safe upper bound).
- **libpdx-font#5 (R102.M2-003) — metrics record.**
  `src/font_metrics.pdx` (`Module FontMetrics`) with
  `font_emit_metrics_record`, emitting the 128-byte
  `MetricsRecord@0.1` shape to fd 2 via a raw `sys_write` (mirrors
  libpdx-argv's `VersionBackend`/`HelpBackend` fd-1 carve-out: this
  library holds no capability of its own for the write, it inherits
  whatever fd-2 authority the linking executable already holds).

### Encoder-arity adaptation (feedback_pdx_encoder_pitfalls)

Two of the issue-specified signatures exceed the 4-curried-arg
encoder ceiling this repo follows (same discipline cp's
`pdxfs_txn_bind` documents): `font_layout_text(text, len, x, y,
out_ops)` is 5 args and `font_emit_metrics_record(op, glyph_count,
bytes_rendered, viewport_w, viewport_h)` is 5 args. Both pack their
two excess u64s into one register: `font_layout_text` takes
`xy0_packed` (x in the high 32 bits, y in the low 32 bits) in place
of separate `x`/`y`; `font_emit_metrics_record` takes
`viewport_wh_packed` (w high, h low) in place of separate
`viewport_w`/`viewport_h`. Unpacked via `shr`/`shl`+`shr` (no `and
reg, imm64`, per the encoder pitfall list). Documented in each
function's own header comment.

### Not landed this pass

- No `tests/` fixtures yet — M4 per the README's milestone table.
- No `doc/<name>.pdxdoc` — M5 (signed 1.0.0 release).
- `manifest.pdxsig` ships as an unsigned source-form placeholder;
  real ML-DSA-65 signing is a release-time (signing-bot) step, same
  posture rm 1.0.0 shipped under.
