# fn-paks

Fortnite IoStore archives (`.utoc`/`.ucas` + `global.umeta`) pulled from the
Android install (Pixel 10a, Fortnite++Release-43.11) for `fn-master/fn-mapreader`.
Transport only; the files are unmodified copies.

- Stored with **Git LFS**. Get them with `git lfs pull` (all) or
  `git lfs pull --include="paks/Startup/*"` (one bundle).
- Layout: `paks/<Bundle>/<file>`; the phone path was
  `files/InstalledBundles/<Bundle>/FortniteGame/Content/Paks/<file>`.
- `MANIFEST.json`: name, size and sha256 of each file (sha256 = LFS oid).
- `LISTING.txt` / `LISTING_ALL_BUNDLES.txt`: phone inventory (`bytes<TAB>path`).
- `NOTES.md`: transport notes (route, what was skipped, anomalies).

Keep this repo private. It's separate from fn-master so its LFS storage can
be freed by deleting this repo when it's no longer needed.
