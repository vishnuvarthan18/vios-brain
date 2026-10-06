# Sensitive-species coordinate policy (set 6 Sep 2026)

## The rule

Exact GPS coordinates for sensitive species (elephant, leopard, tiger, vulture, pangolin, star tortoise, sandalwood, and anything else on the sensitivity registry) must be coarsened **only on production** — the public-facing sathyamangalam.online site.

**Dev and staging may hold exact, uncoarsened coordinates.** This is permitted because the project has permission from the forest department to work with precise location data in non-public environments. Coarsening is a publication-safety control, not a data-handling requirement everywhere.

## Why this was decided

During the D1 → atlas.db sync work (6 Sep 2026), the sync script found and fixed a real bug: harvest-engine's sensitivity matcher checks binomial names (`Elephas maximus`) but GBIF returns trinomials (`Elephas maximus indicus`), so the match never fired and 13 records (5 elephant, 2 leopard, 1 white-rumped vulture, plus others) had exact coordinates sitting unmasked in `public_lat`/`public_lon` fields. The sync patched this on the way in as a safety net.

When flagged, the user clarified: the *leak* risk only applies to what the public can see (production). Since dev/staging aren't public and the project already has forest department permission for precise location handling there, forcing coarsening in every environment is unnecessary and would degrade the data unnecessarily for internal analysis work.

## What this means going forward

- **Production build/deploy path**: must coarsen sensitive-species coordinates before anything reaches the public site. This is non-negotiable and should be enforced as a hard gate (a database constraint or a build-time check), not a policy note anyone could forget.
- **Dev and staging databases**: may carry exact coordinates for sensitive species. No coarsening needed there.
- **The underlying harvest-engine bug (binomial vs trinomial matching) still needs a real fix** — not because dev/staging need it, but because whatever eventually populates production must coarsen correctly, and right now it silently doesn't. This should be fixed before harvest-engine's next full run, not deferred to sync-time patching forever.
- Any future coding-agent instruction touching sensitive-species handling should state this distinction explicitly, so an agent doesn't over-apply coarsening to dev/staging or under-apply it to whatever builds production.
