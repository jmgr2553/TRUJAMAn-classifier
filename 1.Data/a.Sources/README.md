# Sources

TRUJAMAn was trained and evaluated on **110 works by 36 authors** from the Judeo-Arabic portal of the Friedberg Jewish Manuscript Society (FJMS), [fjms.genizah.org](https://fjms.genizah.org).

> Texts courtesy of **The National Library of Israel / The Friedberg Jewish Manuscript Society**. The transcriptions are the copyright of their authors.

**The texts themselves are not included.** The FJMS terms of use allow them to be used for personal study and research only, so this repository publishes only metadata about them.

`sources.csv` lists every work:

| Column | Description |
|---|---|
| `codeName` | Short identifier used throughout the repository (author + transliterated title) |
| `author` | Author, in English |
| `title` | Title as given on the portal |
| `fjmsResourceId` | Resource number of the work on the FJMS portal |
| `pages` | Number of portal pages in the work |
| `split` | `training` (70 works) or `evaluation` (40 works) |

The word counts and the share of Hebrew in each work are in `works.csv` in `../b.Training/b.1.Training/` and `../b.Training/b.2.Evaluation/`.
