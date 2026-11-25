# Transport-Notizen (Mac, 2026-11-25)

Quelle: Pixel 10a, `++Fortnite+Release-43.11-CL-60687381` (laut `Startup/BuildIdentity.txt`).
Ziel: dieses private Repo, Git LFS. Am Handy wurde nur gelesen.

## (a) Transportweg

Kein MTP und kein aShell-Umweg, sondern **adb über USB-Debugging**
(`android-platform-tools` via Homebrew). Die adb-Shell darf `Android/data`
direkt lesen, auf demselben Weg wie Shizuku. Pro Datei:

1. sha256 auf dem Handy (`sha256sum`)
2. `adb pull` in einen Staging-Ordner außerhalb des Repos
3. Größe und sha256 lokal mit den Handy-Werten verglichen
4. ins Repo verschoben, die LFS-Pointer-oid mit derselben sha256 verglichen
5. ein Commit pro Datei (`Add <name>`)

Pushes: Die ersten 4 Dateien wurden einzeln nach jedem Commit gepusht. Das
ergab nur ≈ 1,8 MB/s, weil pro Push nur ein LFS-Upload läuft. Den Rest habe ich
lokal committet und dann in einem einzigen Push hochgeladen, mit 8 parallelen
LFS-Uploads (4,25 GB in 23 Minuten, ≈ 3 MB/s). Ein abgebrochener Push lässt sich trotzdem
fortsetzen, weil LFS nur fehlende Objekte hochlädt.

**Abweichungen vom Auftrag (TRANSPORT.md in fn-master), mit dem Nutzer abgesprochen:**

- Quelle sind **alle** `InstalledBundles/*/FortniteGame/Content/Paks`. Im
  Auftragspfad `GFP_BaseInstallRoot/.../Paks` liegt nur `pakchunk30-*`, siehe (e).
- Ziel ist **dieses eigene Repo mit LFS** statt fn-master mit 90-MB-Splits.
  GitHub Pro enthält 10 GiB LFS-Speicher und 10 GiB Download pro Monat.
  LFS-Speicher wird nur durch Löschen des Repos wieder frei, deshalb ein
  getrenntes Repo.
- `global.umeta` ist mit dabei, weil es auf Android kein `global.utoc`/`.ucas` gibt.

## (b) Nicht transportiert

| Was | Anzahl | Größe | Grund |
|---|---|---|---|
| `pakchunk30-*` (.utoc, 3× .ucas, .pak, .uondemandtoc) | 6 | 810,7 MB | Ausschluss laut Auftrag (eigenes Menü-pak) |
| übrige `*.pak` | 29 | 171,3 MB | Ausschluss laut Auftrag |
| übrige `*.uondemandtoc` | 13 | 77,5 MB | kein .utoc/.ucas; Index für On-Demand-Streaming |
| DefaultTags-Platzhalter, `BuildIdentity.txt`, 2 Replays, `cpm.tflite`, Legal-txt, `ManifestArchive.journal` | 27 | 7,3 MB | keine IoStore-Archive |

Ordner außerhalb von `InstalledBundles` wurden nicht angefasst, darunter
`IoStore/InstallCache` (20 MB, Streaming-Cache), `ias/`, `RHICache/` und
`Saved/`. Alle Dateien mit Größe stehen in `LISTING_ALL_BUNDLES.txt`.

## (c) Gesamtgröße

68 Dateien, **4.720.512.504 Bytes** (4,72 GB / 4,40 GiB):
27 × `.utoc`, 40 × `.ucas`, 1 × `global.umeta`.

## (d) LFS

Ja. `.gitattributes` trackt `*.utoc`, `*.ucas` und `*.umeta`. Es gibt keine
Splits, deshalb ist `parts` in `MANIFEST.json` überall leer. Die sha256 im
Manifest ist gleich der LFS-oid.

Prüfung nach dem Push über die LFS-Batch-API (ohne Download): **68 von 68
Objekten auf GitHub vorhanden**, keines fehlt (2026-11-25 01:50).

Abrufen, ohne die 10 GiB Monatsbandbreite unnötig zu verbrauchen:

    GIT_LFS_SKIP_SMUDGE=1 git clone https://github.com/mashen456/fn-paks.git
    cd fn-paks
    git lfs pull --include="paks/GFP_BRRoot/*,paks/Startup/*,paks/FortniteCatchAll/*"   # gezielt
    git lfs pull                                                                       # alles, 4,4 GiB

## (e) Auffälligkeiten

1. **Layout:** Die Android-Installation ist auf ~38 Bundle-Ordner verteilt
   (`InstalledBundles/<Bundle>/FortniteGame/Content/Paks`), nicht auf einen
   Paks-Ordner wie am PC. Größte Bundles: FortniteBR 1,9 GB, GFP_BlitzRoot
   0,73 GB, GFP_BRRoot 0,61 GB, Startup 0,43 GB (dort liegt `pakchunk0`).
2. **Kein `global.utoc`/`global.ucas`.** Stattdessen gibt es
   `FortniteCatchAll/.../global.umeta` (121 MB, Magic `CONTAINERMETAHDR`,
   Version 1, ≈ 1,49 Mio. Einträge). Das Format ist unbekannt, ob CUE4Parse es
   liest, ist offen.
3. Alle `.utoc` haben **TOC-Version 10** und sind Oodle-komprimiert.
   11 Container sind **unverschlüsselt** (Key-GUID 0). Die 16 Bundles
   `Encrypted<GUID>` sind verschlüsselt, ihre Key-GUID ist der Ordnername.
   Diese Keys sind nicht in den Dateien enthalten. Zusammen sind es nur 72 MB.
4. **On-Demand-Inhalte:** Es gibt einen OnDemand-Container
   (`FatCosmo/pakchunk29ondemand`) und 14 `.uondemandtoc`. Ein Teil der
   Inhalte, vermutlich Textur-Mips, wird zur Laufzeit gestreamt und liegt
   nicht vollständig lokal.
5. Große `.ucas` sind in 256-MiB-Partitionen segmentiert (`_s1`, `_s2`).
6. Alle Dateien haben den Zeitstempel 2026-11-25 01:04–01:07, also eine
   frische Installation bzw. ein frisches Update kurz vor dem Transport.
