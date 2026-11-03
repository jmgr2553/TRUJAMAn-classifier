# TRUJAMAn 🧙

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.23921993.svg)](https://doi.org/10.5281/zenodo.23921993)

**Textual Recognition Unit for Judeo-Arabic (and Hebrew) in Medieval Anthologies**

*Beta release v0.1.0* · Juan Moreno Gonzalez ([ORCID 0009-0003-6528-0736](https://orcid.org/0009-0003-6528-0736)), Faculty of Asian and Middle Eastern Studies, University of Cambridge

TRUJAMAn is a word-level classifier that labels every word in a Hebrew-script text as **Judeo-Arabic (JA)** or **Hebrew (HE)**. It is meant for medieval Judeo-Arabic works, where Hebrew quotations, terms and phrases are woven into Arabic prose.

**Try it online:** [trujaman.streamlit.app](https://trujaman.streamlit.app/), where you can paste a text and download the tagged words as CSV.

**Paper:** [*TRUJAMAn: Word-Level Recognition of Judeo-Arabic and Hebrew in Medieval Texts*](https://doi.org/10.5281/zenodo.23926319) (working paper, v0.2), with the data, method and full results.

> ⚠️ **Beta.** The labels are machine-generated. Check anything important against the text.

---

## Download

| File | Size | Contents |
|---|---|---|
| `trujaman_v0.1-beta.joblib` | 16 MB | the trained model (scikit-learn 1.9.0) |

```python
import re
import joblib

clf = joblib.load("trujaman_v0.1-beta.joblib")

# Same cleaning as in training: remove diacritics and punctuation.
STRIP = re.compile("[\u05c4\u05bf\u05f3\u05f4\u0027\u0022,.;:()\\[\\]\\-\u2013\u2014?!\u2026]")

def tag(text):
    """Return (word, label, probability of Hebrew) for every word in a text."""
    words = [w for w in text.split() if STRIP.sub("", w)]
    norm = [STRIP.sub("", w) for w in words]
    # Each word is classified together with its neighbours: "prev ~word~ next"
    feats = [(norm[i - 1] if i > 0 else "") + " ~" + w + "~ " +
             (norm[i + 1] if i + 1 < len(norm) else "")
             for i, w in enumerate(norm)]
    p_he = clf.predict_proba(feats)[:, 1]
    return [(w, "HE" if p > 0.5 else "JA", round(float(p), 3)) for w, p in zip(words, p_he)]

for row in tag("קאל אללה תעאלי בראשית ברא אלהים את השמים"):
    print(row)
```

The model must be loaded with **scikit-learn 1.9.0** (`pip install -r requirements.txt`).

## How it works

Each word is cleaned (diacritics such as the upper dot, rafe and geresh, quotation marks and punctuation are removed), so the model does not depend on how a manuscript is pointed. It is then classified **together with the word before and after it**, using character sequences of 1 to 5 letters (TF-IDF) and a logistic regression.

## Training data

| | Works | Words |
|---|---|---|
| Training (`1.Data/b.Training/b.1.Training/`) | 70 | 2,852,411 |
| Evaluation (`1.Data/b.Training/b.2.Evaluation/`) | 40 | 1,070,917 |

Words are counted after cleaning, as the model sees them: editorial marks such as `[…]`, `-` and `(?)`, which the transcriptions label with a language, are not counted. The `works.csv` files count every word labelled JA or HE, so their totals are slightly higher (3,932,708 in all, against 3,923,328).

The 110 works by 36 authors come from the Judeo-Arabic portal of the **Friedberg Jewish Manuscript Society (FJMS)**, in which every word of the transcription is marked as Hebrew or Judeo-Arabic. The evaluation set holds one work by each author, plus every work by **Ben Hofni**. Authors with a single work in the corpus therefore appear only in evaluation: 31 of the 40 evaluation works are by 25 authors the model never saw in training.

**The texts are not included in this repository**, because the FJMS terms of use do not allow them to be redistributed. The repository lists the works used (`1.Data/a.Sources/sources.csv`, and `works.csv` in each training folder) with word counts and the share of Hebrew in each. To retrain the model, add the texts from FJMS as described in `1.Training.ipynb` and run its training cell.

## Evaluation

Scores on the 40 held-out works (1,070,917 words, 36% Hebrew):

| | Precision | Recall | F1 | Accuracy | Words |
|---|---|---|---|---|---|
| Judeo-Arabic | 0.980 | 0.964 | 0.972 | | 683,038 |
| Hebrew | 0.938 | 0.965 | 0.951 | | 387,879 |
| **All evaluation works** | 0.959 | 0.964 | 0.962 | **96.4%** | 1,070,917 |
| Authors unseen in training | 0.958 | 0.964 | 0.961 | 96.4% | 974,869 |
| Authors also in training | 0.966 | 0.966 | 0.966 | 96.6% | 95,493 |

For the last three rows, precision, recall and F1 are the average of the two languages (macro average). Accuracy applies only to a whole set of words, not to one language.

91% of the evaluation words come from authors the model has never seen, and it does as well on them as on new works by known authors. Accuracy per work ranges from 89% (Ibn Quraysh, *Risāla*) to 99%. Scores for every work are in [`evaluation_scores.csv`](1.Data/b.Training/b.2.Evaluation/evaluation_scores.csv). In works with almost no Hebrew (e.g. under 1%), F1 for Hebrew is low because a handful of errors weighs heavily; accuracy is the better guide there.

## Repository

```
1.Data/
  a.Sources/          list of the 110 works and their FJMS identifiers
  b.Training/
    b.1.Training/     works used for training (list only)
    b.2.Evaluation/   works held out for evaluation (list only) + evaluation_scores.csv
1.Training.ipynb      trains the model and evaluates it
2.Test.ipynb          tests the published model: per work, precision/recall, seen vs unseen authors,
                      confusion matrix, language switches, confidence, frequent words and chunks
                      (with short five-word examples), and a combined table of error-prone and
                      strongly weighted words
trujaman_v0.1-beta.joblib
```

## Known limitations

- **Two labels only.** Every word is labelled JA or HE. Aramaic is not distinguished from Hebrew, and Arabic script is not handled.
- **Trained on edited prose.** The training texts are transcriptions of literary works (commentaries, philosophy, law, grammar). Letters, documents and poetry may behave differently.
- **Ambiguous words.** Short words shared by both languages (e.g. `לא`, `מן`, `אלא`, `או`, `כל`) are decided by their neighbours and are the most common errors.
- **No word-break correction.** The text is split at spaces only.

## Sources and credits

Texts courtesy of **The National Library of Israel / The Friedberg Jewish Manuscript Society**, Judeo-Arabic portal, [fjms.genizah.org](https://fjms.genizah.org). The transcriptions and their language markup are the copyright of their authors. The texts were used for research under the FJMS terms of use and are not redistributed here.

## License

The **model and data** (`trujaman_v0.1-beta.joblib`, `1.Data/`) are licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/): you may share and adapt them for non-commercial purposes, with credit. The **code** (`1.Training.ipynb`) is under the MIT License. See [`LICENSE`](LICENSE).

## Citation

> Moreno Gonzalez, J. (2026). *TRUJAMAn: Textual Recognition Unit for Judeo-Arabic (and Hebrew) in Medieval Anthologies* (Version v0.1.0-beta) [Computer software]. Zenodo. https://doi.org/10.5281/zenodo.23921994

To cite the working paper that describes the model and its evaluation:

> Moreno Gonzalez, J. (2026). *TRUJAMAn: Word-Level Recognition of Judeo-Arabic and Hebrew in Medieval Texts* (Working paper, v0.2). Zenodo. https://doi.org/10.5281/zenodo.23926319

To cite TRUJAMAn in general (always resolves to the latest version), use [doi:10.5281/zenodo.23921993](https://doi.org/10.5281/zenodo.23921993). See also [`CITATION.cff`](CITATION.cff) or GitHub's **"Cite this repository"** button.
