# How this repository was made (Oman)

This file is written by the build on 2026-10-29, the day the source data was retrieved. Everything below is enough to rebuild this repository hash for hash.

## Sources

| Source | Address | Retrieved | Requests | Licence |
| --- | --- | --- | ---: | --- |
| Qanoon.om | <https://qanoon.om> | 2026-10-29 | 28 | Qanoon.om states that the texts of law are public property in Oman under the Copyright and Related Rights Law and may be used, copied, published and translated freely; it says it is not an official source and refers users to the Official Gazette. |
| Ministry of Justice and Legal Affairs (MJLA), laws section | <https://mjla.gov.om/laws/1> | 2026-10-29 | 0 | Government Open Licence - Sultanate of Oman (stated in the site footer). |

Every response is cached by the sha256 of its URL, and the table above lists every source with its retrieval date. Lexifina keeps its fetch scripts privately; `tools/parse.py` reads only the cache and writes `corpus.json`, so the same cache always gives the same corpus.

## What is in it

| Act | Title | Versions | Span |
| --- | --- | ---: | --- |
| `nizam-asasi` | النظام الأساسي للدولة | 4 | 1996-11-06 to 2026-09-13 |
| `qanun-al-jaza` | قانون الجزاء | 4 | 2018-01-15 to 2026-06-29 |
| `himayat-al-bayanat` | قانون حماية البيانات الشخصية | 2 | 2023-02-13 to 2026-09-07 |

The build made 10 commits:

| Commits | Kind |
| ---: | --- |
| 3 | first version of an act |
| 7 | one law on its in-force day |
| 0 | changes no amendment note explains, credited to the publisher |

Other counts:
- 0 commits on the branch `future`: laws not yet in force on 2026-10-29.
- 0 commits dated 1 January 1970: in force before 1970, with the real day in their `In-force` trailer.
- 10 laws in the record.
- 20 tags.

## Dates

Each commit is dated at noon (Asia/Muscat) on the day the change took effect. The other dates, enacted and published, are trailers.

When a law gives no date of its own, the default rule applies: Basic Statute of the State (RD 6/2021) art. 91: "تنشر القوانين في الجريدة الرسمية خلال أسبوعين من تاريخ إصدارها، ويعمل بها من تاريخ نشرها، ما لم ينص فيها على تاريخ آخر." (Laws take effect from publication in the Official Gazette unless they fix another date.) Every decree built here fixes its own date. https://qanoon.om/p/2021/l2021006/

## Commands, in order

```sh
# fill the cache from the sources listed above (network; Lexifina's fetch scripts are kept privately)
python -X utf8 tools/parse.py                  # cache -> corpus.json (no network)
python -X utf8 tools/build.py <folder> --repo <empty folder> --docs-date 2026-10-29
python -X utf8 tools/check_repo.py <repo> --corpus <folder>/corpus.json --write
python -X utf8 tools/build.py <folder> --repo <another empty folder> --docs-date 2026-10-29 --compare <repo>
```

In the main Lexifina repository these files live as follows:
- `fetch.py`, `parse.py` and `repo/`: in `scripts/oman-law/`.
- `build.py`, `check_repo.py` (`check.py` there) and `on.py`: in `scripts/gitlaw_common/`, with the corpus format in `SCHEMA.md`.

## Exact inputs

- `corpus.json` sha256 `ac02fcf070f3069aee0e36561feb751da36b0b014835af6317f05ffea03c64cf` (462763 bytes)
- Script blob hashes (`git hash-object`):

| Script | Blob |
| --- | --- |
| build.py | `2bfc4ac9df193a7be844556d7f69cdb9d93ec3f2` |
| check.py | `cd346c7ff9996d450240e3bab52bcae03b2bc865` |
| on.py | `644a6c0857fbcc72562032b58d57aa113fee0641` |
| fetch.py | `f6de02ff2f952dbaa778f3aedeeeed6e5f1c1a71` |
| parse.py | `a252b61d0d29ad1321da586fc71cb9451a6a0daf` |

- Tools:
  - Python 3.14.0
  - git version 2.52.0.windows.1
  - curl 8.21.0 (Windows) libcurl/8.21.0 Schannel zlib/1.3.2 WinIDN WinLDAP

## Checks

Not yet run for this build.

A second build from the same corpus into an empty folder gave identical hashes for all 10 history commits and 20 tags (history tip `d1165deaab6becacb52fc846ea0f92bac0b18fe0`).

These results are from the check run on the build before this one, from the same corpus.

## Decisions made by hand

- Texts are the Arabic texts; Qanoon.om's and decree.om's English translations were not used.
- Qanoon.om is not an official source (its own words). Its text is used because the official MJLA PDFs have no usable text layer (Arabic glyphs mapped out of order), and OCR is not used. Every current article was checked against the MJLA PDF letter by letter (letter counts per article; see verify.py and verification.json).
- Versions are rebuilt from each decree as issued: the original text, then each amending decree's new wording, credited to the decree that Qanoon.om's footnote names. The last version is checked to equal Qanoon.om's consolidated text exactly, and every new line is checked to appear in the amending decree.
- In-force days are those the decrees fix themselves (date of issue, the day after publication, or one year after publication); publication dates are those Qanoon.om records for the Official Gazette issue.
- Units: the decree's preamble (preamble), the promulgating decree's articles (isdar-NN), and the law's articles (art-NNN, art-NNN-bis, art-NNN-bis-NN). Book, part and chapter headings are not units.
- Text normalisation: HTML line breaks become lines; NBSP, RLM/LRM, ZWNJ and tatweel removed; Western digits written as Arabic-Indic digits; curly quotes and dashes unified; no space before Arabic punctuation. Words are untouched.
- The Basic Statute of 1996 and that of 2021 are one act (one folder): RD 6/2021 issued a new statute and repealed RD 101/96 and RD 99/2011, so articles of the 1996 text that the 2021 text lacks are removed on 2021-01-11.
- rd-1996-101: issued 1996-11-06, Official Gazette issue 587 of 1996-11-16; in force 1996-11-06 by its own article: "ينشر هذا المرسوم في الجريدة الرسمية، ويعمل به اعتبارا من تاريخ صدوره."
- rd-2011-99: issued 2011-10-19, Official Gazette supplement to issue 948 of 2011-10-26; in force 2011-10-27 by its own article: "ينشر هذا المرسوم في الجريدة الرسمية، ويعمل به من اليوم التالي لتاريخ نشره."
- rd-2021-6: issued 2021-01-11, Official Gazette supplement to issue 1374 of 2021-01-12; in force 2021-01-11 by its own article: "ينشر هذا المرسوم في الجريدة الرسمية، ويعمل به من تاريخ صدوره."
- rd-2026-79: issued 2026-09-13, Official Gazette issue 1666 of 2026-09-20; in force 2026-09-13 by its own article: "ينشر هذا المرسوم في الجريدة الرسمية، ويعمل به من تاريخ صدوره."
- Basic Statute art-086: the consolidated text also makes the words that agree with "النيابة العامة" feminine (e.g. "يتولى" becomes "تتولى"), beyond the literal substitution of RD 79/2026 art. 1; the consolidated wording is used, credited to RD 79/2026.
- Basic Statute: RD 79/2026 changes 2 units (art-082, art-086); Qanoon.om footnotes the change only at art-082, and art. 1 of the decree applies it "wherever it occurs", so every unit where the words occur is credited.
- rd-2018-7: issued 2018-01-11, Official Gazette supplement to issue 1226 of 2018-01-14; in force 2018-01-15 by its own article: "ينشر هذا المرسوم في الجريدة الرسمية، ويعمل به من اليوم التالي لتاريخ نشره."
- rd-2022-68: issued 2022-10-20, Official Gazette issue 1464 of 2022-10-23; in force 2022-10-24 by its own article: "ينشر هذا المرسوم في الجريدة الرسمية، ويعمل به من اليوم التالي لتاريخ نشره."
- rd-2025-11: issued 2025-01-07, Official Gazette issue 1579 of 2025-01-13; in force 2025-01-14 by its own article: "ينشر هذا المرسوم في الجريدة الرسمية، ويعمل به من اليوم التالي لتاريخ نشره."
- rd-2026-66: issued 2026-06-23, Official Gazette issue 1654 of 2026-06-28; in force 2026-06-29 by its own article: "ينشر هذا المرسوم في الجريدة الرسمية، ويعمل به من اليوم التالي لتاريخ نشره."
- qanun-al-jaza: Qanoon.om footnote "عدل العنوان بموجب المرسوم السلطاني رقم ٦٨ / ٢٠٢٢." is on the heading "الفصل الثاني / القرصنة والاعتداء على وسائل النقل والمرافق العامة والمخالفات البلدية"; headings are not units, so it has no commit of its own.
- qanun-al-jaza: Qanoon.om footnote "أضيف بموجب المرسوم السلطاني رقم ٦٦ / ٢٠٢٦." is on the heading "الباب الخامس مكررا / جريمة الرشوة في القطاع الخاص"; headings are not units, so it has no commit of its own.
- qanun-al-jaza: art-029, art-114, art-143 differ between the text as issued and the consolidated page with no footnote naming an amendment (transcription differences: a dropped letter or a line break); the text as issued is kept.
- rd-2022-6: issued 2022-02-09, Official Gazette issue 1429 of 2022-02-13; in force 2023-02-13 by its own article: "ينشر هذا المرسوم في الجريدة الرسمية، ويعمل به بعد انقضاء سنة من تاريخ نشره."
- rd-2026-68: issued 2026-09-03, Official Gazette issue 1664 of 2026-09-06; in force 2026-09-07 by its own article: "ينشر هذا المرسوم في الجريدة الرسمية، ويعمل به من اليوم التالي لتاريخ نشره."

## What AI did and did not do

The fetch, parse and check scripts were written by Claude Code running Claude Opus 5.5 (`claude-opus-5-5`) under Lexifina's direction.

No model wrote, translated or corrected legal text. No model runs at build time. Every word of law in this repository comes from the official sources above, through code that anyone can rerun.

The commit messages are written by `build.py` from the source data.

Model use at data-build time:
- None at data-build time: fetch.py, qpage.py, parse.py and verify.py are deterministic code written with Claude Code (claude-opus-5-5); no model reads, writes, translates or summarises legal text.
