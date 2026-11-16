# Tools

The scripts behind this Oman-only run of global-law's vulnerability detectors. They run inside a copy of
[global-law/vulnerabilities/scripts](https://github.com/lexifina/global-law/tree/main/vulnerabilities/scripts) (Python 3.11+,
`pip install duckdb pyarrow`), with a bare clone of this repository at `sources/corpus/oman-law.git`, a blobless bare
clone of global-law at `sources/global-law.git`, and a `pins.json` that names only oman-law and its commit (language
`ar`). `build_vulns.py`, `flags_chart.py`, `late_ext_select.py` and `late_ext_rebut.py` sit two folders above the scripts,
next to `config.json`.

| Script | What it does |
| --- | --- |
| `run.sh` | Indexes the history and texts, maps moved paths (`map_moved.py`), adds this repository's history rows, and runs the detectors (`bash run.sh <shards>`) |
| `map_moved.py` | Rewrites the paths in the history table to each act's folder since the move of 10 November 2026 (`data/paths.json`: 93 folders, from `rd-<year>-<number>` to `rd/<year>/<number>`), so that an act's history is not split in two at the move |
| `add_root.py` | Builds the history rows from this repository's own commits (global-law replays it only up to an earlier commit), so the history detectors read every change |
| `run_triage.sh` | The model review: the 40 strongest flags per category, the 40 strongest lapsed time limits in law texts, and the empty periods (here 4 flags, the 2 lapsed time limits reviewed in both passes, and no empty periods) |
| `late_ext_select.py` | Chooses the empty periods for global-law's review of that category (there were none here) |
| `late_ext_rebut.py` | Adds the second, arguing-against pass to the empty-period review, as `triage.py` does for the other categories |
| `build_vulns.py` | Builds `data/flags.csv`, `data/reviewed.csv` and `data/by_category.csv` from the findings and the model review |
| `flags_chart.py` | Draws the chart at the top of the READMEs, in Arabic and English, light and dark; the Arabic chart is drawn right to left |
| `config.json` | The repository, the Arabic category names, the files that are not law, and the chart's labels |
| `patch_detectors.py` | Fixes blind spots found in runs on several country repositories in the copy of global-law's scripts (`python -X utf8 patch_detectors.py <scripts dir>`) |
| `patch_detectors_vi.py`, `patch_es.py`, `patch_ca.py`, `patch_en.py`, `patch_de.py`, `patch_en_et.py`, `patch_co.py`, `patch_sv.py`, `patch_pt.py`, `patch_ru.py`, `patch_r18.py`, `patch_xk.py`, `patch_mo.py`, `patch_lu.py`, `patch_li.py`, `patch_mx.py`, `patch_mn.py`, `patch_mt.py`, `patch_np.py`, `patch_en_nr.py`, `patch_dv.py`, `patch_az.py` | The same for the blind spots other runs found in other languages and repositories; this run's copy carries them all. None of them reads Arabic, beyond the reference scan's "المادة (N)", which was already there |
| `patch_ar.py` | The fixes for Arabic, below (`python -X utf8 patch_ar.py <scripts dir>`; each change carries a `patch_ar` marker, and one already in place is skipped) |
| `apply_patches.py` | Applies every patch above once, in order, recording each in `.kit_patches.json` in the scripts folder (`python -X utf8 apply_patches.py <scripts dir>`); `patch_ar.py` is last in its list |
| `patch_build_r7.py` | Brings an older country's `build_vulns.py` up to date (`redo.py` runs it; the `build_vulns.py` here already has its changes) |
| `redo.py` | Runs `apply_patches.py`, then again only the scans and detectors the fixes change, the model review of lapsed time limits, references to deleted provisions and empty periods, and the build (`python -X utf8 redo.py <run folder> [--keep-reviews] [--full]`; `--full` runs every step again through `run.sh`; `EMBED_FONTS` names the script that embeds the chart's fonts); not needed here, as every fix was in place before the run whose results are published |

The model review needs `LLM_BASE_URL`, `LLM_API_KEY`, `LLM_MODEL` and `LLM_AUTH` set for an OpenAI-compatible Responses API.
It was run as `bash run_triage.sh '^(data/|tools/|vulnerabilities/|README|REPRODUCE|LICENSE|jurisdiction)'`; every law is
under `nizam-asasi/`, `qanun-al-jaza/`, `himayat-al-bayanat/` or `rd/`, so the review of lapsed time limits in law texts
drew the same 2 flags as the review of all of them, and `build_vulns.py` keeps the later review of each.

Changes made for this repository:

- `pins.json` names the corpus language as `ar`: every text is in Arabic.
- `add_root.py` is unchanged: the 10 commits credited to the royal decree that made a change (among them the first text of
  the Basic Statute, the Penal Code and the Personal Data Protection Law) carry `Kind: decree`, and the 93 that hold the
  current text of every other act carry `Kind: publisher`; the kit counts both as law. The commit that moved the act
  folders carries `Kind: documentation` and is left out, as are the commits with no `In-force` trailer (READMEs, tables,
  tools, `REPRODUCE.md`).
- `patch_ar.py`, time limits (`ingest/textscan.py`): no Arabic "until" was read, so the search found no lapsed time limit.
  It now reads "حتى", "وحتى", "لغاية", "ولغاية" and "إلى غاية" before a date, "تاريخ" or "يوم" between them ("حتى تاريخ
  ..."), "قبل نهاية سنة/عام" (before the end of the year) before a year, and the Gregorian months (يناير ... ديسمبر, with
  the spellings ابريل/إبريل, يونيه, يوليه, اغسطس, اكتوبر, and the Levantine كانون الثاني ... كانون الأول), with "من شهر"
  before the month and "سنة"/"عام" before the year. Arabic-Indic digits (٠-٩; every one of the 11,585 digits in the law
  texts is one) needed nothing: Python's `\d` and `int()` read them. Lapsed time limits went from 0 to 2. Measured on
  the law texts at this commit: they hold one "until" with a full date ("حتى ١ / ١١ / ١٩٩٦م") and one "before the end of
  the year" ("قبل نهاية سنة ٢٠٢٤م"), both now read; one "until" gives only a month ("حتى شهر يونيو ١٩٨٦م") and is left out,
  as month-only dates are in other languages; of the 14 dates with a month name, none follows an "until". No date in the
  law texts is written in the Hijri calendar ("١٤٤٢هـ"), so Hijri dates were left unread. "قبل" alone (before) is not read
  as a limit: "قبل ١ / ٨ / ١٩٩٤م" in Article 20(d) of Royal Decree 86/96 bounds a period of past service.
- `patch_ar.py`, amounts (`ingest/textscan.py`): amounts in rials and baisas, written as a number (often in brackets),
  the amount in words and then the currency ("(٥٠٠) خمسمائة ريال عماني", "(١٠) عشرة ريالات عمانية"), were not read; they
  are now, in a corpus whose language is `ar`, for the search for stale amounts. It found none: no amendment in the history replaces an amount.
- Not fixed: the reference scan (`ingest/refscan.py`) reads "المادة (N)" in either digit set (974 references in the law
  texts), but not lists and ranges after "المواد" ("المواد (١٥)، (١٦)"; "من (١٥٣) إلى (١٥٨)"), "مكررا" after the number
  (article N bis), or ordinal words ("المادة الأولى", which mostly name the articles of an issuing decree). Its detector
  also needs a deleted article in the history, which the 93 one-version acts cannot show. Read by hand, one reference
  points to an article that is no longer in force: Article 159 bis (21) of the Income Tax Law to Article 135 bis (see
  EDGE-CASES.en.md).
- `flags_chart.py` draws the Arabic chart mirrored, right to left (`direction="rtl"` on each text). The Arabic labels
  use the reader's system font (the embedded fonts, Manrope and JetBrains Mono, have no Arabic glyphs), which shapes the
  letters; the chart was checked in a browser render.
- `config.json` counts as not law `data/`, `tools/`, `vulnerabilities/`, the root files and every `README.md` (each
  folder's list of its articles).
- There are no amounts in a dead currency: the amounts are in rials and baisas.
