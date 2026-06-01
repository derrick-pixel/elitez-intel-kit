# Changelog

## v1.1.0 — 2026-06-01

Security: close CSO findings raised by the 2026-05-30 Elitez Aviation
orchestra audit. Affects every consuming workspace.

- Vendor: bump `vendor/jspdf.umd.min.js` from jsPDF **2.5.1** (2022-01-28)
  to **4.2.1** (2026-03-17). Closes the CVE chain flagged by ESOP CSO-03
  / Aviation CSO NEW-H1 (2 CRIT + 8 HIGH on the 2.x line).
- Auth-gate: document why `shouldCreateUser` remains `true` and the
  Phase 2 hook canary that will let us flip it to `false`
  (Aviation CSO NEW-C2). No behavioural change.

## v1.0.0 — 2026-05-19
Initial extraction. Files copied verbatim from `competitor-intel-template`
(`template/assets/js/`, `template/assets/vendor/`, `auth-gate.js`) with zero
code changes, so byte-identical consumers can migrate mechanically.
- Fixed: `test` script (`node --test`); removed orphan `market-funnel.test.mjs` (its module is slated for a later release).
