# libpdx-font

Bitmap glyph store + text metrics library. Ships the 8x16 font used by the kernel framebuffer console verbatim, plus a metrics API that emits `FontMetricsRecord@0.1` on the compositor's semantic-pipe surface.

Part of the **paideia-os** organization. MIT-licensed.

## Wave

R102 (softarch userland graphical stack) — the CPU-side framebuffer stack
that lands the first graphical UI on paideia-os before the G-series
GPU-accelerated compositor matures. Companion to the osarch R101 kernel-side
plan.

## Design reference

- Design lives in the monorepo at [`design/graphics/r102-user-plan.md`](https://github.com/paideia-os/paideia-os/blob/main/design/graphics/r102-user-plan.md) §2.1 / §4.1.
- Kernel-side companion: `design/graphics/r101-kernel-plan.md`.

## Milestones

Per the plan, this repo lands across five milestones:

- **M1** — repo scaffold; caps.decl (KIND_MEMORY for glyph store); frozen public API stubs
- **M2** — 8x16 glyph store copied bit-exact from fb_font.pdx; font_open/font_glyph/font_layout_line/font_metrics real bodies; fixed-advance layout
- **M3** — FontMetricsRecord@0.1 emission (schema-registry integration when it lands; fallback-line-based until then)
- **M4** — smokes: bit-exact glyph equivalence to kernel-console, UTF-8-out-of-range fallback, per-line layout correctness
- **M5** — second face (16x32) landed as font_open('default-large'); signed 1.0.0 release
  — `src/font_16x32.pdx` (`Font16x32`) ships the 16x32 glyph store; a
  `font_open('default-large')` selector API is future follow-up (not
  wired in this landing). 1.0.0 ships as unsigned source form
  (`v1.0.0-src`) pending a real `paideia-as release --sign` pass.

Every issue is filed against one of these five milestones; see the Issues tab.

## Scaffolding

No code lands with this repo scaffold — scaffolding lives in the M1
issues (`caps.decl`, `src/` skeleton, public API stubs, argv parsing).
Repo shape mirrors R100 satellites: paideia-as manifest at root,
`caps.decl` at root, `src/` module tree, `tests/`, `release/`,
`doc/<name>.pdxdoc`, dual-signed `manifest.pdxsig` at 1.0.0.

## License

MIT. See [LICENSE](LICENSE).
