# Oman: law history in git

The law of the Sultanate of Oman as a git history: one file per article, one commit per royal decree on the day it took effect, and the decree that made each change as the commit's author. `git blame` names the decree behind every paragraph; `git log` lists every change to an article; reading the history on a date gives the text in force that day. The texts are in Arabic, the language of the Official Gazette.

It follows the shared design of [lexifina/global-law](https://github.com/lexifina/global-law), which places it at `om/`.

**What is in it now**

| Folder | Act | Versions | History |
| --- | --- | ---: | --- |
| `nizam-asasi/` | النظام الأساسي للدولة (Basic Statute of the State) | 4 | RD 101/96 (6 November 1996); amended by RD 99/2011 (27 October 2011); replaced by a new statute, RD 6/2021 (11 January 2021); amended by RD 79/2026 (13 September 2026) |
| `qanun-al-jaza/` | قانون الجزاء (Penal Code, RD 7/2018) | 4 | As issued (in force 15 January 2018); amended by RD 68/2022 (24 October 2022), RD 11/2025 (14 January 2025) and RD 66/2026 (29 June 2026) |
| `himayat-al-bayanat/` | قانون حماية البيانات الشخصية (Personal Data Protection Law, RD 6/2022) | 2 | As issued (in force 13 February 2023, one year after publication); amended by RD 68/2026 (7 September 2026) |

Each folder holds the promulgating decree's preamble (`preamble.md`) and articles (`isdar-NN.md`) and the law's articles (`art-NNN.md`, `art-NNN-bis.md`, `art-NNN-bis-NN.md` for مكررا and مكررا (N)). The per-act `README.md` lists every article, how often it changed and when.

## A lawyer's questions as git commands

| Question | Command |
| --- | --- |
| What did art. 86 of the Basic Statute say on 1 June 2026? | `python tools/on.py 2026-06-01 nizam-asasi art-086` |
| The same, with raw git (bash) | `git show $(git rev-list -1 --before="2026-06-01T23:59:59Z" main -- nizam-asasi/art-086.md):nizam-asasi/art-086.md` |
| The same, in PowerShell | `$r = git rev-list -1 --before="2026-06-01T23:59:59Z" main -- nizam-asasi/art-086.md; git show "$($r):nizam-asasi/art-086.md"` |
| Which decree wrote each paragraph of art. 97 of the Penal Code? | `git blame qanun-al-jaza/art-097.md` |
| Every change to art. 42 of the Basic Statute, word by word | `git log -p --word-diff nizam-asasi/art-042.md` |
| Every decree in the record, in order | `git log --reverse --format="%ad %an" --date=short main` |
| Everything one decree changed | `git show law/rd-2022-68` (tags `law/<key>` mark each decree's commit) |
| The whole act on a date | `python tools/on.py 2015-01-01 nizam-asasi` |
| Also name the decree behind the wording shown | `python tools/on.py --law 2026-10-01 himayat-al-bayanat art-010` |

Decree keys are `rd-<year>-<number>`: `rd-1996-101` is المرسوم السلطاني رقم ١٠١ / ٩٦. After a rebuild, refresh tags with `git fetch --force --tags`: a plain fetch keeps stale tags.

## Dates

- Each commit is dated at **noon, Asia/Muscat, on the day the change took effect** (the in-force date). Issue and publication dates are trailers (`Enacted:`, `Published:`), and the in-force day is in `In-force:`, which `tools/on.py` reads.
- **Every decree here fixes its own in-force day** in its last article, and that day is used: "from its date of issue" (RD 101/96, RD 6/2021, RD 79/2026), "from the day after its publication" (RD 99/2011, RD 7/2018, RD 68/2022, RD 11/2025, RD 66/2026, RD 68/2026), or "one year after its publication" (RD 6/2022). Publication dates are the Official Gazette issue dates.
- **Default rule.** Basic Statute (2021) art. 91: laws are published in the Official Gazette within two weeks of issue and take effect from publication unless they fix another date (the 1996 statute said the same in art. 74).
- **One act across two statutes.** RD 6/2021 issued a new Basic Statute and repealed RD 101/96 and RD 99/2011. Both statutes live in `nizam-asasi/`; on 11 January 2021 every article is rewritten, and the 1996 articles the new statute lacks (for example arts 58 bis (1) to (44) on the Council of Oman, which moved to the Council of Oman Law, RD 7/2021) are removed.
- `--before="YYYY-MM-DD"` alone uses your clock's time of day, so a decree that took effect that day may be missed before noon. Use `T23:59:59Z` as above, or the tool.

## How it is checked

- **Replay against the consolidated text.** Each act's history is rebuilt from the decrees as issued: the original text, then each amending decree's wording. The last version is compared with Qanoon.om's consolidated "(معدل)" page, article by article, and every line that an amendment introduced is checked to appear in that amending decree as published.
- **Against the official text.** The Ministry of Justice and Legal Affairs publishes the current text of each act as a PDF. Its text layer keeps every Arabic letter but scrambles their order inside ligatures, so it cannot be read as text without OCR, which is not used. Instead, `tools/verify.py` checks that the letters of each article (alef forms and digits left out) are exactly the letters of some run of the official PDF. On 29 October 2026 every article of the current version matched: 98 of 98 (Basic Statute), 397 of 397 (Penal Code), 34 of 34 (Personal Data Protection Law). The check is sensitive to any added, dropped or changed letter other than an alef; it does not see word order. The result, with the sha256 of each PDF, is recorded with the sources in `REPRODUCE.md`.
- Every version is read back from git on the day it took effect and on the day before (`tools/check_repo.py`); both must match `corpus.json`. No merge commits, no dates going backwards, nothing dated in the future, and every commit's trailers parse with `git interpret-trailers --parse`.
- The repository is rebuilt from the same data a second time and must give identical hashes. Results are in `REPRODUCE.md`.

## What "author" means

The author of a commit is the royal decree that made the change, with an e-mail address that cannot exist (`rd-2022-68@qanoon.invalid`), so `git blame` and `git log --author` name decrees. The committer is always Lexifina.

- **Credits come only from the source.** A changed article is credited to the decree that Qanoon.om's footnote on that article names ("عدلت بموجب المرسوم السلطاني رقم ٦٨ / ٢٠٢٢"). Articles added under a new part (the Penal Code's Part V bis, on bribery in the private sector, RD 66/2026) take the footnote on the part's heading.
- **RD 99/2011** is not footnoted anywhere, since the 1996 statute is repealed and no consolidated text of it is published. Its five items (art. 6 replaced; a sixth paragraph added to art. 42; the fourth paragraph of art. 44 replaced; art. 56 replaced; Part V replaced by arts 58 and 58 bis to 58 bis (44)) were applied to the 1996 text as written, and those articles are credited to it.
- **RD 79/2026** replaces "الادعاء العام" with "النيابة العامة" wherever it occurs in the Basic Statute (arts 82 and 86). In art. 86 the consolidated text also makes the words that agree with it feminine ("يتولى" becomes "تتولى"); the official PDF has the same letters, and that wording is used.
- **Transcription differences** between Qanoon.om's page of a decree as issued and its consolidated page, where no footnote names an amendment, are not changes: Penal Code arts 29 (a dropped alef in the consolidated page), 114 and 143 (line breaks) keep the text as issued.
- Blame credits whole lines (one paragraph per line). Use `git log -p --word-diff` to see exactly which words each decree changed.

## What it does not show

- **Headings.** Book, part, chapter and section headings are not files. Two heading changes are therefore not commits: RD 68/2022 renamed chapter 2 of part 3 of book 2 of the Penal Code ("القرصنة والاعتداء على وسائل النقل والمرافق العامة والمخالفات البلدية"), and RD 66/2026 added the heading of Part V bis.
- **The decrees' signature blocks** (date of issue, signature, Gazette issue) are not in the text; they are in the commit trailers.
- **Earlier law.** The Penal Code of 2018 replaced the Omani Penal Law of 1974 (RD 7/74), which is not included.
- **Other laws.** Only these three acts are here so far.
- **Applicability.** Transitional rules on which cases a change applies to are in the text, not in the dates; no applicability data is recorded.

## How it is built

`tools/fetch.py` downloads every source page once through curl (at most three requests a second, cached by the sha256 of the URL), `tools/qpage.py` and `tools/parse.py` turn the cache into `corpus.json`, `tools/verify.py` checks it against the official PDFs, and the shared builder `tools/build.py` writes the history with `git fast-import`. The same corpus always gives the same hashes. `REPRODUCE.md` records every command, tool version, data hash and decision, and the result of a hash-for-hash rebuild.

No AI model wrote, changed or translated legal text. Claude Opus 5.5 wrote the scripts and chose the sources; see `REPRODUCE.md`.

## Sources

| Source | Kind | Past versions | Names the amending act | Bulk | Notes |
| --- | --- | --- | --- | --- | --- |
| [Qanoon.om](https://qanoon.om) | Unofficial transcription of the Official Gazette (since 2015) | each decree as issued, plus a consolidated page per law | yes, a footnote per amended article | WordPress pages and sitemap | Used for the text. It states it is not an official source. |
| [Ministry of Justice and Legal Affairs](https://mjla.gov.om) (laws section) | Official legislation | current text only | no | PDF per law; a laws dataset on opendata.gov.om | Used for the letter check. Government Open Licence - Sultanate of Oman. Its PDFs' text layer is scrambled (see above). |
| Official Gazette issues on mjla.gov.om (`legislation/`) | Official gazette | n/a | n/a | PDF per issue (2013-) | Not read. |

Research guides that cover every country: [GlobaLex](https://www.nyulawglobal.org/globalex/) and the [Law Library of Congress Guide to Law Online](https://www.loc.gov/research-centers/law-library-of-congress/researcher-resources/guide-to-law-online/).

## Licence

The legal texts are royal decrees and laws of the Sultanate of Oman, as published in the Official Gazette. Qanoon.om states that texts of law are public property in Oman under the Omani Copyright and Related Rights Law and may be used, copied, published and translated freely. The Ministry of Justice and Legal Affairs licenses its site under the Government Open Licence - Sultanate of Oman. The arrangement, tables, commit messages and tools are by Lexifina, under the terms in `LICENSE`.

This repository is not legal advice and not an official publication.

## بالعربية

تاريخ التشريع العماني في git: ملف لكل مادة، و commit لكل مرسوم سلطاني بتاريخ العمل به، ويُنسب كل تعديل إلى المرسوم الذي أجراه (`git blame` يسمي المرسوم وراء كل فقرة). يضم النظام الأساسي للدولة منذ المرسوم السلطاني رقم ١٠١ / ٩٦ وتعديله بالمرسوم السلطاني رقم ٩٩ / ٢٠١١، ثم النظام الأساسي الصادر بالمرسوم السلطاني رقم ٦ / ٢٠٢١ وتعديله بالمرسوم السلطاني رقم ٧٩ / ٢٠٢٦، وقانون الجزاء (المرسوم السلطاني رقم ٧ / ٢٠١٨) وتعديلاته، وقانون حماية البيانات الشخصية (المرسوم السلطاني رقم ٦ / ٢٠٢٢) وتعديله.

- النص النافذ في تاريخ معين: `python tools/on.py 2026-06-01 nizam-asasi art-086`
- التواريخ: تاريخ العمل بكل مرسوم كما نص عليه المرسوم ذاته.
- المصدر: موقع Qanoon.om (وهو ليس مصدرا رسميا)، مع مطابقة النص النافذ حرفا بحرف مع ملفات وزارة العدل والشؤون القانونية. لم يكتب أي نموذج ذكاء اصطناعي نصا قانونيا.
